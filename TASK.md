# Task: Create Semantius Agent Skills Repository

Transform the cloned Supabase agent-skills repo into a Semantius agent-skills repo.

## Background

Semantius is a universal System of Records platform built on PostgreSQL with PostgREST API access. It allows customers to build CRM, ITSM, LMS, and other business applications. Each deployment is different — the skill must teach agents how to discover and work with the platform dynamically.

Key characteristics:
- All data access is via PostgREST (no direct SQL)
- Schema is stored in metadata tables (tables, fields)
- RBAC controls access via RLS policies
- Three MCP tools provide all functionality

---

## Step 1: Clean Up Supabase Content

Delete these directories entirely:
```
rm -rf skills/postgres-best-practices/
rm -rf packages/
rm -rf .agents/
```

Keep the repo structure but we'll replace all content.

---

## Step 2: Update Root Files

### package.json
Change name to `semantius-agent-skills`, update description and author.

### README.md
Replace entirely with Semantius branding and installation instructions:
- `npx skills add semantius/agent-skills`
- Description of the semantius skill
- Link to documentation

### AGENTS.md
Replace with Semantius-specific agent instructions for this repo.

### CLAUDE.md
Replace with Semantius-specific Claude Code instructions for this repo.

### .claude-plugin/ (if exists)
Update plugin configuration for Semantius.

---

## Step 3: Create Directory Structure

```
semantius-agent-skills/
├── skills/
│   └── semantius/
│       ├── SKILL.md
│       ├── README.md
│       ├── metadata.json
│       └── references/
│           ├── postgrest-syntax.md
│           ├── tables-schema.md
│           ├── fields-schema.md
│           └── rbac-schema.md
├── .claude-plugin/
├── AGENTS.md
├── CLAUDE.md
├── README.md
└── package.json
```

---

## Step 4: Create skills/semantius/SKILL.md

```markdown
---
name: semantius
description: |
  Universal System of Records platform management via PostgREST API.
  Use when working with business data modules (CRM, ITSM, LMS, etc.),
  querying or managing records, or configuring the platform (admin).
  All operations use PostgREST — there is no direct SQL access.
---

# Semantius Platform Skill

Semantius is a dynamic platform where each deployment has different modules and tables.
Always introspect to discover what's available before operating on data.

## Available MCP Tools

### getCurrentUser

Retrieves profile information for the authenticated user including email, roles, permissions, accessible modules, and metadata.

**Always call this first** to understand what the current user can access.

**Parameters:** None

**Returns:** User profile with roles, permissions, and accessible modules.

**Usage:**
```
semantius:getCurrentUser()
```

---

### postgrestRequest

Performs an HTTP request against the PostgREST API.

**Parameters:**

| Name | Type | Required | Description |
|------|------|----------|-------------|
| method | `GET` \| `POST` \| `PUT` \| `PATCH` \| `DELETE` | Yes | HTTP method |
| path | string | Yes | PostgREST API path (e.g., `/users`, `/contacts?status=eq.active`) |
| body | object \| array | No | Request body for POST/PUT/PATCH. Object for single record, array for bulk. |

**Examples:**

Query with filters:
```javascript
semantius:postgrestRequest({
  method: "GET",
  path: "/contacts?status=eq.active&order=name.asc"
})
```

Create single record:
```javascript
semantius:postgrestRequest({
  method: "POST",
  path: "/contacts",
  body: { name: "John Doe", email: "john@example.com" }
})
```

Create multiple records:
```javascript
semantius:postgrestRequest({
  method: "POST",
  path: "/contacts",
  body: [
    { name: "John Doe", email: "john@example.com" },
    { name: "Jane Smith", email: "jane@example.com" }
  ]
})
```

Update record:
```javascript
semantius:postgrestRequest({
  method: "PATCH",
  path: "/contacts?id=eq.123",
  body: { status: "inactive" }
})
```

Delete record:
```javascript
semantius:postgrestRequest({
  method: "DELETE",
  path: "/contacts?id=eq.123"
})
```

---

### sqlToRest

Converts a SQL query to a PostgREST API request. Use when SQL is easier to reason about, then execute the result with `postgrestRequest`.

**Parameters:**

| Name | Type | Required | Description |
|------|------|----------|-------------|
| sql | string | Yes | SQL query to convert |

**Example:**
```javascript
semantius:sqlToRest({
  sql: "SELECT name, email FROM contacts WHERE status = 'active' ORDER BY created_at DESC LIMIT 10"
})
```

**Returns:** The equivalent PostgREST method and path to use with `postgrestRequest`.

**Workflow:**
1. Call `sqlToRest` with your SQL query
2. Take the returned method and path
3. Execute with `postgrestRequest`

---

## Introspection — Always Do This First

The platform is dynamic. Before any operation, discover what's available.

### 1. Get Current User Context

```javascript
semantius:getCurrentUser()
```

This returns:
- User's roles
- User's permissions
- Accessible modules

### 2. List Available Modules

```javascript
semantius:postgrestRequest({
  method: "GET",
  path: "/modules?select=id,name,label,description"
})
```

### 3. List Tables in a Module

```javascript
semantius:postgrestRequest({
  method: "GET",
  path: "/tables?module_id=eq.{MODULE_ID}&select=id,table_name,singular_label,plural_label,description"
})
```

### 4. Get Fields for a Table

```javascript
semantius:postgrestRequest({
  method: "GET",
  path: "/fields?table_name=eq.{TABLE_NAME}&select=field_name,title,format,is_nullable,input_type"
})
```

### 5. Understand Any Table's Schema

Always query `tables` and `fields` to understand structure before querying data:

```javascript
// Get table metadata
semantius:postgrestRequest({
  method: "GET",
  path: "/tables?table_name=eq.contacts"
})

