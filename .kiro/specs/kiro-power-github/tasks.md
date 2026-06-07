# Implementation Plan: kiro-power-github Power

## Overview

This implementation plan creates the kiro-power-github Guided MCP Power package incrementally. The Power bundles a Docker-based GitHub MCP server configuration, 5 steering files, 7 documented skills, and a release pipeline. Each task builds on previous ones, starting with the foundational structure (mcp.json, POWER.md skeleton) and layering in steering files, skills documentation, and automation.

Since this is a documentation and configuration package (not executable application code), all tasks involve creating or modifying markdown, JSON, and YAML files.

## Tasks

- [x] 1. Set up foundational Power structure
  - [x] 1.1 Create mcp.json with pinned Docker image configuration
    - Create `mcp.json` at the repository root
    - Declare the GitHub MCP server with `"command": "docker"` and args `["run", "--rm", "-i", "-e", "GITHUB_PERSONAL_ACCESS_TOKEN", "ghcr.io/github/github-mcp-server:<version>@sha256:<digest>"]`
    - Include `"env"` block with `"GITHUB_PERSONAL_ACCESS_TOKEN": "${secret:GITHUB_PERSONAL_ACCESS_TOKEN}"`
    - Verify the actual latest stable image tag and SHA256 digest from `ghcr.io/github/github-mcp-server`
    - _Requirements: 1.2, 1.6, 2.1, 2.2, 2.3, 2.5, 2.7, 8.4_

  - [x] 1.2 Create POWER.md with frontmatter and skeleton structure
    - Create `POWER.md` at the repository root with YAML frontmatter: `name: github-mcp`, `displayName: GitHub MCP`, `description` (≤300 chars, ≤3 sentences), `version: "0.1.0"`, `keywords` (must include github, mcp, pull-request, issues, repository), `author: studiokanslia`
    - Add section headings for all 11 required sections: Overview, Configuration, Available Steering Files, Skills, Tool Usage Examples, Best Practices, Troubleshooting, Image Update Process, Workspace Overrides, Contributing, Feedback & Reporting
    - Leave section content as placeholders to be filled in subsequent tasks
    - _Requirements: 1.1, 1.4, 1.5, 3.1, 11.4_

  - [x] 1.3 Create CHANGELOG.md with initial v0.1.0 entry
    - Create `CHANGELOG.md` at the repository root following Keep a Changelog format
    - Include header with format reference and semver adherence statement
    - Add `[Unreleased]` section
    - Add `[0.1.0] - YYYY-MM-DD` section with all initial features listed under `### Added`
    - Include comparison links at the bottom
    - _Requirements: 11.1, 11.7_

  - [x] 1.4 Create the steering/ directory structure
    - Create `steering/` directory with empty placeholder files: `pull-requests.md`, `issues.md`, `repository-management.md`, `spec-issue-sync.md`, `project-context.md`
    - Each file should have just the top-level heading as placeholder
    - _Requirements: 1.3_

