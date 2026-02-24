# QuillCRM Pro Make API Docs

## Connection

### Base URL

- `{{connection.baseUrl}}`

### Authentication

All requests must include authentication via one of these methods:

1. **Body parameter:** `api_key` in the JSON request body
2. **Header:** `x_api_key`
3. **Header:** `Authorization: Bearer {{connection.apiKey}}`

### Logging

Sensitive fields (`api_key`) are sanitized from logs.

---

## Auth Test

Verify that the API key is valid.

**Endpoint**

```
POST {{connection.baseUrl}}/wp-json/qc/v1/integrations/make/auth/test
```

**Request Body**

```json
{
  "api_key": "{{connection.apiKey}}"
}
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

QuillCRM Pro supports attaching and detaching webhooks for contact and deal events.

### Supported Trigger Types

**Contact Triggers**

| Trigger Type | Description |
|---|---|
| `contact_subscribed` | New contact subscribed |
| `contact_unsubscribed` | Contact unsubscribed |
| `contact_updated` | Contact updated |
| `contact_tags_applied` | Tags applied to contact |
| `contact_tags_removed` | Tags removed from contact |
| `contact_lists_applied` | Contact added to lists |
| `contact_lists_removed` | Contact removed from lists |

**Deal Triggers**

| Trigger Type | Description |
|---|---|
| `deal_created` | New deal created |
| `deal_owner_changed` | Deal owner changed |
| `deal_stage_changed` | Deal stage changed |
| `deal_value_changed` | Deal value changed |

---

### Attach Webhook

**Endpoint**

```
POST {{connection.baseUrl}}/wp-json/qc/v1/integrations/make/webhooks/subscribe
```

**Request Body**

```json
{
  "api_key": "{{connection.apiKey}}",
  "hook_url": "{{webhook.url}}",
  "scenario_id": "{{parameters.scenario_id}}",
  "trigger_type": "<trigger_type>"
}
```

**Optional Filters**

Depending on the trigger type, you can include additional filters:

- `tag_ids` (array) — for `contact_tags_applied` / `contact_tags_removed`
- `list_ids` (array) — for `contact_lists_applied` / `contact_lists_removed`
- `pipeline_id` (integer) — for deal triggers
- `stage_id` (integer) — for deal triggers
- `owner_id` (integer) — for deal triggers

**Example (Contact Tags Applied)**

```json
{
  "api_key": "{{connection.apiKey}}",
  "hook_url": "{{webhook.url}}",
  "scenario_id": "{{parameters.scenario_id}}",
  "trigger_type": "contact_tags_applied",
  "tag_ids": [1, 2, 3]
}
```

**Example (Deal Stage Changed)**

```json
{
  "api_key": "{{connection.apiKey}}",
  "hook_url": "{{webhook.url}}",
  "scenario_id": "{{parameters.scenario_id}}",
  "trigger_type": "deal_stage_changed",
  "pipeline_id": 1,
  "stage_id": 2
}
```

**Success Response**

```json
{
  "success": true,
  "message": "Webhook subscribed successfully!",
  "data": {
    "webhook_id": "uuid-string"
  }
}
```

- `webhook_id` is stored and used for detach requests.

---

### Detach Webhook

**Endpoint**

```
POST {{connection.baseUrl}}/wp-json/qc/v1/integrations/make/webhooks/unsubscribe
```

**Request Body**

```json
{
  "api_key": "{{connection.apiKey}}",
  "webhook_id": "{{webhook.id}}"
}
```

---

### List Webhooks

**Endpoint**

```
POST {{connection.baseUrl}}/wp-json/qc/v1/integrations/make/webhooks
```

**Request Body**

```json
{
  "api_key": "{{connection.apiKey}}"
}
```

---

## Reference Endpoints

These endpoints return data used to populate dropdowns and filters in Make scenarios.

### Get Pipelines

```
POST {{connection.baseUrl}}/wp-json/qc/v1/integrations/make/pipelines
```

```json
{ "api_key": "{{connection.apiKey}}" }
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
POST {{connection.baseUrl}}/wp-json/qc/v1/integrations/make/stages
```

```json
{
  "api_key": "{{connection.apiKey}}",
  "pipeline_id": 1
}
```

**Response**

```json
{
  "success": true,
  "data": [
    { "id": 1, "name": "Qualification" },
    { "id": 2, "name": "Proposal" }
  ]
}
```

### Get Owners

```
POST {{connection.baseUrl}}/wp-json/qc/v1/integrations/make/owners
```

```json
{ "api_key": "{{connection.apiKey}}" }
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

### Get Lists

```
POST {{connection.baseUrl}}/wp-json/qc/v1/integrations/make/lists
```

```json
{ "api_key": "{{connection.apiKey}}" }
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
POST {{connection.baseUrl}}/wp-json/qc/v1/integrations/make/tags
```

