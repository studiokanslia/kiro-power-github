# Pull Request Workflows Steering

## When This Applies

This steering is loaded when Kiro is involved in any pull request activity, including:

- Creating a new pull request
- Updating an existing pull request description or title
- Reviewing a pull request
- Deciding on a merge strategy
- Adding labels to a pull request
- Responding to PR review comments

**Trigger keywords:** pull request, PR, merge, review, code review, submit changes, open PR

---

## Workflows

### Creating a Pull Request

Follow these steps when creating a new pull request:

1. **Compose the title** — Write a concise title of **72 characters or fewer** that summarizes the change. Use imperative mood (e.g., "Add user authentication endpoint").

2. **Write the description** — The PR description MUST contain all three required sections:
   - **Summary** — A brief explanation of what this PR does and why.
   - **Changes** — A list of specific changes made (files, modules, features affected).
   - **Testing** — How the changes were tested (unit tests, manual testing, integration tests).

3. **Apply labels** — Assign at least one type label and one scope label:
   - **Type labels:** `feature`, `bugfix`, `chore`, `hotfix`, `docs`
   - **Scope labels:** `frontend`, `backend`, `infra`, `docs`

4. **Validate completeness** — Before submitting, verify all required sections are present. If any section is missing, prompt the user to provide the missing information before proceeding.

5. **Link related issues** — If the PR addresses a GitHub issue, include `Closes #<number>` or `Relates to #<number>` in the description.

### Reviewing a Pull Request

When reviewing a pull request, check the following in order:

1. **Coding conventions** — Verify the code adheres to the project's coding standards and style guidelines.
2. **Test presence** — Confirm that unit or integration tests are included for any changed or new code paths.
3. **Breaking changes** — Identify any changes to public interfaces, APIs, or contracts that could break consumers.
4. **TODO/FIXME audit** — Scan for unresolved `TODO` or `FIXME` comments that should be addressed before merging.

If any item fails the checklist, leave a review comment describing the issue and request changes.

### Choosing a Merge Strategy

Use the following decision matrix to select the appropriate merge strategy:

| Condition | Strategy | Rationale |
|-----------|----------|-----------|
| Single-feature branch with multiple commits | **Squash merge** | Produces a clean, single commit on the target branch |
| Long-lived branch with meaningful commit history | **Merge commit** | Preserves the full history of the branch development |
| Small change with 3 or fewer commits, linear history | **Rebase** | Maintains a linear commit history without merge bubbles |

**Decision flow:**
1. If the branch has ≤3 commits and all are meaningful → **Rebase**
2. If the branch is a single feature with noisy/WIP commits → **Squash**
3. If the branch is long-lived (e.g., `release/`, integration branch) → **Merge commit**

---

## Templates

### Pull Request Description Template

```markdown
## Summary

[One to three sentences explaining what this PR does and why it is needed.]

## Changes

- [Change 1: describe what was modified and in which area]
- [Change 2: describe what was added or removed]
- [Change 3: any configuration or dependency changes]

## Testing

- [How were these changes tested?]
- [Which test suites were run?]
- [Any manual verification steps performed?]
```

### Populated Example

```markdown
## Summary

Add rate limiting middleware to the API gateway to prevent abuse and ensure fair usage across tenants. This implements a token bucket algorithm with configurable limits per API key.

## Changes

- Added `src/middleware/rate-limiter.ts` with token bucket implementation
- Updated `src/routes/index.ts` to apply rate limiting to all authenticated endpoints
- Added `config/rate-limits.yaml` with default tier configurations
- Updated `package.json` with `ioredis` dependency for distributed state

## Testing

- Unit tests added for token bucket logic (`src/middleware/rate-limiter.test.ts`)
- Integration tests verify rate limiting headers are returned correctly
- Load tested with k6 script at 1000 req/s to confirm limits engage properly
- Manual verification: exceeded limit returns 429 with appropriate Retry-After header
```

---

## Rules & Constraints