// Get field definitions
semantius:postgrestRequest({
  method: "GET",
  path: "/fields?table_name=eq.contacts&order=field_order.asc"
})
```

---

## PostgREST Query Syntax

See `references/postgrest-syntax.md` for complete documentation.

### Quick Reference

**Operators:**

| Operator | Meaning | Example |
|----------|---------|---------|
| eq | Equals | `?status=eq.active` |
| neq | Not equals | `?status=neq.deleted` |
| gt | Greater than | `?age=gt.18` |
| gte | Greater or equal | `?age=gte.21` |
| lt | Less than | `?price=lt.100` |
| lte | Less or equal | `?price=lte.99.99` |
| like | LIKE (use * for %) | `?name=like.*smith*` |
| ilike | Case-insensitive LIKE | `?name=ilike.*smith*` |
| in | In list | `?status=in.(active,pending)` |
| is | IS (for null/bool) | `?deleted_at=is.null` |
| not | Negate | `?status=not.eq.deleted` |

**Logical Operators:**

```
?and=(status.eq.active,type.eq.lead)
?or=(status.eq.active,status.eq.pending)
```

**Ordering:**

```
?order=name.asc
?order=created_at.desc
?order=priority.desc,name.asc
```

**Pagination:**

```
?limit=10
?offset=20
?limit=10&offset=20
```

**Selecting Fields:**

```
?select=id,name,email
?select=*
?select=id,name,company:companies(name,industry)
```

**Embedding Relations:**

```
// Get contacts with their company
?select=id,name,company_id,companies(name)

