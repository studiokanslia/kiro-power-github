# Requirements Document

## Introduction

This document defines the requirements for the **kiro-power-github** Power — a Guided MCP Power that bundles GitHub MCP server configuration, steering files for guiding Kiro's GitHub interactions, and skills for common GitHub workflows. The Power enables Kiro to interact with the GitHub API (creating issues, pull requests, managing repositories, reviewing code, etc.) through the official GitHub MCP server, guided by curated steering rules and reusable skill actions.

The Power MUST run inside a Docker container to ensure compatibility with web agent and cloud sandbox execution environments. The known working Docker command is: `docker run --rm -i -e GITHUB_PERSONAL_ACCESS_TOKEN ghcr.io/github/github-mcp-server`. The `GITHUB_PERSONAL_ACCESS_TOKEN` is provided by Kiro's secret configuration system and passed into the container at runtime via the `-e` flag.

## Glossary

- **Power**: A Kiro Power package that bundles documentation (POWER.md), optional MCP server configuration (mcp.json), and optional steering files to provide capabilities to Kiro.
- **MCP_Server**: A Model Context Protocol server that exposes tools for an AI agent to interact with external services; in this case, the GitHub MCP server (`ghcr.io/github/github-mcp-server`) running inside a Docker container.
- **Steering_File**: A markdown file placed in the `steering/` directory of a Power that provides contextual instructions, workflows, and best practices for Kiro to follow during specific tasks.
- **Skill**: A reusable workflow or action described within the Power documentation or steering files that Kiro can perform, such as creating a pull request with a standardized description.
- **POWER_MD**: The main documentation file (`POWER.md`) that contains frontmatter metadata, an overview, tool usage examples, and best practices for the Power.
- **MCP_JSON**: The configuration file (`mcp.json`) that declares MCP server connection details including command, arguments, and environment variables for launching the Docker container.
- **GitHub_API**: The GitHub REST and GraphQL APIs exposed through the GitHub MCP server's tools (e.g., create_issue, create_pull_request, search_repositories).
- **Frontmatter**: YAML metadata at the top of POWER.md delimited by `---` lines, containing name, displayName, description, keywords, and author fields.
- **Kiro_Secret**: A secret value (such as `GITHUB_PERSONAL_ACCESS_TOKEN`) managed by Kiro's configuration system and injected into the MCP server container at runtime; not a user-managed environment variable but a value provided through Kiro's secrets infrastructure.
- **Docker_Container**: The isolated execution environment (`ghcr.io/github/github-mcp-server`) in which the GitHub MCP server runs, communicating with the host agent via stdio transport.
- **Web_Agent**: The cloud-based sandbox execution environment in which Kiro operates, requiring all MCP servers to run inside Docker containers rather than as native host processes.
- **Semver**: Semantic Versioning 2.0.0 specification (MAJOR.MINOR.PATCH) used to communicate the nature of changes between Power releases.
- **Release_Pipeline**: A GitHub Actions workflow that automates the creation of versioned releases when a semver tag is pushed to the repository.
- **Project_Context**: The workspace-level configuration that identifies which GitHub repository is associated with the current development environment, derived from the workspace's git remote URL.
- **Dogfooding**: The practice of using the Power's own capabilities (steering, skills, spec-issue sync) to develop and manage the Power itself, ensuring real-world validation of its guidance and tools.
- **GitHub_Projects_v2**: GitHub's modern project management system featuring customizable views (Board, Table, Roadmap), custom fields, automation workflows, and iteration tracking.
- **Sub_Issues**: GitHub's native parent-child issue relationship feature allowing hierarchical work breakdown, replacing flat checkbox lists with trackable linked issues.
- **Image_Pinning**: The practice of referencing a Docker container image by a specific version tag or SHA256 digest rather than a mutable tag (e.g., `latest`), ensuring reproducible builds and preventing supply chain attacks through tag mutation.
- **Workspace_Overrides**: Configuration values defined at the workspace or project context level that override the Power's default settings for issue types, priority levels, label taxonomies, and other project-specific conventions (e.g., using MoSCoW priorities instead of P0-P3, or "defect" instead of "bug").

