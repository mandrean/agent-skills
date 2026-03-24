# Jira CLI Cheatsheet

Use this file as a compact map of the CLI. Prefer `jira <command> <subcommand> --help` on the current machine when exact flags matter.

## Contents

1. Command Families
2. Establish Context
3. Issue Workflows
4. Epic Workflows
5. Sprint Workflows
6. Filtering And Querying
7. Output Modes
8. Scripting Patterns
9. Environment And Config
10. Navigation Keys

## Command Families

- Issues: `jira issue` (aliases: `issues`) — list, create, edit, assign, move, view, link, unlink, clone, delete, comment, worklog, watch
- Epics: `jira epic` (aliases: `epics`) — list, create, add, remove
- Sprints: `jira sprint` (aliases: `sprints`) — list, add, close
- Boards: `jira board` (aliases: `boards`) — list
- Projects: `jira project` (aliases: `projects`) — list
- Releases: `jira release` (aliases: `releases`) — list
- Browser: `jira open [ISSUE-KEY]`
- Identity: `jira me`
- Server info: `jira serverinfo`

## Establish Context

- Check CLI version: `jira version`
- Check current user: `jira me`
- Check server info: `jira serverinfo`
- Override project: `-p PROJECT_KEY` or `--project PROJECT_KEY`
- Override config: `-c PATH` or `--config PATH` or `JIRA_CONFIG_FILE` env var
- Shell completion: `jira completion --help`

## Issue Workflows

### List / Search

```sh
jira issue list                                      # recent issues (interactive)
jira issue list --plain                              # plain table
jira issue list --raw                                # JSON output
jira issue list --csv                                # CSV output
jira issue list -a$(jira me)                         # assigned to me
jira issue list -r$(jira me) --created week          # reported by me this week
jira issue list -yHigh -s"In Progress" -lbackend     # filtered by priority, status, label
jira issue list -s~Done --created-before -24w -a~x   # not done, old, assigned
jira issue list -q "summary ~ cli"                   # raw JQL
jira issue list --order-by rank --reverse            # ranked ascending
jira issue list --paginate 10:50                     # 50 items starting from 10
jira issue list --history                            # recently accessed
jira issue list -w                                   # watching
jira issue list --plain --columns key,status,assignee --no-headers  # scripting
```

Aliases: `lists`, `ls`, `search`

### Create

```sh
jira issue create                                     # interactive
jira issue create -tBug -s"Title" -yHigh -b"Desc" --no-input  # non-interactive
jira issue create -tStory -s"Story" -PEPIC-42         # attach to epic
jira issue create --template /path/to/template.tmpl   # from template
echo "Body" | jira issue create -s"Title" -tTask      # from stdin
jira issue create -tStory -s"Story" --custom story-points=3  # custom fields
jira issue create --raw                               # JSON output after creation
```

### Edit

```sh
jira issue edit ISSUE-1                               # interactive
jira issue edit ISSUE-1 -s"New title" -yHigh --no-input  # non-interactive
jira issue edit ISSUE-1 --label -old --label new      # remove/add labels (minus to remove)
jira issue edit ISSUE-1 --component -FE --component BE  # replace components
jira issue edit ISSUE-1 --fix-version -v1.0 --fix-version v2.0  # swap versions
```

Aliases: `update`, `modify`

### Assign

```sh
jira issue assign ISSUE-1 "Jon Doe"                   # by name
jira issue assign ISSUE-1 jon@domain.tld              # by email
jira issue assign ISSUE-1 $(jira me)                  # self
jira issue assign ISSUE-1 default                     # default assignee
jira issue assign ISSUE-1 x                           # unassign
```

Aliases: `asg`

### Move / Transition

```sh
jira issue move ISSUE-1 "In Progress"                 # transition
jira issue move ISSUE-1 Done -RFixed -a$(jira me)     # with resolution and assignee
jira issue move ISSUE-1 "In Progress" --comment "WIP" # with comment
```

Aliases: `transition`, `mv`

### View

```sh
jira issue view ISSUE-1                               # default view (uses pager)
jira issue view ISSUE-1 --comments 5                  # show 5 recent comments
jira issue view ISSUE-1 --raw                         # raw JSON
jira issue view ISSUE-1 --plain                       # plain text
```

Aliases: `show`

### Comment

```sh
jira issue comment add ISSUE-1 "Comment body"         # direct
jira issue comment add ISSUE-1 "Internal" --internal  # internal comment
jira issue comment add ISSUE-1 --template /path.tmpl  # from template
echo "Comment" | jira issue comment add ISSUE-1       # from stdin
```

### Worklog

```sh
jira issue worklog add ISSUE-1 "2d 3h 30m" --no-input           # add time
jira issue worklog add ISSUE-1 "10m" --comment "Note" --no-input # with comment
```

### Link / Unlink

```sh
jira issue link ISSUE-1 ISSUE-2 Blocks                # link issues
jira issue link remote ISSUE-1 https://url "Title"    # remote link
jira issue unlink ISSUE-1 ISSUE-2                     # unlink
```

### Clone

```sh
jira issue clone ISSUE-1                               # basic clone
jira issue clone ISSUE-1 -s"New title" -yHigh -a$(jira me)  # with overrides
jira issue clone ISSUE-1 -H"find:replace"              # with text replacement
```

