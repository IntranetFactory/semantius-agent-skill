---
impact: HIGH
title: Tables Schema Reference
---

## Tables Schema Reference

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
    view_permission: "crm.read",
    edit_permission: "crm.write",
    id_column: "id",
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

**Correct (Query all tables in a module):**
```javascript
postgrestRequest({
  method: "GET",
  path: "/tables?module_id=eq.1&select=id,table_name,singular_label,plural_label,description"
})
```
