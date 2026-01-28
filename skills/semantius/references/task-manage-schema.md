---
impact: HIGH
title: Managing Schema (Modules, Tables, Fields)
---

# Managing Schema

This reference covers creating, updating, and deleting modules, tables, and fields.

**Workflow order:** Modules → Tables → Fields (create in this order, delete in reverse)

---

## 1. Modules

Modules are logical containers that group related tables.

### Module Columns

| Column | Type | Required | Default | Description |
|--------|------|----------|---------|-------------|
| id | serial | Auto | - | Primary key |
| module_name | text | Yes | '' | Unique identifier (e.g., "crm", "inventory") |
| description | text | No | '' | Module description |
| view_permission | text | Yes | 'user:read' | Permission required to view |
| logo_url | text | No | '' | URL or path to module logo |
| logo_color | text | No | '' | Color for branding (hex or name) |
| home_page | text | Yes | '/' | Default landing page |
| alias | text | Yes | '' | URL alias |
| settings | jsonb | No | null | Module-specific settings |

### Create Module

```javascript
postgrestRequest({
  method: "POST",
  path: "/modules",
  body: {
    module_name: "crm",
    description: "Customer Relationship Management",
    view_permission: "crm:read",
    home_page: "/crm/contacts",
    alias: "crm"
  }
})
```

### Update Module

```javascript
postgrestRequest({
  method: "PATCH",
  path: "/modules?id=eq.2",
  body: { description: "Updated description" }
})
```

### Delete Module

```javascript
postgrestRequest({
  method: "DELETE",
  path: "/modules?id=eq.2"
})
```

Tables in the module will have `module_id` set to NULL (not deleted).

---

## 2. Tables

Tables define data structures within modules.

### Table Columns

| Column | Type | Required | Default | Description |
|--------|------|----------|---------|-------------|
| table_name | text | Yes | - | Primary key (plural, lowercase, snake_case) |
| singular | text | Yes | '' | Singular form (e.g., "contact") |
| singular_label | text | Yes | '' | Display label singular (e.g., "Contact") |
| plural_label | text | Yes | '' | Display label plural (e.g., "Contacts") |
| module_id | integer | No | null | Foreign key to modules.id |
| view_permission | text | Yes | 'public:read' | Permission for SELECT |
| edit_permission | text | Yes | 'admin' | Permission for INSERT/UPDATE/DELETE |
| id_column | text | Yes | 'id' | Primary key column name |
| label_column | text | Yes | 'label' | Column for display label |
| description | text | No | '' | Table description |
| managed | boolean | Yes | true | Enable automatic DDL |
| searchable | boolean | Yes | false | Include in search |

### Naming Rules

| Field | Rule | Correct | Incorrect |
|-------|------|---------|-----------|
| table_name | plural, lowercase, snake_case | `contacts`, `order_items` | `Contact`, `orderItems` |
| singular | singular form | `contact`, `order_item` | `contacts` |

### Auto-Created Fields

When you create a table, these 4 fields are automatically added:
- `id` (integer, primary key)
- `label` (text) - or named per your `label_column`
- `created_at` (timestamp)
- `updated_at` (timestamp)

**No other fields exist until you create them.**

### Create Table

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
    view_permission: "crm:read",
    edit_permission: "crm:write",
    label_column: "name",
    description: "Customer contacts"
  }
})
```

### Common Mistakes

```javascript
// WRONG: table_name is singular
{ table_name: "contact" }  // Must be plural: "contacts"

// WRONG: table_name has uppercase
{ table_name: "ContactList" }  // Must be lowercase: "contact_list"

