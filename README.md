# QuillCRM Make API Docs

## Connection

### Base URL

`{{connection.baseUrl}}`

### Authentication

All requests must include `api_key` as a query string parameter:

```
?api_key={{connection.apiKey}}
```

### Logging

Sensitive fields (`api_key`) are sanitized from logs.

---

## Auth Test

Verify that the API key is valid.

**Endpoint**

```
GET {{connection.baseUrl}}/wp-json/qc/v1/integrations/make/auth/test?api_key={{connection.apiKey}}
```

**Success Response**

```json
{
  "success": true,
  "message": "Authentication successful!",
  "data": {
    "site_name": "My WordPress Site",
    "site_url": "https://example.com",
    "version": "1.0.0"
  }
}
```

---

## Webhooks

QuillCRM supports attaching and detaching webhooks for contact and deal events.

### Attach Webhook

**Endpoint**

```
POST {{connection.baseUrl}}/wp-json/qc/v1/integrations/make/webhooks/subscribe?api_key={{connection.apiKey}}
```

**Request Body**

```json
{
  "hook_url": "{{webhook.url}}",
  "trigger_type": "<trigger_type>"
}
```

**Optional filters** (include in body depending on trigger type):

| Parameter | Type | Used with |
|---|---|---|
| `tag_ids` | string (comma-separated) or array | `contact_tags_applied`, `contact_tags_removed` |
| `list_ids` | string (comma-separated) or array | `contact_lists_applied`, `contact_lists_removed` |
| `pipeline_id` | integer | deal triggers |
| `stage_id` | integer | deal triggers |
| `owner_id` | integer | deal triggers |

**Example (Contact Lists Applied)**

```json
{
  "hook_url": "{{webhook.url}}",
  "trigger_type": "contact_lists_applied",
  "list_ids": "{{parameters.list_id}}"
}
```

**Success Response**

```json
{
  "success": true,
  "message": "Webhook subscribed successfully!",
  "data": {
    "externalHookId": "uuid-string",
    "token": "64-char-hex-token"
  }
}
```

- `externalHookId` is stored by Make and used for detach requests (`{{webhook.externalHookId}}`).
- `token` is stored by Make and accessible via `{{webhook.token}}`.

### Detach Webhook

**Endpoint**

```
DELETE {{connection.baseUrl}}/wp-json/qc/v1/integrations/make/webhooks/unsubscribe
```

**Query Parameters**

- `api_key`: `{{connection.apiKey}}`
- `webhook_id`: `{{webhook.externalHookId}}`

**Example**

```
DELETE {{connection.baseUrl}}/wp-json/qc/v1/integrations/make/webhooks/unsubscribe?api_key=XXXX&webhook_id=uuid-string
```

---

## Supported Hooks

### Contact Triggers

| # | Trigger Type | Description |
|---|---|---|
| 1 | `contact_subscribed` | New contact subscribed |
| 2 | `contact_unsubscribed` | Contact unsubscribed |
| 3 | `contact_updated` | Contact updated |
| 4 | `contact_tags_applied` | Tags applied to contact |
| 5 | `contact_tags_removed` | Tags removed from contact |
| 6 | `contact_lists_applied` | Contact added to lists |
| 7 | `contact_lists_removed` | Contact removed from lists |

### Deal Triggers

| # | Trigger Type | Description |
|---|---|---|
| 1 | `deal_created` | New deal created |
| 2 | `deal_owner_changed` | Deal owner changed |
| 3 | `deal_stage_changed` | Deal stage changed |
| 4 | `deal_value_changed` | Deal value changed |

---

## Make App Configuration

### Attach (Remote Procedure)

```json
{
  "url": "{{connection.baseUrl}}/wp-json/qc/v1/integrations/make/webhooks/subscribe",
  "method": "POST",
  "qs": {
    "api_key": "{{connection.apiKey}}"
  },
  "body": {
    "hook_url": "{{webhook.url}}",
    "trigger_type": "contact_lists_applied",
    "list_ids": "{{parameters.list_id}}"
  },
  "response": {
    "data": {
      "externalHookId": "{{body.data.externalHookId}}",
      "token": "{{body.data.token}}"
    }
  }
}
```

### Detach (Remote Procedure)

```json
{
  "url": "{{connection.baseUrl}}/wp-json/qc/v1/integrations/make/webhooks/unsubscribe",
  "method": "DELETE",
  "qs": {
    "api_key": "{{connection.apiKey}}",
    "webhook_id": "{{webhook.externalHookId}}"
  }
}
```

---

## Reference Endpoints

These endpoints return data used to populate dropdowns in Make scenarios.

### Get Lists

