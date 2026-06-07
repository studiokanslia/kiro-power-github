# Repository Management

Guides Kiro through branch management, release workflows, repository configuration, and GitHub Actions automation for modern GitHub repositories (2026+).

---

## When This Applies

This steering applies when Kiro is asked to:

- Create, rename, or validate branch names
- Manage releases (tagging, changelog entries, release notes)
- Configure or advise on repository settings (branch protection, merge strategies)
- Set up or modify GitHub Actions workflows for automation
- Advise on repository best practices

**Trigger keywords:** branch, release, tag, repo settings, repository, merge strategy, protection rules, automation, workflow, changelog

---

## Workflows

### Branch Naming Convention

All branches MUST follow the naming pattern:

```
{prefix}/{description}
```

**Allowed Prefixes:**

| Prefix | Purpose | Example |
|--------|---------|---------|
| `feature/` | New functionality or capability | `feature/add-user-authentication` |
| `bugfix/` | Fix for a known defect | `bugfix/resolve-login-timeout` |
| `chore/` | Maintenance, refactoring, dependency updates | `chore/update-docker-image-pin` |
| `release/` | Release preparation and version bumps | `release/v1.2.0` |
| `hotfix/` | Urgent production fix | `hotfix/patch-token-validation` |

**Rules:**

- Maximum branch name length: **100 characters** (including prefix)
- Separator between prefix and description: `/` (forward slash)
- Description uses **kebab-case** (lowercase, words separated by hyphens)
- No uppercase letters, underscores, or special characters in the description portion
- No trailing slashes or double slashes

### Branch Name Validation

WHEN a user requests a branch name that violates the convention:

1. **Inform** the user of the specific violation (e.g., "Branch name exceeds 100 characters" or "Missing required prefix")
2. **Suggest** a corrected branch name that conforms to the convention
3. **Confirm** with the user before proceeding with the corrected name

**Common violations and corrections:**

| Violation | Example | Suggested Correction |
|-----------|---------|---------------------|
| Missing prefix | `add-login-page` | `feature/add-login-page` |
| Wrong separator | `feature_add-login` | `feature/add-login` |
| Too long (>100 chars) | `feature/implement-the-entire-...` | Shorten description to essential terms |
| Uppercase letters | `feature/Add-Login-Page` | `feature/add-login-page` |
| Spaces in name | `feature/add login page` | `feature/add-login-page` |

---

### Release Workflow

Releases follow Semantic Versioning 2.0.0 and are automated via the release pipeline.

#### Semantic Versioning Tag Format

Tags MUST use the format: `vMAJOR.MINOR.PATCH`

| Increment | When to Use | Example |
|-----------|-------------|---------|
| PATCH | Bug fixes, documentation updates, steering file clarifications, image version bumps | `v0.1.0` → `v0.1.1` |
| MINOR | New steering files, new skills, new MCP configuration options, new override capabilities | `v0.1.0` → `v0.2.0` |
| MAJOR | Breaking changes to Power structure, mcp.json format changes, steering file renames | `v0.2.0` → `v1.0.0` |

#### Release Steps

1. **Create release branch**: `release/vMAJOR.MINOR.PATCH`
2. **Update CHANGELOG.md**: Move items from `[Unreleased]` to the new version section
3. **Update POWER.md frontmatter**: Set `version` field to match the release version
4. **Verify Docker image pin**: Confirm mcp.json references a pinned image (tag + SHA256 digest)
5. **Create PR** to merge release branch into `main`
6. **After merge**: Push semver tag `vMAJOR.MINOR.PATCH` to trigger the release pipeline
7. **Verify**: Confirm GitHub Release is created with assets and release notes

#### Changelog Entry Structure

