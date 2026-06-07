# Spec-Issue Synchronization Steering

Guides Kiro through bidirectional synchronization between spec documents (requirements.md) and GitHub issues, ensuring traceability between specifications and issue tracker state.

---

## When This Applies

- When a new spec (requirements.md) is created or updated in the workspace
- When Kiro is asked to create GitHub issues from spec requirements
- When a GitHub issue linked to a spec is updated (comments, status, labels)
- When the user wants to sync spec state with issue tracker state
- When creating sub-issues or breaking down spec tasks into trackable issues
- When the user asks about spec-to-issue mapping or traceability

**Trigger keywords:** spec, sync, issue, requirement, traceability, spec-driven, link issue, create issue from spec, update spec

---

## Workflows

### 1. Spec → Issue: Creating Issues from Specs

When a spec (requirements.md) is created or a new requirement is added, offer to create a corresponding GitHub issue.

**Steps:**

1. **Identify requirements** — Parse the spec document for requirement headings and their acceptance criteria.
2. **Offer issue creation** — Ask the user: "I found N requirements in this spec. Would you like me to create GitHub issues for them?"
3. **Resolve context** — Use the workspace project context (see `project-context.md`) to determine `owner` and `repo`.
4. **Check for overrides** — Load workspace-level overrides for issue types and priority levels before creating issues.
5. **Create issues** — For each requirement, create a GitHub issue with:
   - **Title:** The requirement title (e.g., "Power Package Structure")
   - **Body:** Acceptance criteria formatted as a tasklist, spec link, and metadata
   - **Labels:** `spec-driven`, appropriate type label, priority label
   - **Sub-issues:** If the requirement has sub-tasks, create them as sub-issues (not flat checkboxes)
6. **Add linking comment** — Insert the `<!-- github-issue: #N -->` comment into the spec document next to the corresponding requirement.
7. **Confirm completion** — Report created issues to the user with issue numbers and URLs.

**Issue Body Structure:**

```markdown
**Spec:** `.kiro/specs/{name}/requirements.md`

## Acceptance Criteria

- [ ] Criterion 1 description
- [ ] Criterion 2 description
- [ ] Criterion 3 description

## Context

[User story or requirement description from the spec]

---
_This issue was created from a spec document and is managed via spec-issue sync._
_Do not remove the spec link above — it enables bidirectional traceability._
```

### 2. Issue → Spec: Reflecting Issue Updates into Specs

When a GitHub issue linked to a spec is updated, reflect relevant changes back into the spec document.

**Triggers for sync-back:**

- New comments that refine acceptance criteria or add requirements
- Status changes (closed → mark requirement as completed in spec)
- Label changes (priority or type change → update spec metadata)
- New sub-issues added to a linked parent issue

**Steps:**

1. **Identify linked spec** — From the issue body, extract the spec file path (`Spec: .kiro/specs/{name}/requirements.md`).
2. **Determine change type** — Classify the update (comment, status, label, sub-issue).
3. **Propose spec update** — Show the user the proposed change to the spec and ask for confirmation before modifying.
4. **Apply update** — Modify the spec document with the confirmed changes.
5. **Preserve issue state** — Do NOT modify the issue when syncing back — only update the spec.

**Change Type Mapping:**

| Issue Change | Spec Update |
|--------------|-------------|
| New comment refining acceptance criteria | Add or update acceptance criterion |
| Issue closed | Add `<!-- status: completed -->` annotation to requirement |
| Priority label changed | Update priority metadata in spec |
| Type label changed | Update type metadata in spec |
| New sub-issue created | Add sub-task entry under the requirement |
| Comment with design decision | Add note to requirement context or constraints |

### 3. Hierarchical Breakdown: Sub-Issues and Tasklists

When a spec requirement has sub-tasks or a complex acceptance criteria set, use GitHub's modern hierarchical features instead of flat checkboxes.

**Mapping Rules:**

