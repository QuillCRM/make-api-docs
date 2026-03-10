# QuillCRM Make API Docs

## Connection

### Website URL

```
{{connection.websiteUrl}}/
```

The website URL is the WordPress site root (e.g. `https://example.com/`). All requests are sent as `POST` to this single URL.

### Authentication

All requests include `api_key` in the JSON request body (not as a query parameter):

```json
{
  "quillcrm_make_action": "<action>",
  "api_key": "{{connection.apiKey}}"
}
```

The API key is prefixed with `qc_` followed by 64 hex characters.

### Routing

Instead of separate REST API endpoints, all actions are routed through a single POST endpoint using the `quillcrm_make_action` field in the JSON body. The plugin reads `php://input` and dispatches to the appropriate handler websited on the action value.

### Response Format

All responses use WordPress `wp_send_json_success` / `wp_send_json_error` format:

**Success:**

```json
{
  "success": true,
  "data": { ... }
}
```

**Error:**

```json
{
  "success": false,
  "data": {
    "code": "optional_machine_code",
    "message": "Human-readable message"
  }
}
```

---

## Auth

### Verify API Key

Validates whether a provided API key is correct. This action does **not** require prior authentication (the key itself is being verified).

**Action:** `verify_api_key`

**Request Body**

```json
{
  "quillcrm_make_action": "verify_api_key",
  "api_key": "{{connection.apiKey}}"
}
```

**Success Response**

```json
{
  "success": true,
  "data": {
    "message": "API key is valid.",
    "site_name": "My WordPress Site",
    "site_url": "https://example.com",
    "version": "1.0.0"
  }
}
```

**Error Response (401)**

```json
{
  "success": false,
  "data": {
    "message": "Invalid API key."
  }
}
```

### Test Authentication

Verify that a previously authenticated connection is working. Requires a valid API key.

**Action:** `test_authentication`

**Request Body**

```json
{
  "quillcrm_make_action": "test_authentication",
  "api_key": "{{connection.apiKey}}"
}
```

**Success Response**

```json
{
  "success": true,
  "data": {
    "message": "Authentication successful!",
    "site_name": "My WordPress Site",
    "site_url": "https://example.com",
    "version": "1.0.0"
  }
}
```

---

## Webhooks

QuillCRM supports attaching and detaching webhooks for contact and deal events.

### Subscribe (Attach Webhook)

**Action:** `subscribe`

**Request Body**

```json
{
  "quillcrm_make_action": "subscribe",
  "api_key": "{{connection.apiKey}}",
  "hook_url": "{{webhook.url}}",
  "trigger_type": "<trigger_type>"
}
```

**Optional filters** (include in body depending on trigger type):

| Parameter | Type | Used with |
|---|---|---|
| `tag_ids` | array of integers | `contact_tags_applied`, `contact_tags_removed` |
| `list_ids` | array of integers | `contact_lists_applied`, `contact_lists_removed` |
| `pipeline_id` | integer | deal triggers |
| `stage_id` | integer | deal triggers |
| `owner_id` | integer | deal triggers |

**Example (Contact Lists Applied)**

```json
{
  "quillcrm_make_action": "subscribe",
  "api_key": "{{connection.apiKey}}",
  "hook_url": "{{webhook.url}}",
  "trigger_type": "contact_lists_applied",
  "list_ids": [1, 3]
}
```

**Success Response**

```json
{
  "success": true,
  "data": {
    "externalHookId": "uuid-string",
    "token": "64-char-hex-token",
    "message": "Webhook subscribed successfully."
  }
}
```

- `externalHookId` is stored by Make and used for unsubscribe requests (`{{webhook.externalHookId}}`).
- `token` is stored by Make and accessible via `{{webhook.token}}`.

### Unsubscribe (Detach Webhook)

**Action:** `unsubscribe`

**Request Body**

```json
{
  "quillcrm_make_action": "unsubscribe",
  "api_key": "{{connection.apiKey}}",
  "webhook_id": "{{webhook.externalHookId}}"
}
```

**Success Response**

```json
{
  "success": true,
  "data": {
    "message": "Webhook unsubscribed successfully."
  }
}
```

### List Webhooks

**Action:** `get_webhooks`

**Request Body**

```json
{
  "quillcrm_make_action": "get_webhooks",
  "api_key": "{{connection.apiKey}}"
}
```

**Success Response**

```json
{
  "success": true,
  "data": {
    "message": "Webhooks retrieved successfully.",
    "data": [ ... ]
  }
}
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
  "url": "{{connection.websiteUrl}}/",
  "method": "POST",
  "headers": {
    "Content-Type": "application/json"
  },
  "body": {
    "quillcrm_make_action": "subscribe",
    "api_key": "{{connection.apiKey}}",
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
  "url": "{{connection.websiteUrl}}/",
  "method": "POST",
  "headers": {
    "Content-Type": "application/json"
  },
  "body": {
    "quillcrm_make_action": "unsubscribe",
    "api_key": "{{connection.apiKey}}",
    "webhook_id": "{{webhook.externalHookId}}"
  }
}
```