// Get company with all contacts
?select=id,name,contacts(id,name,email)
```

---

## Data Dictionary Schema

The platform schema is stored in metadata tables. See `references/tables-schema.md` and `references/fields-schema.md` for complete documentation.

### modules

Defines available modules (CRM, ITSM, LMS, etc.)

Key columns: `id`, `name`, `label`, `description`

### tables

Defines tables within modules.

Key columns:
- `id` - Primary key
- `table_name` - Physical table name (plural, lowercase)
- `singular` - Singular form for labels
- `singular_label`, `plural_label` - Display names
- `module_id` - Reference to modules
- `view_permission`, `edit_permission` - Required permissions
- `id_column` - Primary key column name (usually "id")
- `label_column` - Column to use as display label

### fields

Defines fields/columns within tables.

Key columns:
- `id` - Primary key
- `table_name` - Reference to table
- `field_name` - Physical column name
- `title` - Display label
- `format` - Data type/format
- `is_nullable` - Whether NULL is allowed
- `input_type` - UI input behavior
- `field_order` - Display order

---

## RBAC Tables

See `references/rbac-schema.md` for complete documentation.

### users

User accounts.

### roles

Role definitions.

### permissions

Permission definitions (e.g., "crm.read", "crm.write", "crm.manage").

### user_roles

Maps users to roles (many-to-many).

### roles_permissions

Maps roles to permissions (many-to-many).

### permission_hierarchy

Defines permission inheritance (e.g., "crm.manage" implies "crm.read" and "crm.write").

---

## Common Workflows

### Query Records from Any Table

1. Introspect: Get table info from `/tables`, get fields from `/fields`
2. Build query using PostgREST syntax
3. Execute with `postgrestRequest`

```javascript
// Example: Get all active contacts with company info
semantius:postgrestRequest({
  method: "GET",
  path: "/contacts?status=eq.active&select=id,name,email,companies(name)&order=name.asc&limit=50"
})
```

### Create a Record

```javascript
semantius:postgrestRequest({
  method: "POST",
  path: "/contacts",
  body: {
    name: "John Doe",
    email: "john@example.com",
    status: "active",
    company_id: 123
  }
})
```

### Update a Record

```javascript
semantius:postgrestRequest({
  method: "PATCH",
  path: "/contacts?id=eq.456",
  body: {
    status: "inactive"
  }
})
```

### Delete a Record

```javascript
semantius:postgrestRequest({
  method: "DELETE",
  path: "/contacts?id=eq.456"
})
```

### Complex Query with SQL

When PostgREST syntax is complex, use sqlToRest:

```javascript
// Step 1: Convert SQL
semantius:sqlToRest({
  sql: `
    SELECT c.name, c.email, COUNT(o.id) as order_count
    FROM contacts c
    LEFT JOIN orders o ON c.id = o.contact_id
    WHERE c.status = 'active'
    GROUP BY c.id
    ORDER BY order_count DESC
    LIMIT 10
  `
})

// Step 2: Execute the returned request
semantius:postgrestRequest({
  method: "GET",
  path: "<path from sqlToRest result>"
})
```

---

## Admin Workflows

**Important:** Always call `getCurrentUser` first to verify admin permissions.

### Create a New Module

```javascript
semantius:postgrestRequest({
  method: "POST",
  path: "/modules",
  body: {
    name: "crm",
    label: "Customer Relationship Management",
    description: "CRM module for managing customers, contacts, and deals"
  }
})
```

### Create a New Table

```javascript
semantius:postgrestRequest({
  method: "POST",
  path: "/tables",
  body: {
    table_name: "contacts",
    singular: "contact",
    singular_label: "Contact",
    plural_label: "Contacts",
    module_id: 1,
    view_permission: "crm.read",
    edit_permission: "crm.write",
    id_column: "id",
    label_column: "name",
    description: "Contact records"
  }
})
```

Note: This automatically creates the physical database table.

### Add Fields to a Table

```javascript
// Add email field
semantius:postgrestRequest({
  method: "POST",
  path: "/fields",
  body: {
    table_name: "contacts",
    field_name: "email",
    title: "Email Address",
    format: "email",
    is_nullable: false,
    input_type: "required",
    field_order: 10
  }
})