### Delete

```sh
jira issue delete ISSUE-1                              # delete issue
jira issue delete ISSUE-1 --cascade                    # delete with subtasks
```

Aliases: `remove`, `rm`, `del`

## Epic Workflows

```sh
jira epic list                                        # explorer view
jira epic list --table                                # table view
jira epic list KEY-1                                  # issues in epic
jira epic list KEY-1 -ax -yHigh                       # unassigned, high priority in epic
jira epic create -n"Name" -s"Summary" -yHigh -b"Desc" # create
jira epic add EPIC-KEY ISSUE-1 ISSUE-2                # add issues (up to 50)
jira epic remove ISSUE-1 ISSUE-2                      # remove issues (up to 50)
```

## Sprint Workflows

```sh
jira sprint list                                      # explorer view (25 recent)
jira sprint list --table                              # table view
jira sprint list --current                            # active sprint
jira sprint list --current -a$(jira me)               # my items in current sprint
jira sprint list --prev                               # previous sprint
jira sprint list --next                               # next planned sprint
jira sprint list --state future,active                # by state
jira sprint list SPRINT_ID                            # specific sprint
jira sprint list SPRINT_ID --order-by rank --reverse  # ranked ascending
jira sprint add SPRINT_ID ISSUE-1 ISSUE-2             # add issues (up to 50)
```

## Filtering And Querying

### Filter Flags (on `issue list`, `epic list`, `sprint list`)

| Flag | Short | Description |
|------|-------|-------------|
| `--type` | `-t` | Issue type (Bug, Story, Task, Epic...) |
| `--status` | `-s` | Status (repeatable, negate with `~`) |
| `--priority` | `-y` | Priority (High, Medium, Low...) |
| `--assignee` | `-a` | Assignee (`x` = unassigned, `~x` = assigned) |
| `--reporter` | `-r` | Reporter |
| `--label` | `-l` | Label (repeatable) |
| `--component` | `-C` | Component |
| `--parent` | `-P` | Parent issue key |
| `--resolution` | `-R` | Resolution type |
| `--watching` | `-w` | Issues you are watching |
| `--history` | | Recently accessed issues |
| `--created` | | Created date filter |
| `--updated` | | Updated date filter |
| `--jql` | `-q` | Raw JQL query |

### Date Filter Syntax

- Keywords: `today`, `week`, `month`, `year`
- Relative periods: `-7d` (7 days ago), `-1h` (1 hour ago), `-30m` (30 minutes ago), `-24w` (24 weeks ago)
- Absolute dates: `2024-01-15` or `2024/01/15`
- Range variants: `--created-after`, `--created-before`, `--updated-after`, `--updated-before`

### Negation Patterns

- Status not Done: `-s~Done`
- Unassigned: `-ax`
- Assigned to someone (not unassigned): `-a~x`

## Output Modes

| Mode | Flag | Description |
|------|------|-------------|
| Interactive | (default) | TUI with navigation |
| Plain table | `--plain` | Tab-delimited text |
| No headers | `--plain --no-headers` | Plain without header row |
| All columns | `--plain --no-truncate` | Plain with all fields |
| Select columns | `--plain --columns KEY,STATUS` | Plain with specific columns |
| Custom delimiter | `--plain --delimiter "\|"` | Plain with custom separator |
| JSON | `--raw` | Raw JSON API response |
| CSV | `--csv` | CSV format |

## Scripting Patterns

### Count issues created per day this month

```bash
jira issue list --created month --plain --columns created --no-headers \
  | awk '{print $2}' | awk -F'-' '{print $3}' | sort -n | uniq -c
```

### Count issues per sprint

```bash
jira sprint list --table --plain --columns id,name --no-headers | while IFS=$'\t' read -r id name; do
  count=$(jira sprint list "${id}" --plain --no-headers 2>/dev/null | wc -l)
  printf "%10s: %3d\n" "${name}" $((count))
done
```

### List all my open issues as plain text

```bash
jira issue list -a$(jira me) -s~Done --plain --no-headers --columns key,summary,status
```

## Environment And Config

- Config file: `~/.config/.jira/.config.yml` (default)
- Override config: `JIRA_CONFIG_FILE` env var or `--config` flag
- Required env vars: `JIRA_API_TOKEN`, `JIRA_AUTH_TYPE`, `JIRA_EMAIL`, `JIRA_ORG`
- Auth types: `basic` (default), `bearer` (PAT), `mtls` (client certificates)
- Multiple projects: use `-c` flag or `JIRA_CONFIG_FILE` to load alternate configs

## Navigation Keys (Interactive Mode)

| Key | Action |
|-----|--------|
| `j` / `k` / arrows | Navigate up/down |
| `h` / `l` / arrows | Navigate left/right |
| `g` / `G` | Jump to top/bottom |
| `CTRL+f` / `CTRL+b` | Page down/up |
| `v` | View issue details |
| `m` | Transition/move issue |
| `ENTER` | Open in browser |
| `c` | Copy issue URL |
| `CTRL+k` | Copy issue key |
| `CTRL+r` / `F5` | Refresh |
| `w` / `TAB` | Toggle sidebar focus |
| `q` / `ESC` / `CTRL+c` | Quit |
| `?` | Help |
