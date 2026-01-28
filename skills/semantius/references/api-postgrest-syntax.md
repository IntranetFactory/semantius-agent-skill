---
impact: HIGH
title: PostgREST Query Syntax Reference
---

## PostgREST Query Syntax Reference

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

**Correct (AND default):**
Multiple conditions are AND'd by default:
```http
/contacts?status=eq.active&type=eq.lead
```

**Correct (Explicit AND):**
```http
/contacts?and=(status.eq.active,type.eq.lead)
```

**Correct (OR usage):**
```http
/contacts?or=(status.eq.active,status.eq.pending)
```

**Correct (Complex combinations):**
```http
/contacts?and=(or=(status.eq.active,status.eq.pending),type.eq.lead)
```

**Incorrect (Missing operator):**
```http
/contacts?status=active,pending
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