// Add status field
semantius:postgrestRequest({
  method: "POST",
  path: "/fields",
  body: {
    table_name: "contacts",
    field_name: "status",
    title: "Status",
    format: "choice",
    is_nullable: false,
    input_type: "required",
    field_order: 20
  }
})
```

Note: Each field insert triggers an ALTER TABLE operation.

### Create a Permission

```javascript
semantius:postgrestRequest({
  method: "POST",
  path: "/permissions",
  body: {
    name: "crm.read",
    description: "Read access to CRM module"
  }
})
```

### Assign Permission to Role

```javascript
semantius:postgrestRequest({
  method: "POST",
  path: "/roles_permissions",
  body: {
    role_id: 2,
    permission_id: 5
  }
})
```

### Create Permission Hierarchy

```javascript
// crm.manage implies crm.read
semantius:postgrestRequest({
  method: "POST",
  path: "/permission_hierarchy",
  body: {
    parent_permission_id: 10,
    child_permission_id: 5
  }
})
```

---

## Naming Conventions

**Tables:**
- Always plural lowercase: `contacts`, `companies`, `orders`
- Snake_case for multi-word: `order_items`, `user_roles`

**Fields:**
- Always snake_case lowercase: `first_name`, `created_at`, `company_id`
- Foreign keys: `{singular_table}_id` (e.g., `company_id`, `user_id`)

**Primary Key:**
- Always named `id`

---

## Error Handling

Common PostgREST errors:

| Code | Meaning | Check |
|------|---------|-------|
| 400 | Bad request | Invalid syntax, constraint violation |
| 403 | Forbidden | User lacks permission (check RBAC) |
| 404 | Not found | Table or resource doesn't exist |
| 409 | Conflict | Unique constraint or foreign key violation |

When errors occur:
1. Verify user has required permissions via `getCurrentUser`
2. Check table/field names follow conventions
3. Verify foreign key references exist
4. Ensure required fields are provided
5. Confirm data types match field formats

---

## Best Practices

1. **Always introspect first** — Call `getCurrentUser`, then query `modules`, `tables`, `fields` to understand the schema
2. **Use sqlToRest for complex queries** — When PostgREST syntax gets complicated
3. **Follow naming conventions** — Plural tables, snake_case fields, lowercase everything
4. **Always filter DELETE/UPDATE** — Never omit the WHERE clause equivalent
5. **Check permissions before admin operations** — Use `getCurrentUser` to verify
6. **Query metadata to understand schema** — Don't assume tables/fields exist
7. **Test incrementally** — Start with simple queries, add complexity gradually
```

---

## Step 5: Create skills/semantius/references/postgrest-syntax.md

