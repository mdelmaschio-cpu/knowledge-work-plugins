# CLAUDE.md — knowledge-work-plugins

## Repository Overview

This is the official Anthropic plugin marketplace for **Claude Cowork** and **Claude Code**. It contains a collection of open-source plugins that extend Claude with domain-specific skills, slash commands, and tool integrations for different professional roles and workflows.

Plugins are entirely file-based — markdown and JSON, no compiled code, no build steps. Each plugin bundles:
- **Skills** — domain expertise Claude draws on automatically when relevant
- **Commands** (legacy) or skill-based slash commands users invoke explicitly
- **MCP server connections** — external tool integrations (Slack, HubSpot, Jira, etc.)
- **Agents** — sub-agents for autonomous multi-step tasks (uncommon)
- **Hooks** — event-triggered automation (rare)

The marketplace index lives at `.claude-plugin/marketplace.json`.

---

## Repository Structure

```
knowledge-work-plugins/
├── .claude-plugin/
│   └── marketplace.json          # Master plugin registry (all plugins listed here)
├── .github/
│   ├── policy/
│   │   ├── prompt.md             # AI policy review prompt for marketplace submissions
│   │   └── schema.json           # Schema for policy review output
│   └── workflows/
│       ├── bump-plugin-shas.yml  # Nightly: bumps pinned SHAs for external plugins
│       ├── check-mcp-urls.yml    # Validates MCP server URLs are live
│       ├── scan-plugins.yml      # Policy scan for new marketplace entries (required check)
│       └── revert-failed-bumps.yml
│
├── bio-research/                 # Life sciences R&D plugin (Anthropic-authored)
├── cowork-plugin-management/     # Plugin creation/customization meta-plugin
├── customer-support/             # Support ticket triage, response drafting, KB authoring
├── data/                         # SQL, analytics, dashboards
├── design/                       # Figma, design critique, UX, accessibility
├── engineering/                  # Code review, incident response, architecture, standups
├── enterprise-search/            # Cross-tool unified search
├── finance/                      # Accounting, reconciliation, close management
├── human-resources/              # Recruiting, onboarding, performance reviews
├── legal/                        # Contract review, NDA triage, compliance
├── marketing/                    # Content, campaigns, brand voice, competitive intel
├── operations/                   # Vendor management, process docs, capacity planning
├── pdf-viewer/                   # PDF annotation, forms, signing
├── product-management/           # Specs, roadmaps, user research synthesis
├── productivity/                 # Tasks, calendar, daily workflows
├── sales/                        # Prospecting, call prep, outreach, pipeline review
├── small-business/               # Payroll, close, growth campaigns
└── partner-built/                # Third-party contributed plugins
    ├── apollo/
    ├── brand-voice/
    ├── common-room/
    ├── slack/
    └── zoom-plugin/
```

---

## Plugin Directory Structure

Every plugin follows the same layout:

```
plugin-name/
├── .claude-plugin/
│   └── plugin.json        # Required manifest (name, version, description, author)
├── .mcp.json              # MCP server connections (tool integrations)
├── skills/                # Preferred format for new plugins
│   └── skill-name/
│       ├── SKILL.md       # Core skill instructions (required)
│       ├── references/    # Detailed docs loaded on demand
│       ├── examples/      # Working examples
│       └── scripts/       # Utility scripts (Python, shell)
├── commands/              # Legacy slash command format (still supported)
│   └── command-name.md
├── agents/                # Sub-agent definitions (uncommon)
│   └── agent-name.md
├── hooks/                 # Event hooks (rare)
│   └── hooks.json
├── CONNECTORS.md          # Explains ~~ placeholders (for distributable plugins)
└── README.md
```

**Important:** Only create directories for components the plugin actually uses.

---

## Key Files and Their Roles

### `.claude-plugin/plugin.json`
Manifest required for every plugin. Minimal required field: `name` (kebab-case).

```json
{
  "name": "plugin-name",
  "version": "0.1.0",
  "description": "Brief explanation",
  "author": { "name": "Author Name" }
}
```

Version follows semver. Start new plugins at `0.1.0`.

### `skills/*/SKILL.md`
The primary component type. Contains YAML frontmatter + markdown body.

**Frontmatter:**
```yaml
---
name: skill-name
description: >
  Third-person description with specific trigger phrases in quotes.
  Use when the user asks to "do X", "create Y", or "review Z".
argument-hint: "<argument description>"   # optional, for slash commands
---
```

**Body:** Imperative instructions for Claude (not documentation for users). Keep under 3,000 words; move detailed content to `references/`.