## Requirements

### Requirement 1: Power Package Structure

**User Story:** As a Kiro user, I want the GitHub Power to follow the standard Kiro Power structure, so that it can be installed and discovered correctly by Kiro.

#### Acceptance Criteria

1. THE Power SHALL contain a `POWER.md` file at the root of the Power directory with YAML frontmatter delimited by `---` lines, parseable as valid YAML, and containing `name`, `displayName`, and `description` fields each with a non-empty string value.
2. THE Power SHALL contain an `mcp.json` file at the root of the Power directory that declares the GitHub MCP server configuration as a valid JSON object with at least one server entry containing `command`, `args`, and `env` fields.
3. THE Power SHALL contain a `steering/` directory with at least one `.md` file providing workflow guidance.
4. THE Frontmatter SHALL use the name `github-mcp` with a displayName of `GitHub MCP` and a description summarizing the Power's capabilities in no more than 3 sentences and no more than 300 characters.
5. THE Frontmatter SHALL include a `keywords` array containing at minimum the terms `github`, `mcp`, `pull-request`, `issues`, and `repository`.
6. THE mcp.json SHALL configure the GitHub MCP server to run inside a Docker container using the `ghcr.io/github/github-mcp-server` image, ensuring the Power is compatible with containerized web agent execution environments.



### Requirement 2: MCP Server Configuration

**User Story:** As a Kiro user, I want the Power to configure the GitHub MCP server correctly, so that Kiro can connect to GitHub and use its API tools.

#### Acceptance Criteria

1. THE MCP_JSON SHALL declare a server entry with `"command": "docker"` and an arguments array starting with `"run"`, `"--rm"`, `"-i"` followed by `-e`, `GITHUB_PERSONAL_ACCESS_TOKEN`, and the image name `ghcr.io/github/github-mcp-server`.
2. THE MCP_JSON SHALL reference the `GITHUB_PERSONAL_ACCESS_TOKEN` secret as provided by Kiro's configuration system, and SHALL pass it into the Docker container using the `-e GITHUB_PERSONAL_ACCESS_TOKEN` argument so the container receives the token at runtime.
3. THE MCP_JSON SHALL use the `stdio` transport type for communication with the MCP server, and the Docker container SHALL run in interactive mode (the `-i` flag) without detaching to enable stdin/stdout communication.
4. THE POWER_MD SHALL document the minimum required GitHub personal access token scopes (at minimum: `repo`, `read:org`, `issues`, `pull_requests`) and SHALL include instructions for configuring the token in Kiro's secrets configuration.
5. THE MCP_JSON SHALL specify `"docker"` as the command and SHALL include `"--rm"` in the arguments array to ensure the container is removed after the MCP server process exits, preventing accumulation of stopped containers.
6. IF the Docker daemon is not running or the `ghcr.io/github/github-mcp-server` image cannot be pulled, THEN THE POWER_MD SHALL document the error condition and instruct the user to verify Docker is installed, running, and has network access to `ghcr.io`.
7. THE MCP_JSON SHALL pin the GitHub MCP server Docker image to a specific version tag or SHA256 digest (e.g., `ghcr.io/github/github-mcp-server:v1.2.0` or `ghcr.io/github/github-mcp-server@sha256:abc123...`) rather than using the `latest` tag, to prevent supply chain attacks from tag mutation or image replacement.



### Requirement 3: Power Documentation (POWER.md)

**User Story:** As a Kiro user, I want comprehensive documentation in POWER.md, so that I can understand how to use the GitHub Power effectively.

#### Acceptance Criteria