Each version entry follows the [Keep a Changelog](https://keepachangelog.com/) format:

```markdown
## [MAJOR.MINOR.PATCH] - YYYY-MM-DD

### Added
- New features or capabilities introduced

### Changed
- Modifications to existing functionality

### Fixed
- Bug fixes and corrections

### Removed
- Features or capabilities that were removed
```

**Rules for changelog entries:**

- Date format: `YYYY-MM-DD` (ISO 8601)
- Only include categories that have entries (omit empty categories)
- Each entry starts with a capital letter and describes the change concisely
- Reference issue numbers where applicable (e.g., "Fixed token validation error (#42)")

#### Release Notes Format

Release notes (generated for GitHub Releases) include:

```markdown
# Release vMAJOR.MINOR.PATCH

## Summary
[2-3 sentence overview of what this release delivers]

## Changes

### Added
- [list of new features/capabilities]

### Changed
- [list of modifications]

### Fixed
- [list of bug fixes]

### Removed
- [list of removed items]

## Upgrade Notes
[Any steps users need to take when upgrading, or "No action required."]
```

---

### Repository Settings Guidance

#### Default Branch Naming

- The default branch SHOULD be named `main`
- Avoid `master`, `develop`, or non-standard default branch names for new repositories
- If migrating, create `main` from the existing default and update branch protection rules

#### Branch Protection Rules

Recommended protection rules for the `main` branch:

| Setting | Recommended Value | Rationale |
|---------|-------------------|-----------|
| Require pull request reviews | Enabled, minimum 1 reviewer | Ensures code review for all changes |
| Require status checks | Enabled | Prevents merging broken code |
| Require branches to be up to date | Enabled | Prevents merge conflicts in CI |
| Restrict push access | Enabled (limit to maintainers) | Prevents direct pushes to main |
| Require linear history | Optional (team preference) | Cleaner git log if enabled |
| Allow force pushes | Disabled | Prevents history rewriting |
| Allow deletions | Disabled | Prevents accidental branch deletion |

#### Merge Strategy Defaults

| Scenario | Recommended Strategy | Rationale |
|----------|---------------------|-----------|
| Single-feature branches | **Squash merge** | Clean single commit per feature |
| Long-lived branches with meaningful history | **Merge commit** | Preserves branch history |
| Small changes (≤3 commits), linear | **Rebase** | Maintains linear history |
| Release branches | **Merge commit** | Preserves release history |
| Hotfix branches | **Squash merge** | Single commit for the fix |

**Repository-level defaults:**
- Enable "Squash merging" as the default merge button option
- Enable "Allow merge commits" for release and long-lived branches
- Enable "Allow rebase merging" for small, linear changes
- Configure "Default to PR title" for squash commit messages

---

### GitHub Actions Automation Guidance

Modern repositories leverage GitHub Actions for automation beyond CI/CD. The following workflows are recommended for repository management.

#### Issue Triage Automation

Automate initial issue processing to reduce manual toil:

```yaml
# .github/workflows/issue-triage.yml
name: Issue Triage

on:
  issues:
    types: [opened]

jobs:
  triage:
    runs-on: ubuntu-latest
    permissions:
      issues: write
      contents: read
    steps:
      - name: Add to project board
        uses: actions/add-to-project@v1
        with:
          project-url: https://github.com/orgs/{org}/projects/{number}
          github-token: ${{ secrets.PROJECT_TOKEN }}

      - name: Apply initial labels
        uses: actions/github-script@v7
        with:
          script: |
            const issue = context.payload.issue;
            const labels = [];
            
            // Auto-detect type from issue type or title keywords
            if (issue.type === 'Bug' || issue.title.toLowerCase().includes('bug')) {
              labels.push('bug');
            } else if (issue.type === 'Feature') {
              labels.push('enhancement');
            }
            
            // Add needs-triage label for human review
            labels.push('needs-triage');
            
            if (labels.length > 0) {
              await github.rest.issues.addLabels({
                owner: context.repo.owner,
                repo: context.repo.repo,
                issue_number: issue.number,
                labels: labels
              });
            }
```

#### PR Labeling Automation

Automatically label PRs based on changed files:

```yaml
# .github/workflows/pr-labeler.yml
name: PR Labeler

on:
  pull_request:
    types: [opened, synchronize]

jobs:
  label:
    runs-on: ubuntu-latest
    permissions:
      pull-requests: write
      contents: read
    steps:
      - uses: actions/labeler@v5
        with:
          repo-token: ${{ secrets.GITHUB_TOKEN }}
          configuration-path: .github/labeler.yml
```

Supporting labeler configuration (`.github/labeler.yml`):

```yaml
steering:
  - changed-files:
      - any-glob-to-any-file: 'steering/**'

documentation:
  - changed-files:
      - any-glob-to-any-file: ['*.md', 'docs/**']

infrastructure:
  - changed-files:
      - any-glob-to-any-file: ['.github/**', 'mcp.json']
```

#### Stale Issue Management

Automatically manage inactive issues and PRs:

```yaml
# .github/workflows/stale.yml
name: Stale Issues

on:
  schedule:
    - cron: '0 6 * * 1'  # Every Monday at 6am UTC

jobs:
  stale:
    runs-on: ubuntu-latest
    permissions:
      issues: write
      pull-requests: write
    steps:
      - uses: actions/stale@v9
        with:
          stale-issue-message: >
            This issue has been inactive for 30 days. It will be closed in 7 days
            if no further activity occurs. Add a comment to keep it open.
          stale-pr-message: >
            This PR has been inactive for 14 days. Please update or close it.
          days-before-stale: 30
          days-before-close: 7
          days-before-pr-stale: 14
          days-before-pr-close: -1
          stale-issue-label: stale
          stale-pr-label: stale
          exempt-issue-labels: 'pinned,security,Must have'
          exempt-pr-labels: 'pinned,do-not-close'
```

#### Project Board Automation

Automate project board status transitions using GitHub Projects v2 workflows:

- **Auto-add to project**: Issues and PRs are automatically added when created
- **Status transitions**:
  - Issue opened → Status: "Backlog"
  - Issue assigned → Status: "In Progress"
  - PR opened (linked to issue) → Status: "In Review"
  - PR merged (linked to issue) → Status: "Done"
  - Issue closed → Status: "Done"

Configure these via **GitHub Projects v2 built-in workflows** (Settings → Workflows in the project):

| Workflow | Trigger | Action |
|----------|---------|--------|
| Item added | Issue/PR added to project | Set Status = "Backlog" |
| Item reopened | Issue reopened | Set Status = "In Progress" |
| PR merged | PR merged | Set Status = "Done" |
| Item closed | Issue/PR closed | Set Status = "Done" |

> **Note:** GitHub Projects v2 built-in workflows are preferred over custom Actions for status management. Use custom Actions only for logic that exceeds built-in capabilities (e.g., cross-project sync, external notifications).

---

## Templates

### Release PR Template

```markdown
## Release vX.Y.Z

### Summary
[Brief description of what this release includes]

### Checklist
- [ ] CHANGELOG.md updated with all changes
- [ ] POWER.md frontmatter version updated to X.Y.Z
- [ ] Docker image pin verified in mcp.json (tag + SHA256)
- [ ] All steering files reviewed for accuracy
- [ ] Release notes draft prepared

### Changes Included
[List PRs or commits included in this release]
```

### Branch Creation Template

When creating a branch for the user:

```
Creating branch: {prefix}/{kebab-case-description}

Prefix: {prefix} ({purpose})
Description: {description}
Full name: {full-branch-name} ({length} characters)
```

---

## Rules & Constraints

1. **Branch names** MUST NOT exceed 100 characters total (including prefix and separator)
2. **Branch names** MUST start with one of the allowed prefixes (`feature/`, `bugfix/`, `chore/`, `release/`, `hotfix/`)
3. **Release tags** MUST follow the `vMAJOR.MINOR.PATCH` format exactly
4. **CHANGELOG entries** MUST include the date in `YYYY-MM-DD` format
5. **Docker image references** in mcp.json MUST be pinned with both a version tag and SHA256 digest — never use `latest` or untagged references
6. **POWER.md version** MUST match the release tag at the time of release
7. **Never** reference deprecated GitHub features such as classic Projects (v1) — always use Projects v2
8. **Merge strategy** defaults to squash unless the user explicitly requests otherwise or the scenario matches merge commit/rebase criteria
9. **Branch protection** recommendations apply to `main` — other branches may have relaxed rules
10. **Stale issue management** MUST exempt security issues and highest-priority items from auto-closure
11. **GitHub Actions** workflows SHOULD use the latest stable major versions of official actions (e.g., `actions/checkout@v4`, `actions/stale@v9`)
12. **Project board automation** SHOULD prefer GitHub Projects v2 built-in workflows over custom Actions where possible
13. All release validations (version match, CHANGELOG presence, image pinning) are enforced by the release pipeline and MUST pass before a release is created

---

## Examples

### Example 1: Creating a Feature Branch

**User:** "Create a branch for adding dark mode support"

**Kiro Response:**
```
Creating branch: feature/add-dark-mode-support

Prefix: feature/ (new functionality)
Description: add-dark-mode-support
Full name: feature/add-dark-mode-support (32 characters)
```

### Example 2: Validating an Invalid Branch Name

**User:** "Create a branch called Add_User_Authentication_Module_For_Enterprise_Customers_With_SSO_Support_And_MFA_Integration_Complete"

**Kiro Response:**
> The requested branch name has the following issues:
> - Missing a required prefix (`feature/`, `bugfix/`, `chore/`, `release/`, `hotfix/`)
> - Contains uppercase letters (should be kebab-case)
> - Contains underscores (should use hyphens)
> - Would exceed 100 characters with a prefix
>
> Suggested correction: `feature/add-enterprise-sso-mfa-auth`
>
> Would you like me to use this corrected name?

### Example 3: Preparing a Release

**User:** "Prepare a release for version 0.2.0"

**Kiro performs:**
1. Creates branch `release/v0.2.0`
2. Updates `CHANGELOG.md`:
   ```markdown
   ## [0.2.0] - 2026-01-15

   ### Added
   - New steering file for code review workflows
   - Skill: automated-review for PR quality checks

   ### Fixed
   - Branch name validation now catches trailing slashes
   ```
3. Updates `POWER.md` frontmatter: `version: "0.2.0"`
4. Verifies mcp.json image pin is valid
5. Opens PR: "Release v0.2.0" with the release checklist template

### Example 4: Recommending a Merge Strategy

**User:** "What merge strategy should I use for this PR?"

**Kiro evaluates:**
- Number of commits on the branch
- Branch type (feature, release, hotfix)
- Whether history is meaningful

**Example response for a 1-commit feature branch:**
> I recommend **squash merge** for this PR because:
> - It's a single-feature branch (`feature/add-dark-mode-support`)
> - Squash merge will produce one clean commit on `main`
> - The branch history isn't meaningful to preserve

### Example 5: Setting Up Issue Triage Automation

**User:** "Set up automated issue triage for our repo"

**Kiro creates:**
1. `.github/workflows/issue-triage.yml` with auto-labeling and project board integration
2. Configures the workflow to use GitHub's native issue types for categorization
3. Adds `needs-triage` label for items requiring human review
4. Integrates with GitHub Projects v2 for automatic board placement
