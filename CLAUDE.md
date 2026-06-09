# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What This Repository Is

A **curated plugin marketplace for Claude Cowork and Claude Code**. There is no compiled code — everything is declarative Markdown and JSON. Plugins bundle skills, commands, MCP tool connections, and metadata so Claude can assist with domain-specific workflows (sales, engineering, finance, legal, etc.).

## Repository Layout

```
<plugin-name>/
├── .claude-plugin/plugin.json   # Manifest: name, version, description, author
├── .mcp.json                    # MCP server connections (Slack, Notion, HubSpot, etc.)
├── README.md                    # User-facing docs
├── CONNECTORS.md                # Tool reference with agnostic placeholders
├── commands/[name].md           # Slash commands users invoke explicitly
└── skills/[name]/SKILL.md       # Domain knowledge Claude auto-applies

partner-built/                   # Third-party plugins pulled via git URL + SHA pin
.claude-plugin/marketplace.json  # Master registry of all ~32 Anthropic + partner plugins
.github/
├── policy/prompt.md             # Security review guidelines used by CI scanner
├── policy/schema.json           # JSON schema for scan verdicts
└── workflows/                   # GitHub Actions pipelines (see CI section)
```

## Core Architectural Concepts

### Skills vs. Commands

**Skills** (`skills/[name]/SKILL.md`) — Domain knowledge Claude draws on *automatically* based on trigger patterns declared in the file. Example triggers from `sales/skills/draft-outreach/SKILL.md`: `"draft outreach to [person/company]"`, `"write cold email to [prospect]"`. Claude reads the skill file and executes the described multi-step workflow.

**Commands** (`commands/[name].md`) — Explicit slash commands users invoke, e.g., `/sales:call-summary`, `/data:write-query`. Each command file defines purpose, inputs, and workflow steps.

### Connector Abstraction

Plugins reference tool *categories* in CONNECTORS.md (e.g., `~~CRM`, `~~email`, `~~knowledge base`) rather than hard-coding specific products. This lets users swap HubSpot ↔ Close ↔ Salesforce without changing plugin files. The actual server bindings live in `.mcp.json` and can be customized per installation.

### Marketplace Manifest

`.claude-plugin/marketplace.json` is the single source of truth listing all plugins. Each entry specifies:
- `source`: `local` path, `git-url` with SHA, `git-subdir`, or `remote-url`
- `sha`: pinned commit hash for external/partner plugins (auto-updated by CI)

Partner plugins under `partner-built/` are referenced via `git-url` source type and their SHAs are bumped nightly.

### Plugin State & Persistence

Some plugins create local files for state:
- `productivity/` — Creates `TASKS.md`, `CLAUDE.md` (memory), `dashboard.html`
- `pdf-viewer/` — Stores annotated PDFs locally
- `small-business/` — Creates financial models and Canva assets

### Settings Customization

Plugins that support personalization read from `settings.local.json`. Example: the `sales/` plugin reads user name, title, company, quota, and competitor names to tailor outputs.

## Development Workflows

There is no build step, test runner, or linter — this is a markdown-first repository. Development means editing `.md` and `.json` files.

### Adding or Modifying a Plugin

1. Create the directory structure above (or copy an existing plugin as a template)
2. Update `.claude-plugin/marketplace.json` to register the plugin — this is what triggers CI scans
3. Add/update skills in `skills/[name]/SKILL.md` with trigger patterns and workflow prose
4. Add/update commands in `commands/[name].md` with invocation syntax and steps
5. Update `.mcp.json` for any new tool connections required

### Plugin Manifest Schema

`plugin.json` fields: `name`, `version` (semver), `description`, `author`. Most Anthropic plugins are at `1.2.0`; newer standalone plugins (`pdf-viewer`, `small-business`) use `0.2.x`.

## CI/CD Pipelines

All workflows live in `.github/workflows/`.

**`scan-plugins.yml`** — Security gate that runs on every PR touching `marketplace.json` or the policy files. Calls the Claude API with the policy prompt (`.github/policy/prompt.md`) to review each changed plugin for: malicious intent, undisclosed telemetry, overly broad hook scope, and misalignment between description and behavior. Verdicts are cached by `(plugin, SHA, policy_hash)`. A `FAIL` verdict blocks merge.

**`bump-plugin-shas.yml`** — Nightly job that checks external plugin repos for new commits and opens a PR updating SHAs in `marketplace.json`. That PR then runs through the security scan.

**`check-mcp-urls.yml`** — Validates that all MCP server URLs in `.mcp.json` files respond.

**`revert-failed-bumps.yml`** — When a scan fails on a bump PR, removes only the failing plugin from the update so other plugins can still ship.

## Policy Review Guidelines

The security scanner (`.github/policy/prompt.md`) flags plugins that:
- Execute or exfiltrate data without user awareness
- Declare hooks with broad scope that aren't disclosed
- Collect telemetry without explicit disclosure
- Behave differently from what their description claims
- Request permissions beyond what their workflows require

When writing new skills or commands, ensure the behavior is fully disclosed in the plugin's README and that any hook registrations are scoped narrowly.