```markdown
# PostgREST Query Syntax Reference

Complete reference for PostgREST query syntax used in Semantius.

## Filtering

### Comparison Operators

| Operator | Description | Example |
|----------|-------------|---------|
| eq | Equals | `/contacts?status=eq.active` |
| neq | Not equals | `/contacts?status=neq.deleted` |
| gt | Greater than | `/orders?total=gt.1000` |
| gte | Greater than or equal | `/orders?total=gte.1000` |
| lt | Less than | `/orders?total=lt.100` |
| lte | Less than or equal | `/orders?total=lte.100` |
| like | Pattern match (case-sensitive) | `/contacts?name=like.*Smith*` |
| ilike | Pattern match (case-insensitive) | `/contacts?name=ilike.*smith*` |
| match | POSIX regex (case-sensitive) | `/contacts?name=match.^John` |
| imatch | POSIX regex (case-insensitive) | `/contacts?name=imatch.^john` |
| in | In list | `/contacts?status=in.(active,pending)` |
| is | IS comparison (null, true, false) | `/contacts?deleted_at=is.null` |
| isdistinct | IS DISTINCT FROM | `/contacts?status=isdistinct.null` |
| fts | Full-text search | `/posts?title=fts.database` |
| plfts | Phrase full-text search | `/posts?title=plfts.the%20database` |
| wfts | Websearch full-text search | `/posts?title=wfts.database` |
| cs | Contains (arrays/ranges) | `/events?tags=cs.{meeting,important}` |
| cd | Contained by | `/events?tags=cd.{meeting,call,email}` |
| ov | Overlaps | `/events?period=ov.[2023-01-01,2023-12-31]` |
| sl | Strictly left of | `/ranges?range=sl.[10,20]` |
| sr | Strictly right of | `/ranges?range=sr.[10,20]` |
| nxr | Not right of | `/ranges?range=nxr.[10,20]` |
| nxl | Not left of | `/ranges?range=nxl.[10,20]` |
| adj | Adjacent | `/ranges?range=adj.[10,20]` |

### Negation

Prefix any operator with `not.`:

```
/contacts?status=not.eq.deleted
/contacts?email=not.is.null
/contacts?status=not.in.(archived,deleted)
```

### Logical Operators

**AND (default):**
Multiple conditions are AND'd by default:
```
/contacts?status=eq.active&type=eq.lead
```

**Explicit AND:**
```
/contacts?and=(status.eq.active,type.eq.lead)
```

**OR:**
```
/contacts?or=(status.eq.active,status.eq.pending)
```

**Complex combinations:**
```
/contacts?and=(or=(status.eq.active,status.eq.pending),type.eq.lead)
```

## Ordering

### Basic ordering

```
/contacts?order=name.asc
/contacts?order=created_at.desc
```

### Multiple columns

```
/contacts?order=priority.desc,name.asc
```

### Null handling

```
/contacts?order=name.asc.nullsfirst
/contacts?order=name.asc.nullslast
```

## Pagination

### Limit and offset

```
/contacts?limit=10
/contacts?limit=10&offset=20
```

### Range header (alternative)

Request header: `Range: 0-24` returns first 25 records.

## Selecting Columns

### Specific columns

```
/contacts?select=id,name,email
```

### All columns

```
/contacts?select=*
```

### Renaming columns

```
/contacts?select=contact_name:name,contact_email:email
```

### Computed/cast columns

```
/contacts?select=id,name,age::text
```

## Embedding (Joins)

### Many-to-one (child to parent)

```
/contacts?select=id,name,company:companies(id,name)
```

### One-to-many (parent to children)

```
/companies?select=id,name,contacts(id,name,email)
```

### Filtering on embedded resources

```
/companies?select=*,contacts(*)&contacts.status=eq.active
```

### Nested embedding

```
/companies?select=*,contacts(*,orders(*))
```

### Limiting embedded resources

```
/companies?select=*,contacts(*)&contacts.limit=5
```

### Ordering embedded resources

```
/companies?select=*,contacts(*)&contacts.order=name.asc
```

## Inserting Data

### Single record

```
POST /contacts
Content-Type: application/json

{"name": "John", "email": "john@example.com"}
```

### Multiple records

```
POST /contacts
Content-Type: application/json

[
  {"name": "John", "email": "john@example.com"},
  {"name": "Jane", "email": "jane@example.com"}
]
```

### Return inserted record

```
POST /contacts?select=id,name,email
Prefer: return=representation
```

### Upsert (insert or update)

```
POST /contacts
Prefer: resolution=merge-duplicates

{"id": 123, "name": "John Updated"}
```

## Updating Data

### Update with filter

```
PATCH /contacts?id=eq.123
Content-Type: application/json

{"status": "inactive"}
```

### Update multiple records

```
PATCH /contacts?status=eq.pending
Content-Type: application/json

{"status": "active"}
```

### Return updated records

```
PATCH /contacts?id=eq.123&select=id,name,status
Prefer: return=representation
```

## Deleting Data

### Delete with filter

```
DELETE /contacts?id=eq.123
```

### Delete multiple records

```
DELETE /contacts?status=eq.archived
```

### Return deleted records

```
DELETE /contacts?id=eq.123&select=*
Prefer: return=representation
```

## Special Features

### Count

Request header: `Prefer: count=exact`

Response header includes: `Content-Range: 0-24/1000`

### Singular response

For queries expected to return one row:
```
GET /contacts?id=eq.123
Accept: application/vnd.pgrst.object+json
```

### Bulk insert with CSV

```
POST /contacts
Content-Type: text/csv

name,email
John,john@example.com
Jane,jane@example.com
```
```

---

## Step 6: Create skills/semantius/references/tables-schema.md

