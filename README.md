# FormHug Skills

Agent Skills for working with FormHug — connecting to its MCP server, building good forms, applying recommended patterns. Built to the [Agent Skills open standard](https://github.com/anthropics/skills), so they work with any agent that reads `SKILL.md`: Claude Code, Codex CLI, Cursor, Windsurf, Claude Desktop, OpenClaw, Hermes, and others.

The MCP server itself (`https://formhug.ai/mcp`) ships its own reference resources (`formhug://guide`, `formhug://field-types`, …) covering tool usage. This repository complements them with skills that the agent **proactively** activates from triggers — installation flows, workflow guides, opinionated best practices — material that's better delivered as procedural skills than as on-demand reference docs.

## Available skills

| Skill | What it does |
|-------|--------------|
| [`setup-formhug-mcp`](skills/setup-formhug-mcp) | Configure the current agent to talk to `https://formhug.ai/mcp`. Picks OAuth (local agents) or Personal Access Token (remote/chat agents) automatically, writes the right config for that agent, and verifies with `get_me`. |
| [`migrate-form-to-formhug`](skills/migrate-form-to-formhug) | Recreate an existing form in FormHug from another platform or a document (Word/PDF/Excel/screenshot). Inventory-first workflow with a field-type mapping table and post-create verification. |
| [`analyze-formhug-entries`](skills/analyze-formhug-entries) | Analyze a form's submissions: full pagination, api_code→label decoding, per-type aggregation (NPS, likert, checkbox), and honest reporting rules. |
| [`audit-formhug-form`](skills/audit-formhug-form) | Review an existing form against a fixed checklist (field-type fit, required discipline, structure, settings, privacy) and apply approved fixes safely on live forms. |
| [`formhug-survey-methodology`](skills/formhug-survey-methodology) | Survey design methodology — question wording, scale design, structure, response-quality safeguards — mapped to the FormHug fields that implement each pattern. |

## Install

The recommended way is the [`skills`](https://www.npmjs.com/package/skills) CLI from Vercel Labs — it detects which agents you have installed and copies the skills to the right place for each:

```bash
npx skills add formhug/skills
```

Install only specific skills, or only to specific agents:

```bash
npx skills add formhug/skills --skill setup-formhug-mcp
npx skills add formhug/skills -a claude-code -a cursor
```

### Manual install

If you'd rather not use the CLI, copy a skill directory into your agent's skills location. Path varies by agent:

| Agent          | Skills directory                                         |
| -------------- | -------------------------------------------------------- |
| Claude Code    | `~/.claude/skills/` (user) or `.claude/skills/` (project) |
| Codex CLI      | `~/.codex/skills/`                                       |
| Cursor         | `~/.cursor/skills/`                                      |
| Gemini CLI     | `~/.gemini/skills/`                                      |
| OpenClaw       | via `clawhub install <skill>`                            |

```bash
git clone https://github.com/formhug/skills.git
cp -r skills/skills/setup-formhug-mcp ~/.claude/skills/
```

## Usage

After installation, skills activate automatically when the agent recognises a matching request (each skill declares its triggers in its frontmatter). Agents that expose skills as slash commands can also invoke them directly, e.g. `/setup-formhug-mcp`.

## Repository layout

```
skills/
  <skill-name>/
    SKILL.md          # frontmatter (name, description) + instructions
    references/       # optional — files the skill pulls in on demand
    scripts/          # optional — executable helpers
```

Each skill is a self-contained directory. The agent only loads `SKILL.md` up front; everything else is referenced from inside it and read on demand (progressive disclosure).

## What kinds of skills belong here

- **Setup / integration** — connecting agents to FormHug (`setup-formhug-mcp`)
- **Workflow guides** — opinionated, agent-driven flows (e.g. "design a registration form from a description", "audit an existing form's entries")
- **Best practices** — codified conventions agents should follow when using FormHug tools (field naming, scoring rubrics, theme choices, webhook patterns)

Reference material that the agent needs only *after* it's already decided what to do (field type catalogues, theme tokens, error code tables) generally belongs in the MCP server's resources rather than here.
