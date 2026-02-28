# FormHug Skills

Reusable agent skills for FormHug — works with any MCP-compatible agent or IDE (Claude Code, Cursor, Windsurf, Claude Desktop, OpenClaw, and others).

## Available Skills

| Skill | Audience | Description |
|-------|----------|-------------|
| [setup-formhug-mcp](skills/setup-formhug-mcp) | Humans | Step-by-step guide for connecting any MCP-compatible agent to FormHug |
| [formhug-mcp-openclaw](skills/formhug-mcp-openclaw) | Agents | Agent-driven setup — the agent adds the MCP server and initiates OAuth itself |

## Installation

### ClawHub (OpenClaw)

```bash
clawhub install formhug-mcp-openclaw
```

### Claude Code

Copy or symlink a skill directory into your Claude Code skills location:

```bash
# User-level (available across all projects)
cp -r skills/setup-formhug-mcp ~/.claude/skills/

# Or project-level (available in a specific project)
cp -r skills/setup-formhug-mcp /path/to/project/.claude/skills/
```

### Other Agents

Each skill's `SKILL.md` contains the full instructions. You can read it directly or adapt it to your agent's skill/prompt system.

## Usage

Once installed, invoke a skill by its trigger phrases or slash command. For example in Claude Code:

```
/setup-formhug-mcp
```

Or simply ask your agent to "set up FormHug MCP" and the skill will activate automatically.

## Adding New Skills

Create a new directory under `skills/` with a `SKILL.md` file:

```
skills/
  your-skill-name/
    SKILL.md          # Required — frontmatter + instructions
    ...               # Optional supporting files
```
