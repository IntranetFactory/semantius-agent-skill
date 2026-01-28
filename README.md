# Semantius Agent Skills

Agent Skills to help developers using AI agents with the Semantius platform. Agent Skills are
folders of instructions, scripts, and resources that agents like Claude Code,
Cursor, Github Copilot, etc. can discover and use to work more accurately
and efficiently with Semantius deployments.

The skills in this repo follow the [Agent Skills](https://agentskills.io/) format.

## Installation

```bash
npx skills add semantius/agent-skills
```

### Claude Code Plugin

You can also install the skills in this repo as Claude Code plugins:

```bash
/plugin marketplace add semantius/agent-skills
/plugin install semantius@semantius-agent-skills
```

## Available Skills

<details>
<summary><strong>semantius</strong></summary>

Universal System of Records platform skill for PostgREST-based data management.

**Use when:**

- Working with Semantius platform deployments
- Querying or managing data via PostgREST
- Discovering modules, tables, and fields dynamically
- Administering the platform (creating tables, managing permissions)
- Working with RBAC and RLS policies

**Key concepts:**

- All access is via PostgREST — no direct SQL
- Schema is dynamic — always introspect before operating
- Data dictionary approach — modifying `tables`/`fields` creates actual database objects
- RBAC via RLS — permissions control what users can access

</details>

## Usage

Skills are automatically available once installed. The agent will use them when
relevant tasks are detected.

**Examples:**

```
Query contacts from the CRM module
```

```
Create a new table in the ITSM module
```

```
Set up RBAC permissions for the sales team
```

## Documentation

- [Semantius Platform Documentation](https://docs.semantius.io)
- [PostgREST Syntax Reference](./skills/semantius/references/postgrest-syntax.md)
- [RBAC Schema Reference](./skills/semantius/references/rbac-schema.md)

## Skill Structure

Each skill follows the [Agent Skills Open Standard](https://agentskills.io/):

- `SKILL.md` - Required skill manifest with frontmatter (name, description, metadata)
- `references/` - Individual reference files

## License

MIT