| Spec Structure | GitHub Structure |
|----------------|-----------------|
| Top-level requirement | Parent issue |
| Sub-tasks within a requirement | Sub-issues (child issues) |
| Acceptance criteria | Tasklist items in parent issue body |
| Related requirements | `tracked-by` / `tracks` relationships |
| Dependent requirements | Issue references with relationship context |

**Creating Sub-Issues:**

1. Create the parent issue from the requirement.
2. For each sub-task in the spec, create a sub-issue linked to the parent.
3. Use GitHub's tasklist syntax to create trackable sub-issues:

```markdown
```[tasklist]
### Sub-tasks
- [ ] #101 — Implement authentication flow
- [ ] #102 — Add rate limiting middleware
- [ ] #103 — Write integration tests
```​
```

4. Link sub-issues to the parent using `tracked-by` / `tracks` relationships.
5. Each sub-issue gets its own `spec-driven` label and references the parent requirement.

### 4. Linking Convention

Maintain bidirectional traceability between specs and issues using these conventions:

**In the spec document (requirements.md):**

Add an HTML comment immediately after the requirement heading:

```markdown
### Requirement 3: Authentication Flow
<!-- github-issue: #42 -->
```

For requirements with multiple linked issues (sub-issues):

```markdown
### Requirement 3: Authentication Flow
<!-- github-issue: #42 -->
<!-- sub-issues: #43, #44, #45 -->
```

**In the GitHub issue body:**

Include the spec file path in the issue body:

```markdown
**Spec:** `.kiro/specs/user-auth/requirements.md`
```

For sub-issues, also reference the parent requirement:

```markdown
**Spec:** `.kiro/specs/user-auth/requirements.md` (Requirement 3)
**Parent:** #42
```

### 5. Update Preservation

When syncing spec changes to existing issues, preserve any manual additions:

**What to preserve:**
- Comments added by team members (discussions, design decisions, context)
- Additional labels added manually (beyond `spec-driven`, type, priority)
- Assignees set on the issue
- Milestone assignments
- Project board placements and custom field values
- Reactions and emoji responses
- Additional checklist items added manually to the issue body

**What to update:**
- Acceptance criteria checklist (add new items, update wording of existing items)
- Issue title (if requirement title changed in spec)
- `spec-driven` label and related labels (priority, type)
- Spec link in issue body (if spec path changed)

**Update strategy:**
1. Fetch the current issue body.
2. Identify the "Acceptance Criteria" section by its heading.
3. Merge spec criteria with existing issue criteria — add new items, update changed items, do NOT remove manually-added items.
4. Preserve all content outside the "Acceptance Criteria" section unchanged.
5. If in doubt about a change, ask the user before modifying the issue.

### 6. Labels on Sync

When creating or updating issues from specs, apply the following labels:

**Always applied:**
- `spec-driven` — Indicates the issue originated from and is managed by a spec document

**Type labels** (one per issue — respects workspace overrides):
- Default: `bug`, `feature`, `chore`, `documentation`, `enhancement`
- With overrides: Use workspace-configured type labels (e.g., `defect`, `story`, `task`)

**Priority labels** (one per issue — respects workspace overrides):
- Default: `P0-critical`, `P1-high`, `P2-medium`, `P3-low`
- With overrides: Use workspace-configured priority labels (e.g., `Must have`, `Should have`, etc.)

**Relationship references:**
- When issues are related, add `Related to #N` in the issue body
- When issues have dependencies, use `tracked-by` / `tracks` relationships
- When issues are part of a parent, use sub-issue linking

### 7. Override Awareness

Issue types and priority labels used during sync MUST respect workspace-level overrides:

1. **Before any sync operation**, check for workspace overrides (see `project-context.md` for detection order).
2. **If overrides are detected**, use the overridden values for:
   - Type labels applied to created issues
   - Priority labels applied to created issues
   - Terminology in issue body text (e.g., "defect" instead of "bug")
   - Triage suggestions when syncing back from issues to specs
3. **If no overrides are detected**, use Power defaults (P0-P3, bug/feature/chore).

---

