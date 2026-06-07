# Project Context Detection

Instructs Kiro on detecting and using the current workspace's repository context, and configuring workspace-level overrides for project-specific conventions.

---

## When This Applies

- When any GitHub operation is requested (creating issues, PRs, releases, syncing specs, reviewing code)
- When a new workspace is entered or switched
- When a GitHub MCP tool invocation requires `owner` and `repo` parameters
- When the user asks about repository context or workspace configuration
- When workspace-level overrides need to be detected and applied
- **Trigger keywords:** repository, context, workspace, owner, override, config

---

## Workflows

### 1. Repository Context Detection

Before executing any GitHub operation, Kiro MUST determine the target repository.

**Detection Method:**

1. Inspect the workspace's git configuration for the `origin` remote URL
2. Parse the URL to extract `owner` and `repo`
3. Apply the result to all subsequent MCP tool invocations

**Supported URL Formats:**

| Format | Pattern | Extraction |
|--------|---------|------------|
| HTTPS | `https://github.com/{owner}/{repo}.git` | Split path segments after `github.com` |
| HTTPS (no .git) | `https://github.com/{owner}/{repo}` | Split path segments after `github.com` |
| SSH | `git@github.com:{owner}/{repo}.git` | Split the part after `:` on `/` |
| SSH (no .git) | `git@github.com:{owner}/{repo}` | Split the part after `:` on `/` |

**Parsing Rules:**
- Strip trailing `.git` suffix if present
- The first path segment after the host is `owner`
- The second path segment is `repo`
- Both values must be non-empty strings

### 2. Fallback: Prompt User

If any of the following conditions are true, prompt the user for `owner` and `repo` before proceeding:

- No `.git` directory exists in the workspace
- No `origin` remote is configured
- The remote URL is not a GitHub URL (e.g., GitLab, Bitbucket, self-hosted)
- The remote URL cannot be parsed into valid `owner`/`repo` segments

**Prompt format:**
> I couldn't detect a GitHub repository from this workspace. Which repository should I target?
> Please provide the owner and repository name (e.g., `octocat/hello-world`).

### 3. Context Scoping

Once `owner` and `repo` are resolved:

- ALL GitHub MCP tool invocations automatically receive the resolved `owner` and `repo` parameters
- Users do not need to specify repository details in subsequent requests
- Tools such as `create_issue`, `create_pull_request`, `list_issues`, `search_code`, `get_pull_request_files`, etc., all inherit the resolved context

### 4. Re-Detection on Workspace Change

Kiro MUST re-detect repository context when:

- A new workspace is opened or entered
- The user explicitly changes the working directory to a different repository
- A GitHub operation is initiated and no context has been cached for the current workspace

Do NOT cache context across different workspaces. Each workspace must have its own resolved context.

### 5. First Write-Operation Confirmation

On the **first write operation** in a session (any operation that modifies GitHub state), confirm the detected repository with the user:

> I detected you're working in `{owner}/{repo}`. Is this correct?

**Write operations that trigger confirmation:**
- Creating an issue
- Creating or updating a pull request
- Creating a release or tag
- Pushing commits
- Modifying labels, milestones, or project boards
- Filing feedback issues (for skills targeting `studiokanslia/kiro-power-github`)

**Read operations that do NOT require confirmation:**
- Listing issues or PRs
- Reading file contents
- Searching repositories
- Getting issue or PR details

After the user confirms (or corrects), the confirmed context is used for the remainder of the session without re-prompting.

### 6. Skill Context Inheritance

All skills inherit the detected project context automatically:

| Skill | Context Source |
|-------|---------------|
| `create-feature-pr` | Workspace detection (current repo) |
| `file-bug-report` | Workspace detection (current repo) |
| `create-release` | Workspace detection (current repo) |
| `review-pull-requests` | Workspace detection (current repo) |
| `create-spec-from-issue` | Workspace detection (current repo) |
| `report-power-bug` | Hardcoded: `studiokanslia/kiro-power-github` |
| `request-power-feature` | Hardcoded: `studiokanslia/kiro-power-github` |

Users never need to manually specify `owner` or `repo` as inputs to skills. The only exceptions are the feedback loop skills which always target the Power's own repository regardless of workspace context.

---

## Override Configuration

### What Can Be Overridden

Projects may override the Power's default conventions at the workspace level:

| Category | Power Default | Override Examples |
|----------|--------------|------------------|
| Priority levels | P0-critical, P1-high, P2-medium, P3-low | MoSCoW (Must have, Should have, Could have, Won't have) |
| Issue types | bug, feature, chore | defect, story, task, spike, epic |
| Type labels | `bug`, `feature`, `chore`, `documentation`, `enhancement` | `defect`, `story`, `spike`, `tech-debt`, `improvement` |
| Effort sizing | (none by default) | T-shirt (XS, S, M, L, XL), Story points (1, 2, 3, 5, 8, 13) |
| Scope labels | frontend, backend, infra, docs | Custom domain labels per project |

### Override Detection Order

Overrides are detected from workspace-level configuration in the following priority order (first match wins):

1. **Workspace steering file** — `.kiro/steering/github-overrides.md` in the project root
2. **Kiro workspace config** — `.kiro/config.json` with a `github-power` section
3. **Power defaults** — Built-in values from `steering/issues.md`

Kiro MUST check for overrides before any issue creation, triage, or labeling operation.

### Override Format: Steering File (Preferred)

Create `.kiro/steering/github-overrides.md` in the workspace:

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

## Label Mapping
| Override Type | Override Label | Maps To |
|--------------|---------------|---------|
| defect | `defect` | (replaces `bug`) |
| story | `story` | (replaces `feature`) |
| task | `task` | (replaces `chore`) |
```

### Override Format: JSON Config (Alternative)

Create or add a `github-power` section in `.kiro/config.json`:

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

### Override Behavior Rules

1. **Full replacement** — Overrides replace the entire category, not individual items. If you override priorities, define all levels you want used.
2. **Label mapping** — When types are overridden, labels in templates and triage workflows use the new names.
3. **Template adaptation** — Issue templates adapt to overridden terminology (e.g., "Priority: Must have" instead of "Priority: P1-high").
4. **Backward compatibility** — If overrides are removed, the Power falls back to built-in defaults without any configuration change.

---

## Templates

### Context Detection Summary (shown to user on first operation)

```
Repository Context:
  Owner: {owner}
  Repository: {repo}
  Source: git remote origin ({format})
  Overrides: {detected | none}
```

### Override Detection Summary

```
Workspace Overrides Detected:
  Source: .kiro/steering/github-overrides.md
  Priorities: Must have, Should have, Could have, Won't have
  Issue Types: defect, story, task, spike
  Effort Sizing: T-shirt (XS, S, M, L, XL)
```

---

## Rules & Constraints

1. **Never assume repository context.** Always detect or confirm before write operations.
2. **Re-detect on workspace change.** Do not carry context from one workspace to another.
3. **Confirm before writing.** The first write operation in a session must trigger user confirmation of the detected context.
4. **Skills inherit context.** No skill should require the user to manually specify `owner`/`repo` (except feedback skills targeting the Power's own repo).
5. **Override precedence is strict.** Workspace steering file wins over JSON config, which wins over Power defaults. Never merge across levels.
6. **Full category replacement.** Overrides replace an entire category (e.g., all priorities), not individual items within a category.
7. **Non-GitHub remotes are unsupported.** If the remote URL points to a non-GitHub host, prompt the user rather than guessing.
8. **Token is separate from context.** Repository context detection is independent of authentication. A valid token is required separately.
9. **Feedback skills are exempt.** The `report-power-bug` and `request-power-feature` skills always target `studiokanslia/kiro-power-github` regardless of workspace context.
10. **Overrides apply to all issue operations.** Once overrides are loaded, they apply to issue creation, triage, labeling, spec-issue sync, and any skill that uses type/priority values.

---

## Examples

### Example 1: HTTPS Remote Detection

**Workspace git config:**
```
[remote "origin"]
    url = https://github.com/acme-corp/web-platform.git
```

**Resolved context:**
- Owner: `acme-corp`
- Repo: `web-platform`

**First write operation prompt:**
> I detected you're working in `acme-corp/web-platform`. Is this correct?

### Example 2: SSH Remote Detection

**Workspace git config:**
```
[remote "origin"]
    url = git@github.com:octocat/hello-world.git
```

**Resolved context:**
- Owner: `octocat`
- Repo: `hello-world`

### Example 3: No Remote Configured

**Workspace:** A local git repository with no remote.

**Kiro response:**
> I couldn't detect a GitHub repository from this workspace. Which repository should I target?
> Please provide the owner and repository name (e.g., `octocat/hello-world`).

### Example 4: MoSCoW Priority Override

**File:** `.kiro/steering/github-overrides.md`
```markdown
# GitHub Power Overrides

## Priority Levels
- Must have (critical path, release blocker)
- Should have (high value, release goal)
- Could have (desirable, defer if needed)
- Won't have (acknowledged, not this cycle)
```

**Effect:** When Kiro triages issues or creates bug reports in this workspace, it uses "Must have / Should have / Could have / Won't have" instead of "P0 / P1 / P2 / P3".

### Example 5: Custom Type Vocabulary

**File:** `.kiro/config.json`
```json
{
  "github-power": {
    "issueTypes": [
      { "name": "defect", "replaces": "bug", "description": "Something broken" },
      { "name": "story", "replaces": "feature", "description": "User-facing capability" },
      { "name": "task", "replaces": "chore", "description": "Technical work" }
    ]
  }
}
```

**Effect:** When Kiro creates issues, it uses the label `defect` instead of `bug`, `story` instead of `feature`, etc.

### Example 6: T-shirt Effort Sizing

**File:** `.kiro/steering/github-overrides.md`
```markdown
## Effort Sizing
- XS (< 1 hour)
- S (1-4 hours)
- M (1-2 days)
- L (3-5 days)
- XL (> 1 week, should be broken down)
```

**Effect:** Kiro includes effort sizing labels (e.g., `size:M`) when creating issues in this workspace.

### Example 7: Skill Using Inherited Context

**User says:** "Create a bug report for the login timeout issue"

**Kiro's internal flow:**
1. Detect context from git remote → `owner: acme-corp`, `repo: web-platform`
2. Check for overrides → `.kiro/steering/github-overrides.md` found with MoSCoW priorities
3. First write operation → Confirm: "I detected you're working in `acme-corp/web-platform`. Is this correct?"
4. User confirms
5. Execute `file-bug-report` skill with `owner: acme-corp`, `repo: web-platform`, priority options: Must have / Should have / Could have / Won't have
