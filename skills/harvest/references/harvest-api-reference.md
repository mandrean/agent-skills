# Harvest API v2 Reference

Base URL: `https://api.harvestapp.com/v2`

## Authentication

All requests require these headers:

```
Authorization: Bearer <HARVEST_TOKEN>
Harvest-Account-Id: <HARVEST_ACCOUNT_ID>
User-Agent: YourApp (contact@example.com)
```

Get a Personal Access Token and Account ID at: https://id.getharvest.com/developers

## Endpoints

### GET /v2/users/me

Returns the currently authenticated user.

**Key response fields:**
```json
{
  "id": 1234,
  "first_name": "Jane",
  "last_name": "Doe",
  "email": "jane@example.com",
  "timezone": "Europe/Stockholm",
  "weekly_capacity": 144000
}
```

---

### GET /v2/users/me/project_assignments

Lists projects (and their tasks) assigned to the current user.

**Query params:**
- `is_active=true` — only active assignments (recommended)
- `per_page=2000` — max page size

**Key response fields:**
```json
{
  "project_assignments": [
    {
      "id": 100,
      "is_active": true,
      "project": { "id": 200, "name": "Acme Redesign", "code": "ACME-1" },
      "client":  { "id": 300, "name": "Acme Corp" },
      "task_assignments": [
        { "id": 400, "task": { "id": 500, "name": "Development" }, "billable": true, "hourly_rate": 150.0 },
        { "id": 401, "task": { "id": 501, "name": "Meetings" },    "billable": false }
      ]
    }
  ],
  "per_page": 2000,
  "total_pages": 1,
  "total_entries": 5
}
```

**Note:** `task_assignments[].task.id` is the `task_id` you use when creating entries.

---

### GET /v2/time_entries

Lists time entries for the authenticated user.

**Query params:**
| Param | Type | Description |
|-------|------|-------------|
| `from` | date (YYYY-MM-DD) | Entries on or after this date |
| `to` | date (YYYY-MM-DD) | Entries on or before this date |
| `project_id` | integer | Filter by project |
| `task_id` | integer | Filter by task |
| `is_billed` | boolean | Filter billed entries |
| `user_id` | integer | Filter by user (admins only) |
| `per_page` | integer | Max 2000, default 2000 |
| `page` | integer | Page number |

**Key response fields:**
```json
{
  "time_entries": [
    {
      "id": 999,
      "spent_date": "2026-03-17",
      "hours": 7.5,
      "rounded_hours": 7.5,
      "notes": "Feature work",
      "project":  { "id": 200, "name": "Acme Redesign" },
      "task":     { "id": 500, "name": "Development" },
      "user":     { "id": 1234, "name": "Jane Doe" },
      "billable": true,
      "is_locked": false,
      "is_billed": false,
      "approval_status": "pending",
      "created_at": "2026-03-17T09:00:00Z",
      "updated_at": "2026-03-17T17:30:00Z"
    }
  ],
  "per_page": 2000,
  "total_pages": 1,
  "total_entries": 9
}
```

**Important:** `is_locked: true` entries cannot be duplicated via the API. Skip them.

---

### POST /v2/time_entries

Creates a new time entry.

**Required body fields:**
| Field | Type | Description |
|-------|------|-------------|
| `project_id` | integer | ID of the project |
| `task_id` | integer | ID of the task |
| `spent_date` | string (YYYY-MM-DD) | Date of the entry |

**Optional body fields:**
| Field | Type | Description |
|-------|------|-------------|
| `hours` | decimal | Duration in hours (e.g. `7.5`) |
| `notes` | string | Description / notes |
| `user_id` | integer | Defaults to the authenticated user |

**Example request:**
```bash
curl -s -X POST \
  -H "Authorization: Bearer $HARVEST_TOKEN" \
  -H "Harvest-Account-Id: $HARVEST_ACCOUNT_ID" \
  -H "User-Agent: ClaudeHarvestSkill (claude@anthropic.com)" \
  -H "Content-Type: application/json" \
  -d '{
    "project_id": 200,
    "task_id": 500,
    "spent_date": "2026-03-23",
    "hours": 7.5,
    "notes": "Feature work"
  }' \
  https://api.harvestapp.com/v2/time_entries
```

**Success response:** `201 Created` with the full entry object.

**Error responses:**
- `401` — bad or missing token
- `403` — forbidden (locked period, insufficient permissions)
- `404` — project or task not found / not assigned
- `422` — validation error (see `message` field in response body)

---

## Pagination

All list endpoints return:
```json
{
  "per_page": 2000,
  "total_pages": 1,
  "total_entries": 42,
  "next_page": null,
  "previous_page": null,
  "page": 1,
  "links": { "first": "...", "next": null, "previous": null, "last": "..." }
}
```

For most users, `per_page=2000` covers a full week with a single request.

---

## Tips

- Always use `spent_date` in `YYYY-MM-DD` format.
- `hours` can be a decimal: `0.25` = 15 min, `0.5` = 30 min, `1.0` = 1h.
- To copy last week → this week, add exactly 7 days to each `spent_date`.
- The API is rate-limited; check the `Retry-After` header on `429` responses.
