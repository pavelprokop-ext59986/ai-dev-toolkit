---
name: ktlint-format-maven
description: Run repository-approved ktlint formatting and verification commands for Maven Kotlin projects.
license: Proprietary
---

# Skill: ktlint-format-maven

## Purpose
Automatically fix Kotlin formatting issues by running the repository-approved ktlint formatter Maven task instead of manual edits, then re-running verification to ensure the formatting gate is clean.

## When to use
Use this skill when:
- `./mvnw test` (or `verify`) fails due to ktlint/formatting violations.
- The user asks to "fix ktlint", "format Kotlin", "resolve formatting failures", or similar.
- CI indicates a ktlint check failure.

## Preconditions / Safety
- Prefer the Maven Wrapper (`./mvnw`) not system Maven.
- Do not run publishing or release goals (`deploy`, `release:*`, etc.).
- Keep diffs minimal; formatting tools may touch many files—call this out if it happens.
- If the repo uses a different formatting tool (Spotless, Gradle), stop and ask or fall back to repository docs.

## Primary command (repo-specific)
Run the formatter:

```bash
./mvnw antrun:run@ktlint-format
```

If the above goal is not configured or fails with:
- `No plugin found for prefix`
- `No plugin execution found`

Then:
- Search for the correct formatter goal in the repo build docs or `pom.xml`
- Look for `ktlint`, `spotless`, `antrun`, `fmt`, `format`
- If you find an alternative standard command, use it and note the change

## Verification
After applying formatting, re-run the same command that originally failed (default: unit tests):

```bash
./mvnw test
```

If CI uses `verify` instead:

```bash
./mvnw verify
```

## Procedure
1. Identify that the failure is formatting-related:
   - Look for output mentioning `ktlint`, `formatting`, `lint`, `Kotlin`, or formatter plugin goals
2. Run:

```bash
./mvnw antrun:run@ktlint-format
```

3. Inspect changes:

```bash
git diff --stat
git diff
```

4. If the formatter changed many unrelated files:
   - report that clearly
   - consider whether formatting should run only for a specific module

5. Re-run verification:

```bash
./mvnw test
```

or:

```bash
./mvnw verify
```

6. If verification still fails due to ktlint:
   - re-run the formatter once more
   - if still failing, investigate plugin configuration:
     - version mismatch
     - rule set mismatch
     - wrong module
     - multiple formatters

7. Provide a final summary:
   - commands executed
   - number of files changed
   - confirmation whether final verification passed

## Output expectations
Report:
- formatter command used
- verification command used
- short summary of changed files/modules
- final pass/fail status

## Notes / Common pitfalls
- Multi-module repos may require running from repository root
- Generated sources should usually not be formatted unless explicitly expected
- If ktlint rules are enforced only in `verify`, do not stop at `test`
