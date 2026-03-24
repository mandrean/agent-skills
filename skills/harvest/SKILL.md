---
name: harvest
description: Fill out time reporting in Harvest. Copies last week's time entries to the current week by default. Supports picking projects and tasks interactively. Use when the user wants to log time, fill in a timesheet, copy last week's hours, or report time in Harvest.
argument-hint: "[copy-last-week | add | list | --from YYYY-MM-DD]"
allowed-tools: Bash(curl *), Bash(date *), Bash(python3 *), Bash(jq *)
---

# Harvest Time Reporting

Fill in Harvest timesheets by copying last week's entries or interactively picking projects and tasks.

## Prerequisites

Require these environment variables before doing anything else. If either is missing, stop and tell the user:

```
HARVEST_TOKEN        # Personal Access Token from https://id.getharvest.com/developers
HARVEST_ACCOUNT_ID   # Account ID from the same page
```

Check with:
```bash
echo "Token set: ${HARVEST_TOKEN:+yes}" && echo "Account ID set: ${HARVEST_ACCOUNT_ID:+yes}"
```

## Helper: Harvest API Calls

All API calls share the same headers. Use this pattern:

```bash
curl -s \
  -H "Authorization: Bearer $HARVEST_TOKEN" \
  -H "Harvest-Account-Id: $HARVEST_ACCOUNT_ID" \
  -H "User-Agent: ClaudeHarvestSkill (claude@anthropic.com)" \
  -H "Content-Type: application/json"
```

Base URL: `https://api.harvestapp.com/v2`

## Helper: Date Calculations

Use Python for reliable cross-platform date arithmetic:

```bash
# Current week Monday (ISO: week starts Monday)
python3 -c "
from datetime import date, timedelta
today = date.today()
monday = today - timedelta(days=today.weekday())
print(monday)
"

# Previous week Monday and Sunday
python3 -c "
from datetime import date, timedelta
today = date.today()
this_monday = today - timedelta(days=today.weekday())
last_monday = this_monday - timedelta(weeks=1)
last_sunday = this_monday - timedelta(days=1)
print(last_monday, last_sunday)
"
```

## Default Workflow: Copy Last Week

When invoked without arguments (or with `copy-last-week`), follow this flow:

### Step 1 — Compute date ranges

```bash
python3 - <<'EOF'
from datetime import date, timedelta
today = date.today()
this_monday = today - timedelta(days=today.weekday())
last_monday = this_monday - timedelta(weeks=1)
last_sunday = this_monday - timedelta(days=1)
print(f"THIS_MONDAY={this_monday}")
print(f"THIS_SUNDAY={this_monday + timedelta(days=6)}")
print(f"LAST_MONDAY={last_monday}")
print(f"LAST_SUNDAY={last_sunday}")
EOF
```

### Step 2 — Check if current week already has entries

```bash
curl -s \
  -H "Authorization: Bearer $HARVEST_TOKEN" \
  -H "Harvest-Account-Id: $HARVEST_ACCOUNT_ID" \
  -H "User-Agent: ClaudeHarvestSkill (claude@anthropic.com)" \
  "https://api.harvestapp.com/v2/time_entries?from=$THIS_MONDAY&to=$THIS_SUNDAY" \
  | jq '.total_entries'
```

If entries already exist for the current week, warn the user and ask whether to continue (avoid duplicating hours).

### Step 3 — Fetch last week's time entries

```bash
curl -s \
  -H "Authorization: Bearer $HARVEST_TOKEN" \
  -H "Harvest-Account-Id: $HARVEST_ACCOUNT_ID" \
  -H "User-Agent: ClaudeHarvestSkill (claude@anthropic.com)" \
  "https://api.harvestapp.com/v2/time_entries?from=$LAST_MONDAY&to=$LAST_SUNDAY" \
  | jq '.time_entries[] | {
      id,
      spent_date,
      hours,
      notes,
      project: .project.name,
      project_id: .project.id,
      task: .task.name,
      task_id: .task.id,
      billable,
      is_locked
    }'
```

### Step 4 — Present entries to the user

Display a table grouped by day, for example:

