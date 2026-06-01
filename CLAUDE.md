# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What This Repository Is

**knowledge-work-plugins** is an open-source collection of 11+ Claude plugins for professional roles (sales, finance, legal, marketing, product management, customer support, data, enterprise search, bio-research, HR, engineering, etc.). Built for Claude Cowork and compatible with Claude Code.

Every component is file-based — Markdown and JSON, no code, no build steps, no infrastructure.

## Repository Layout

```
knowledge-work-plugins/
├── productivity/            # Task, calendar, and daily workflow management
├── sales/                   # Prospect research, pipeline review, call prep
├── customer-support/        # Ticket triage, responses, KB article generation
├── product-management/      # Specs, roadmaps, user research synthesis
├── marketing/               # Content, campaigns, brand voice, performance
├── legal/                   # Contract review, NDAs, compliance, risk
├── finance/                 # Journal entries, reconciliation, close, audit
├── data/                    # SQL, statistical analysis, dashboards
├── enterprise-search/       # Cross-tool unified search
├── bio-research/            # Life sciences R&D: literature, genomics, targets
├── human-resources/         # HR workflows
├── engineering/             # Engineering team workflows
├── operations/              # Operations workflows
├── design/                  # Design team workflows
├── cowork-plugin-management/ # Create and customize plugins
├── partner-built/           # Third-party maintained plugins
├── small-business/          # Small business workflows
└── pdf-viewer/              # PDF viewer plugin
```

## Plugin Structure

Every plugin follows the same convention:

```
plugin-name/
├── .claude-plugin/plugin.json   # Plugin manifest (name, version, author, skills)
├── .mcp.json                    # MCP server connections (tool connectors)
├── commands/                    # Slash commands (explicitly invoked)
│   └── <command-name>.md
└── skills/                      # Domain knowledge (auto-invoked when relevant)
    └── <skill-name>/SKILL.md
```

- **Skills** encode domain expertise and workflows; Claude draws on them automatically
- **Commands** are explicit slash commands (e.g., `/sales:call-prep`, `/finance:reconciliation`)
- **Connectors** (`.mcp.json`) wire Claude to external tools via MCP servers

## Installing Plugins

**Claude Cowork:**
```
Install from claude.com/plugins
```

**Claude Code:**
```bash
# Add the marketplace
claude plugin marketplace add anthropics/knowledge-work-plugins

# Install a specific plugin
claude plugin install sales@knowledge-work-plugins
claude plugin install legal@knowledge-work-plugins
```

## Key Connectors by Plugin

| Plugin | Key Connectors |
|--------|---------------|
| sales | Slack, HubSpot, Close, Clay, ZoomInfo, Notion, Fireflies |
| finance | Snowflake, Databricks, BigQuery, Microsoft 365 |
| data | Snowflake, Databricks, BigQuery, Amplitude, Hex |
| product-management | Linear, Jira, Figma, Amplitude, Intercom, Notion |
| marketing | Canva, HubSpot, Ahrefs, Amplitude, Klaviyo |
| legal | Box, Egnyte, Jira, Microsoft 365 |
| bio-research | PubMed, ClinicalTrials.gov, ChEMBL, Benchling |

## Customization

These plugins are generic starting points. To adapt for a specific organization:

1. **Swap connectors** — Edit `.mcp.json` to point at your tool stack
2. **Add company context** — Drop terminology, org structure, processes into skill files
3. **Adjust workflows** — Modify skill instructions to match how your team actually works
4. **Build new plugins** — Use `cowork-plugin-management/` or follow the plugin structure above

## Key Conventions for Editing

- **Plugin manifest stays in sync**: when adding or renaming skills/commands, update `.claude-plugin/plugin.json`
- **`.mcp.json` is per-plugin**: each plugin has its own connector config — do not create a global one
- **Skills fire automatically; commands do not**: write skill bodies to be useful without explicit invocation; write command bodies assuming the user specifically triggered them
- **No code, no build steps**: this repo is pure Markdown and JSON — do not introduce scripts or dependencies
- **Each skill is one job**: if a skill handles multiple distinct workflows, split it into separate skill files
- **Formatting**: 2-space indent in JSON; final newline in all text files; no trailing whitespace

## Contributing

Fork the repo, edit plugin Markdown/JSON files, and submit a PR. No build step required.