---

## Reference Endpoints

These endpoints return data used to populate dropdowns in Make scenarios. All use `POST` to `{{connection.websiteUrl}}/` with `quillcrm_make_action` and `api_key` in the JSON body.

### Get Lists

**Action:** `get_lists`

```json
{
  "quillcrm_make_action": "get_lists",
  "api_key": "{{connection.apiKey}}"
}
```

**Response**

```json
{
  "success": true,
  "data": {
    "message": "Lists retrieved successfully.",
    "lists": [
      { "id": 1, "name": "Newsletter" }
    ]
  }
}
```

### Get Tags

**Action:** `get_tags`

```json
{
  "quillcrm_make_action": "get_tags",
  "api_key": "{{connection.apiKey}}"
}
```

**Response**

```json
{
  "success": true,
  "data": {
    "message": "Tags retrieved successfully.",
    "tags": [
      { "id": 1, "name": "VIP" }
    ]
  }
}
```

### Get Pipelines

**Action:** `get_pipelines`

```json
{
  "quillcrm_make_action": "get_pipelines",
  "api_key": "{{connection.apiKey}}"
}
```

**Response**

```json
{
  "success": true,
  "data": {
    "message": "Pipelines retrieved successfully.",
    "pipelines": [
      { "id": 1, "name": "Sales Pipeline" }
    ]
  }
}
```

### Get Stages

**Action:** `get_stages`

```json
{
  "quillcrm_make_action": "get_stages",
  "api_key": "{{connection.apiKey}}",
  "pipeline_id": 1
}
```

**Response**

```json
{
  "success": true,
  "data": {
    "message": "Stages retrieved successfully.",
    "stages": [
      { "id": 1, "name": "Qualification", "pipeline_id": 1, "win_probability": 10 },
      { "id": 2, "name": "Proposal", "pipeline_id": 1, "win_probability": 50 }
    ]
  }
}
```

### Get Owners

**Action:** `get_owners`

```json
{
  "quillcrm_make_action": "get_owners",
  "api_key": "{{connection.apiKey}}"
}
```

**Response**

```json
{
  "success": true,
  "data": {
    "message": "Owners retrieved successfully.",
    "owners": [
      { "id": 1, "display_name": "John Doe", "email": "john@example.com" }
    ]
  }
}
```

### Polling

**Action:** `polling`

```json
{
  "quillcrm_make_action": "polling",
  "api_key": "{{connection.apiKey}}",
  "trigger_type": "<trigger_type>"
}
```

**Response**

```json
{
  "success": true,
  "data": {
    "message": "Polling data retrieved successfully.",
    "data": [ ... ]
  }
}
```

Returns sample/fallback data for the specified trigger type if no real data is available.

---

## Contact Actions

All contact actions use `POST` to `{{connection.websiteUrl}}/` with the JSON body containing `quillcrm_make_action` and `api_key`. On success, the response includes the full updated contact object.

### Create Contact

**Action:** `create_contact`