```
POST {{connection.baseUrl}}/wp-json/qc/v1/integrations/make/lists?api_key={{connection.apiKey}}
```

**Response**

```json
{
  "success": true,
  "data": [
    { "id": 1, "name": "Newsletter" }
  ]
}
```

### Get Tags

```
POST {{connection.baseUrl}}/wp-json/qc/v1/integrations/make/tags?api_key={{connection.apiKey}}
```

**Response**

```json
{
  "success": true,
  "data": [
    { "id": 1, "name": "VIP" }
  ]
}
```

### Get Pipelines

```
POST {{connection.baseUrl}}/wp-json/qc/v1/integrations/make/pipelines?api_key={{connection.apiKey}}
```

**Response**

```json
{
  "success": true,
  "data": [
    { "id": 1, "name": "Sales Pipeline" }
  ]
}
```

### Get Stages

```
POST {{connection.baseUrl}}/wp-json/qc/v1/integrations/make/stages?api_key={{connection.apiKey}}
```

**Request Body**

```json
{
  "pipeline_id": 1
}
```

**Response**

```json
{
  "success": true,
  "data": [
    { "id": 1, "name": "Qualification", "pipeline_id": 1, "win_probability": 10 },
    { "id": 2, "name": "Proposal", "pipeline_id": 1, "win_probability": 50 }
  ]
}
```

### Get Owners

```
POST {{connection.baseUrl}}/wp-json/qc/v1/integrations/make/owners?api_key={{connection.apiKey}}
```

**Response**

```json
{
  "success": true,
  "data": [
    { "id": 1, "display_name": "John Doe", "email": "john@example.com" }
  ]
}
```

### Polling

```
GET {{connection.baseUrl}}/wp-json/qc/v1/integrations/make/polling/{trigger_type}?api_key={{connection.apiKey}}
```

---

## Contact Actions

### Create Contact

```
POST {{connection.baseUrl}}/wp-json/qc/v1/integrations/make/contacts/create?api_key={{connection.apiKey}}
```

**Request Body**

```json
{
  "email": "jane@example.com",
  "first_name": "Jane",
  "last_name": "Doe",
  "phone": "+1234567890",
  "whatsapp_phone": "+1234567890",
  "address_1": "123 Main St",
  "address_2": "Suite 100",
  "city": "New York",
  "state": "NY",
  "country": "US",
  "zip": "10001",
  "email_status": "subscribed",
  "sms_status": "subscribed",
  "whatsapp_status": "subscribed",
  "source": "make",
  "tag_ids": [1, 2],
  "list_ids": [1]
}
```

### Add Tags to Contact

```
POST {{connection.baseUrl}}/wp-json/qc/v1/integrations/make/contacts/add-tags?api_key={{connection.apiKey}}
```

```json
{
  "email": "jane@example.com",
  "tag_ids": [1, 2]
}
```

### Remove Tags from Contact

```
POST {{connection.baseUrl}}/wp-json/qc/v1/integrations/make/contacts/remove-tags?api_key={{connection.apiKey}}
```

```json
{
  "email": "jane@example.com",
  "tag_ids": [1, 2]
}
```

### Add Contact to Lists

```
POST {{connection.baseUrl}}/wp-json/qc/v1/integrations/make/contacts/add-to-lists?api_key={{connection.apiKey}}
```

```json
{
  "email": "jane@example.com",
  "list_ids": [1, 3]
}
```

### Remove Contact from Lists

```
POST {{connection.baseUrl}}/wp-json/qc/v1/integrations/make/contacts/remove-from-lists?api_key={{connection.apiKey}}
```

```json
{
  "email": "jane@example.com",
  "list_ids": [1, 3]
}
```

### Change Contact Status

```
POST {{connection.baseUrl}}/wp-json/qc/v1/integrations/make/contacts/change-status?api_key={{connection.apiKey}}
```

```json
{
  "email": "jane@example.com",
  "status_type": "email_status",
  "status": "subscribed"
}
```

**`status_type`** options: `email_status`, `sms_status`, `whatsapp_status`

**`status`** options: `subscribed`, `unsubscribed`, `blocked`

---

## Deal Actions

### Create Deal

```
POST {{connection.baseUrl}}/wp-json/qc/v1/integrations/make/deals/create?api_key={{connection.apiKey}}
```

```json
{
  "title": "New Deal",
  "email": "jane@example.com",
  "pipeline_id": 1,
  "stage_id": 1,
  "value": 5000,
  "owner_id": 1,
  "priority": "high",
  "expected_close_date": "2026-03-15",
  "source": "make"
}
```

### Update Deal Title

```
POST {{connection.baseUrl}}/wp-json/qc/v1/integrations/make/deals/update-title?api_key={{connection.apiKey}}
```