### `commands/*.md` (legacy)
Single-file slash command format. Supports `$ARGUMENTS`, `$1`, `$2`, `@path` file inclusion, and inline bash with backtick syntax. Still works — do not migrate unless changing the file anyway.

### `.mcp.json`
Defines MCP server connections. Supports `stdio` (local), `sse` (remote with OAuth), and `http` (REST) transports. Use `${CLAUDE_PLUGIN_ROOT}` for local paths, `${VAR_NAME}` for env var expansion.

```json
{
  "mcpServers": {
    "slack": {
      "type": "http",
      "url": "https://mcp.slack.com/mcp",
      "oauth": { "clientId": "...", "callbackPort": 3118 }
    }
  }
}
```

### `CONNECTORS.md`
Only needed when the plugin uses `~~category` placeholders instead of specific tool names. Documents placeholder-to-tool mapping. Required for plugins intended for broad distribution.

### `.claude-plugin/marketplace.json`
Top-level index of all plugins. In-repo plugins use `"source": "./plugin-dir"`. External plugins pin a git SHA:

```json
{
  "name": "external-plugin",
  "source": {
    "source": "url",
    "url": "https://github.com/org/repo.git",
    "sha": "abc123..."
  }
}
```

---

## Plugin Conventions and Patterns

### `~~` Connector Placeholders
Plugins intended for external distribution reference tools generically using `~~category` syntax (e.g., `~~project tracker`, `~~chat`, `~~literature`). This makes plugins tool-agnostic. The `cowork-plugin-customizer` skill replaces these with specific tool names during setup.

Do **not** use `~~` placeholders for internal/company-specific plugins — use real tool names directly.

### `${CLAUDE_PLUGIN_ROOT}` Variable
Always use this variable for intra-plugin path references in hooks and MCP configs. Never hardcode absolute paths.

### Progressive Disclosure for Skills
- **Frontmatter description** (~100 words) — always in context
- **SKILL.md body** (<3,000 words) — loaded when skill triggers
- **`references/` files** — loaded on demand for depth

### Writing Style
- **Skill/command bodies**: Imperative form for Claude. "Parse the config file." Not "You should parse the config file."
- **Frontmatter descriptions**: Third-person. "This skill should be used when the user asks to..."
- **User-facing output in plugin creation/customization skills**: Plain language, no technical implementation details.

### Naming Conventions
- All directory and file names: `kebab-case`
- Plugin names: lowercase, hyphens only, no spaces or special chars
- Skills: directory name must match the `name` frontmatter field

---

## The Three Focus Plugins

### `bio-research/`
Life sciences R&D plugin. Connects 11 MCP servers (PubMed, BioRender, bioRxiv, ChEMBL, Open Targets, ClinicalTrials.gov, Synapse, Wiley, Owkin, Benchling, Consensus). Skills include:
- `single-cell-rna-qc` — MAD-based QC for scRNA-seq `.h5ad`/`.h5` files with Python scripts
- `scvi-tools` — Deep learning for single-cell omics (scVI, scANVI, totalVI, PeakVI, etc.)
- `nextflow-development` — Run nf-core pipelines (rnaseq, sarek, atacseq); includes Python scripts for sample sheet generation and GEO/SRA data fetching
- `instrument-data-to-allotrope` — Converts 40+ lab instrument formats to Allotrope Simple Model (ASM); includes Python conversion scripts
- `scientific-problem-selection` — 9-step framework for research problem evaluation (Fischbach & Walsh)
- `start` — Orientation skill that inventories connected MCP servers

Skills in `bio-research` often include actual Python scripts under `scripts/` — this is more code-heavy than most other plugins.

### `cowork-plugin-management/`
Meta-plugin for building and customizing other plugins. No MCP servers. Skills:
- `create-cowork-plugin` — Guides users through 5-phase plugin creation: Discovery → Component Planning → Design → Implementation → Review & Package. Outputs a `.plugin` zip file.
- `cowork-plugin-customizer` — Customizes existing plugins by replacing `~~` placeholders using knowledge MCPs (Slack, docs) and user input. Outputs a `.plugin` zip file.

This plugin encodes the authoritative plugin architecture specification. When in doubt about plugin structure, read `cowork-plugin-management/skills/create-cowork-plugin/SKILL.md` and `references/component-schemas.md`.

### `customer-support/`
Support team plugin with 5 skills (no sub-`references/` directories — simpler single-SKILL.md pattern):
- `ticket-triage` — P1–P4 priority framework, category taxonomy, routing rules, duplicate detection, auto-response templates
- `customer-research` — Multi-source research methodology with confidence scoring
- `draft-response` — Communication best practices, tone guidelines, scenario templates
- `customer-escalation` — Escalation tiers, structured escalation format, impact assessment
- `kb-article` — Article structure standards, searchability, maintenance cadence

