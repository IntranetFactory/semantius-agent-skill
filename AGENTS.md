# AGENTS.md

Guidance for AI coding agents working with this repository.

> **Note:** `CLAUDE.md` is a symlink to this file.

## Repository Structure

```
skills/
  semantius/
    SKILL.md              # Required: skill manifest (Agent Skills spec)
    README.md             # Skill overview
    metadata.json         # Skill metadata
    references/
      postgrest-syntax.md # PostgREST query syntax reference
      tables-schema.md    # Tables metadata schema
      fields-schema.md    # Fields metadata schema
      rbac-schema.md      # RBAC tables reference
```

## About Semantius

Semantius is a universal System of Records platform built on PostgreSQL with PostgREST API access. Key characteristics:

- **All data access is via PostgREST** — there is no direct SQL access
- **Schema is stored in metadata tables** — `modules`, `tables`, `fields`
- **RBAC controls access via RLS policies**
- **Three MCP tools provide all functionality**: `getCurrentUser`, `postgrestRequest`, `sqlToRest`

## Working with This Skill

When agents work with Semantius:

1. **Always introspect first** — call `getCurrentUser` and query metadata tables
2. **Schema is dynamic** — each deployment has different modules and tables
3. **Use PostgREST syntax** — see `references/postgrest-syntax.md`
4. **Follow naming conventions** — plural tables, snake_case fields, lowercase everything

## Commands

```bash
npm run build    # Regenerate AGENTS.md files
npm run check    # Format and lint (auto-fix)
npm run ci:check # CI validation
```

**Before completing any task**, run `npm run check` and `npm run build` to ensure CI passes.

## Creating Reference Files

Reference files in `references/` extend skills with detailed documentation.

```markdown
# Title

Description of the reference content.

## Section

Content with examples...
```

Keep reference files focused on a single topic. Use tables and code examples for clarity.
