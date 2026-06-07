---
name: github-mcp
displayName: GitHub MCP
description: "Guided Power for GitHub workflows — bundles Docker-based MCP server, steering files for PRs/issues/repos, and reusable skills."
version: "0.1.0"
keywords:
  - github
  - mcp
  - pull-request
  - issues
  - repository
  - docker
  - automation
author: studiokanslia
---

## Overview

The **GitHub MCP** Power provides three core capabilities:

1. **GitHub API Interaction** — A pre-configured Docker-based MCP server (`ghcr.io/github/github-mcp-server`) that exposes GitHub tools (issues, pull requests, repositories, code search, and more) via stdio transport, fully compatible with web agent and cloud sandbox environments.
2. **Guided Workflows via Steering Files** — Five markdown steering files that instruct Kiro on best practices for pull requests, issue management, repository operations, spec-issue synchronization, and workspace project context detection.
3. **Reusable Skills** — Seven documented skill workflows (`create-feature-pr`, `file-bug-report`, `create-release`, `review-pull-requests`, `create-spec-from-issue`, `report-power-bug`, `request-power-feature`) that Kiro can execute as multi-step guided actions through the Docker-based MCP server.

## Configuration

### Prerequisites

- **Docker** installed and running (the MCP server runs inside a container)
- **Kiro** installed (provides the agent runtime and secrets management)
- **GitHub Personal Access Token** configured in Kiro's secrets system

### Token Setup

Register your GitHub personal access token in Kiro's secrets configuration using the secret reference syntax:

```
${secret:GITHUB_PERSONAL_ACCESS_TOKEN}
```

Kiro resolves this reference at runtime and injects the token into the container via the `-e` flag. The token is never embedded in the Docker image and must never be stored in repository files.

#### Minimum Required Token Scopes

**Classic tokens** — the following scopes are required at minimum:

| Scope | Purpose |
|-------|---------|
| `repo` | Repository read/write (code, commits, status) |
| `read:org` | Read organization membership |
| `issues` | Issue read/write |
| `pull_requests` | Pull request read/write |

**Fine-grained tokens (recommended)** — fine-grained personal access tokens are preferred over classic tokens because they can be scoped to specific repositories rather than granting organization-wide access:

| Permission | Access Level | Purpose |
|------------|-------------|---------|
| Contents | Read and Write | Repository content operations |
| Issues | Read and Write | Issue creation, triage, updates |
| Pull Requests | Read and Write | PR creation, review, merge |
| Organization | Read | Organization membership queries |

> **Recommendation:** Use fine-grained tokens scoped to only the repositories you need. This follows the principle of least privilege and limits exposure if the token is compromised.

### Docker Image Reference

The Power uses a pinned Docker image for the GitHub MCP server:

```
ghcr.io/github/github-mcp-server:v0.2.0@sha256:placeholder_replace_with_actual_digest_before_release
```

#### Image Pinning Policy

The image reference MUST use both a version tag AND a SHA256 digest — never `latest` or an untagged reference:

- **Version tag** (e.g., `:v0.2.0`) provides human-readable identification
- **SHA256 digest** (e.g., `@sha256:<64-char-hex>`) ensures immutable content verification

This defense-in-depth approach prevents supply chain attacks via tag mutation. Even if a tag is maliciously re-pushed, the digest verification ensures the exact expected image bytes are used.

**Format:** `ghcr.io/github/github-mcp-server:<version>@sha256:<digest>`

### Docker Run Command

The MCP server is launched via Docker with the following command:

```bash
docker run --rm -i -e GITHUB_PERSONAL_ACCESS_TOKEN ghcr.io/github/github-mcp-server:v0.2.0@sha256:placeholder_replace_with_actual_digest_before_release
```

This is configured in `mcp.json` and executed automatically by Kiro when the Power is activated.

### Stdio Transport and Container Networking

The Docker container communicates with the Kiro agent via **stdio transport** (stdin/stdout):

- **`--rm`** — Removes the container automatically after the MCP server process exits, preventing accumulation of stopped containers
- **`-i`** — Runs the container in interactive mode, keeping stdin open for stdio communication (no `-t` flag since there is no TTY)
- **No `-d` flag** — The container runs in the foreground, not detached, so stdin/stdout are directly connected to the agent
- **No port mappings** — Communication is entirely via stdio, not network ports. No `-p` flags are needed

This architecture is specifically designed for web agent and cloud sandbox environments where the MCP server must run inside a container rather than as a native host process.

### Configuring the Token in Kiro

To register your `GITHUB_PERSONAL_ACCESS_TOKEN` in Kiro's secrets system:

