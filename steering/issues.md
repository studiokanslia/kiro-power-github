# Issue Management Workflows

Guides Kiro through issue creation, triage, lifecycle management, and modern GitHub issue features. Defines **overridable defaults** for issue types and priority levels.

---

## When This Applies

- When the user asks Kiro to create, update, or close a GitHub issue
- When the user asks Kiro to triage, prioritize, or categorize issues
- When the user asks to file a bug report, feature request, or task
- When the user asks to organize work breakdown using issues
- When the user asks about issue templates, labels, or projects
- **Trigger keywords:** issue, bug, feature, task, triage, priority, label, report, defect, enhancement, sub-issue, project board

---

## Workflows

### 1. Issue Creation

When creating any issue, follow these steps:

1. **Detect project context** — Resolve `owner` and `repo` from workspace (see `project-context.md`)
2. **Check for workspace overrides** — Load custom types/priorities if configured (see Override Behavior below)
3. **Determine issue type** — Classify as Bug, Feature, or Task (or overridden equivalents)
4. **Compose title** — Max 72 characters, summarizing the issue concisely
5. **Fill template** — Use the appropriate template for the issue type (see Templates section)
6. **Assign labels** — Apply type label, priority label, and any scope labels
7. **Set issue type** — Use GitHub's native issue type system where available; fall back to labels
8. **Link related issues** — Reference related issues by number if applicable
9. **Add to project** — If a GitHub Projects v2 board is configured, auto-add the issue and set custom fields (Status, Priority, Type)

### 2. Issue Triage

When triaging one or more issues:

1. **Assess priority** — Assign one priority level based on impact and urgency:
   - **P0-critical** — System down, data loss, security vulnerability; requires immediate action
   - **P1-high** — Major feature broken, significant user impact; address within current sprint
   - **P2-medium** — Moderate impact, workaround exists; schedule for upcoming sprint
   - **P3-low** — Minor inconvenience, cosmetic issue; address when capacity allows

2. **Categorize by type** — Assign one type label:
   - **Bug** (`bug`) — Something is broken or behaving incorrectly
   - **Feature** (`feature`) — New capability or user-facing functionality
   - **Task/Chore** (`chore`) — Technical maintenance, refactoring, infrastructure

3. **Link related issues** — Find and reference issues that share the same component, root cause, or user workflow. Use `Related to #N` in the issue body.

4. **Break down large issues** — If an issue is too large for a single sprint:
   - Create sub-issues for each discrete unit of work
   - Use tasklists (`- [ ] #N`) for trackable progress
   - Assign the parent issue as an epic or tracking issue

5. **Set project fields** — Update GitHub Projects v2 custom fields:
   - Status: Triage → Ready → In Progress → Done
   - Priority: Set from triage assessment
   - Sprint/Iteration: Assign based on priority and capacity

### 3. Issue Lifecycle

| Stage | Actions |
|-------|---------|
| **Open** | Create issue, assign labels and priority, add to project |
| **Triage** | Review, assess priority, categorize, break down if needed |
| **Ready** | Requirements clear, acceptance criteria defined, sized |
| **In Progress** | Linked to branch/PR, sub-issues being worked |
| **Review** | PR submitted, linked via `Closes #N` or `Fixes #N` |
| **Done** | PR merged, issue auto-closed, verified in production |

### 4. Using Sub-Issues for Hierarchical Breakdown

Instead of flat checkbox lists, use GitHub's native sub-issue feature for hierarchical work breakdown:

1. **Create the parent issue** — Describes the overall feature or epic
2. **Create sub-issues** — Each sub-issue is a discrete, assignable unit of work
3. **Link via tasklist** — In the parent issue body, use tasklist syntax for trackable sub-issues:

```markdown
- [ ] #101 Design the API schema
- [ ] #102 Implement the endpoint
- [ ] #103 Write integration tests
- [ ] #104 Update documentation
```

4. **Track progress** — Sub-issues provide automatic progress tracking on the parent
5. **Use tracked-by/tracks relationships** — Express dependencies between issues using GitHub's native linking rather than manual cross-references