```json
{ "api_key": "{{connection.apiKey}}" }
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

---

## Contact Actions

### Create Contact

**Endpoint**

```
POST {{connection.baseUrl}}/wp-json/qc/v1/integrations/make/contacts/create
```

**Request Body**

```json
{
  "api_key": "{{connection.apiKey}}",
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

**Success Response**

```json
{
  "id": 123,
  "email": "jane@example.com",
  "first_name": "Jane",
  "last_name": "Doe",
  "phone": "+1234567890",
  "whatsapp_phone": "+1234567890",
  "tags": [
    { "id": 1, "name": "VIP" }
  ],
  "lists": [
    { "id": 1, "name": "Newsletter" }
  ],
  "contact_url": "https://example.com/wp-admin/admin.php?page=quillcrm&path=contacts/123"
}
```

---

### Add Tags to Contact

**Endpoint**

```
POST {{connection.baseUrl}}/wp-json/qc/v1/integrations/make/contacts/add-tags
```

**Request Body**

```json
{
  "api_key": "{{connection.apiKey}}",
  "email": "jane@example.com",
  "tag_ids": [1, 2]
}
```

---

### Remove Tags from Contact

**Endpoint**

```
POST {{connection.baseUrl}}/wp-json/qc/v1/integrations/make/contacts/remove-tags
```

**Request Body**

```json
{
  "api_key": "{{connection.apiKey}}",
  "email": "jane@example.com",
  "tag_ids": [1, 2]
}
```

---

### Add Contact to Lists

**Endpoint**

```
POST {{connection.baseUrl}}/wp-json/qc/v1/integrations/make/contacts/add-to-lists
```

**Request Body**

```json
{
  "api_key": "{{connection.apiKey}}",
  "email": "jane@example.com",
  "list_ids": [1, 3]
}
```

---

### Remove Contact from Lists

**Endpoint**

```
POST {{connection.baseUrl}}/wp-json/qc/v1/integrations/make/contacts/remove-from-lists
```

**Request Body**

```json
{
  "api_key": "{{connection.apiKey}}",
  "email": "jane@example.com",
  "list_ids": [1, 3]
}
```

---

### Change Contact Status

**Endpoint**

```
POST {{connection.baseUrl}}/wp-json/qc/v1/integrations/make/contacts/change-status
```

**Request Body**

```json
{
  "api_key": "{{connection.apiKey}}",
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

**Endpoint**

```
POST {{connection.baseUrl}}/wp-json/qc/v1/integrations/make/deals/create
```

**Request Body**

```json
{
  "api_key": "{{connection.apiKey}}",
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

**Success Response**

```json
{
  "id": 456,
  "title": "New Deal",
  "value": 5000,
  "contact_email": "jane@example.com",
  "pipeline_name": "Sales Pipeline",
  "stage_name": "Qualification",
  "owner_name": "John Doe",
  "deal_url": "https://example.com/wp-admin/admin.php?page=quillcrm&path=deals/456"
}
```

---

### Update Deal Title

**Endpoint**

```
POST {{connection.baseUrl}}/wp-json/qc/v1/integrations/make/deals/update-title
```

**Request Body**

```json
{
  "api_key": "{{connection.apiKey}}",
  "deal_id": 456,
  "title": "Updated Deal Title"
}
```

---

### Update Deal Value

**Endpoint**

```
POST {{connection.baseUrl}}/wp-json/qc/v1/integrations/make/deals/update-value
```

**Request Body**

```json
{
  "api_key": "{{connection.apiKey}}",
  "deal_id": 456,
  "value": 7500
}
```

---

### Update Deal Owner

**Endpoint**

```
POST {{connection.baseUrl}}/wp-json/qc/v1/integrations/make/deals/update-owner
```

**Request Body**

```json
{
  "api_key": "{{connection.apiKey}}",
  "deal_id": 456,
  "owner_id": 2
}
```

---

### Update Deal Stage

**Endpoint**

```
POST {{connection.baseUrl}}/wp-json/qc/v1/integrations/make/deals/update-stage
```

**Request Body**

```json
{
  "api_key": "{{connection.apiKey}}",
  "deal_id": 456,
  "stage_id": 3,
  "pipeline_id": 1
}
```

`pipeline_id` is optional — if omitted, the deal's current pipeline is used.

---

## Webhook Payload Format

When a trigger fires, QuillCRM sends a POST request to the registered `hook_url` with the following structure.

### Headers

| Header | Value |
|---|---|
| `Content-Type` | `application/json` |
| `User-Agent` | `QuillCRM-Make/1.0` |
| `X-QuillCRM-Trigger` | e.g. `contact.subscribed`, `deal.created` |

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
  "city": "New York",
  "country": "US",
  "source": "form",
  "email_status": "subscribed",
  "sms_status": "subscribed",
  "whatsapp_status": "subscribed",
  "tags": [
    { "id": 1, "name": "VIP" }
  ],
  "lists": [
    { "id": 1, "name": "Newsletter" }
  ],
  "trigger_type": "contact_subscribed",
  "created_at": "2026-01-15 10:30:00",
  "updated_at": "2026-02-20 14:00:00",
  "contact_url": "https://example.com/wp-admin/admin.php?page=quillcrm&path=contacts/123",
  "_make": {
    "webhook_id": "uuid-string",
    "name": "My Webhook",
    "sent_at": "2026-02-24 12:00:00",
    "scenario_id": "scenario-123"
  }
}
```

### Deal Payload

```json
{
  "id": 456,
  "title": "New Deal",
  "value": 5000,
  "contact_email": "jane@example.com",
  "pipeline_name": "Sales Pipeline",
  "stage_name": "Qualification",
  "owner_name": "John Doe",
  "trigger_type": "deal_created",
  "deal_url": "https://example.com/wp-admin/admin.php?page=quillcrm&path=deals/456",
  "_make": {
    "webhook_id": "uuid-string",
    "name": "My Webhook",
    "sent_at": "2026-02-24 12:00:00",
    "scenario_id": "scenario-123"
  }
}
```

For change triggers (`deal_stage_changed`, `deal_value_changed`, `deal_owner_changed`), additional fields are included:

- `old_stage_id` / `new_stage_id`
- `old_value` / `new_value`
- `old_owner_id` / `new_owner_id`

---

## Error Responses

All endpoints return errors in this format:

```json
{
  "success": false,
  "code": "error_code",
  "message": "Human-readable error message."
}
```

Common HTTP status codes: `400` (bad request), `401` (unauthorized), `404` (not found), `500` (server error).