1. THE POWER_MD SHALL include an Overview section explaining what the Power provides, listing at least 3 capabilities (e.g., GitHub API interaction, guided workflows via steering files, reusable skills) each described in 1-2 sentences.
2. THE POWER_MD SHALL include an Available Steering Files section listing all steering files, where each entry includes the file name and a 1-2 sentence description of its purpose.
3. THE POWER_MD SHALL include a Tool Usage Examples section demonstrating at least 3 GitHub MCP tool invocations, where each example includes the tool name, required parameters with sample values, and a one-sentence description of what the invocation accomplishes.
4. THE POWER_MD SHALL include a Configuration section documenting prerequisites, the `GITHUB_PERSONAL_ACCESS_TOKEN` secret (provided via Kiro configuration), Docker image reference, the Docker run command with environment variable passing, container stdio transport setup, and step-by-step instructions for launching the MCP server inside a container.
5. THE POWER_MD SHALL include a Troubleshooting section covering at least 2 scenarios per category for: container connectivity errors (e.g., container not starting, stdio transport failures), authentication failures (e.g., invalid or expired token), and token permission issues (e.g., insufficient scopes for requested operation), where each scenario includes the symptom and a resolution step.
6. THE POWER_MD SHALL include a Best Practices section with guidance on when to use specific GitHub tools and recommended workflows, including at least 3 concrete recommendations.
7. THE POWER_MD Configuration section SHALL document Docker-specific networking considerations for the web agent environment, including how the containerized MCP server communicates with the host agent via stdio and any required Docker flags for the container runtime.



### Requirement 4: Steering for Pull Request Workflows

**User Story:** As a developer, I want steering guidance for pull request workflows, so that Kiro follows consistent practices when creating, reviewing, and managing pull requests.

#### Acceptance Criteria

1. THE Power SHALL include a steering file named `pull-requests.md` for pull request workflows in the `steering/` directory.
2. WHEN Kiro creates a pull request, THE Steering_File SHALL instruct Kiro to include a title of 72 characters or fewer, a description containing at minimum a Summary section, a Changes section, and a Testing section, and one or more labels from the categories: type (e.g., feature, bugfix, chore) and scope (e.g., frontend, backend, docs).
3. WHEN Kiro reviews a pull request, THE Steering_File SHALL instruct Kiro to check for: adherence to project coding conventions, presence of unit or integration tests for changed code, breaking changes to public interfaces, and unresolved TODO or FIXME comments, before approving.
4. WHEN Kiro is asked which merge strategy to use for a pull request, THE Steering_File SHALL provide selection criteria specifying squash for single-feature branches, merge commit for long-lived branches with meaningful history, and rebase for linear history on small changes with fewer than 3 commits.
5. THE Steering_File SHALL include at least one template showing a pull request description with populated Summary, Changes, and Testing sections.
6. IF a pull request description is missing any of the required sections (Summary, Changes, or Testing), THEN THE Steering_File SHALL instruct Kiro to prompt the user to provide the missing information before proceeding.



### Requirement 5: Steering for Issue Management Workflows

**User Story:** As a developer, I want steering guidance for issue management, so that Kiro follows consistent practices when creating and triaging issues.

#### Acceptance Criteria