### 5. Using GitHub Issue Types

GitHub's native issue type system provides first-class categorization beyond labels:

- **Bug** — Use for defects, regressions, broken behavior
- **Feature** — Use for new capabilities and enhancements
- **Task** — Use for maintenance, chores, and infrastructure work

**Behavior:**
- When the repository has issue types enabled, set the issue type directly via the API
- When issue types are unavailable, fall back to applying type labels (`bug`, `feature`, `chore`)
- When workspace overrides are active, use the overridden type names (e.g., `defect` instead of `bug`)

### 6. GitHub Projects v2 Integration

When a GitHub Projects v2 board is associated with the repository:

1. **Auto-add issues** — Newly created issues should be added to the active project
2. **Set custom fields** — Apply values to project-specific custom fields:
   - `Status`: Set to "Triage" or "Ready" based on completeness
   - `Priority`: Set from the assigned priority level
   - `Type`: Set from the issue type
   - `Sprint`/`Iteration`: Set if the issue is scheduled
   - `Size`/`Effort`: Set if effort sizing is configured (see workspace overrides)
3. **Board views** — Issues appear in Board view columns matching their Status field
4. **Automation** — Leverage project automation for status transitions:
   - Issue closed → Status moves to "Done"
   - PR linked → Status moves to "In Progress"
   - Issue labeled with priority → Priority field set automatically

---

## Default Types and Priorities

### Default Issue Types

These are the Power's built-in issue types. They serve as **overridable defaults** — see Override Behavior for customization.

| Type | GitHub Issue Type | Label Fallback | Description |
|------|-------------------|----------------|-------------|
| Bug | Bug | `bug` | Something is broken or behaving incorrectly |
| Feature | Feature | `feature` | New capability or user-facing functionality |
| Task | Task | `chore` | Technical maintenance, refactoring, infrastructure |

**Additional labels** (not types, used for further classification):
- `documentation` — Documentation improvements
- `enhancement` — Improvement to existing functionality

### Default Priority Levels

| Priority | Label | Description | Response Expectation |
|----------|-------|-------------|---------------------|
| P0-critical | `priority:P0` | System down, data loss, security vulnerability | Immediate |
| P1-high | `priority:P1` | Major feature broken, significant user impact | Current sprint |
| P2-medium | `priority:P2` | Moderate impact, workaround exists | Upcoming sprint |
| P3-low | `priority:P3` | Minor inconvenience, cosmetic | When capacity allows |

---

## Override Behavior

The default issue types and priority levels defined above are **workspace-overridable**. Projects may define custom taxonomies that replace these defaults entirely.

### How Overrides Work

1. Before any issue creation, triage, or labeling operation, Kiro checks for workspace-level overrides
2. Override detection follows the priority order defined in `project-context.md`:
   - `.kiro/steering/github-overrides.md` (preferred)
   - `.kiro/config.json` with `github-power` section (alternative)
   - Power defaults from this file (fallback)
3. When overrides are detected, the overridden values **replace** the defaults for ALL issue operations in that workspace

### Override Application

When workspace overrides are active:
- Issue creation uses overridden type names and labels
- Triage workflow uses overridden priority levels
- Templates adapt to use overridden terminology
- GitHub Projects v2 fields use overridden values
- Spec-issue sync uses overridden types and priorities

### Common Alternative Taxonomies

**MoSCoW Priorities** (replaces P0-P3):
| Priority | Description |
|----------|-------------|
| Must have | Critical path, release blocker |
| Should have | High value, release goal |
| Could have | Desirable, defer if needed |
| Won't have | Acknowledged, not this cycle |

**T-shirt Effort Sizing** (supplementary, not a priority replacement):
| Size | Estimate |
|------|----------|
| XS | < 1 hour |
| S | 1-4 hours |
| M | 1-2 days |
| L | 3-5 days |
| XL | > 1 week (should be broken down) |