- [x] 2. Implement steering files (core workflows)
  - [x] 2.1 Write steering/project-context.md
    - Implement the full steering file for workspace context detection
    - Include "When This Applies" section with trigger conditions
    - Document detection method: parse `origin` remote URL from workspace git config
    - Support both HTTPS (`https://github.com/{owner}/{repo}.git`) and SSH (`git@github.com:{owner}/{repo}.git`) formats
    - Document fallback: prompt user if no remote or unparseable URL
    - Document scoping: all MCP tool invocations receive resolved owner/repo automatically
    - Document re-detection on workspace change
    - Document first-write-operation confirmation with user
    - Document that all skills inherit detected context
    - Document override configuration: detection order (workspace `.kiro/steering/github-overrides.md` → `.kiro/config.json` → Power defaults)
    - Document what can be overridden: issue types, priority levels, label taxonomies, effort sizing
    - Include override format examples (steering file preferred, JSON alternative)
    - Include "Rules & Constraints" and "Examples" sections
    - _Requirements: 12.1, 12.2, 12.3, 12.4, 12.5, 12.6, 12.7, 12.8, 5.6, 5.7, 5.8_

  - [x] 2.2 Write steering/issues.md
    - Implement the full steering file for issue management workflows
    - Include "When This Applies" section with trigger keywords
    - Document title constraint (max 72 characters)
    - Define default issue types (Bug, Feature, Task) as GitHub native types with label fallbacks (`bug`, `feature`, `chore`, `documentation`, `enhancement`)
    - Define default priority levels (P0-critical, P1-high, P2-medium, P3-low)
    - Document these as overridable defaults — reference project-context.md for override mechanism
    - Document override behavior: when workspace overrides detected, use overridden values for all operations
    - Provide examples of alternative taxonomies (MoSCoW, T-shirt sizing, custom type vocabularies)
    - Include templates for: bug reports (title, description, steps to reproduce, expected/actual behavior, environment), feature requests (title, description, motivation, acceptance criteria), tasks (title, description, subtasks, definition of done)
    - Document triage workflow: assign priority, categorize by type, link related issues
    - Document modern GitHub features: sub-issues for hierarchical breakdown, tasklists for trackable subtasks, issue types as first-class categorization
    - Document GitHub Projects v2 integration: auto-add to project, set custom fields
    - Include "Rules & Constraints" and "Examples" sections
    - _Requirements: 5.1, 5.2, 5.3, 5.4, 5.5, 5.6, 5.7, 5.8, 13.1, 13.3, 13.4, 13.5_

  - [x] 2.3 Write steering/pull-requests.md
    - Implement the full steering file for PR workflows
    - Include "When This Applies" section with trigger conditions
    - Document PR title constraint: max 72 characters
    - Document required description sections: Summary, Changes, Testing
    - Define label taxonomy: type labels (feature, bugfix, chore, hotfix, docs) and scope labels (frontend, backend, infra, docs)
    - Document review checklist: coding conventions, test presence, breaking changes, TODO/FIXME audit
    - Document merge strategy decision matrix: squash (single-feature), merge commit (long-lived branches), rebase (≤3 commits, linear)
    - Include a populated template with example Summary, Changes, and Testing sections
    - Document validation rule: prompt user if required sections missing
    - Include "Rules & Constraints" and "Examples" sections
    - _Requirements: 4.1, 4.2, 4.3, 4.4, 4.5, 4.6_

  - [x] 2.4 Write steering/repository-management.md
    - Implement the full steering file for repository management
    - Include "When This Applies" section with trigger conditions
    - Document branch naming convention: prefixes (`feature/`, `bugfix/`, `chore/`, `release/`, `hotfix/`), separator `/`, max 100 chars
    - Document branch name validation: inform user of violation, suggest correction
    - Document release workflow: semver tag format `vMAJOR.MINOR.PATCH`, changelog entry structure (date + categories: Added, Changed, Fixed, Removed), release notes with summary + change list
    - Document repository settings guidance: default branch naming, branch protection (required reviews, status checks), merge strategy defaults
    - Document GitHub Actions automation guidance: issue triage, PR labeling, stale issue management, project board automation
    - Ensure no references to deprecated features (Projects v1, etc.)
    - Include "Rules & Constraints" and "Examples" sections
    - _Requirements: 6.1, 6.2, 6.3, 6.4, 6.5, 13.6, 13.7_

  - [x] 2.5 Write steering/spec-issue-sync.md
    - Implement the full steering file for bidirectional spec-issue synchronization
    - Include "When This Applies" section with trigger conditions
    - Document Spec → Issue direction: when spec created, offer to create GitHub issue with requirement title as issue title, acceptance criteria as checklist, `spec-driven` label, priority and type labels
    - Document Issue → Spec direction: when issue updated (comments, status, labels), reflect changes back into spec document
    - Define linking convention: `<!-- github-issue: #42 -->` in spec, spec file path in issue body (`Spec: .kiro/specs/{name}/requirements.md`)
    - Define mapping rules: one requirement → one GitHub issue; sub-tasks → sub-issues (not flat checkboxes)
    - Document update preservation: preserve manual additions/comments on issues when syncing
    - Document labels on sync: `spec-driven`, priority label, type label, related issue references
    - Document modern features: sub-issues and tasklists for hierarchical breakdown, tracked-by/tracks relationships
    - Document override awareness: types and priorities used during sync respect workspace overrides
    - Include "Rules & Constraints" and "Examples" sections
    - _Requirements: 9.1, 9.2, 9.3, 9.4, 9.5, 9.6, 9.7, 13.1, 13.4, 13.5_

- [x] 3. Checkpoint - Validate steering files
  - Ensure all 5 steering files exist in `steering/` directory with correct filenames
  - Ensure each steering file follows the consistent structure: When This Applies, Workflows, Templates, Rules & Constraints, Examples
  - Ensure all steering files reference modern GitHub features (sub-issues, issue types, Projects v2, tasklists) where appropriate
  - Ensure no deprecated features (Projects v1) are referenced
  - Ensure issues.md documents overridable defaults and project-context.md documents override configuration
  - Ensure all tests pass, ask the user if questions arise.