1. THE Power SHALL include a steering file for issue management workflows in the `steering/` directory.
2. WHEN Kiro creates an issue, THE Steering_File SHALL instruct Kiro to include a title of no more than 72 characters summarizing the issue, reproduction steps (for bugs), acceptance criteria (for features), and labels selected from a defined set (e.g., `bug`, `feature`, `chore`, `documentation`, `enhancement`).
3. WHEN Kiro triages issues, THE Steering_File SHALL provide guidance on assigning one of the defined priority levels (P0-critical, P1-high, P2-medium, P3-low), categorizing by type (bug, feature, chore), and linking related issues by referencing issue numbers that share the same component or root cause.
4. THE Steering_File SHALL include templates for bug reports (with sections: title, description, steps to reproduce, expected behavior, actual behavior, environment), feature requests (with sections: title, description, motivation, acceptance criteria), and task issues (with sections: title, description, subtasks, definition of done).
5. WHEN Kiro creates a bug report issue, THE Steering_File SHALL instruct Kiro to include at least the steps to reproduce, expected behavior, and actual behavior sections, and to assign a priority level.
6. THE Steering_File SHALL define default issue types (e.g., `bug`, `feature`, `chore`) and default priority levels (e.g., P0-critical, P1-high, P2-medium, P3-low) as overridable defaults that can be replaced at the workspace context level, allowing projects to use alternative taxonomies such as MoSCoW priorities (Must have, Should have, Could have, Won't have) or alternative type names (e.g., `defect` instead of `bug`, `story` instead of `feature`).
7. WHEN workspace-level overrides for issue types or priority levels are detected in the Project_Context, THE Steering_File SHALL instruct Kiro to use the overridden values instead of the Power's built-in defaults for all issue creation, triage, and labeling operations.
8. THE Steering_File SHALL document the override mechanism, specifying where workspace-level configuration for custom issue types and priority levels is defined (e.g., in the project's steering or context configuration), and SHALL provide examples of common alternative taxonomies (MoSCoW, T-shirt sizing, custom type vocabularies).



### Requirement 6: Steering for Repository Management Workflows

**User Story:** As a developer, I want steering guidance for repository management, so that Kiro can help with branch management, releases, and repository configuration.

#### Acceptance Criteria

1. THE Power SHALL include a steering file for repository management workflows in the `steering/` directory with a filename that clearly identifies it as the repository management guide.
2. WHEN Kiro creates a branch, THE Steering_File SHALL instruct Kiro to follow a naming convention that specifies required prefixes (`feature/`, `bugfix/`, `chore/`, `release/`, `hotfix/`), a separator character between prefix and description, and a maximum branch name length of 100 characters.
3. IF a requested branch name does not conform to the defined naming convention, THEN THE Steering_File SHALL instruct Kiro to inform the user of the violation and suggest a corrected branch name before proceeding.
4. WHEN Kiro manages releases, THE Steering_File SHALL provide instructions on semantic versioning tag format (e.g., `vMAJOR.MINOR.PATCH`), changelog entry structure including date and category grouping (Added, Changed, Fixed, Removed), and release note formatting with a summary section and a list of included changes.
5. WHEN Kiro configures or advises on repository settings, THE Steering_File SHALL include guidance covering default branch naming, branch protection rules (required reviews, status checks), and merge strategy configuration (squash, merge, rebase defaults).



### Requirement 7: Skills for Common GitHub Actions

**User Story:** As a developer, I want predefined skills for common GitHub actions, so that Kiro can quickly perform routine tasks without requiring detailed instructions each time.

#### Acceptance Criteria

1. THE POWER_MD SHALL document reusable skills that Kiro can perform, including: creating a feature branch with PR, filing a bug report, creating a release, and reviewing open PRs.
2. WHEN a user requests a skill by name, THE Power SHALL provide Kiro with all necessary steps and tool invocations to complete the action, where completion is defined as the skill producing its documented expected output (e.g., a PR URL, an issue number, a release tag, or a summary of reviewed PRs).
3. THE POWER_MD SHALL describe each skill with a name, description, required inputs (specifying which are user-provided and which are derived from repository context), the sequence of GitHub MCP server tools invoked, and expected outputs.
4. THE Power SHALL document at least five skills: `create-feature-pr`, `file-bug-report`, `create-release`, `review-pull-requests`, and `create-spec-from-issue`.
5. THE POWER_MD SHALL specify that all skill tool invocations execute through the Docker-based GitHub MCP server container (`ghcr.io/github/github-mcp-server`) and are compatible with containerized web agent environments.



### Requirement 8: Authentication and Security

**User Story:** As a developer, I want clear authentication guidance, so that the Power connects to GitHub securely with appropriate permissions.

#### Acceptance Criteria

1. THE POWER_MD SHALL document the minimum required GitHub token scopes for each of the following operation categories: repository (read/write), issues (read/write), pull requests (read/write), and releases (read/write), listing no more than the scopes necessary for the tools exposed by the MCP server in that category.
2. THE POWER_MD SHALL recommend using fine-grained personal access tokens over classic tokens, and SHALL explain how to scope the token to specific repositories rather than granting organization-wide access.
3. IF an MCP tool call fails due to insufficient permissions, THEN THE Steering_File SHALL instruct Kiro to identify the failed operation, map it to the required token scope from the documented scope table, and inform the user which specific scope must be added to their token.
4. THE POWER_MD SHALL document that the `GITHUB_PERSONAL_ACCESS_TOKEN` is provided by Kiro's configuration system as a secret, and SHALL state that the token is passed into the Docker container via the `-e` flag at runtime, and SHALL state that the token must never be embedded in the Docker image or stored in configuration files within the repository.
5. THE POWER_MD SHALL document how to configure the `GITHUB_PERSONAL_ACCESS_TOKEN` secret in Kiro's configuration, and SHALL provide setup instructions for registering the token within the Kiro secrets system.
6. THE POWER_MD SHALL document the image pinning policy, specifying that the `ghcr.io/github/github-mcp-server` image MUST be referenced by a specific version tag or SHA256 digest in mcp.json, and SHALL include instructions for verifying the image digest and updating it when a new trusted version is released.
7. THE POWER_MD SHALL document a process for updating the pinned image version, including: verifying the new image's provenance (checking GitHub's official release), updating the digest/tag in mcp.json, testing the new version, and updating the CHANGELOG with the version bump.



### Requirement 9: Steering for Spec-Issue Synchronization

**User Story:** As a developer, I want steering guidance for synchronizing specs with GitHub issues, so that Kiro can keep my specification documents and GitHub issue tracker in sync.

#### Acceptance Criteria

1. THE Power SHALL include a steering file named `spec-issue-sync.md` in the `steering/` directory that provides guidance on creating and synchronizing specs with GitHub issues.
2. WHEN Kiro creates a spec (requirements.md), THE Steering_File SHALL instruct Kiro to offer creating a corresponding GitHub issue that references the spec, including the requirement title as the issue title, acceptance criteria as a checklist in the issue body, and a label indicating it originated from a spec (e.g., `spec-driven`).
3. WHEN a spec is updated (requirements added, modified, or removed), THE Steering_File SHALL instruct Kiro to identify the corresponding GitHub issue(s) and update them to reflect the current spec state, preserving any manual additions or comments on the issue.
4. WHEN a GitHub issue is updated (new comments, status changes, label changes), THE Steering_File SHALL provide guidance on how Kiro should reflect those changes back into the relevant spec document (e.g., updating acceptance criteria based on issue discussion).
5. THE Steering_File SHALL define a linking convention between specs and issues (e.g., including the issue number in the spec document and the spec file path in the issue body) to enable bidirectional traceability.
6. THE Steering_File SHALL instruct Kiro to use a consistent mapping: one requirement maps to one GitHub issue, with sub-tasks in the spec mapping to issue task lists (checkboxes).
7. WHEN Kiro syncs a spec to issues, THE Steering_File SHALL instruct Kiro to create issues with appropriate labels (`spec-driven`, priority label, type label) and to link related issues using GitHub issue references (e.g., "Related to #123").



### Requirement 10: Skill for Creating Specs from GitHub Issues

**User Story:** As a developer, I want to retrieve a GitHub issue and have Kiro generate a spec/requirements document from it, so that I can quickly formalize issue descriptions into structured specifications.

#### Acceptance Criteria

1. THE POWER_MD SHALL document a skill named `create-spec-from-issue` that retrieves a GitHub issue by number and generates a structured requirements document from its content.
2. WHEN the user invokes the `create-spec-from-issue` skill with a repository and issue number, THE Power SHALL instruct Kiro to fetch the issue (title, body, labels, comments, linked issues) using the GitHub MCP server tools.
3. THE skill SHALL instruct Kiro to transform the issue content into a requirements document following the standard spec format, including: extracting the user story from the issue title and description, deriving acceptance criteria from the issue body (task lists, described behaviors), incorporating relevant information from issue comments, and mapping issue labels to requirement metadata.
4. IF the GitHub issue contains task lists (checkboxes), THEN THE skill SHALL instruct Kiro to convert each task item into an individual acceptance criterion in the generated requirements document.
5. IF the GitHub issue references other issues (via `#number` syntax), THEN THE skill SHALL instruct Kiro to fetch those related issues and include them as related requirements or dependencies in the spec document.
6. THE skill SHALL instruct Kiro to add a traceability link in the generated spec document referencing the source GitHub issue number, and to add a comment on the GitHub issue linking back to the created spec.
7. THE POWER_MD SHALL describe the `create-spec-from-issue` skill with: name, description, required inputs (repository owner, repository name, issue number), the sequence of GitHub MCP server tools invoked (get_issue, list_issue_comments, optionally get related issues), and expected outputs (a requirements.md file in the spec directory).



### Requirement 11: Semantic Versioning and Release Pipeline

**User Story:** As a Power maintainer, I want the Power to be semantically versioned and released via GitHub Actions pipelines, so that versions are tracked consistently and releases are automated from the beginning.

#### Acceptance Criteria

1. THE Power SHALL follow Semantic Versioning 2.0.0 (semver) with the format `MAJOR.MINOR.PATCH`, starting at version `0.1.0` as the initial release.
2. THE Power repository SHALL contain a GitHub Actions workflow file (e.g., `.github/workflows/release.yml`) that automates the release process, including creating a GitHub release with a version tag when triggered.
3. THE release workflow SHALL be triggered by a push of a semver-formatted tag (e.g., `v0.1.0`, `v1.0.0`) to the repository, and SHALL create a GitHub release with the tag name, auto-generated release notes, and the Power package files as release assets.
4. THE Power SHALL include a version field in the POWER.md frontmatter that matches the current release version tag.
5. THE POWER_MD SHALL document the versioning policy, specifying that: PATCH increments are for bug fixes and documentation updates to steering files, MINOR increments are for new steering files, new skills, or new MCP configuration options, and MAJOR increments are for breaking changes to the Power structure or MCP configuration format.
6. THE release workflow SHALL validate that the version tag matches the version declared in POWER.md frontmatter before creating the release, and SHALL fail with a descriptive error if they do not match.
7. THE Power repository SHALL include a CHANGELOG.md at the root that follows the Keep a Changelog format, with entries grouped by version and categorized as Added, Changed, Fixed, or Removed.
8. THE release workflow SHALL include a step that verifies the Docker image reference in mcp.json is pinned to a specific version tag or SHA256 digest (not `latest` or untagged), and SHALL fail the release if an unpinned image reference is detected.



### Requirement 12: Project Context Awareness

**User Story:** As a developer using the Power in different workspaces, I want the Power to automatically detect and respect the project context, so that all GitHub operations target the correct repository without requiring me to specify it every time.

#### Acceptance Criteria

1. THE Power SHALL include a steering file named `project-context.md` in the `steering/` directory that instructs Kiro on how to detect and use the current workspace's repository context.
2. WHEN Kiro performs any GitHub operation (creating issues, PRs, syncing specs, etc.), THE Steering_File SHALL instruct Kiro to first determine the target repository by inspecting the workspace's git remote configuration (e.g., `origin` remote URL) to extract the repository owner and name.
3. THE Steering_File SHALL instruct Kiro to derive the repository owner and repository name from the git remote URL, supporting both HTTPS format (`https://github.com/{owner}/{repo}.git`) and SSH format (`git@github.com:{owner}/{repo}.git`).
4. IF the workspace does not have a git remote configured or the remote URL cannot be parsed, THEN THE Steering_File SHALL instruct Kiro to prompt the user for the target repository owner and name before proceeding with any GitHub operation.
5. THE Steering_File SHALL instruct Kiro to scope all GitHub MCP tool invocations to the detected repository context, passing the resolved `owner` and `repo` parameters automatically to tools like `create_issue`, `create_pull_request`, `list_issues`, etc.
6. WHEN the Power is globally installed and used across multiple workspaces, THE Steering_File SHALL instruct Kiro to re-detect the repository context each time a new workspace is entered or a GitHub operation is initiated, ensuring operations always target the correct repository.
7. THE Steering_File SHALL instruct Kiro to confirm the detected repository context with the user on first use in a session (e.g., "I detected you're working in `owner/repo`. Is this correct?") before performing write operations (creating issues, PRs, releases).
8. THE Steering_File SHALL document that all skills (create-feature-pr, file-bug-report, create-release, review-pull-requests, create-spec-from-issue) inherit the detected project context and do not require the user to manually specify repository owner and name as inputs.



### Requirement 13: Modern GitHub Best Practices (2026+)

**User Story:** As a developer, I want the steering files to reflect current GitHub best practices (2026 and beyond), so that Kiro leverages modern platform features rather than outdated patterns.

#### Acceptance Criteria

1. THE steering files SHALL reference and utilize GitHub's modern issue features including: sub-issues (parent/child relationships), issue types (Bug, Feature, Task as first-class types), and tasklists (markdown tasklist syntax that creates trackable sub-issues).
2. THE steering files SHALL provide guidance on using GitHub Projects v2, including: creating project views (Board, Table, Roadmap), using custom fields (Status, Priority, Sprint, Size), and configuring project automation workflows (auto-add issues, auto-set fields on status change).
3. WHEN Kiro creates issues, THE Steering_File SHALL instruct Kiro to use issue types (where available) instead of relying solely on labels for categorization, leveraging GitHub's native issue type system, and SHALL apply any workspace-level overrides to the default type names (e.g., using "defect" instead of "bug" if configured in the Project_Context).
4. WHEN Kiro organizes work, THE Steering_File SHALL provide guidance on using tasklists within issues to break down work into trackable sub-issues, rather than using flat checkbox lists that lack traceability.
5. THE steering files SHALL instruct Kiro to use GitHub's native linking features (sub-issues, tracked-by/tracks relationships) for expressing dependencies between issues rather than manual cross-references in issue bodies.
6. THE steering files SHALL provide guidance on leveraging GitHub Actions for automation beyond releases, including: issue triage automation, PR labeling, stale issue management, and project board automation.
7. THE steering files SHALL NOT reference deprecated GitHub features (e.g., classic Projects v1) and SHALL prefer native platform capabilities over third-party tooling where equivalent functionality exists.
8. WHEN Kiro manages milestones or iterations, THE Steering_File SHALL instruct Kiro to use GitHub Projects v2 iterations/sprints fields rather than legacy milestone features, unless the repository specifically uses milestones.



### Requirement 14: Self-Sustained Dogfooding Development

**User Story:** As a Power maintainer, I want the Power to be developed using its own capabilities (dogfooding), so that the Power's quality is validated through real-world usage on its own development.

#### Acceptance Criteria

1. THE POWER_MD SHALL document that the Power's own development is managed through GitHub issues in the `studiokanslia/kiro-power-github` repository, using the Power's own steering files and skills for issue creation, PR management, and spec synchronization.
2. WHEN new features or improvements are planned for the Power, THE development process SHALL use the `create-spec-from-issue` skill to generate specs from GitHub issues filed in the Power's own repository.
3. WHEN specs are created for Power development, THE spec-issue-sync steering SHALL be applied to keep the Power's own specs synchronized with its GitHub issues.
4. THE Power repository SHALL maintain a GitHub Project (v2) board that tracks the Power's own development backlog, with columns/views for: Backlog, In Progress, In Review, and Done.
5. THE POWER_MD SHALL include a "Contributing" section that describes the dogfooding workflow: file an issue → create spec from issue → implement → sync spec back to issue → close issue, demonstrating the Power's own capabilities.
6. THE Power's own issues SHALL use the same templates, labels, and conventions defined in the steering files (Requirements 5, 6, 13), serving as living examples of the Power's guidance in action.
7. AFTER the initial bootstrap (v0.1.0 release), ALL subsequent feature development on the Power SHALL be tracked as GitHub issues and managed using the Power's steering and skills, validating that the Power is sufficient for real project management workflows.
