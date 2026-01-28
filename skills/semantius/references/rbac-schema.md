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