```json
{
  "deal_id": 456,
  "title": "Updated Deal Title"
}
```

### Update Deal Value

```
POST {{connection.baseUrl}}/wp-json/qc/v1/integrations/make/deals/update-value?api_key={{connection.apiKey}}
```

```json
{
  "deal_id": 456,
  "value": 7500
}
```

### Update Deal Owner

```
POST {{connection.baseUrl}}/wp-json/qc/v1/integrations/make/deals/update-owner?api_key={{connection.apiKey}}
```

```json
{
  "deal_id": 456,
  "owner_id": 2
}
```

### Update Deal Stage

```
POST {{connection.baseUrl}}/wp-json/qc/v1/integrations/make/deals/update-stage?api_key={{connection.apiKey}}
```

```json
{
  "deal_id": 456,
  "stage_id": 3,
  "pipeline_id": 1
}
```

`pipeline_id` is optional — if omitted, the deal's current pipeline is used.

---

## Webhook Payload Format

When a trigger fires, QuillCRM sends a POST request to the registered `hook_url`.

### Headers

| Header | Value |
|---|---|
| `Content-Type` | `application/json` |
| `User-Agent` | `QuillCRM-Make/1.0` |
| `X-QuillCRM-Trigger` | e.g. `contact.contact_subscribed`, `deal.deal_created` |

### Contact Payload

```json
{
  "id": 123,
  "hash_id": "abc123...",
  "email": "jane@example.com",
  "first_name": "Jane",
  "last_name": "Doe",
  "phone": "+1234567890",
  "whatsapp_phone": "+1234567890",
  "address_1": "123 Main St",
  "address_2": "Suite 100",
  "city": "New York",
  "state": "NY",
  "country": "US",
  "zip": "10001",
  "source": "form",
  "email_status": "subscribed",
  "sms_status": "subscribed",
  "whatsapp_status": "subscribed",
  "tags": [{ "id": 1, "name": "VIP" }],
  "lists": [{ "id": 1, "name": "Newsletter" }],
  "trigger_type": "contact_subscribed",
  "created_at": "2026-01-15 10:30:00",
  "updated_at": "2026-02-20 14:00:00",
  "contact_url": "https://example.com/wp-admin/admin.php?page=quillcrm&path=contacts/123",
  "_make": {
    "webhook_id": "uuid-string",
    "name": "Unknown",
    "sent_at": "2026-02-24T12:00:00+00:00"
  }
}
```

**Additional fields for tag triggers** (`contact_tags_applied`, `contact_tags_removed`):

- `tags_applied`: array of `{ "id", "name" }`
- `tags_removed`: array of `{ "id", "name" }`

**Additional fields for list triggers** (`contact_lists_applied`, `contact_lists_removed`):

- `lists_applied`: array of `{ "id", "name" }`
- `lists_removed`: array of `{ "id", "name" }`

### Deal Payload

```json
{
  "id": 456,
  "title": "New Deal",
  "value": 5000,
  "currency": "USD",
  "status": "open",
  "priority": "high",
  "probability": 75,
  "expected_close_date": "2026-04-01",
  "source": "website",
  "lost_reason": "",
  "contact_id": 123,
  "contact_email": "jane@example.com",
  "contact_name": "Jane Doe",
  "pipeline_id": 1,
  "pipeline_name": "Sales Pipeline",
  "stage_id": 1,
  "stage_name": "Qualification",
  "owner_id": 1,
  "owner_name": "John Doe",
  "trigger_type": "deal_created",
  "created_at": "2026-01-15 10:30:00",
  "updated_at": "2026-02-20 14:00:00",
  "deal_url": "https://example.com/wp-admin/admin.php?page=quillcrm&path=deals/456",
  "_make": {
    "webhook_id": "uuid-string",
    "name": "Unknown",
    "sent_at": "2026-02-24T12:00:00+00:00"
  }
}
```

**Additional fields for change triggers:**

- `deal_stage_changed`: `old_stage_id`, `old_stage_name`, `new_stage_id`, `new_stage_name`
- `deal_value_changed`: `old_value`, `new_value`
- `deal_owner_changed`: `old_owner_id`, `old_owner_name`, `new_owner_id`, `new_owner_name`

---

## Error Responses

### WordPress REST API Errors

```json
{
  "code": "invalid_api_key",
  "message": "Invalid API key.",
  "data": { "status": 401 }
}
```

### Action Handler Errors

```json
{
  "success": false,
  "code": "error_code",
  "message": "Human-readable error message."
}
```

Common HTTP status codes: `400` (bad request), `401` (unauthorized), `404` (not found), `500` (server error).