1. Open Kiro's settings or configuration panel
2. Navigate to the Secrets section
3. Add a new secret with the key `GITHUB_PERSONAL_ACCESS_TOKEN`
4. Paste your GitHub personal access token as the value
5. Save — Kiro will resolve `${secret:GITHUB_PERSONAL_ACCESS_TOKEN}` at runtime and pass it into the container

> **Security:** The token is injected into the container at runtime via the `-e` environment variable flag. It is never baked into the Docker image, never written to disk inside the container, and must never be committed to repository files.

### Troubleshooting Docker Setup

| Problem | Resolution |
|---------|------------|
| Docker not installed | Install Docker Desktop (macOS/Windows) or Docker Engine (Linux) from [docker.com](https://docker.com). Verify with `docker --version`. |
| Docker daemon not running | Start Docker Desktop or run `sudo systemctl start docker` (Linux). Verify with `docker info`. |
| Image cannot be pulled | Verify network access to `ghcr.io`. Check that Docker can reach the GitHub Container Registry: `docker pull ghcr.io/github/github-mcp-server:v0.2.0`. If behind a corporate proxy, configure Docker's proxy settings. |
| Permission denied on Docker socket | Add your user to the `docker` group: `sudo usermod -aG docker $USER`, then log out and back in. |

## Available Steering Files

| File | Description |
|------|-------------|
| `steering/pull-requests.md` | Guides Kiro through PR creation, code review, and merge strategy selection. Includes PR description templates, label taxonomy, and review checklists. |
| `steering/issues.md` | Guides Kiro through issue creation, triage, and lifecycle management. Defines overridable defaults for issue types and priority levels. |
| `steering/repository-management.md` | Guides Kiro through branch naming conventions, release workflows, and repository configuration. Includes GitHub Actions automation guidance. |
| `steering/spec-issue-sync.md` | Guides Kiro through bidirectional synchronization between spec documents and GitHub issues. Maintains traceability between requirements and issue tracker. |
| `steering/project-context.md` | Instructs Kiro on workspace repository detection, context scoping, and workspace-level override configuration. Enables global Power installation with automatic per-project targeting. |

## Skills

All skills execute through the Docker-based GitHub MCP server container (`ghcr.io/github/github-mcp-server`). Skills 1–5 inherit the detected workspace project context (owner/repo) automatically via the `project-context.md` steering file — users do not need to manually specify repository owner and name. Skills 6–7 always target `studiokanslia/kiro-power-github` regardless of workspace context.

### Skill: create-feature-pr

**Description:** Creates a feature branch, commits staged changes, and opens a pull request with a structured description following the PR steering conventions.

**Required Inputs:**
- `branch_name` (source: user) — Name for the feature branch (e.g., `feature/add-dark-mode`)
- `description` (source: user) — Summary of the changes for the PR body (Summary, Changes, Testing sections)
- `owner` (source: context) — Repository owner, auto-detected from git remote
- `repo` (source: context) — Repository name, auto-detected from git remote

**Steps:**
1. Validate branch name against naming convention (prefix/description, max 100 chars) → Tool: `create_branch(owner, repo, branch_name, from_branch: "main")`
2. Stage and commit changes to the new branch → Tool: `create_or_update_file(owner, repo, path, content, message, branch)`
3. Open a pull request with structured description (Summary, Changes, Testing), type/scope labels → Tool: `create_pull_request(owner, repo, title, body, head: branch_name, base: "main")`

**Expected Output:** Pull request URL (e.g., `https://github.com/{owner}/{repo}/pull/42`)

**Example Invocation:** "Create a feature PR for the dark mode implementation on branch `feature/add-dark-mode`"

---

### Skill: file-bug-report

**Description:** Creates a bug report issue with reproduction steps, expected/actual behavior, environment details, and priority assignment following the issue steering conventions.

**Required Inputs:**
- `summary` (source: user) — One-sentence description of the bug (max 72 characters for title)
- `steps` (source: user) — Steps to reproduce the bug
- `priority` (source: user) — Priority level (P0-critical, P1-high, P2-medium, P3-low, or workspace-overridden values)
- `owner` (source: context) — Repository owner, auto-detected from git remote
- `repo` (source: context) — Repository name, auto-detected from git remote

**Steps:**
1. Format the issue body using the bug report template (title, description, steps to reproduce, expected/actual behavior, environment) → Tool: `create_issue(owner, repo, title, body, labels: ["bug", priority_label])`
2. Add environment context or additional details as a follow-up comment if needed → Tool: `add_issue_comment(owner, repo, issue_number, body)`

**Expected Output:** Issue URL (e.g., `https://github.com/{owner}/{repo}/issues/15`)

**Example Invocation:** "File a bug report: the login form crashes when submitting empty fields, priority P1"

---

### Skill: create-release

**Description:** Tags a semantic version release, generates a changelog summary, and creates a GitHub release with auto-generated release notes.

**Required Inputs:**
- `version` (source: user) — Semantic version for the release (e.g., `v1.2.0`)
- `owner` (source: context) — Repository owner, auto-detected from git remote
- `repo` (source: context) — Repository name, auto-detected from git remote

**Steps:**
1. Create an annotated tag on the latest commit with the semver version → Tool: `create_tag(owner, repo, tag: version, message, sha)`
2. Create the GitHub release with tag, auto-generated release notes, and changelog summary → Tool: `create_release(owner, repo, tag_name: version, name: version, generate_release_notes: true)`

**Expected Output:** Release URL (e.g., `https://github.com/{owner}/{repo}/releases/tag/v1.2.0`)

**Example Invocation:** "Create release v1.2.0 with the latest changes"

---

### Skill: review-pull-requests

**Description:** Reviews all open pull requests checking for coding conventions, test presence, breaking changes, and unresolved TODO/FIXME comments, then provides a structured review summary.

**Required Inputs:**
- `owner` (source: context) — Repository owner, auto-detected from git remote
- `repo` (source: context) — Repository name, auto-detected from git remote

**Steps:**
1. Fetch all open pull requests for the repository → Tool: `list_pull_requests(owner, repo, state: "open")`
2. For each open PR, retrieve the changed files to analyze diff scope → Tool: `get_pull_request_files(owner, repo, pull_number)`
3. Submit a review with findings (conventions, tests, breaking changes, TODOs) → Tool: `create_pull_request_review(owner, repo, pull_number, body, event: "COMMENT")`

**Expected Output:** Review summary listing each PR with findings (conventions adherence, test coverage, breaking changes, unresolved TODOs)

**Example Invocation:** "Review all open pull requests and check for issues"

---

### Skill: create-spec-from-issue

**Description:** Fetches a GitHub issue and generates a structured requirements.md specification document from its content, including acceptance criteria derived from task lists and related issue context.

**Required Inputs:**
- `issue_number` (source: user) — The GitHub issue number to convert into a spec
- `owner` (source: context) — Repository owner, auto-detected from git remote
- `repo` (source: context) — Repository name, auto-detected from git remote

**Steps:**
1. Fetch the issue details (title, body, labels, assignees) → Tool: `get_issue(owner, repo, issue_number)`
2. Fetch all comments on the issue for additional context and refinements → Tool: `list_issue_comments(owner, repo, issue_number)`
3. If the issue body references other issues (`#number` syntax), fetch related issues for dependency context → Tool: `get_issue(owner, repo, related_issue_number)`
4. Transform issue content into a structured requirements.md: extract user story from title/description, derive acceptance criteria from task lists and described behaviors, incorporate relevant comment insights, map labels to requirement metadata
5. Add traceability link (`<!-- github-issue: #N -->`) in the generated spec and comment on the issue linking back to the spec

**Expected Output:** A `requirements.md` file in the spec directory (e.g., `.kiro/specs/{feature-name}/requirements.md`) with structured user story, acceptance criteria, and traceability link

**Example Invocation:** "Create a spec from issue #42"

---

### Skill: report-power-bug

**Description:** Reports a bug in the GitHub MCP Power itself to the `studiokanslia/kiro-power-github` repository using an AI-processable template with metadata comments that enable automated conversion to specs/requirements.

**Required Inputs:**
- `summary` (source: user) — One-sentence description of the bug
- `steps` (source: user) — Steps to reproduce the bug
- `expected` (source: user) — What should have happened
- `actual` (source: user) — What actually happened

**Steps:**
1. Gather bug details from user through conversational prompts
2. Auto-detect environment context (Kiro version, Docker version, Power version) if available
3. Format the issue body using the AI-processable bug template with metadata comments (`<!-- ai-processable: true -->`, `<!-- source: report-power-bug skill -->`, `<!-- convertible-to-spec: yes -->`)
4. Confirm the formatted issue with the user
5. Either file directly or present formatted content → Tool: `create_issue(owner: "studiokanslia", repo: "kiro-power-github", title, body, labels: ["bug", "user-reported"])`

**Expected Output:** Issue URL (e.g., `https://github.com/studiokanslia/kiro-power-github/issues/7`) or formatted markdown content ready to paste manually

**Example Invocation:** "Report a bug in the GitHub Power — the create-release skill fails when version has no 'v' prefix"

---

### Skill: request-power-feature

**Description:** Requests a feature for the GitHub MCP Power itself to the `studiokanslia/kiro-power-github` repository using an AI-processable template with metadata comments that enable automated conversion to specs/requirements.

**Required Inputs:**
- `title` (source: user) — Short description of the desired feature
- `motivation` (source: user) — Why this feature is needed / what problem it solves
- `desired_behavior` (source: user) — What the feature should do

**Steps:**
1. Gather feature details from user through conversational prompts
2. Help user articulate acceptance criteria (suggest if user is unsure)
3. Format the issue body using the AI-processable feature template with metadata comments (`<!-- ai-processable: true -->`, `<!-- source: request-power-feature skill -->`, `<!-- convertible-to-spec: yes -->`)
4. Confirm the formatted issue with the user
5. Either file directly or present formatted content → Tool: `create_issue(owner: "studiokanslia", repo: "kiro-power-github", title, body, labels: ["enhancement", "user-requested"])`

**Expected Output:** Issue URL (e.g., `https://github.com/studiokanslia/kiro-power-github/issues/8`) or formatted markdown content ready to paste manually

**Example Invocation:** "Request a feature for the GitHub Power — I want a skill that auto-labels PRs based on file paths changed"

## Tool Usage Examples

The following examples demonstrate common GitHub MCP tool invocations through the Docker-based server. All tools automatically receive `owner` and `repo` parameters from the detected project context (see [steering/project-context.md](steering/project-context.md)).

### 1. `create_issue` — Create a new GitHub issue

**Parameters:**
- `owner`: `"acme-corp"`
- `repo`: `"web-platform"`
- `title`: `"Fix login timeout after session expiry"`
- `body`: `"## Bug Report\n\n**Steps to reproduce:**\n1. Log in to the application\n2. Wait 30 minutes without activity\n3. Attempt any action\n\n**Expected:** Session refresh prompt\n**Actual:** Unhandled 401 error\n\n**Environment:** Production, Chrome 128"`
- `labels`: `["bug", "priority:P1"]`

Creates a structured bug report issue with reproduction steps, expected/actual behavior, and appropriate labels for triage.

### 2. `create_pull_request` — Open a new pull request

**Parameters:**
- `owner`: `"acme-corp"`
- `repo`: `"web-platform"`
- `head`: `"feature/add-dark-mode"`
- `base`: `"main"`
- `title`: `"Add dark mode support for dashboard"`
- `body`: `"## Summary\nImplements user-toggleable dark mode for the dashboard UI.\n\n## Changes\n- Added CSS custom properties for theme colors\n- Created ThemeProvider context component\n- Added toggle button to settings panel\n\n## Testing\n- Unit tests for ThemeProvider\n- Visual regression tests for both themes"`

Opens a PR from a feature branch to main with a description following the required Summary/Changes/Testing structure.

### 3. `list_issues` — List issues with filters for triage

**Parameters:**
- `owner`: `"acme-corp"`
- `repo`: `"web-platform"`
- `state`: `"open"`
- `labels`: `"bug,priority:P0"`

Retrieves all open critical bug issues for immediate triage, enabling the team to focus on the highest-priority defects first.

### 4. `search_code` — Search repository code for patterns

**Parameters:**
- `q`: `"TODO+repo:acme-corp/web-platform+language:typescript"`
- `per_page`: `10`

Finds TODO comments across TypeScript files in the codebase, useful for technical debt tracking and cleanup planning.

## Best Practices

### 1. Search before creating

Always search existing issues before creating a new one to avoid duplicates. Use `list_issues` with label filters or `search_code` with issue-related keywords to check whether the problem or feature request has already been reported. Duplicate issues fragment discussion and waste triage effort.

### 2. Use fine-grained token scopes

Prefer fine-grained personal access tokens scoped to specific repositories over classic tokens with broad organization access. This follows the principle of least privilege — if a token is compromised, exposure is limited to the repositories explicitly granted. Review and rotate tokens quarterly.

### 3. Keep PRs atomic and focused

Each pull request should address a single concern — one feature, one bugfix, or one refactor. If a PR touches multiple unrelated areas, split it into smaller PRs. Atomic PRs are easier to review, faster to merge, and simpler to revert if issues arise.

### 4. Batch related operations for consistency

When syncing specs to issues, triaging multiple items, or performing bulk label updates, batch related operations together. This reduces API calls, maintains consistency across related items, and avoids partial-update states where some issues are updated but others are not.

### 5. Pin images, never use `latest`

Always reference Docker images by both a version tag and a SHA256 digest (e.g., `ghcr.io/github/github-mcp-server:v0.2.0@sha256:<digest>`). This prevents supply chain attacks via tag mutation and ensures reproducible builds. Never use the `latest` tag in production configurations — it provides no guarantee about what code is running.

## Troubleshooting

### Container Connectivity Errors

| # | Symptom | Resolution |
|---|---------|------------|
| 1 | `Cannot connect to the Docker daemon` — container fails to start | Start the Docker service. On macOS/Windows, launch Docker Desktop. On Linux, run `sudo systemctl start docker`. Verify with `docker info`. |
| 2 | MCP connection drops immediately after container launch | Ensure the Docker run command uses the `-i` (interactive) flag and does NOT use the `-d` (detach) flag. Stdio transport requires the container to run in the foreground with stdin open. The correct flags are `docker run --rm -i ...`. |
| 3 | `manifest unknown` error or timeout when pulling the image | Verify network access to `ghcr.io`. Run `docker pull ghcr.io/github/github-mcp-server:v0.2.0` manually. If behind a corporate proxy, configure Docker's proxy settings in `~/.docker/config.json`. If the digest has changed, verify the image reference in `mcp.json` matches the current published digest. |

### Authentication Failures

| # | Symptom | Resolution |
|---|---------|------------|
| 1 | `401 Unauthorized` on all API requests | The `GITHUB_PERSONAL_ACCESS_TOKEN` secret is missing or not configured. Register your token in Kiro's secrets system (Settings → Secrets → add `GITHUB_PERSONAL_ACCESS_TOKEN`). Kiro resolves the `${secret:GITHUB_PERSONAL_ACCESS_TOKEN}` reference at runtime and injects it into the container. |
| 2 | `401 Bad credentials` after token was previously working | The token has expired or been revoked. Navigate to GitHub Settings → Developer settings → Personal access tokens, regenerate the token, and update the value in Kiro's secrets configuration. |
| 3 | `401` error on the very first request with a newly configured token | The token value may contain trailing whitespace, newline characters, or invisible Unicode characters. Copy the token again from GitHub, paste it cleanly into Kiro's secrets system, and ensure no extra characters are included before or after the token string. |

### Token Permission Issues

| # | Symptom | Resolution |
|---|---------|------------|
| 1 | `403 Resource not accessible by personal access token` | The token lacks the required scope for the operation. Identify the operation that failed, consult the token scopes table in the Configuration section above, and add the missing scope (e.g., `repo` for repository operations, `issues` for issue management, `pull_requests` for PR operations). Regenerate or edit the token in GitHub Settings. |
| 2 | `404 Not Found` when accessing a repository that exists | For fine-grained personal access tokens, the token must be explicitly granted access to the specific repository. Go to GitHub Settings → Developer settings → Fine-grained tokens → edit your token → ensure the target repository is selected under "Repository access". For classic tokens, ensure the `repo` scope is present. |
| 3 | `403 Forbidden` when accessing organization resources | The token is missing the `read:org` scope required for organization membership queries. For classic tokens, add the `read:org` scope. For fine-grained tokens, ensure the "Organization" permission is set to "Read". Additionally, check if the organization has restricted third-party access — an org admin may need to approve the token. |

## Image Update Process

When a new version of the GitHub MCP server image is released, the pinned reference in `mcp.json` must be updated carefully. Follow these steps to safely update the image while maintaining supply chain integrity.

### Step 1: Verify Provenance

Check the new release on the official GitHub repository ([github/github-mcp-server](https://github.com/github/github-mcp-server)):

- Confirm the release is published by the official maintainers
- Review the changelog for breaking changes or deprecations
- Verify no security advisories affect the new version

### Step 2: Pull and Verify the New Image

Pull the image by tag and capture the immutable SHA256 digest:

```bash
docker pull ghcr.io/github/github-mcp-server:v1.3.0
docker inspect --format='{{index .RepoDigests 0}}' ghcr.io/github/github-mcp-server:v1.3.0
```

Record the full digest output (e.g., `ghcr.io/github/github-mcp-server@sha256:<64-char-hex>`). This digest ensures the exact image bytes are verified regardless of tag mutations.

### Step 3: Test the New Version

Run the container and verify MCP tool availability by sending an initialization request:

```bash
echo '{"jsonrpc":"2.0","method":"initialize","params":{},"id":1}' | \
  docker run --rm -i -e GITHUB_PERSONAL_ACCESS_TOKEN ghcr.io/github/github-mcp-server:v1.3.0
```

Confirm:
- The container starts without errors
- The MCP server responds with a valid JSON-RPC response
- Tool listings include the expected tools (issues, pull requests, repositories, etc.)

### Step 4: Update mcp.json

Replace the image reference in `mcp.json` with the new version tag and the verified SHA256 digest:

```json
"ghcr.io/github/github-mcp-server:v1.3.0@sha256:<new-digest>"
```

The image reference MUST include both the version tag (for human readability) and the digest (for immutable verification).

### Step 5: Update CHANGELOG

Add an entry under the next version in `CHANGELOG.md` documenting the image version bump:

```markdown
### Changed
- Bumped GitHub MCP server image from v1.2.0 to v1.3.0
```

### Step 6: Create PR

File a pull request with:
- The updated `mcp.json` containing the new image reference
- The updated `CHANGELOG.md` entry

### Step 7: Tag and Release

After the PR is merged, tag a new version (PATCH for bug-fix image updates, MINOR for feature-adding image updates) to trigger the release pipeline:

```bash
git tag v0.2.0
git push origin v0.2.0
```

### Important Notes

> **Release validation:** The release workflow validates that the image in `mcp.json` is properly pinned (version tag + SHA256 digest). If the image is unpinned or uses only a tag without a digest, the release will fail.
>
> **Never use `latest`:** The `:latest` tag is mutable and provides no reproducibility or security guarantees. Always pin to a specific version tag with a digest.

## Workspace Overrides

The Power ships with sensible defaults (P0-P3 priorities, bug/feature/chore types) that work for most projects. However, many teams use different conventions — MoSCoW priorities, "defect" instead of "bug", T-shirt sizing for effort, etc. Workspace-level overrides allow you to replace these defaults entirely without modifying the Power itself.

### What Can Be Overridden

| Category | Power Default | Override Examples |
|----------|--------------|------------------|
| Priority levels | P0-critical, P1-high, P2-medium, P3-low | MoSCoW (Must have, Should have, Could have, Won't have) |
| Issue types | bug, feature, chore | defect, story, task, spike, epic |
| Type labels | `bug`, `feature`, `chore`, `documentation`, `enhancement` | `defect`, `story`, `spike`, `tech-debt`, `improvement` |
| Effort sizing | (none by default) | T-shirt (XS, S, M, L, XL), Story points (1, 2, 3, 5, 8, 13) |
| Scope labels | frontend, backend, infra, docs | Custom domain labels per project |

### Override Detection Order

Overrides are detected in the following priority order (first match wins):

1. **Workspace steering file** (preferred) — `.kiro/steering/github-overrides.md` in the project root
2. **JSON config** (alternative) — `.kiro/config.json` with a `github-power` section
3. **Power defaults** — Built-in values from `steering/issues.md`

### Override Format: Steering File (Preferred)

Create `.kiro/steering/github-overrides.md` in your workspace:

```markdown
# GitHub Power Overrides

## Priority Levels
- Must have (critical, blocks release)
- Should have (important, needed for release)
- Could have (nice to have, defer if needed)
- Won't have (explicitly out of scope for now)

## Issue Types
- defect (replaces "bug" — something broken)
- story (replaces "feature" — user-facing capability)
- task (replaces "chore" — technical/maintenance work)
- spike (time-boxed research/investigation)

## Effort Sizing
- XS (< 1 hour)
- S (1-4 hours)
- M (1-2 days)
- L (3-5 days)
- XL (> 1 week, should be broken down)
```

### Override Format: JSON Config (Alternative)

For teams preferring structured configuration, add a `github-power` section to `.kiro/config.json`:

```json
{
  "github-power": {
    "priorities": [
      { "name": "Must have", "description": "Critical, blocks release" },
      { "name": "Should have", "description": "Important, needed for release" },
      { "name": "Could have", "description": "Nice to have, defer if needed" },
      { "name": "Won't have", "description": "Explicitly out of scope" }
    ],
    "issueTypes": [
      { "name": "defect", "replaces": "bug", "description": "Something broken" },
      { "name": "story", "replaces": "feature", "description": "User-facing capability" },
      { "name": "task", "replaces": "chore", "description": "Technical/maintenance work" }
    ],
    "effortSizing": {
      "enabled": true,
      "scale": "tshirt",
      "values": ["XS", "S", "M", "L", "XL"]
    }
  }
}
```

### Common Override Examples

**MoSCoW Priorities:**
Replace P0-P3 with Must/Should/Could/Won't to align with agile frameworks that use MoSCoW prioritization.

**"Defect" Instead of "Bug":**
Many enterprise teams use "defect" terminology. Override the issue type so all issue creation, triage, and labeling operations use "defect" labels instead of "bug".

**T-shirt Sizing:**
Add effort estimation to issues using XS/S/M/L/XL categories, enabling capacity planning in GitHub Projects v2 views.

### Override Behavior

- **Full replacement** — Overrides replace the entire category. If you override priorities, define all levels you want used.
- **Label mapping** — Overridden type names are used as GitHub labels in templates and triage workflows.
- **Template adaptation** — Issue templates adapt to overridden terminology automatically.
- **Backward compatible** — Removing overrides falls back to Power defaults without configuration changes.

For full details on the override mechanism, configuration format, and detection flow, see [`steering/project-context.md`](steering/project-context.md).

## Contributing

### Dogfooding Workflow

After the v0.1.0 bootstrap, the Power's own development is managed using its own steering files and skills. This ensures real-world validation of all guidance and workflows.

The development lifecycle follows this pattern:

1. **File issue** — Report a bug or request a feature using the `report-power-bug` or `request-power-feature` skills (or file directly to `studiokanslia/kiro-power-github`)
2. **Create spec from issue** — Use the `create-spec-from-issue` skill to generate a structured requirements document from the filed issue
3. **Implement** — Develop the change following the generated spec, guided by the Power's own steering files
4. **Sync spec back to issue** — Use the spec-issue sync steering to keep the GitHub issue updated with implementation progress
5. **Close issue** — Once the change is released, the issue is closed with a reference to the release

### Development Management

- The Power's development is managed through GitHub issues in the [`studiokanslia/kiro-power-github`](https://github.com/studiokanslia/kiro-power-github) repository
- A GitHub Projects v2 board tracks the development backlog with status, priority, and iteration fields
- After v0.1.0, all changes go through the standard PR workflow guided by `steering/pull-requests.md`
- The Power uses its own steering and skills for its own development — ensuring that any friction or gaps are discovered and addressed

### How to Contribute

1. Check existing issues in `studiokanslia/kiro-power-github` for duplicates
2. File a new issue (bug or feature request) — use the Power's feedback skills or file directly
3. Reference the issue in your PR when submitting changes
4. Follow the steering conventions documented in `steering/pull-requests.md` for PR structure

## Feedback & Reporting

### Reporting Bugs or Requesting Features

To report bugs or request features **for the Power itself**, use the built-in feedback skills:

- **`report-power-bug`** — Say "Report a bug in the GitHub Power" and Kiro will guide you through filing a structured bug report
- **`request-power-feature`** — Say "Request a feature for the GitHub Power" and Kiro will help you articulate and file a feature request

Both skills always target `studiokanslia/kiro-power-github` regardless of which workspace you are currently in.

### Filing Options

- **File directly** — If you have write access to `studiokanslia/kiro-power-github`, the skill can create the issue for you immediately
- **Get formatted content** — If you lack write access, the skill provides well-formatted markdown content that you can copy and post as a new issue manually

### AI-Processable Issue Format

Issues filed through these skills are structured with metadata comments that make them **AI-processable**:

```markdown
<!-- ai-processable: true -->
<!-- source: report-power-bug skill -->
<!-- convertible-to-spec: yes -->
```

This means:
- Issues can be automatically converted into formal specs/requirements using the `create-spec-from-issue` skill
- The structured format (reproduction steps, acceptance criteria, severity) enables consistent automated processing
- Maintainers can batch-process user feedback into development specs without manual reformatting

### What Makes a Good Report

Whether you use the skills or file directly, include:

- **For bugs:** Clear reproduction steps, expected vs. actual behavior, affected component (steering file, skill, mcp.json, etc.)
- **For features:** Motivation (what problem it solves), desired behavior, and suggested acceptance criteria

All feedback is welcome — the dogfooding workflow ensures every issue becomes actionable through the spec-driven development process.

## Security

### Token Scopes

The following table lists the minimum required GitHub token scopes per operation category. Grant only the scopes necessary for your use case to follow the principle of least privilege.

| Operation | Fine-Grained Permission | Classic Scope |
|-----------|------------------------|---------------|
| Repository (read/write) | Contents: Read and Write | `repo` |
| Issues (read/write) | Issues: Read and Write | `repo` (included) |
| Pull Requests (read/write) | Pull requests: Read and Write | `repo` (included) |
| Releases (read/write) | Contents: Read and Write | `repo` (included) |
| Organization (read) | Organization: Read | `read:org` |

### Token Recommendations

**Use fine-grained personal access tokens over classic tokens.** Fine-grained tokens provide superior security because they can be:

- **Scoped to specific repositories** — Rather than granting access to all repositories in an organization, select only the repositories the Power needs to interact with. Navigate to GitHub Settings → Developer settings → Fine-grained personal access tokens → select "Only select repositories" and choose the specific repos.
- **Limited to exact permissions** — Grant only the permission categories listed in the Token Scopes table above, rather than broad scopes like `repo` that include write access to all repository features.
- **Set with short expiration** — Choose the shortest expiration appropriate for your use case. For active development, 30-90 days is recommended. For CI/CD automation, align expiration with your rotation schedule. Tokens can always be regenerated when they expire.

> **Recommendation:** If you work across multiple repositories, create one fine-grained token scoped to all of them rather than multiple tokens. This simplifies secret management while maintaining repository-level access control.

### Permission Error Handling

When a GitHub MCP tool call fails with a `403 Forbidden` response, follow this decision flowchart to resolve the issue:

```
403 Forbidden received
    │
    ├─→ Identify the failed operation (e.g., create_issue, create_pull_request)
    │
    ├─→ Map the operation to required scope from the Token Scopes table above
    │       • Issue operations → Issues: Read and Write (or `repo` for classic)
    │       • PR operations → Pull requests: Read and Write (or `repo` for classic)
    │       • Repository operations → Contents: Read and Write (or `repo` for classic)
    │       • Organization operations → Organization: Read (or `read:org` for classic)
    │
    ├─→ Check current token permissions
    │       • Fine-grained: GitHub Settings → Developer settings → Fine-grained tokens → edit token
    │       • Classic: GitHub Settings → Developer settings → Tokens (classic) → view scopes
    │
    └─→ Add the missing permission and regenerate/update the token in Kiro's secrets
```

**Common 403 scenarios:**

| Failed Operation | Missing Permission | Resolution |
|------------------|--------------------|------------|
| `create_issue`, `update_issue` | Issues: Read and Write | Add Issues permission to fine-grained token, or ensure `repo` scope on classic token |
| `create_pull_request`, `merge_pull_request` | Pull requests: Read and Write | Add Pull requests permission to fine-grained token, or ensure `repo` scope on classic token |
| `create_release`, `push_file` | Contents: Read and Write | Add Contents permission to fine-grained token, or ensure `repo` scope on classic token |
| `list_organization_repos` | Organization: Read | Add Organization permission to fine-grained token, or add `read:org` scope to classic token |

**Note:** A `404 Not Found` response when accessing a repository that definitely exists usually indicates the fine-grained token has not been granted access to that specific repository. Edit the token and add the repository under "Repository access".

### Image Pinning Policy

The Docker image for the GitHub MCP server MUST be pinned with both a version tag AND a SHA256 digest in `mcp.json`:

**Required format:**
```
ghcr.io/github/github-mcp-server:<version>@sha256:<64-hex-chars>
```

**Rules:**
- **MUST** use a specific version tag (e.g., `:v0.2.0`) for human-readable identification
- **MUST** include a SHA256 digest (e.g., `@sha256:a1b2c3d4...`) for immutable content verification
- **MUST NOT** use the `latest` tag — it is mutable and provides no reproducibility guarantee
- **MUST NOT** use bare image names without a tag or digest
- The release pipeline validates image pinning — **releases will fail if an unpinned image reference is detected** in `mcp.json`

**Why both tag and digest?** The version tag provides human readability ("we're running v0.2.0"), while the SHA256 digest provides cryptographic verification that the exact expected image bytes are used. Even if a tag is maliciously re-pushed to a different image, the digest check will fail, preventing supply chain attacks via tag mutation.

### Versioning Policy

The Power follows [Semantic Versioning 2.0.0](https://semver.org/) starting from version `0.1.0`:

| Increment | When to use | Examples |
|-----------|-------------|----------|
| **PATCH** (0.1.x) | Bug fixes, documentation updates, steering clarifications, Docker image version bumps | Fix typo in steering file, update image digest, clarify troubleshooting step |
| **MINOR** (0.x.0) | New steering files, new skills, new MCP configuration options, new sections in POWER.md | Add new steering file, document a new skill, add new tool example |
| **MAJOR** (x.0.0) | Breaking changes to Power structure, mcp.json format changes, steering file renames or removals | Rename steering files, change mcp.json schema, remove a skill |

**Release process:**
1. Update `version` in POWER.md frontmatter to the new version
2. Update CHANGELOG.md with the new version entry
3. Create and push a semver tag (e.g., `git tag v0.2.0 && git push origin v0.2.0`)
4. The release pipeline validates version consistency and creates the GitHub release automatically