## Templates

### Spec Requirement → Issue Body Template

```markdown
**Spec:** `.kiro/specs/{spec-name}/requirements.md`

## User Story

{user_story_from_requirement}

## Acceptance Criteria

- [ ] {criterion_1}
- [ ] {criterion_2}
- [ ] {criterion_3}

## Related Issues

{related_issue_references_or_none}

---
_Managed by spec-issue sync. Source: `.kiro/specs/{spec-name}/requirements.md` (Requirement {N})._
```

### Sub-Issue Body Template

```markdown
**Spec:** `.kiro/specs/{spec-name}/requirements.md` (Requirement {N}, Sub-task {M})
**Parent:** #{parent_issue_number}

## Description

{sub_task_description}

## Definition of Done

- [ ] {done_criterion_1}
- [ ] {done_criterion_2}

---
_Sub-issue of #{parent_issue_number}. Managed by spec-issue sync._
```

### Spec Document Linking Template

```markdown
### Requirement {N}: {Title}
<!-- github-issue: #{issue_number} -->
<!-- sub-issues: #{sub_1}, #{sub_2} -->

**User Story:** ...

#### Acceptance Criteria

1. ...
2. ...
```

### Sync Status Summary Template

```
Spec-Issue Sync Summary:
  Spec: .kiro/specs/{name}/requirements.md
  Requirements: {total_count}
  Linked issues: {linked_count}
  Unlinked requirements: {unlinked_count}
  Stale issues (spec changed): {stale_count}
  
  Actions available:
  - Create issues for {unlinked_count} unlinked requirements
  - Update {stale_count} stale issues with latest spec changes
  - Sync back {pending_count} issue updates into spec
```

---

## Rules & Constraints

1. **One requirement → one issue.** Each top-level requirement in a spec maps to exactly one GitHub issue. Do not combine multiple requirements into a single issue.

2. **Sub-tasks → sub-issues.** Sub-tasks within a requirement MUST be created as sub-issues (child issues) using GitHub's native sub-issue feature. Do NOT flatten sub-tasks into checkbox lists on the parent issue.

3. **Always offer, never auto-sync.** When a spec is created or updated, offer to sync with issues. Do not automatically create or modify issues without user confirmation.

4. **Preserve manual additions.** When syncing spec changes to issues, never remove content that was manually added to the issue (comments, extra labels, assignees, custom fields, additional checklist items).

5. **Bidirectional linking is mandatory.** Every spec-driven issue MUST have the spec file path in its body, and every linked requirement MUST have the `<!-- github-issue: #N -->` comment in the spec.

6. **`spec-driven` label is required.** All issues created from specs MUST have the `spec-driven` label applied. This label identifies issues that are managed by the sync process.

7. **Respect workspace overrides.** Always check for workspace-level overrides before creating or updating issues. Use overridden type and priority labels when configured.

8. **Confirm before modifying specs.** When syncing issue updates back into a spec document, always show the proposed changes to the user and get confirmation before modifying the file.

9. **Use modern GitHub features.** Prefer sub-issues, tasklists, and `tracked-by`/`tracks` relationships over manual cross-references. Do NOT use deprecated features (Projects v1, classic issue references without relationship semantics).

10. **Linking format is stable.** The `<!-- github-issue: #N -->` comment format is a contract — do not change the format as it is used for programmatic parsing during sync operations.

11. **Issue title matches requirement title.** When the requirement title changes in the spec, update the linked issue title to match. The spec is the source of truth for titles.

12. **Handle missing issues gracefully.** If a linked issue has been deleted or transferred, inform the user and offer to create a replacement issue or update the spec link.

---

## Examples

### Example 1: Creating Issues from a New Spec

**User creates a spec with 3 requirements:**

```markdown
### Requirement 1: User Authentication
### Requirement 2: Role-Based Access Control
### Requirement 3: Session Management
```