**Custom Type Vocabularies** (replaces bug/feature/chore):
| Custom Type | Replaces | Description |
|-------------|----------|-------------|
| defect | bug | Something broken or regressed |
| story | feature | User-facing capability |
| task | chore | Technical/maintenance work |
| spike | (new) | Time-boxed research/investigation |
| epic | (new) | Large body of work to break down |

For full override configuration details and format examples, see `project-context.md` → Override Configuration section.

---

## Templates

### Bug Report Template

```markdown
## Bug Report

**Title:** [max 72 chars — concise summary of the defect]

**Type:** Bug
**Priority:** [P0-critical | P1-high | P2-medium | P3-low]
**Labels:** `bug`, `priority:{level}`, `{scope}`

### Description
[Brief description of what is broken]

### Steps to Reproduce
1. [First step]
2. [Second step]
3. [Third step]

### Expected Behavior
[What should happen when following the steps above]

### Actual Behavior
[What actually happens — include error messages, screenshots if applicable]

### Environment
- OS: [e.g., macOS 14.5, Ubuntu 24.04]
- Browser/Runtime: [e.g., Chrome 126, Node.js 22]
- Version: [application version or commit SHA]
- Relevant config: [any non-default configuration]

### Additional Context
[Any other information — logs, related issues, workarounds discovered]
```

### Feature Request Template

```markdown
## Feature Request

**Title:** [max 72 chars — concise summary of desired capability]

**Type:** Feature
**Priority:** [P1-high | P2-medium | P3-low]
**Labels:** `feature`, `priority:{level}`, `{scope}`

### Description
[What capability is being requested]

### Motivation
[Why this feature is needed — what problem does it solve? Who benefits?]

### Acceptance Criteria
- [ ] [Criterion 1 — specific, testable condition]
- [ ] [Criterion 2 — specific, testable condition]
- [ ] [Criterion 3 — specific, testable condition]

### Alternatives Considered
[Other approaches considered and why they were rejected or deferred]

### Additional Context
[Mockups, related features, dependencies, links to discussions]
```

### Task Template

```markdown
## Task

**Title:** [max 72 chars — concise summary of the work]

**Type:** Task
**Priority:** [P1-high | P2-medium | P3-low]
**Labels:** `chore`, `priority:{level}`, `{scope}`

### Description
[What needs to be done and why]

### Subtasks
- [ ] #[sub-issue-number] [Subtask 1 description]
- [ ] #[sub-issue-number] [Subtask 2 description]
- [ ] #[sub-issue-number] [Subtask 3 description]

### Definition of Done
- [ ] [Completion criterion 1]
- [ ] [Completion criterion 2]
- [ ] [Completion criterion 3]

### Additional Context
[Related documentation, design decisions, dependencies]
```

---

## Rules & Constraints

1. **Title length limit.** Issue titles MUST NOT exceed 72 characters. If a proposed title is longer, truncate or rephrase it before creating the issue.
2. **Type assignment is mandatory.** Every issue MUST have a type assigned — either via GitHub's native issue type system or via a type label.
3. **Priority assignment for bugs.** Bug reports MUST have a priority level assigned at creation time.
4. **Use sub-issues, not flat checkboxes.** When breaking down work, create sub-issues and link them via tasklists. Do not use flat checkbox lists (`- [ ] do thing`) for trackable work items.
5. **Respect workspace overrides.** When workspace-level overrides are detected, use the overridden values for ALL operations. Never mix default and overridden values.
6. **Override replaces entirely.** Overrides replace a full category (all priorities or all types), not individual items. Partially overriding one priority level is not supported.
7. **Labels match types.** The label applied to an issue MUST correspond to its assigned type. If the type is overridden (e.g., "defect"), use the overridden label (e.g., `defect`), not the default (`bug`).
8. **No deprecated features.** Do not use GitHub Projects v1 (classic project boards). Use Projects v2 exclusively.
9. **Native linking over manual references.** Use GitHub's tracked-by/tracks relationships and sub-issues for dependencies rather than manual "See also #N" text in bodies.
10. **Issue type fallback.** If the repository does not support GitHub's native issue types, fall back to labels for categorization. Do not fail or skip categorization.
11. **One issue per concern.** Each issue should address a single bug, feature, or task. If multiple concerns are bundled, split them into separate issues linked via sub-issues.
12. **Reproduction steps required for bugs.** Bug reports MUST include steps to reproduce, expected behavior, and actual behavior. Prompt the user if any section is missing.