```markdown
# Tables Schema Reference

The `tables` table defines all data tables in the system.

## Columns

| Column | Type | Required | Description |
|--------|------|----------|-------------|
| id | integer | Auto | Primary key |
| table_name | text | Yes | Physical table name (plural, lowercase, snake_case) |
| singular | text | Yes | Singular form (e.g., "contact" for "contacts") |
| singular_label | text | Yes | Display label singular (e.g., "Contact") |
| plural_label | text | Yes | Display label plural (e.g., "Contacts") |
| module_id | integer | Yes | Foreign key to modules.id |
| view_permission | text | No | Permission required to view records |
| edit_permission | text | No | Permission required to edit records |
| id_column | text | Yes | Primary key column name (default: "id") |
| label_column | text | No | Column to use as display label |
| description | text | No | Description of the table |
| icon | text | No | Icon identifier for UI |
| is_system | boolean | No | Whether this is a system table |
| created_at | timestamp | Auto | Creation timestamp |
| updated_at | timestamp | Auto | Last update timestamp |

## Naming Conventions

**table_name:**
- Always plural: `contacts`, `companies`, `orders`
- Always lowercase
- Use snake_case for multi-word: `order_items`, `user_roles`

**singular:**
- Singular form of table_name: `contact`, `company`, `order`

## Behavior

When a row is inserted into `tables`:
1. A physical PostgreSQL table is created
2. An `id` column (primary key) is automatically added
3. RLS policies are configured based on permissions

When a row is deleted from `tables`:
1. The physical table is dropped
2. All related field definitions are removed

## Examples

### Create a contacts table

```javascript
postgrestRequest({
  method: "POST",
  path: "/tables",
  body: {
    table_name: "contacts",
    singular: "contact",
    singular_label: "Contact",
    plural_label: "Contacts",
    module_id: 1,
    view_permission: "crm.read",
    edit_permission: "crm.write",
    id_column: "id",
    label_column: "name",
    description: "Customer contacts"
  }
})
```

### Query all tables in a module

```javascript
postgrestRequest({
  method: "GET",
  path: "/tables?module_id=eq.1&select=id,table_name,singular_label,plural_label,description"
})
```
```

---

## Step 7: Create skills/semantius/references/fields-schema.md

```markdown
# Fields Schema Reference

The `fields` table defines all columns/fields for data tables.

## Columns

| Column | Type | Required | Description |
|--------|------|----------|-------------|
| id | integer | Auto | Primary key |
| table_name | text | Yes | Reference to tables.table_name |
| field_name | text | Yes | Physical column name (snake_case) |
| title | text | Yes | Display label |
| format | text | Yes | Data type/format (see below) |
| is_nullable | boolean | Yes | Whether NULL is allowed |
| input_type | text | No | UI input behavior (see below) |
| width | text | No | Display width hint (xs, s, m, l, xl) |
| field_order | integer | No | Display order (lower = first) |
| description | text | No | Help text for the field |
| default_value | text | No | Default value expression |
| reference_table | text | No | For foreign keys: referenced table |
| reference_column | text | No | For foreign keys: referenced column |
| choices | jsonb | No | For choice fields: array of options |
| validation | jsonb | No | Validation rules |
| is_system | boolean | No | Whether this is a system field |
| created_at | timestamp | Auto | Creation timestamp |
| updated_at | timestamp | Auto | Last update timestamp |

## Format Values

| Format | PostgreSQL Type | Description |
|--------|-----------------|-------------|
| text | text | Free-form text |
| textarea | text | Multi-line text |
| email | text | Email address (with validation) |
| url | text | URL (with validation) |
| phone | text | Phone number |
| integer | integer | Whole number |
| decimal | numeric | Decimal number |
| currency | numeric | Money value |
| boolean | boolean | True/false |
| date | date | Date only |
| datetime | timestamp | Date and time |
| time | time | Time only |
| choice | text | Single choice from list |
| multichoice | text[] | Multiple choices from list |
| reference | integer | Foreign key to another table |
| json | jsonb | JSON data |
| file | text | File attachment reference |
| image | text | Image attachment reference |

## Input Type Values

| Input Type | Description |
|------------|-------------|
| required | Field is required, shown prominently |
| optional | Field is optional, shown normally |
| readonly | Field is read-only |
| hidden | Field is not shown in UI |
| calculated | Field is computed, not editable |

## Naming Conventions

**field_name:**
- Always snake_case lowercase: `first_name`, `email_address`
- Foreign keys: `{singular_table}_id` (e.g., `company_id`)
- Timestamps: `created_at`, `updated_at`, `deleted_at`
- Booleans: `is_active`, `has_subscription`

## Behavior

When a row is inserted into `fields`:
1. An ALTER TABLE adds the column to the physical table
2. Constraints are applied based on is_nullable, format, etc.

When a row is updated:
1. Column definition may be altered

When a row is deleted:
1. The physical column is dropped

## Examples

### Add an email field

```javascript
postgrestRequest({
  method: "POST",
  path: "/fields",
  body: {
    table_name: "contacts",
    field_name: "email",
    title: "Email Address",
    format: "email",
    is_nullable: false,
    input_type: "required",
    width: "m",
    field_order: 10
  }
})
```

### Add a foreign key field

```javascript
postgrestRequest({
  method: "POST",
  path: "/fields",
  body: {
    table_name: "contacts",
    field_name: "company_id",
    title: "Company",
    format: "reference",
    is_nullable: true,
    input_type: "optional",
    reference_table: "companies",
    reference_column: "id",
    field_order: 20
  }
})
```

### Add a choice field

```javascript
postgrestRequest({
  method: "POST",
  path: "/fields",
  body: {
    table_name: "contacts",
    field_name: "status",
    title: "Status",
    format: "choice",
    is_nullable: false,
    input_type: "required",
    choices: ["active", "inactive", "pending", "archived"],
    default_value: "active",
    field_order: 30
  }
})
```

### Query all fields for a table

```javascript
postgrestRequest({
  method: "GET",
  path: "/fields?table_name=eq.contacts&order=field_order.asc&select=field_name,title,format,is_nullable,input_type"
})
```
```