MCP connections: Slack, Intercom, HubSpot, Guru, Atlassian, Notion, Microsoft 365.

---

## Development Workflows

### Installing a Plugin (Claude Code)
```bash
# Add the marketplace
claude plugin marketplace add anthropics/knowledge-work-plugins

# Install a specific plugin
claude plugin install customer-support@knowledge-work-plugins
```

### Validating a Plugin
```bash
claude plugin validate <path-to-plugin.json>
```

Manual validation checklist:
- `.claude-plugin/plugin.json` exists and has valid JSON with a `name` field
- `name` is kebab-case (lowercase letters, numbers, hyphens only)
- All referenced component directories exist (`skills/`, `agents/`, `commands/`, `hooks/`)
- Each `skills/*/` subdirectory contains a `SKILL.md`
- `.md` for commands/skills/agents; `.json` for hooks

### Packaging a Plugin
```bash
# Always create in /tmp first, then copy to outputs
cd /path/to/plugin-dir && \
  zip -r /tmp/plugin-name.plugin . -x "*.DS_Store" "setup/*" && \
  cp /tmp/plugin-name.plugin /path/to/outputs/plugin-name.plugin
```

### Contributing to the Marketplace
- Fork the repo, make changes, submit a PR
- Plugins are just markdown and JSON — no code review for compilation
- External plugin entries in `marketplace.json` are automatically scanned by `.github/workflows/scan-plugins.yml` (required check on `main`)
- Pinned SHAs for external plugins are bumped nightly by the `bump-plugin-shas.yml` workflow

---

## CI/CD and Automation

### `scan-plugins.yml` (required PR check)
Runs a Claude-powered policy review on any new or changed external marketplace entries. Uses `.github/policy/prompt.md` as the review prompt. Checks for:
- Malicious code, data exfiltration, privacy violations
- Broad-scope hooks that observe sessions unrelated to the plugin's purpose
- Undisclosed telemetry (outbound network calls not mentioned in the description)
- Description/behavior mismatches

Results are cached per `(plugin, sha)` pair — same SHA is never re-scanned under the same policy.

### `bump-plugin-shas.yml`
Runs nightly at 07:23 UTC. Checks each external plugin for upstream changes, validates the new SHA with `claude plugin validate`, and opens a PR with all passing bumps.

### `check-mcp-urls.yml`
Periodically checks that MCP server URLs in `.mcp.json` files are still live and returns valid responses.

### `revert-failed-bumps.yml`
When `scan-plugins.yml` fails on a bump PR, this workflow automatically drops the failing entries so one bad upstream doesn't block the rest.

---

## Notes for AI Assistants

- **No build step**: Every plugin is pure markdown and JSON. Never suggest running `npm install`, `pip install`, or any build command for the plugin itself. Individual skills may have `requirements.txt` for their Python scripts — those are separate concerns.
- **Skills vs Commands**: New plugins should always use `skills/*/SKILL.md`. The legacy `commands/*.md` format still works but should not be used for new work. The Cowork UI presents both identically.
- **Skill bodies are instructions for Claude, not users**: When writing or editing skill content, frame everything as imperative directives. The body of a SKILL.md is essentially a system prompt injected when the skill triggers.
- **~~placeholders are for external distribution only**: Only add `~~` connectors if the plugin is meant to be shared outside one organization. Internal plugins should use real tool names.
- **`${CLAUDE_PLUGIN_ROOT}` is mandatory** for any path reference in hooks or MCP configs. Hardcoded absolute paths will break portability.
- **Component schemas are authoritative**: `cowork-plugin-management/skills/create-cowork-plugin/references/component-schemas.md` is the single source of truth for exact format specs (frontmatter fields, JSON schemas, hook events, etc.).
- **External plugins are SHA-pinned**: Entries in `marketplace.json` with `"source": {"source": "url", "sha": "..."}` reference external repos at a specific commit. The SHA is updated automatically by CI; don't change SHAs manually in PRs unless fixing a specific upstream issue.
- **Benchling MCP URL is a placeholder**: In `bio-research/.mcp.json`, the `benchling` entry has an empty URL — this is intentional and documented as pending configuration.
- **Partner-built plugins**: `partner-built/` contains plugins contributed by third parties (Salesforce/Slack, Apollo, Common Room, Zoom). These follow the same plugin structure but are not Anthropic-authored.
