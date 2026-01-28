---
impact: HIGH
title: Modules Schema Reference
---

## Modules Schema Reference

The `modules` table defines logical modules that group related tables, roles, and permissions.

## Columns

| Column | Type | Required | Default | Description |
|--------|------|----------|---------|-------------|
| id | serial | Auto | - | Primary key |
| module_name | text | Yes | '' | Unique module identifier (e.g., "crm", "inventory") |
| description | text | No | '' | Module description |
| view_permission | text | Yes | 'user:read' | Permission required to view this module |
| logo_url | text | No | '' | URL or path to module logo/icon |
| logo_color | text | No | '' | Color for module branding (hex or name) |
| home_page | text | Yes | '/' | Default landing page for this module |
| alias | text | Yes | '' | URL alias for the module |
| settings | jsonb | No | null | Module-specific settings |
| created_at | timestamptz | Auto | CURRENT_TIMESTAMP | Creation timestamp |
| updated_at | timestamptz | Auto | CURRENT_TIMESTAMP | Last update timestamp |

## Purpose

Modules serve as logical containers that:
- Group related tables together
- Define module-level permissions
- Provide branding (logo, color) for UI
- Configure module home pages and navigation

## Relationships

- `tables.module_id` references `modules.id` (SET NULL on delete)
- `permissions.module_id` references `modules.id`

## Core Module

Module ID `1` is reserved for `_core` - the Semantius system module containing:
- `tables` and `fields` (schema metadata)
- `users`, `roles`, `permissions` (RBAC)
- `webhook_receivers`, `webhook_receiver_logs`

## Examples

**Correct (Create a CRM module):**
```javascript
postgrestRequest({
  method: "POST",
  path: "/modules",
  body: {
    module_name: "crm",
    description: "Customer Relationship Management",
    view_permission: "crm:read",
    logo_url: "/icons/crm.svg",
    logo_color: "#4CAF50",
    home_page: "/crm/contacts",
    alias: "crm"
  }
})
```

**Correct (Create module with custom settings):**
```javascript
postgrestRequest({
  method: "POST",
  path: "/modules",
  body: {
    module_name: "inventory",
    description: "Inventory Management",
    view_permission: "inventory:read",
    home_page: "/inventory/products",
    alias: "inv",
    settings: {
      default_currency: "USD",
      low_stock_threshold: 10,
      enable_barcode_scanning: true
    }
  }
})
```

**Correct (Query all modules with their tables):**
```javascript
postgrestRequest({
  method: "GET",
  path: "/modules?select=id,module_name,description,tables(table_name,singular_label)"
})
```

**Correct (Get module by name):**
```javascript
postgrestRequest({
  method: "GET",
  path: "/modules?module_name=eq.crm&select=*"
})
```

**Correct (Update module settings):**
```javascript
postgrestRequest({
  method: "PATCH",
  path: "/modules?id=eq.2",
  body: {
    settings: {
      default_currency: "EUR",
      low_stock_threshold: 5
    }
  }
})
```