1. **Title length** — PR titles MUST be 72 characters or fewer. If a title exceeds this limit, shorten it while preserving meaning.

2. **Required sections** — Every PR description MUST contain a Summary, Changes, and Testing section. If any section is missing when Kiro is creating or updating a PR, prompt the user to provide the missing information before proceeding.

3. **Label requirements** — Every PR MUST have at least one type label and one scope label applied.

4. **Type labels** (mutually exclusive — choose one):
   - `feature` — New functionality or capability
   - `bugfix` — Fix for a defect or regression
   - `chore` — Maintenance, refactoring, or dependency updates
   - `hotfix` — Urgent production fix requiring expedited review
   - `docs` — Documentation-only changes

5. **Scope labels** (one or more):
   - `frontend` — UI, client-side, or presentation layer changes
   - `backend` — Server-side, API, or business logic changes
   - `infra` — Infrastructure, CI/CD, deployment, or configuration changes
   - `docs` — Documentation, README, or steering file changes

6. **Review checklist** — Before approving any PR, all four review items must pass:
   - Coding conventions are followed
   - Tests are present for changed code
   - No undocumented breaking changes to public interfaces
   - No unresolved TODO/FIXME comments remain

7. **Merge strategy** — Always recommend the appropriate merge strategy based on the decision matrix. Do not default to a single strategy for all PRs.

8. **Validation rule** — If a pull request description is missing any required section (Summary, Changes, or Testing), Kiro MUST prompt the user to provide the missing information before proceeding with PR creation or submission.

9. **Issue linking** — When a PR resolves an issue, use GitHub closing keywords (`Closes #N`, `Fixes #N`) in the description body.

10. **Draft PRs** — For work-in-progress changes, recommend creating a draft PR to signal that the code is not yet ready for review.

---

## Examples

### Example 1: Feature PR with correct structure

**Title:** `Add webhook retry logic with exponential backoff`

**Labels:** `feature`, `backend`

**Description:**
```markdown
## Summary

Implement retry logic for failed webhook deliveries using exponential backoff with jitter. Failed webhooks are retried up to 5 times over a 24-hour window.

## Changes

- Added `WebhookRetryService` in `src/services/webhook-retry.ts`
- Added dead letter queue for webhooks that exhaust all retries
- Updated `WebhookDispatcher` to delegate failures to the retry service
- Added database migration for `webhook_delivery_attempts` table

## Testing

- Unit tests cover retry timing calculations and max-attempt logic
- Integration test verifies end-to-end retry flow with a failing endpoint
- Verified in staging with intentionally failing webhook endpoint
```

### Example 2: Bugfix PR

**Title:** `Fix race condition in session token refresh`

**Labels:** `bugfix`, `backend`

**Description:**
```markdown
## Summary

Fix a race condition where concurrent API requests could trigger multiple token refreshes simultaneously, causing some requests to fail with 401 errors.

## Changes

- Added mutex lock around token refresh in `src/auth/token-manager.ts`
- Added request queuing during active refresh cycle
- Removed redundant token validation check that caused unnecessary refreshes

## Testing

- Added concurrent request test reproducing the race condition
- Verified with 50 parallel requests that only one refresh occurs
- Existing auth test suite passes without modification
```

### Example 3: Kiro prompting for missing section

If a user asks Kiro to create a PR with only a title and summary:

> **Kiro:** I notice the PR description is missing the **Changes** and **Testing** sections. These are required for all pull requests. Could you provide:
>
> 1. **Changes** — What specific files or modules were modified?
> 2. **Testing** — How were these changes verified?
>
> I'll complete the PR once these sections are filled in.

### Example 4: Merge strategy recommendation

> **User:** "What merge strategy should I use for this PR?"
>
> **Kiro:** This PR has 7 commits on a single-feature branch (`feature/add-rate-limiting`), with several WIP and fixup commits. I recommend **squash merge** — it will combine all commits into one clean commit on main, keeping the history readable while preserving the full context in the PR itself.