**Kiro offers:**
> I found 3 requirements in this spec. Would you like me to create GitHub issues for them?
>
> 1. "User Authentication" — 4 acceptance criteria
> 2. "Role-Based Access Control" — 5 acceptance criteria
> 3. "Session Management" — 3 acceptance criteria
>
> I'll apply the `spec-driven` label and appropriate type/priority labels to each.

**After creation:**
> Created 3 issues:
> - #101 — User Authentication (`feature`, `P1-high`)
> - #102 — Role-Based Access Control (`feature`, `P1-high`)
> - #103 — Session Management (`feature`, `P2-medium`)
>
> I've added issue references to the spec document.

### Example 2: Sync with Workspace Overrides (MoSCoW)

**Workspace has `.kiro/steering/github-overrides.md` with MoSCoW priorities.**

**Kiro creates issues using overridden values:**
> Created issue #45 — "Payment Gateway Integration"
> - Labels: `spec-driven`, `story`, `Must have`
> - Spec link added: `<!-- github-issue: #45 -->`

### Example 3: Issue Update → Spec Sync-Back

**A team member adds a comment on issue #42:**
> "We also need to handle the case where the OAuth provider is unreachable. Adding a 30-second timeout with graceful degradation."

**Kiro detects the update and proposes:**
> Issue #42 (User Authentication) has a new comment suggesting an additional requirement. I propose adding this acceptance criterion to the spec:
>
> > "IF the OAuth provider is unreachable, THEN the system SHALL time out after 30 seconds and degrade gracefully to cached credentials."
>
> Shall I add this to Requirement 1's acceptance criteria?

### Example 4: Hierarchical Breakdown with Sub-Issues

**Spec requirement with sub-tasks:**

```markdown
### Requirement 5: API Rate Limiting
<!-- github-issue: #50 -->
<!-- sub-issues: #51, #52, #53 -->

#### Sub-tasks:
- Implement token bucket algorithm (#51)
- Add per-endpoint configuration (#52)
- Create rate limit dashboard (#53)
```

**GitHub parent issue #50 body:**

```markdown
**Spec:** `.kiro/specs/api-gateway/requirements.md`

## Acceptance Criteria

- [ ] Rate limiting applies to all authenticated API endpoints
- [ ] Configurable limits per endpoint and per API key tier
- [ ] Rate limit headers included in all responses

```[tasklist]
### Sub-tasks
- [ ] #51 — Implement token bucket algorithm
- [ ] #52 — Add per-endpoint configuration
- [ ] #53 — Create rate limit dashboard
```​

---
_Managed by spec-issue sync._
```

### Example 5: Preserving Manual Additions During Sync

**Original issue #42 body (created from spec):**
```markdown
## Acceptance Criteria
- [ ] System authenticates users via OAuth 2.0
- [ ] System supports Google and GitHub providers
```

**Team member manually adds a checklist item:**
```markdown
## Acceptance Criteria
- [ ] System authenticates users via OAuth 2.0
- [ ] System supports Google and GitHub providers
- [ ] (Manual) Rate limit OAuth callback endpoint to 10 req/min
```

**Spec is updated with a new criterion. Kiro syncs:**
```markdown
## Acceptance Criteria
- [ ] System authenticates users via OAuth 2.0
- [ ] System supports Google and GitHub providers
- [ ] (Manual) Rate limit OAuth callback endpoint to 10 req/min
- [ ] System falls back to cached credentials when provider unreachable
```

The manually-added item `(Manual) Rate limit...` is preserved during the sync.

### Example 6: Sync Status Check

**User asks:** "What's the sync status of my auth spec?"

**Kiro responds:**
> Spec-Issue Sync Summary:
>   Spec: .kiro/specs/user-auth/requirements.md
>   Requirements: 5
>   Linked issues: 4
>   Unlinked requirements: 1 (Requirement 5: Session Timeout)
>   Stale issues (spec changed): 1 (#42 — title changed)
>   
>   Actions available:
>   - Create issue for "Session Timeout" requirement
>   - Update issue #42 title to match spec change
>
> Would you like me to proceed with these actions?
