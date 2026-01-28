# Semantius Agent Skill

Universal System of Records platform skill for AI agents.

## Overview

Semantius is a dynamic platform where each deployment has different modules (CRM, ITSM, LMS, etc.) and tables. This skill teaches agents how to:

- Discover available modules, tables, and fields
- Query and manage data via PostgREST
- Administer the platform (create tables, manage permissions)

## MCP Tools

The skill works with three MCP tools:

| Tool | Description |
|------|-------------|
| `getCurrentUser` | Get authenticated user's profile, roles, and permissions |
| `postgrestRequest` | Execute PostgREST API requests |
| `sqlToRest` | Convert SQL queries to PostgREST format |

## Key Concepts

- **All access is via PostgREST** — no direct SQL
- **Schema is dynamic** — always introspect before operating
- **Data dictionary approach** — modifying `tables`/`fields` creates actual database objects
- **RBAC via RLS** — permissions control what users can access

## Quick Start

1. Call `getCurrentUser` to understand permissions
2. Query `/modules` to see available modules
3. Query `/tables` to see tables in a module
4. Query `/fields` to understand table structure
5. Perform CRUD operations with `postgrestRequest`

## Documentation

- [SKILL.md](./SKILL.md) — Main skill instructions
- [references/postgrest-syntax.md](./references/postgrest-syntax.md) — PostgREST query reference
- [references/tables-schema.md](./references/tables-schema.md) — Tables metadata schema
- [references/fields-schema.md](./references/fields-schema.md) — Fields metadata schema
- [references/rbac-schema.md](./references/rbac-schema.md) — RBAC tables reference
