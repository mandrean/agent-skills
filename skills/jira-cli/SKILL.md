---
name: jira-cli
description: Operate Jira through the `jira` command-line tool (ankitpokhrel/jira-cli). Use when a task involves Jira CLI, `jira` commands, issues, epics, sprints, boards, projects, releases, worklogs, or issue transitions from a terminal or automation.
argument-hint: "[jira subcommand or task description]"
allowed-tools: Bash(jira *), Read
---

# Jira CLI

Use `jira` to inspect and operate Jira resources from the shell. Prefer live help from the installed CLI for exact flags on the current machine, then use the reference cheatsheet when you need broader command discovery.

## Establish Context

- Run `jira version` when behavior may depend on the installed release.
- Run `jira me` to confirm the authenticated user.
- Run `jira serverinfo` to check the Jira instance type (Cloud vs Server) since some features differ.
- The default project is read from `~/.config/.jira/.config.yml`. Override per-command with `-p PROJECT_KEY`.
- For alternate configs, use `--config PATH` or `JIRA_CONFIG_FILE` environment variable.
- Required environment variables: `JIRA_API_TOKEN`, `JIRA_AUTH_TYPE`, `JIRA_EMAIL`, `JIRA_ORG`.

## Choose The Right Command Family

- Use `jira issue` for all issue operations: list, create, edit, assign, move, view, link, clone, delete, comment, worklog.
- Use `jira epic` for epic management: list, create, add/remove issues.
- Use `jira sprint` for sprint management: list, add issues, close.
- Use `jira board list`, `jira project list`, and `jira release list` for discovery.
- Use `jira open ISSUE-KEY` to open an issue in the browser quickly.
- Use `jira <command> <subcommand> --help` for exact usage and flags.

## Prefer Structured Output

- Use `--plain` for scriptable tab-delimited table output.
- Add `--no-headers` with `--plain` to suppress header rows in pipelines.
- Add `--no-truncate` with `--plain` to show all available columns.
- Use `--columns KEY,STATUS,ASSIGNEE` with `--plain` to select specific columns.
- Use `--raw` for raw JSON output when the result will be parsed programmatically.
- Use `--csv` for CSV-formatted output.
- Use `--delimiter` with `--plain` for custom column separators.

## Filter And Query Issues

- Combine short flags for compact queries: `-yHigh -s"In Progress" --created month -lbackend -a$(jira me)`.
- Use `-q` / `--jql` for raw JQL when built-in filters are not enough: `jira issue list -q "summary ~ cli"`.
- Negate filters with `~` prefix: `-s~Done` (status is NOT Done), `-ax` (unassigned), `-a~x` (assigned to someone).
- Use `--order-by` and `--reverse` to control sort order: `--order-by rank --reverse`.
- Use `--created` and `--updated` with relative periods: `-7d`, `-1h`, `week`, `month`, `year`.
- Use `--paginate FROM:LIMIT` for pagination (max 100): `--paginate 10:50`.

## Non-Interactive Mode

- Many commands are interactive by default and will prompt for input. Use `--no-input` to disable prompts for automation.
- Supply all required fields via flags when using `--no-input`.
- Pipe description/body from stdin: `echo "Body text" | jira issue create -s"Summary" -tTask --no-input`.
- Use `--template PATH` to load descriptions from a file, or `--template -` to read from stdin.

## Protect The User

- Start with read-only commands (`list`, `view`) before mutations when the target state is uncertain.
- Confirm with the user before running `jira issue delete`, especially with `--cascade` which also deletes subtasks.
- Confirm before `jira issue move` to a terminal state like "Done" since transitions may not be reversible.
- Use `--web` on create/edit/move commands to let the user verify in the browser after mutations.
- Prefer explicit `-p PROJECT_KEY` over implicit inference when a mistake would touch the wrong project.

## Read References As Needed

- Read [jira-cli-cheatsheet.md](./references/jira-cli-cheatsheet.md) for command families, common workflows, filtering patterns, output modes, and scripting examples.
