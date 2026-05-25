---
name: pr-reviewer
description: >
  Reviews a GitHub pull request by fetching its metadata, file list, and per-file diffs via
  the GitHub CLI and API. Use this skill when the user asks to review a pull request or a PR URL,
  and especially when the diff may be large. Produces a structured code review with findings
  grouped by severity.
---

## Goal

Perform a thorough, high-signal code review of a GitHub pull request. Avoid noise — only surface
genuine issues: bugs, logic errors, security vulnerabilities, risky behavior changes, and
significant omissions.

## Steps

### 1. Fetch PR metadata

```bash
gh pr view <PR_URL_OR_NUMBER> --json title,body,files,commits,additions,deletions,baseRefName,headRefName,author
```

Note the total additions/deletions and the list of changed files.

### 2. Fetch per-file diffs (avoids large monolithic output)

Instead of `gh pr diff` (which produces a single large output that can be hard to read), fetch
each file's patch individually via the GitHub API:

```bash
# Extract owner/repo from the PR URL, e.g. csas-dev/corp-le-ca-limits-service
gh api repos/<owner>/<repo>/pulls/<number>/files --jq '.[] | {filename: .filename, additions: .additions, deletions: .deletions, patch: .patch}' 2>/dev/null
```

If the API response is large, pipe through `jq` to get just filenames first:
```bash
gh api repos/<owner>/<repo>/pulls/<number>/files --jq '.[].filename'
```

Then fetch individual file patches selectively:
```bash
gh api repos/<owner>/<repo>/pulls/<number>/files --jq '.[] | select(.filename == "path/to/file") | .patch'
```

For very large patches, save to a temp file and use `view` with `view_range` to read in sections:
```bash
gh api repos/<owner>/<repo>/pulls/<number>/files --jq '.[].patch' > /tmp/pr-patches.txt
```

### 3. Analyse the changes

Read each file's diff. For each changed file, consider:

- **Correctness**: Does the logic do what it claims? Are there edge cases or off-by-one errors?
- **Behavior changes**: Could a seemingly mechanical change (rename, config removal, dependency bump) silently alter runtime behavior?
- **Security**: Are credentials, tokens, or sensitive data handled safely?
- **API/library upgrades**: For version bumps, check if breaking changes or default behavior changes could affect the service.
- **Config changes**: Removal of config properties (e.g. YAML keys) — verify the new default matches the old explicit value.
- **Test coverage**: Are new behaviors covered by tests? Are existing tests still meaningful?
- **Consistency**: Is the change applied consistently across all affected files?

### 4. Project-specific checks (CSAS / corp-le ecosystem)

When reviewing services in the `csas-dev` organization (corp-le-* repositories):

- **Spring Boot upgrades**: Check removed YAML properties for changed defaults (especially
  `jackson.*`, `spring.jpa.*`, `spring.security.*`). Verify package relocations are complete
  (e.g. `org.springframework.boot.autoconfigure.*` → restructured in SB4).
- **Jackson 3.x migration** (`tools.jackson` vs `com.fasterxml.jackson`): Confirm `JsonMapper`
  builder pattern is used correctly; check `write-dates-as-timestamps` default changed.
- **Kafka / Avro**: Check serializer/deserializer config is compatible with upgraded connector versions.
- **ktlint**: Flag any multi-line `when` blocks missing blank lines, or trailing commas missing on
  multi-line `setOf()`/`when` groups.
- **New API parameters**: When a dependency bumps a major version and a method gains new parameters,
  verify the values passed are intentional (not just placeholders).
- **Flyway migrations**: If schema changes are present, confirm they have explicit user approval
  before creation.
- **Test config credentials**: Placeholder credentials in `application-test.yml` are acceptable;
  flag any real-looking credentials.

### 5. Format the review

Structure the output as follows:

```
## PR Review: <title>

**Branch:** <head> → <base>  
**Files changed:** N (+additions / -deletions)

---

### ✅ What's done well
- <bullet points of good patterns, correctness, consistency>

---

### ⚠️ Items worth double-checking
For each item:
**File** — short description
> Snippet or line reference
Explanation of the concern and suggested fix/question.

---

### 🐛 Bugs / Breaking issues  (only if present)
Same format as above, but for genuine bugs or breaking changes.

---

### 🟢 Verdict
Overall assessment + merge recommendation.
```

Only include sections that have content. Omit empty sections.
Keep findings concise — one clear sentence per finding, with a code snippet only when it aids
understanding.