```
Last week's time entries (2026-03-16 → 2026-03-22):

Mon 03-16
  8.0h  Acme Corp / Development        "Feature work"     [billable]
  0.5h  Internal / Meetings            "Standup"

Tue 03-17
  7.5h  Acme Corp / Development        "Code review"      [billable]
  ...

Total: 38.5h across 9 entries
```

**Skip any entries where `is_locked: true`** — locked entries cannot be created via API. Note them as skipped.

### Step 5 — Confirm before creating

Ask: "Copy these N entries to this week (2026-03-23 → 2026-03-29)?"

### Step 6 — Create entries for the current week

Use Python (not a shell loop) to avoid zsh newline-splitting issues when iterating dates:

```bash
python3 - "$HARVEST_TOKEN" "$HARVEST_ACCOUNT_ID" <<'PYEOF'
import sys, json, urllib.request, urllib.error

token, account_id = sys.argv[1], sys.argv[2]
headers = {
    "Authorization": f"Bearer {token}",
    "Harvest-Account-Id": account_id,
    "User-Agent": "ClaudeHarvestSkill (claude@anthropic.com)",
    "Content-Type": "application/json",
}

entries = [
    # list of dicts: {"project_id":..., "task_id":..., "spent_date":"YYYY-MM-DD", "hours":..., "notes":"..."}
]

ok = 0; fail = 0
for entry in entries:
    req = urllib.request.Request(
        "https://api.harvestapp.com/v2/time_entries",
        data=json.dumps(entry).encode(), headers=headers, method="POST"
    )
    try:
        with urllib.request.urlopen(req) as r:
            created = json.load(r)
            print(f"OK  {entry['spent_date']}  id={created['id']}")
            ok += 1
    except urllib.error.HTTPError as e:
        err = json.load(e)
        print(f"ERR {entry['spent_date']}  ({e.code}) {err.get('message','?')}")
        fail += 1

print(f"\n{ok} created, {fail} failed | {ok * 8}h total")
PYEOF
```

Report each result: success (show created entry ID) or failure (show error message from response body).

### Step 7 — Summary

After all creates, show:
- How many entries were created successfully
- Total hours logged
- Any failures or skipped locked entries

---

## Alternative: No Last-Week Entries

If last week has no entries, fall through to the interactive flow:

1. Fetch the user's active project assignments:

```bash
curl -s \
  -H "Authorization: Bearer $HARVEST_TOKEN" \
  -H "Harvest-Account-Id: $HARVEST_ACCOUNT_ID" \
  -H "User-Agent: ClaudeHarvestSkill (claude@anthropic.com)" \
  "https://api.harvestapp.com/v2/users/me/project_assignments?is_active=true" \
  | jq '.project_assignments[] | {
      client: .client.name,
      project: .project.name,
      project_id: .project.id,
      tasks: [.task_assignments[] | {name, task_id: .id, billable}]
    }'
```

2. Present a numbered list of projects and their tasks to the user.

3. Ask the user to specify entries:
   - Which project + task (by number or name)
   - Which date(s) (default: today)
   - How many hours
   - Notes (optional)

4. Create the entries.

---

## Alternative: `add` Subcommand

When invoked as `/harvest add`, go directly to the interactive project/task picker above and create entries for a specific day.

## Alternative: `list` Subcommand

When invoked as `/harvest list`, show this week's entries (same as Step 3 but for current week range), formatted as a table.

---

## Error Handling

| Situation | Action |
|-----------|--------|
| Missing env vars | Stop immediately, print setup instructions |
| 401 Unauthorized | Tell user to check `HARVEST_TOKEN` and `HARVEST_ACCOUNT_ID` |
| 403 Forbidden | The entry may be locked or in a closed period; skip and note |
| 404 Not Found | Project or task may have been deactivated; skip and note |
| 422 Unprocessable | Show the API error message; likely missing required field |
| jq not installed | Use `python3 -c "import json,sys; ..."` for JSON parsing instead |

## Read References As Needed

- Read [harvest-api-reference.md](./references/harvest-api-reference.md) for full endpoint details, pagination, and field definitions.
