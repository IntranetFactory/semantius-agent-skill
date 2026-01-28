---
impact: HIGH
title: Fields Schema Reference
---

## Fields Schema Reference

The `fields` table defines all columns/fields for data tables.

## Columns

| Column | Type | Required | Default | Description |
|--------|------|----------|---------|-------------|
| id | varchar | Auto | - | Generated PK: `table_name.field_name` |
| table_name | text | Yes | - | Reference to tables.table_name (CASCADE delete) |
| field_name | text | Yes | '' | Physical column name (snake_case) |
| title | text | Yes | '' | Human-readable display name |
| description | text | No | '' | Detailed description (used for COMMENT ON COLUMN) |
| format | text | Yes | 'text' | Data type/format (see below) |
| is_pk | boolean | Yes | false | Whether this field is the primary key |
| is_nullable | boolean | Yes | true | Whether NULL is allowed |
| default_value | text | No | '' | Default value (as SQL expression) |
| field_order | integer | Yes | 0 | Display order (lower = first) |
| input_type | text | Yes | 'default' | UI input behavior (see below) |
| width | text | Yes | 'm' | Display width: s (small), m (medium), w (wide) |
| ctype | text | No | '' | Special column type: '', 'id', or 'label' |
| is_core | boolean | Yes | false | Core system field (cannot be deleted/modified structurally) |
| searchable | boolean | Yes | false | Whether field is included in search |
| enum_values | jsonb | No | null | Array of allowed enum values |
| reference_table | text | Yes | '' | Referenced table for foreign keys (empty = none) |
| reference_delete_mode | text | Yes | 'restrict' | ON DELETE behavior: 'restrict' or 'clear' |
| created_at | timestamp | Auto | CURRENT_TIMESTAMP | Creation timestamp |
| updated_at | timestamp | Auto | CURRENT_TIMESTAMP | Last update timestamp |

## Constraints

- `field_name` must match pattern `^[a-z_][a-z0-9_]*$`
- Only one field per table can have `is_pk = true`
- When `format = 'reference'`, `reference_table` must not be empty
- `reference_table` must reference an existing table in `tables`

## Format Values

| Format | Category | Description |
|--------|----------|-------------|
| **Custom Formats** | | |
| json | Custom | JSON data |
| html | Custom | HTML content |
| text | Custom | Free-form text |
| code | Custom | Source code |
| jsonata | Custom | JSONata expression |
| reference | Custom | Foreign key to another table |
| enum | Custom | Single choice from enum_values |
| **Date/Time** | | |
| date | JSON Schema | Date only (YYYY-MM-DD) |
| time | JSON Schema | Time only (HH:MM:SS) |
| date-time | JSON Schema | Date and time (ISO 8601) |
| duration | JSON Schema | Duration (ISO 8601) |
| **URI/URL** | | |
| uri | JSON Schema | Full URI |
| uri-reference | JSON Schema | URI reference |
| uri-template | JSON Schema | URI template |
| url | JSON Schema | URL (alias for uri) |
| **Network** | | |
| email | JSON Schema | Email address |
| hostname | JSON Schema | Hostname |
| ipv4 | JSON Schema | IPv4 address |
| ipv6 | JSON Schema | IPv6 address |
| **Identifiers** | | |
| uuid | JSON Schema | UUID |
| regex | JSON Schema | Regular expression |
| **JSON Pointers** | | |
| json-pointer | JSON Schema | JSON pointer |
| json-pointer-uri-fragment | JSON Schema | JSON pointer URI fragment |
| relative-json-pointer | JSON Schema | Relative JSON pointer |
| **Numeric** | | |
| byte | OpenAPI | Base64-encoded bytes |
| int32 | OpenAPI | 32-bit integer |
| int64 | OpenAPI | 64-bit integer |
| float | OpenAPI | Float |
| double | OpenAPI | Double |
| **Primitives** | | |
| string | JSON Schema | String primitive |
| number | JSON Schema | Number primitive |
| integer | JSON Schema | Integer primitive |
| boolean | JSON Schema | Boolean primitive |
| object | JSON Schema | Object primitive |
| array | JSON Schema | Array primitive |
| null | JSON Schema | Null primitive |
| **Other** | | |
| password | OpenAPI | Password (masked in UI) |
| binary | OpenAPI | Binary data |

## Input Type Values

| Input Type | Description |
|------------|-------------|
| default | Normal optional field |
| required | Field is required, shown prominently |
| readonly | Field is read-only (visible but not editable) |
| disabled | Field is disabled (greyed out) |
| hidden | Field is not shown in UI |

## Width Values

| Width | Description |
|-------|-------------|
| s | Small (compact fields like checkboxes, short text) |
| m | Medium (default, standard input width) |
| w | Wide (full-width fields like descriptions, textareas) |

## CType Values

| CType | Description |
|-------|-------------|
| '' | Normal field (default) |
| id | Primary key column |
| label | Display label column |

## Reference Delete Modes

| Mode | SQL Equivalent | Description |
|------|----------------|-------------|
| restrict | ON DELETE RESTRICT | Prevent deletion if referenced |
| clear | ON DELETE SET NULL | Set to NULL when referenced row deleted |

## Naming Conventions

**field_name:**
- Always snake_case lowercase: `first_name`, `email_address`
- Foreign keys: `{singular_table}_id` (e.g., `company_id`)
- Timestamps: `created_at`, `updated_at`, `deleted_at`
- Booleans: `is_active`, `has_subscription`

## Core Fields

When a table is created, these core fields are auto-generated with `is_core = true`:
- `id` - Primary key (ctype: 'id')
- `label` - Display label (ctype: 'label')
- `created_at` - Creation timestamp
- `updated_at` - Last update timestamp

Core fields cannot be deleted or have structural changes.

## Behavior

When a row is inserted into `fields`:
1. ALTER TABLE adds the column to the physical table
2. Constraints are applied based on is_nullable, format, reference_table

When a row is updated:
1. Column definition may be altered (if not is_core)

When a row is deleted:
1. The physical column is dropped (if not is_core)

## Examples

**Correct (Add an email field):**
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

**Incorrect (Invalid format for email):**
```javascript
postgrestRequest({
  method: "POST",
  path: "/fields",
  body: {
    table_name: "contacts",
    field_name: "email",
    title: "Email Address",
    format: "text", // Should be "email" for validation
    input_type: "required"
  }
})
```

**Correct (Add a foreign key field):**
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
    input_type: "default",
    reference_table: "companies",
    reference_delete_mode: "clear",
    field_order: 20
  }
})
```

**Correct (Add an enum field):**
```javascript
postgrestRequest({
  method: "POST",
  path: "/fields",
  body: {
    table_name: "contacts",
    field_name: "status",
    title: "Status",
    format: "enum",
    is_nullable: false,
    input_type: "required",
    enum_values: ["active", "inactive", "pending", "archived"],
    default_value: "'active'",
    field_order: 30
  }
})
```

**Correct (Add a searchable description field):**
```javascript
postgrestRequest({
  method: "POST",
  path: "/fields",
  body: {
    table_name: "contacts",
    field_name: "notes",
    title: "Notes",
    format: "text",
    width: "w",
    searchable: true,
    field_order: 40
  }
})
```

**Correct (Query all fields for a table):**
```javascript
postgrestRequest({
  method: "GET",
  path: "/fields?table_name=eq.contacts&order=field_order.asc&select=id,field_name,title,format,is_nullable,input_type"
})
```

**Correct (Get field by composite ID):**
```javascript
postgrestRequest({
  method: "GET",
  path: "/fields?id=eq.contacts.email"
})
```
