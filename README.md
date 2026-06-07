# kiro-power-github

[![Release](https://img.shields.io/github/v/release/studiokanslia/kiro-power-github?style=flat-square)](https://github.com/studiokanslia/kiro-power-github/releases)

**Guided MCP Power for GitHub workflows** — bundles a Docker-based GitHub MCP server, steering files for PRs/issues/repos, and reusable skills for Kiro.

## What This Power Provides

- **GitHub API Tools** — Pre-configured Docker-based MCP server (`ghcr.io/github/github-mcp-server`) with 50+ GitHub tools
- **5 Steering Files** — Best practice guidance for PRs, issues, repository management, spec-issue sync, and project context
- **7 Skills** — Reusable multi-step workflows (create-feature-pr, file-bug-report, create-release, review-pull-requests, create-spec-from-issue, report-power-bug, request-power-feature)
- **Supply Chain Security** — Docker image pinned with version tag + SHA256 digest
- **Workspace Overrides** — Customizable issue types, priorities, and labels per project

## Quick Start

### Prerequisites
- Docker installed and running
- Kiro installed
- GitHub personal access token

### Setup
1. Install this Power in Kiro
2. Configure your `GITHUB_PERSONAL_ACCESS_TOKEN` in Kiro's secrets system
3. Start using GitHub tools and skills in any workspace

## Documentation

See [POWER.md](POWER.md) for full documentation including:
- Configuration details and token setup
- All 5 steering files with descriptions
- Detailed skill documentation (inputs, steps, outputs)
- Troubleshooting guide
- Image update process
- Workspace override configuration
- Contributing and feedback

## License

See [LICENSE](LICENSE).
