---
impact: HIGH
title: Fields Schema Reference
---

## Fields Schema Reference

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