---

## Step 8: Create skills/semantius/references/rbac-schema.md

```markdown
# RBAC Schema Reference

Role-Based Access Control tables that manage user permissions.

## Tables Overview

```
users
  └── user_roles (many-to-many)
        └── roles
              └── roles_permissions (many-to-many)
                    └── permissions
                          └── permission_hierarchy (self-referential)
```

## users

User accounts.

| Column | Type | Required | Description |
|--------|------|----------|-------------|
| id | integer | Auto | Primary key |
| email | text | Yes | Unique email address |
| name | text | No | Display name |
| is_active | boolean | Yes | Whether user can log in |
| created_at | timestamp | Auto | Creation timestamp |
| updated_at | timestamp | Auto | Last update timestamp |

## roles

Role definitions.

| Column | Type | Required | Description |
|--------|------|----------|-------------|
| id | integer | Auto | Primary key |
| name | text | Yes | Unique role name (e.g., "admin", "sales_rep") |
| label | text | Yes | Display label (e.g., "Administrator", "Sales Representative") |
| description | text | No | Role description |
| is_system | boolean | No | Whether this is a system role |
| created_at | timestamp | Auto | Creation timestamp |

## permissions

Permission definitions.

| Column | Type | Required | Description |
|--------|------|----------|-------------|
| id | integer | Auto | Primary key |
| name | text | Yes | Unique permission name (e.g., "crm.read", "crm.write") |
| description | text | No | Permission description |
| module_id | integer | No | Associated module (optional) |
| created_at | timestamp | Auto | Creation timestamp |

### Permission Naming Convention

Use dot notation: `{module}.{action}`

Examples:
- `crm.read` - Read CRM data
- `crm.write` - Create/update CRM data
- `crm.delete` - Delete CRM data
- `crm.manage` - Full CRM access (typically implies read, write, delete)
- `admin.users` - Manage users
- `admin.roles` - Manage roles

## user_roles

Maps users to roles (many-to-many).

| Column | Type | Required | Description |
|--------|------|----------|-------------|
| id | integer | Auto | Primary key |
| user_id | integer | Yes | Foreign key to users.id |
| role_id | integer | Yes | Foreign key to roles.id |
| created_at | timestamp | Auto | Creation timestamp |

## roles_permissions

Maps roles to permissions (many-to-many).

| Column | Type | Required | Description |
|--------|------|----------|-------------|
| id | integer | Auto | Primary key |
| role_id | integer | Yes | Foreign key to roles.id |
| permission_id | integer | Yes | Foreign key to permissions.id |
| created_at | timestamp | Auto | Creation timestamp |

## permission_hierarchy

Defines permission inheritance (e.g., "crm.manage" implies "crm.read").

| Column | Type | Required | Description |
|--------|------|----------|-------------|
| id | integer | Auto | Primary key |
| parent_permission_id | integer | Yes | The higher-level permission |
| child_permission_id | integer | Yes | The implied permission |
| created_at | timestamp | Auto | Creation timestamp |

When a user has `parent_permission`, they automatically have `child_permission`.

## Common Workflows

### Create a new role

```javascript
postgrestRequest({
  method: "POST",
  path: "/roles",
  body: {
    name: "sales_manager",
    label: "Sales Manager",
    description: "Can manage sales team and view reports"
  }
})
```

### Create permissions for a module

```javascript
// Create base permissions
postgrestRequest({
  method: "POST",
  path: "/permissions",
  body: [
    { name: "crm.read", description: "View CRM data" },
    { name: "crm.write", description: "Create and edit CRM data" },
    { name: "crm.delete", description: "Delete CRM data" },
    { name: "crm.manage", description: "Full CRM access" }
  ]
})
```

### Set up permission hierarchy

```javascript
// crm.manage implies crm.read, crm.write, crm.delete
postgrestRequest({
  method: "POST",
  path: "/permission_hierarchy",
  body: [
    { parent_permission_id: 4, child_permission_id: 1 }, // manage -> read
    { parent_permission_id: 4, child_permission_id: 2 }, // manage -> write
    { parent_permission_id: 4, child_permission_id: 3 }  // manage -> delete
  ]
})
```

### Assign permissions to a role

```javascript
postgrestRequest({
  method: "POST",
  path: "/roles_permissions",
  body: [
    { role_id: 2, permission_id: 1 },
    { role_id: 2, permission_id: 2 }
  ]
})
```

### Assign role to user

```javascript
postgrestRequest({
  method: "POST",
  path: "/user_roles",
  body: {
    user_id: 10,
    role_id: 2
  }
})
```

### Query user's effective permissions

```javascript
// Get user with roles and permissions
postgrestRequest({
  method: "GET",
  path: "/users?id=eq.10&select=*,user_roles(roles(name,roles_permissions(permissions(name))))"
})
```

### Check if role has specific permission

```javascript
postgrestRequest({
  method: "GET",
  path: "/roles_permissions?role_id=eq.2&permissions.name=eq.crm.read&select=id,permissions(name)"
})
```
```

---

## Step 9: Create skills/semantius/metadata.json

```json
{
  "name": "semantius",
  "version": "1.0.0",
  "description": "Universal System of Records platform skill for PostgREST-based data management",
  "author": "Semantius",
  "license": "MIT",
  "keywords": ["postgrest", "database", "crm", "itsm", "system-of-records", "rbac"],
  "repository": {
    "type": "git",
    "url": "https://github.com/semantius/agent-skills"
  }
}
```

---

## Step 10: Create skills/semantius/README.md

```markdown
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
```

---

## Final Checklist

After Claude Code completes the task, verify:

- [ ] `skills/postgres-best-practices/` is deleted
- [ ] `packages/` directory is deleted
- [ ] Root README.md references Semantius
- [ ] package.json has name `semantius-agent-skills`
- [ ] `skills/semantius/SKILL.md` exists with all sections
- [ ] `skills/semantius/references/` contains all 4 reference files
- [ ] `skills/semantius/metadata.json` exists
- [ ] `skills/semantius/README.md` exists
- [ ] No Supabase references remain in any file