- [x] 4. Write POWER.md content sections
  - [x] 4.1 Write POWER.md Overview and Configuration sections
    - Fill in Overview section: explain 3 capabilities (GitHub API interaction, guided workflows via steering files, reusable skills) each in 1-2 sentences
    - Fill in Configuration section: prerequisites (Docker, Kiro, GitHub token), Docker setup, Kiro secret configuration (`GITHUB_PERSONAL_ACCESS_TOKEN` via `${secret:...}`), minimum required token scopes (repo, read:org, issues, pull_requests), fine-grained token recommendation, networking/stdio transport explanation, image pinning policy, step-by-step launch instructions
    - Document Docker-specific networking for web agent environment (stdio communication, required Docker flags)
    - Document how to configure the token in Kiro's secrets system
    - Document that token must never be embedded in images or stored in repo files
    - _Requirements: 3.1, 3.4, 3.7, 2.4, 2.6, 8.1, 8.2, 8.4, 8.5_

  - [x] 4.2 Write POWER.md Available Steering Files section
    - Create a table listing all 5 steering files with filename and 1-2 sentence description of purpose
    - _Requirements: 3.2_

  - [x] 4.3 Write POWER.md Skills section (7 skills)
    - Document all 7 skills following the schema: name, description, required inputs (user-provided vs context-derived), steps with tool invocations, expected outputs, example invocation
    - Skills: `create-feature-pr`, `file-bug-report`, `create-release`, `review-pull-requests`, `create-spec-from-issue`, `report-power-bug`, `request-power-feature`
    - Document that all skill tool invocations execute through the Docker-based GitHub MCP server container
    - Document that all skills inherit detected project context (except feedback skills which target `studiokanslia/kiro-power-github`)
    - For `create-spec-from-issue`: document inputs (owner, repo, issue number), tools (get_issue, list_issue_comments, optionally get related issues), and outputs (requirements.md)
    - For `report-power-bug`: document AI-processable bug template with metadata comments (`<!-- ai-processable: true -->`, `<!-- convertible-to-spec: yes -->`)
    - For `request-power-feature`: document AI-processable feature template with metadata comments
    - _Requirements: 7.1, 7.2, 7.3, 7.4, 7.5, 10.1, 10.2, 10.3, 10.4, 10.5, 10.6, 10.7_

  - [x] 4.4 Write POWER.md Tool Usage Examples and Best Practices sections
    - Write Tool Usage Examples: at least 3 GitHub MCP tool invocations (e.g., create_issue, create_pull_request, search_repositories) with tool name, required parameters with sample values, and 1-sentence description
    - Write Best Practices: at least 3 concrete recommendations for GitHub workflows (e.g., when to use squash merge, how to structure issue descriptions, PR review cadence)
    - _Requirements: 3.3, 3.6_

  - [x] 4.5 Write POWER.md Troubleshooting section
    - Cover at least 2 scenarios per category:
      - Container connectivity errors (container not starting, stdio transport failures)
      - Authentication failures (invalid/expired token, missing token)
      - Token permission issues (insufficient scopes, wrong repo access)
    - Each scenario includes symptom and resolution step
    - _Requirements: 3.5, 2.6_

  - [x] 4.6 Write POWER.md Image Update Process section
    - Document the step-by-step process for safely updating the pinned container image
    - Steps: verify provenance, pull and verify new image, test new version, update mcp.json, update CHANGELOG, create PR, tag and release
    - Document that the release workflow validates image pinning
    - _Requirements: 8.6, 8.7_

  - [x] 4.7 Write POWER.md Workspace Overrides, Contributing, and Feedback sections
    - Workspace Overrides: document how to configure custom issue types, priorities, labels at workspace level; reference override formats (steering file, JSON config); list what can be overridden
    - Contributing: describe the dogfooding workflow (file issue → create-spec-from-issue → implement → sync → close); state that Power development uses its own steering and skills
    - Feedback & Reporting: document how to report bugs or request features for the Power itself using the `report-power-bug` and `request-power-feature` skills; explain AI-processable format enables automated spec generation
    - _Requirements: 14.1, 14.2, 14.3, 14.5, 14.6, 14.7, 5.6, 5.8_

  - [x] 4.8 Write POWER.md Security and Image Pinning documentation
    - Document minimum required GitHub token scopes per operation category in a table (repository, issues, PRs, releases, organization)
    - Recommend fine-grained personal access tokens over classic tokens
    - Explain how to scope token to specific repositories
    - Document permission error handling: identify failed operation, map to required scope, inform user
    - Document image pinning policy: MUST use version tag + SHA256 digest, MUST NOT use `latest`, format `ghcr.io/github/github-mcp-server:<tag>@sha256:<digest>`
    - Document versioning policy: PATCH for fixes/docs, MINOR for new steering/skills, MAJOR for breaking changes
    - _Requirements: 8.1, 8.2, 8.3, 8.6, 11.5_

