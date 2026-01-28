---
impact: MEDIUM
title: Webhook Schema Reference
---

## Webhook Schema Reference

Tables for receiving and logging external webhook events.

## Tables Overview

```
webhook_receivers
  └── webhook_receiver_logs (one-to-many)
```

## webhook_receivers

Configuration for webhook endpoints.

| Column | Type | Required | Default | Description |
|--------|------|----------|---------|-------------|
| id | integer | Auto | - | Primary key |
| label | text | Yes | - | Display label (core field) |
| table_name | text | Yes | - | Target table for webhook data |
| description | text | Yes | '' | Description of webhook receiver purpose |
| auth_type | enum | Yes | 'none' | Authentication type: "none", "hmac", "header" |
| secret | text | Yes | '' | Secret for HMAC authentication |
| header_name | text | Yes | '' | Custom header name for header auth |
| header_value | text | Yes | '' | Expected value for header auth |
| jsonata | jsonata | Yes | '' | JSONata expression to transform incoming data |
| created_at | timestamp | Auto | - | Creation timestamp |
| updated_at | timestamp | Auto | - | Last update timestamp |

### Authentication Types

| Type | Description | Required Fields |
|------|-------------|-----------------|
| none | No authentication | - |
| hmac | HMAC signature verification | `secret` |
| header | Custom header validation | `header_name`, `header_value` |

### JSONata Transform

The `jsonata` field contains an optional JSONata expression to transform incoming webhook payloads before inserting into the target table. If empty, the payload is inserted as-is.

## webhook_receiver_logs

Log of webhook receiver events.

| Column | Type | Required | Default | Description |
|--------|------|----------|---------|-------------|
| id | integer | Auto | - | Primary key |
| label | text | Yes | - | Display label (same as webhook_id) |
| webhook_receiver_id | int32 | Yes | - | Reference to webhook receiver |
| webhook_timestamp | date-time | Yes | - | Timestamp from webhook source |
| received_timestamp | date-time | Yes | CURRENT_TIMESTAMP | When webhook was received |
| payload | json | Yes | - | Webhook payload data |
| result | enum | Yes | '10' | Processing result code |
| error_message | text | Yes | '' | Error message if processing failed |
| created_at | timestamp | Auto | - | Creation timestamp |
| updated_at | timestamp | Auto | - | Last update timestamp |

### Result Codes

| Code | Meaning | Description |
|------|---------|-------------|
| 10 | Received | Webhook received, not yet processed |
| 20 | Processed | Successfully processed and data inserted |
| 90 | Failed | Processing failed (see error_message) |

## Examples

**Correct (Create a webhook receiver):**
```javascript
postgrestRequest({
  method: "POST",
  path: "/webhook_receivers",
  body: {
    label: "Stripe Payments",
    table_name: "payments",
    description: "Receives payment events from Stripe",
    auth_type: "hmac",
    secret: "whsec_xxxxxxxxxxxxx",
    jsonata: `{
      "amount": payload.data.object.amount / 100,
      "currency": payload.data.object.currency,
      "status": payload.type = "payment_intent.succeeded" ? "completed" : "pending",
      "external_id": payload.data.object.id
    }`
  }
})
```

**Correct (Create webhook with header auth):**
```javascript
postgrestRequest({
  method: "POST",
  path: "/webhook_receivers",
  body: {
    label: "GitHub Events",
    table_name: "github_events",
    description: "Receives webhook events from GitHub",
    auth_type: "header",
    header_name: "X-Hub-Signature-256",
    header_value: "sha256=xxxxxxxx"
  }
})
```

**Correct (Query webhook logs with errors):**
```javascript
postgrestRequest({
  method: "GET",
  path: "/webhook_receiver_logs?result=eq.90&select=id,webhook_receiver_id,received_timestamp,error_message&order=received_timestamp.desc&limit=50"
})
```

**Correct (Query logs for specific receiver):**
```javascript
postgrestRequest({
  method: "GET",
  path: "/webhook_receiver_logs?webhook_receiver_id=eq.1&order=received_timestamp.desc&limit=20&select=*"
})
```

**Correct (Get receiver with recent logs):**
```javascript
postgrestRequest({
  method: "GET",
  path: "/webhook_receivers?id=eq.1&select=*,webhook_receiver_logs(id,result,received_timestamp,error_message)&webhook_receiver_logs.order=received_timestamp.desc&webhook_receiver_logs.limit=10"
})
```