// WRONG: assuming fields exist
// After creating "contacts" table, you cannot immediately insert
// { email: "test@example.com" } - the email field doesn't exist yet!
```

### Update Table

```javascript
postgrestRequest({
  method: "PATCH",
  path: "/tables?table_name=eq.contacts",
  body: { description: "Updated description" }
})
```

### Delete Table

```javascript
postgrestRequest({
  method: "DELETE",
  path: "/tables?table_name=eq.contacts"
})
```

**Warning:** This drops the physical table and all its data.

---

## 3. Fields

Fields define columns within tables.

### Field Columns

| Column | Type | Required | Default | Description |
|--------|------|----------|---------|-------------|
| id | varchar | Auto | - | Generated: `table_name.field_name` |
| table_name | text | Yes | - | Reference to tables.table_name |
| field_name | text | Yes | '' | Column name (snake_case) |
| title | text | Yes | '' | Display name |
| description | text | No | '' | Detailed description |
| format | text | Yes | 'text' | Data type - **see valid values below** |
| is_nullable | boolean | Yes | true | Allow NULL |
| default_value | text | No | '' | Default (SQL expression) |
| field_order | integer | Yes | 0 | Display order |
| input_type | text | Yes | 'default' | UI behavior - **see valid values below** |
| width | text | Yes | 'm' | Display width - **see valid values below** |
| searchable | boolean | Yes | false | Include in search |
| enum_values | jsonb | No | null | Allowed values for enum format |
| reference_table | text | Yes | '' | Target table for reference format |
| reference_delete_mode | text | Yes | 'restrict' | ON DELETE behavior |

### Valid `input_type` Values

| Value | Description |
|-------|-------------|
| `default` | Normal optional field |
| `required` | Required, shown prominently |
| `readonly` | Visible but not editable |
| `disabled` | Greyed out |
| `hidden` | Not shown in UI |

**Only use these 5 values. Do NOT invent values like "mandatory", "optional", "visible".**

### Valid `format` Values

| Format | Description | Notes |
|--------|-------------|-------|
| `text` | Free-form text | Default |
| `string` | String primitive | |
| `email` | Email address | Validation applied |
| `url` / `uri` | URL | |
| `integer` | Integer | |
| `int32` | 32-bit integer | |
| `int64` | 64-bit integer | |
| `number` | Number | |
| `float` | Float | |
| `double` | Double | |
| `boolean` | Boolean | |
| `date` | Date (YYYY-MM-DD) | |
| `time` | Time (HH:MM:SS) | |
| `date-time` | ISO 8601 datetime | |
| `enum` | Single choice | Requires `enum_values` |
| `reference` | Foreign key | Requires `reference_table` |
| `json` | JSON data | |
| `html` | HTML content | |
| `code` | Source code | |
| `uuid` | UUID | |
| `password` | Masked in UI | |
| `ipv4` | IPv4 address | |
| `ipv6` | IPv6 address | |
| `hostname` | Hostname | |

**Only use these values. Do NOT invent values like "emailAddress", "varchar", "bigint".**

### Valid `width` Values

| Value | Description |
|-------|-------------|
| `s` | Small (checkboxes, short text) |
| `m` | Medium (default) |
| `w` | Wide (full-width, textareas) |

### Valid `reference_delete_mode` Values

| Value | SQL | Description |
|-------|-----|-------------|
| `restrict` | ON DELETE RESTRICT | Prevent deletion if referenced |
| `clear` | ON DELETE SET NULL | Set to NULL when deleted |

### Naming Rules

| Pattern | Example |
|---------|---------|
| snake_case lowercase | `first_name`, `email_address` |
| Foreign keys | `company_id`, `user_id` |
| Timestamps | `created_at`, `deleted_at` |
| Booleans | `is_active`, `has_subscription` |

### Create Field Examples

**Text field (email):**
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

**Enum field:**
```javascript
postgrestRequest({
  method: "POST",
  path: "/fields",
  body: {
    table_name: "contacts",
    field_name: "status",
    title: "Status",
    format: "enum",
    enum_values: ["active", "inactive", "pending"],
    is_nullable: false,
    input_type: "required",
    default_value: "'active'",
    field_order: 20
  }
})
```

**Foreign key field:**
```javascript
postgrestRequest({
  method: "POST",
  path: "/fields",
  body: {
    table_name: "contacts",
    field_name: "company_id",
    title: "Company",
    format: "reference",
    reference_table: "companies",
    reference_delete_mode: "clear",
    is_nullable: true,
    input_type: "default",
    field_order: 30
  }
})
```

**Searchable notes field:**
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

### Common Mistakes

```javascript
// WRONG: Invalid input_type
{ input_type: "mandatory" }  // Use "required"
{ input_type: "optional" }   // Use "default"

// WRONG: Invalid format
{ format: "varchar" }        // Use "text" or "string"
{ format: "emailAddress" }   // Use "email"
{ format: "bigint" }         // Use "int64"

// WRONG: reference without reference_table
{ format: "reference" }      // Must include reference_table

// WRONG: enum without enum_values
{ format: "enum" }           // Must include enum_values
```

### Update Field

```javascript
postgrestRequest({
  method: "PATCH",
  path: "/fields?id=eq.contacts.email",
  body: { title: "Email", searchable: true }
})
```

### Delete Field

```javascript
postgrestRequest({
  method: "DELETE",
  path: "/fields?id=eq.contacts.notes"
})
```

Core fields (`id`, `label`, `created_at`, `updated_at`) cannot be deleted.

---

## Query Examples

**List all modules:**
```javascript
postgrestRequest({
  method: "GET",
  path: "/modules?select=id,module_name,description"
})
```

**List tables in a module:**
```javascript
postgrestRequest({
  method: "GET",
  path: "/tables?module_id=eq.1&select=table_name,singular_label,plural_label"
})
```

**List fields for a table:**
```javascript
postgrestRequest({
  method: "GET",
  path: "/fields?table_name=eq.contacts&order=field_order.asc&select=field_name,title,format,input_type"
})
```