---

## Examples

### Example 1: Creating a Bug Report

**User says:** "File a bug — the login page crashes when you enter an email with a plus sign"

**Kiro's actions:**
1. Detect context → `owner: acme-corp`, `repo: web-platform`
2. Check overrides → None found, using Power defaults
3. Compose issue:
   - Title: "Login page crashes with plus sign in email address" (52 chars)
   - Type: Bug (native) / Label: `bug`
   - Priority: P1-high (major feature broken)
4. Fill bug report template with details from user
5. Create issue via `create_issue` tool
6. Add to Projects v2 board with Status: "Triage", Priority: "P1"

**Result:** `Created issue #47: "Login page crashes with plus sign in email address"`

### Example 2: Creating a Feature Request

**User says:** "I want to request dark mode support for the dashboard"

**Kiro's actions:**
1. Detect context → `owner: acme-corp`, `repo: web-platform`
2. Check overrides → None found
3. Compose issue:
   - Title: "Add dark mode support for dashboard" (36 chars)
   - Type: Feature / Label: `feature`
   - Priority: P2-medium
4. Prompt user for acceptance criteria
5. Create issue with feature request template

### Example 3: Triage with MoSCoW Overrides

**Workspace has `.kiro/steering/github-overrides.md` with MoSCoW priorities.**

**User says:** "Triage the open issues"

**Kiro's actions:**
1. Detect context → `owner: acme-corp`, `repo: web-platform`
2. Check overrides → MoSCoW priorities detected
3. List open issues via `list_issues`
4. For each issue, assess and assign:
   - Issue #50: Security vulnerability → Priority: "Must have"
   - Issue #51: UI alignment bug → Priority: "Could have"
   - Issue #52: Performance optimization → Priority: "Should have"
5. Apply priority labels using overridden names: `priority:must-have`, `priority:should-have`, `priority:could-have`

### Example 4: Breaking Down Work with Sub-Issues

**User says:** "Create an issue for the user authentication feature and break it down"

**Kiro's actions:**
1. Create parent issue: "Implement user authentication" (Type: Feature, P1-high)
2. Create sub-issues:
   - #61: "Design authentication API schema"
   - #62: "Implement login endpoint"
   - #63: "Implement token refresh flow"
   - #64: "Add rate limiting to auth endpoints"
   - #65: "Write integration tests for auth"
3. Update parent issue body with tasklist:
   ```markdown
   - [ ] #61 Design authentication API schema
   - [ ] #62 Implement login endpoint
   - [ ] #63 Implement token refresh flow
   - [ ] #64 Add rate limiting to auth endpoints
   - [ ] #65 Write integration tests for auth
   ```
4. Add all issues to Projects v2 board

### Example 5: Using Custom Type Vocabulary

**Workspace has overrides: types = defect, story, task, spike**

**User says:** "File a defect for the payment timeout issue"

**Kiro's actions:**
1. Detect context and overrides → Custom types active
2. Recognize "defect" as the overridden type (replaces "bug")
3. Use `defect` label instead of `bug`
4. Apply bug report template (same structure, adapted terminology)
5. Title: "Payment gateway timeout on large cart checkout" (48 chars)
6. Create issue with labels: `defect`, `priority:P1`

### Example 6: GitHub Projects v2 Auto-Integration

**User says:** "Create a task to upgrade the database to PostgreSQL 17"

**Kiro's actions:**
1. Create issue: "Upgrade database to PostgreSQL 17" (Type: Task, P2-medium)
2. Auto-add to the repository's active Projects v2 board
3. Set custom fields:
   - Status: "Ready"
   - Priority: "P2-medium"
   - Type: "Task"
   - Sprint: (leave unset — to be assigned during planning)
4. Result: Issue appears on board in "Ready" column with all fields populated
