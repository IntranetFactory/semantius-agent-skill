# semantius

> **Note:** `CLAUDE.md` is a symlink to this file.

## Overview

Universal System of Records platform management via PostgREST API.
Use when working with business data modules (CRM, ITSM, LMS, etc.),
querying or managing records, or configuring the platform (admin).
All operations use PostgREST — there is no direct SQL access.

## Structure

```
semantius.skill  # Ready-to-upload skill bundle
semantius/
  SKILL.md       # Main skill file - read this first
  AGENTS.md      # This navigation guide
  CLAUDE.md      # Symlink to AGENTS.md
  references/    # Detailed reference files
```

## Usage

1. Read `SKILL.md` for the main skill instructions
2. Browse `references/` for detailed documentation on specific topics
3. Reference files are loaded on-demand - read only what you need

## Reference Categories

| Priority | Category | Impact | Prefix |
|----------|----------|--------|--------|
| 1 | API Syntax | high | `api-` |
| 2 | Schema Metadata | high | `schema-` |
| 3 | Access Control | high | `rbac-` |

Reference files are named `{prefix}-{topic}.md` (e.g., `query-missing-indexes.md`).

## Available References

**API Syntax** (`api-`):
- `references/api-postgrest-syntax.md`

**Access Control** (`rbac-`):
- `references/rbac-rbac-schema.md`

**Schema Metadata** (`schema-`):
- `references/schema-fields-schema.md`
- `references/schema-tables-schema.md`

---

*4 reference files across 3 categories*