- [x] 5. Checkpoint - Validate POWER.md completeness
  - Ensure POWER.md has valid YAML frontmatter with all required fields
  - Ensure all 11 content sections are present and populated
  - Ensure 7 skills are documented with full schema (name, description, inputs, steps, outputs)
  - Ensure at least 3 tool usage examples are present
  - Ensure at least 3 best practices are documented
  - Ensure troubleshooting covers all 3 error categories with at least 2 scenarios each
  - Ensure all tests pass, ask the user if questions arise.

- [x] 6. Create release pipeline and finalize
  - [x] 6.1 Create .github/workflows/release.yml
    - Create `.github/workflows/release.yml` with tag-triggered pipeline
    - Trigger on push of semver tags matching `v[0-9]+.[0-9]+.[0-9]+`
    - Add step: extract version from tag
    - Add step: validate version matches POWER.md frontmatter
    - Add step: validate CHANGELOG entry exists for the version
    - Add step: validate Docker image is pinned in mcp.json (version tag + SHA256 digest)
    - Add step: create GitHub release with tag name, auto-generated release notes, and Power package files as assets (POWER.md, mcp.json, CHANGELOG.md, steering/*)
    - Set permissions to `contents: write`
    - _Requirements: 11.2, 11.3, 11.6, 11.8_

  - [x] 6.2 Update README.md with Power overview
    - Update the repository README.md with:
      - Brief description of what the Power does
      - Installation instructions (how to install as a Kiro Power)
      - Quick start guide (token setup, activation)
      - Link to POWER.md for full documentation
      - Badge for latest release version
    - _Requirements: 3.1 (supporting documentation)_

- [x] 7. Final checkpoint - Validate complete Power package
  - Verify all files exist: POWER.md, mcp.json, CHANGELOG.md, README.md, steering/pull-requests.md, steering/issues.md, steering/repository-management.md, steering/spec-issue-sync.md, steering/project-context.md, .github/workflows/release.yml
  - Verify POWER.md frontmatter version is `0.1.0`
  - Verify mcp.json image reference includes both version tag and SHA256 digest
  - Verify CHANGELOG has [0.1.0] entry
  - Verify release workflow validates version consistency, CHANGELOG presence, and image pinning
  - Verify all 14 requirements are covered by the implementation
  - Ensure all tests pass, ask the user if questions arise.

## Notes

- This is a documentation and configuration Power package — all deliverables are markdown, JSON, and YAML files
- No executable application code is produced, so property-based tests do not apply
- The Docker image tag and SHA256 digest in mcp.json must be verified against the actual `ghcr.io/github/github-mcp-server` registry at implementation time
- After v0.1.0 bootstrap, subsequent development should use the Power's own skills and steering (dogfooding)
- All steering files should reference modern GitHub 2026+ features (sub-issues, issue types, Projects v2, tasklists) and avoid deprecated features
- Workspace-level overrides for issue types and priorities are a cross-cutting concern — referenced in issues.md, project-context.md, spec-issue-sync.md, and POWER.md
- The feedback loop skills (`report-power-bug`, `request-power-feature`) always target `studiokanslia/kiro-power-github` regardless of workspace context

## Task Dependency Graph

```json
{
  "waves": [
    { "id": 0, "tasks": ["1.1", "1.3", "1.4"] },
    { "id": 1, "tasks": ["1.2"] },
    { "id": 2, "tasks": ["2.1", "2.3", "2.4"] },
    { "id": 3, "tasks": ["2.2", "2.5"] },
    { "id": 4, "tasks": ["4.1", "4.2"] },
    { "id": 5, "tasks": ["4.3", "4.4", "4.5", "4.6"] },
    { "id": 6, "tasks": ["4.7", "4.8"] },
    { "id": 7, "tasks": ["6.1", "6.2"] }
  ]
}
```