```json
{
  "quillcrm_make_action": "create_contact",
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

If a contact with the given email already exists, it is updated (upsert behavior via `createOrUpdate`). If `source` is not provided, defaults to `make`.

**Success Response**

```json
{
  "success": true,
  "data": {
    "id": 123,
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
    "source": "make",
    "email_status": "subscribed",
    "sms_status": "subscribed",
    "whatsapp_status": "subscribed",
    "tags": [{ "id": 1, "name": "VIP" }],
    "lists": [{ "id": 1, "name": "Newsletter" }],
    "created_at": "2026-01-15 10:30:00",
    "updated_at": "2026-02-20 14:00:00",
    "contact_url": "https://example.com/wp-admin/admin.php?page=quillcrm&path=contacts/123",
    "message": "Contact created successfully."
  }
}
```

### Add Tags to Contact

**Action:** `add_tags`

```json
{
  "quillcrm_make_action": "add_tags",
  "api_key": "{{connection.apiKey}}",
  "email": "jane@example.com",
  "tag_ids": [1, 2]
}
```

### Remove Tags from Contact

**Action:** `remove_tags`

```json
{
  "quillcrm_make_action": "remove_tags",
  "api_key": "{{connection.apiKey}}",
  "email": "jane@example.com",
  "tag_ids": [1, 2]
}
```

### Add Contact to Lists

**Action:** `add_to_lists`

```json
{
  "quillcrm_make_action": "add_to_lists",
  "api_key": "{{connection.apiKey}}",
  "email": "jane@example.com",
  "list_ids": [1, 3]
}
```

### Remove Contact from Lists

**Action:** `remove_from_lists`

```json
{
  "quillcrm_make_action": "remove_from_lists",
  "api_key": "{{connection.apiKey}}",
  "email": "jane@example.com",
  "list_ids": [1, 3]
}
```

### Change Contact Status

**Action:** `change_status`

```json
{
  "quillcrm_make_action": "change_status",
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

All deal actions use `POST` to `{{connection.websiteUrl}}/` with the JSON body containing `quillcrm_make_action` and `api_key`. On success, the response includes the full updated deal object.

### Create Deal

**Action:** `add_deal`

```json
{
  "quillcrm_make_action": "add_deal",
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

`pipeline_id` is required. If `stage_id` is omitted, the first stage of the pipeline is used. If `source` is not provided, defaults to `make`. The deal's `status` and `probability` are automatically derived from the stage's `win_probability`.

**Success Response**

```json
{
  "success": true,
  "data": {
    "id": 456,
    "title": "New Deal",
    "value": 5000,
    "currency": "USD",
    "status": "open",
    "priority": "high",
    "probability": 10,
    "expected_close_date": "2026-03-15",
    "source": "make",
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
    "created_at": "2026-01-15 10:30:00",
    "updated_at": "2026-02-20 14:00:00",
    "deal_url": "https://example.com/wp-admin/admin.php?page=quillcrm&path=deals/456",
    "message": "Deal created successfully."
  }
}
```

### Update Deal Title

**Action:** `update_deal_title`

```json
{
  "quillcrm_make_action": "update_deal_title",
  "api_key": "{{connection.apiKey}}",
  "deal_id": 456,
  "title": "Updated Deal Title"
}
```

### Update Deal Value

**Action:** `update_deal_value`

```json
{
  "quillcrm_make_action": "update_deal_value",
  "api_key": "{{connection.apiKey}}",
  "deal_id": 456,
  "value": 7500
}
```

### Update Deal Owner

**Action:** `update_deal_owner`

```json
{
  "quillcrm_make_action": "update_deal_owner",
  "api_key": "{{connection.apiKey}}",
  "deal_id": 456,
  "owner_id": 2
}
```

### Update Deal Stage

**Action:** `update_deal_stage`

```json
{
  "quillcrm_make_action": "update_deal_stage",
  "api_key": "{{connection.apiKey}}",
  "deal_id": 456,
  "stage_id": 3,
  "pipeline_id": 1
}
```

`pipeline_id` is optional — if omitted, the deal's current pipeline is used. The deal's `status` and `probability` are automatically updated websited on the new stage's `win_probability`.

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

## Complete Action Reference

| Action | Auth Required | Description |
|---|---|---|
| `verify_api_key` | No | Validate API key |
| `test_authentication` | Yes | Test connection |
| `subscribe` | Yes | Attach webhook |
| `unsubscribe` | Yes | Detach webhook |
| `get_webhooks` | Yes | List all webhooks |
| `get_pipelines` | Yes | List pipelines |
| `get_stages` | Yes | List stages (requires `pipeline_id`) |
| `get_owners` | Yes | List owners |
| `get_lists` | Yes | List lists |
| `get_tags` | Yes | List tags |
| `polling` | Yes | Get sample data for trigger type |
| `create_contact` | Yes | Create or update contact |
| `add_tags` | Yes | Add tags to contact |
| `remove_tags` | Yes | Remove tags from contact |
| `add_to_lists` | Yes | Add contact to lists |
| `remove_from_lists` | Yes | Remove contact from lists |
| `change_status` | Yes | Change contact status |
| `add_deal` | Yes | Create deal |
| `update_deal_title` | Yes | Update deal title |
| `update_deal_value` | Yes | Update deal value |
| `update_deal_owner` | Yes | Update deal owner |
| `update_deal_stage` | Yes | Update deal stage |

---

## Error Responses

### Authentication Error

```json
{
  "success": false,
  "data": {
    "message": "Invalid api key."
  }
}
```

HTTP status: `401`

### Unknown Action

```json
{
  "success": false,
  "data": {
    "message": "Unknown action."
  }
}
```

HTTP status: `422`

### Validation / Action Handler Errors

```json
{
  "success": false,
  "data": {
    "code": "error_code",
    "message": "Human-readable error message."
  }
}
```

**Error codes:**

| Code | Description |
|---|---|
| `unknown_action` | Unknown action type for handler |
| `not_implemented` | Action handler method not implemented |
| `action_failed` | Generic action failure |
| `not_found` | Contact or deal not found |
| `handler_unavailable` | Contact/deal handler not available |
| `invalid_email` | Invalid email address |
| `contact_not_found` | Contact not found by email |

**Common HTTP status codes:** `400` (bad request/validation), `401` (unauthorized), `404` (not found), `422` (missing parameters), `500` (server error)
