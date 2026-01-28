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

> ⚠️ **Do not create artifacts for data operations. Use direct tool calls.**

## Usage Guidelines

**Direct tool calls vs Artifacts:**
- Use direct tool calls (`Semantius:postgrestRequest`, `Semantius:getCurrentUser`) for:
  - Creating modules, tables, fields
  - Inserting, updating, or deleting records
  - Querying data
  - Any one-off or administrative task

- Only create artifacts when the user explicitly requests an interactive UI or app

**Always introspect using metadata tables:**
- Query `/tables?table_name=eq.{name}` to understand table configuration (permissions, label columns, etc.)
- Query `/fields?table_name=eq.{name}&order=field_order.asc` to get full field definitions (formats, enums, searchable, required, etc.)
- Do not use `?select=*&limit=1` on data tables - this only shows raw values, not schema metadata
- Read the reference files in `/references/` for authoritative schema details - do not rely solely on the summary column lists in this file

**PostgREST bulk insert constraint:**
- When inserting multiple records as an array, all objects must have identical keys
- If objects have different optional fields, insert them one at a time

---

## Available MCP Tools

---

## Quick Reference: What to Read When

Before performing these tasks, you MUST read the corresponding reference file:

| Task | Read First | Critical Info |
|------|------------|---------------|
| **Adding fields to tables** | `/references/schema-fields-schema.md` | Lines 42-93: Valid format values (40+ formats) |
| **Creating tables** | `/references/schema-tables-schema.md` | Required fields, auto-generated columns |
| **Querying data** | `/references/api-postgrest-syntax.md` | PostgREST operators, filters |
| **Configuring RBAC** | `/references/rbac-rbac-schema.md` | Users, roles, permissions structure |

---

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

> **Note:** For bulk inserts, all objects MUST have identical keys (PGRST102 error).
> See `references/api-postgrest-syntax.md` for details.

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

## Introspection — The ONLY Way to Discover Schema

⚠️ **Important:** There is no schema discovery endpoint at `/`. You cannot query the root path or use database introspection commands directly. The metadata tables (`modules`, `tables`, `fields`) ARE the schema discovery mechanism.

**Never attempt:**
- ❌ `GET /` (requires secret API key, will return 401 Unauthorized)
- ❌ Querying table paths without first verifying they exist in `/tables`
- ❌ Using column names without first checking `/fields`

**The Rule:** Always query the metadata tables first to discover what exists, then query the actual data.

> **CRITICAL:** The platform is dynamic. Before ANY data operation, you MUST introspect
> to discover actual table and column names. Never guess or assume based on documentation
> examples — always verify by querying the `/tables` and `/fields` metadata tables.

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

See `references/api-postgrest-syntax.md` for complete documentation.

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

The platform schema is stored in metadata tables. See `references/schema-tables-schema.md` and `references/schema-fields-schema.md` for complete documentation.

### modules

Defines available modules (CRM, ITSM, LMS, etc.)

Key columns: `id`, `module_name`, `description`, `view_permission`, `home_page`

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

See `references/rbac-rbac-schema.md` for complete documentation.

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

**Naming rules:**
- `table_name`: plural, lowercase, snake_case (e.g., `contacts`, `order_items`)
- `singular`: singular form (e.g., `contact`, `order_item`)
- `label_column`: must be a field that will exist (default: `label`, or specify like `name`)

⚠️ Table names MUST be plural lowercase. `Contact` or `contact` will cause issues.

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

**⚠️ What Gets Created Automatically:**

A new table contains ONLY these 4 core fields:
- `id` (integer, primary key)
- `label` (text) - Named according to your `label_column` value (e.g., "name" if you specified `label_column: "name"`)
- `created_at` (timestamp)
- `updated_at` (timestamp)

**Common mistake:** Assuming fields like `email`, `phone`, `description`, `status` exist automatically. They don't - you must create them.


### Add Fields to a Table

**Valid `input_type` values:** `default` | `required` | `readonly` | `disabled` | `hidden`

**Valid `format` values:** `text` | `email` | `enum` | `reference` | `date` | `date-time` | `time` | `boolean` | `integer` | `int32` | `int64` | `float` | `double` | `number` | `string` | `uuid` | `uri` | `url` | `json` | `html` | `code` | `password` | `ipv4` | `ipv6` | `hostname`

**Valid `width` values:** `s` (small) | `m` (medium) | `w` (wide)

⚠️ **Do NOT use values outside these lists.** If unsure, read `/references/schema-fields-schema.md`.

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
    format: "enum",
    enum_values: ["active", "inactive"],
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

1. **Always introspect first** — Call `getCurrentUser`, then query `modules`, `tables`, `fields` metadata to understand the schema
2. **Use sqlToRest for complex queries** — When PostgREST syntax gets complicated
3. **Follow naming conventions** — Plural tables, snake_case fields, lowercase everything
4. **Always filter DELETE/UPDATE** — Never omit the WHERE clause equivalent
5. **Check permissions before admin operations** — Use `getCurrentUser` to verify
6. **Query metadata to understand schema** — Don't assume tables/fields exist
7. **Test incrementally** — Start with simple queries, add complexity gradually
8. **Bulk inserts require identical keys** — All objects in a bulk insert array must have the exact same keys (PGRST102 error)
