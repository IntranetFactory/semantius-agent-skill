---
impact: HIGH
title: Tables Schema Reference
---

## Tables Schema Reference

The `tables` table defines all data tables in the system.

## Columns

| Column | Type | Required | Default | Description |
|--------|------|----------|---------|-------------|
| table_name | text | Yes | - | Primary key. Physical table name (plural, lowercase, snake_case) |
| singular | text | Yes | '' | Singular form (e.g., "contact" for "contacts") |
| plural | text | Auto | table_name | Plural form, auto-assigned to table_name |
| singular_label | text | Yes | '' | Display label singular (e.g., "Contact") |
| plural_label | text | Yes | '' | Display label plural (e.g., "Contacts") |
| icon_url | text | No | '' | URL or path to icon for this table |
| description | text | No | '' | Description of the table |
| module_id | integer | No | null | Foreign key to modules.id (SET NULL on delete) |
| view_permission | text | Yes | 'public:read' | Permission required to SELECT from this table |
| edit_permission | text | Yes | 'admin' | Permission required to INSERT/UPDATE/DELETE |
| id_column | text | Yes | 'id' | Primary key column name |
| label_column | text | Yes | 'label' | Column to use as display label |
| managed | boolean | Yes | true | When false, automatic DDL execution is disabled |
| searchable | boolean | Yes | false | Whether table is included in search |
| created_at | timestamp | Auto | CURRENT_TIMESTAMP | Creation timestamp |
| updated_at | timestamp | Auto | CURRENT_TIMESTAMP | Last update timestamp |

## Constraints

- `table_name` must match pattern `^[a-z_][a-z0-9_]*$`
- `id_column` must match pattern `^[a-z_][a-z0-9_]*$`
- `label_column` must match pattern `^[a-z_][a-z0-9_]*$`
- `plural` must equal `table_name` (auto-enforced)

## Naming Conventions

**table_name:**
- Always plural: `contacts`, `companies`, `orders`
- Always lowercase
- Use snake_case for multi-word: `order_items`, `user_roles`
- Must start with letter or underscore

**singular:**
- Singular form of table_name: `contact`, `company`, `order`

## Behavior

When a row is inserted into `tables`:
1. A physical PostgreSQL table is created
2. Core columns (`id`, `label`, `created_at`, `updated_at`) are automatically added
3. RLS policies are configured based on view_permission and edit_permission

When `managed` is false:
- Automatic DDL execution for table and field changes is disabled
- Use for tables managed externally or with custom DDL

When a row is deleted from `tables`:
1. The physical table is dropped
2. All related field definitions are cascade deleted

## Examples

**Correct (Create a contacts table):**
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

**Incorrect (Table name is singular):**
```javascript
postgrestRequest({
  method: "POST",
  path: "/tables",
  body: {
    table_name: "contact", // Should be plural: "contacts"
    singular: "contact",
    singular_label: "Contact"
  }
})
```

**Incorrect (Invalid characters in table_name):**
```javascript
postgrestRequest({
  method: "POST",
  path: "/tables",
  body: {
    table_name: "ContactList", // Must be lowercase: "contact_list"
    singular: "contact_list",
    singular_label: "Contact List"
  }
})
```

**Correct (Query all tables in a module):**
```javascript
postgrestRequest({
  method: "GET",
  path: "/tables?module_id=eq.1&select=table_name,singular_label,plural_label,description"
})
```

**Correct (Create unmanaged table for external DDL):**
```javascript
postgrestRequest({
  method: "POST",
  path: "/tables",
  body: {
    table_name: "external_data",
    singular: "external_datum",
    singular_label: "External Data",
    plural_label: "External Data",
    managed: false
  }
})
```
