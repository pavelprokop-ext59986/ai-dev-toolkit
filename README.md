# AI Dev Toolkit

A collection of reusable GitHub Copilot CLI agents and skills for AI-assisted software engineering workflows.

The repository is focused on:
- structured AI-assisted development
- orchestration of specialized agents
- safe Git workflows
- local developer tooling integration
- pragmatic enterprise-friendly workflows

The toolkit is designed primarily for local developer workstation usage with GitHub Copilot CLI.

---

# Repository Contents

## Agents

### Coordination

#### `workflow-orchestrator`
Coordinates workflows between specialized agents and helps organize implementation phases.

#### `solution-architect`
Analyzes requirements, refines architecture, and produces implementation-ready documentation and guidance.

---

### Implementation

#### `kotlin-backend-developer`
Specialized Kotlin/Spring backend implementation agent.

Focused on:
- Spring Boot
- Kotlin
- Flyway
- jOOQ
- repositories/services/controllers
- Kafka/event integration
- enterprise backend workflows

#### `angular-frontend-developer`
Specialized Angular frontend implementation agent.

Focused on:
- Angular
- TypeScript
- forms
- API integration
- frontend architecture
- UI workflows

---

### Quality & Safety

#### `test-engineer`
Focused on:
- testing strategy
- unit tests
- integration tests
- E2E testing
- test reliability
- test coverage validation

#### `code-reviewer`
Performs implementation reviews with focus on:
- architecture alignment
- code quality
- maintainability
- migration safety
- testing adequacy
- enterprise development practices

#### `git-watchdog`
Git safety and repository recovery assistant.

Focused on:
- rebases
- stash recovery
- repository cleanup
- branch extraction
- commit hygiene
- preventing accidental work loss

---

## Skills

### `ktlint-format-maven`
Runs repository-approved ktlint formatting and verification workflows for Maven Kotlin projects.

---

### `playwright-browser-automation`
Uses Playwright from command line tooling to:
- inspect web applications
- automate browser interactions
- validate UI behavior
- capture screenshots
- assist frontend debugging

---

### `postgres-database-inspector`
Provides safe PostgreSQL inspection workflows using:
- `psql`
- `.pgpass`
- local workstation authentication

Focused primarily on:
- schema inspection
- Flyway troubleshooting
- debugging
- query validation
- safe read-only database investigation

---

# Installation

## macOS / Linux (recommended)

Clone repository:

```bash
cd ~/projects
git clone https://github.com/<YOUR-USERNAME>/ai-dev-toolkit.git
```

Create workspace-level symlinks:

```bash
cd ~/projects

ln -s ~/projects/ai-dev-toolkit/copilot/.github .github
```

Optional user-level configuration:

```bash
ln -s ~/projects/ai-dev-toolkit/copilot/.github/skills ~/.copilot/skills
ln -s ~/projects/ai-dev-toolkit/copilot/.github/agents ~/.copilot/agents
```

Verify configuration:

```bash
copilot
/agent list
/skill list
```

---

## Windows

### Option 1 — Copy files (simplest)

Copy:

```text
copilot/.github
```

into workspace root.

Example:

```text
C:\projects\
  .github\
```

---

### Option 2 — Junctions / symbolic links

Run PowerShell as Administrator:

```powershell
New-Item -ItemType Junction `
  -Path C:\projects\.github `
  -Target C:\projects\ai-dev-toolkit\copilot\.github
```

---

# Repository Structure

```text
ai-dev-toolkit/
  copilot/
    .github/
      agents/
      skills/
```

---

# Typical Workflow

Example development workflow:

1. `solution-architect`
   - analyzes requirements
   - proposes architecture
   - prepares implementation guidance

2. `kotlin-backend-developer` and/or `angular-frontend-developer`
   - implement requested functionality

3. `test-engineer`
   - validates testing strategy and implementation

4. `code-reviewer`
   - reviews implementation quality and alignment

5. `git-watchdog`
   - assists with Git safety, cleanup, and recovery workflows

6. `workflow-orchestrator`
   - coordinates overall workflow when needed

---

# Updating Toolkit

When using symlinks:
- changes are applied immediately
- no reinstall is needed

Update repository normally:

```bash
cd ~/projects/ai-dev-toolkit
git pull
```

---

# Security Notes

This repository should not contain:
- credentials
- tokens
- production secrets
- sensitive company/internal information

Recommended:
- use `.gitignore`
- keep Playwright auth states local-only
- prefer local workstation tooling over remote integrations

---

# Philosophy

The toolkit intentionally focuses on:
- local-first workflows
- user-controlled tooling
- explicit command execution
- pragmatic orchestration
- enterprise-friendly AI adoption

The goal is not autonomous development, but structured and safe AI-assisted engineering.
