---
name: gh-cli
description: Operate GitHub through the `gh` command-line tool. Use when a task involves GitHub CLI, `gh` commands, repositories, pull requests, issues, releases, workflow runs, codespaces, extensions, authentication, or `gh api` requests from a terminal or automation.
---

# GH CLI

Use `gh` to inspect and operate GitHub resources from the shell. Prefer live help from the installed CLI for exact flags on the current machine, then use the official manual when you need stable documentation links or broader command discovery.

## Establish Context

- Run `gh --version` when behavior may depend on the installed release.
- Identify the target repository and host before acting.
- In a checked-out repository, let `gh` infer `{owner}` and `{repo}` from the current git remote when that is clearly correct.
- Outside a repository, or when inference would be ambiguous, set `GH_REPO=[HOST/]OWNER/REPO` and `GH_HOST` explicitly.
- Check authentication with `gh auth status` before a task that mutates state or calls a private API.
- Prefer environment tokens such as `GH_TOKEN` or `GH_ENTERPRISE_TOKEN` for non-interactive automation.

## Choose The Right Command Family

- Use `gh pr`, `gh issue`, `gh repo`, `gh release`, `gh workflow`, `gh run`, `gh search`, and `gh codespace` before reaching for raw API calls.
- Use `gh browse` or `--web` when a browser view is faster than reconstructing a URL manually.
- Use `gh api` when the top-level commands do not expose the needed field, filter, or mutation.
- Use `gh help reference` to enumerate available commands, and `gh <command> [<subcommand>] --help` for exact usage.

## Prefer Structured Output

- Prefer `--json` when the result will be filtered, reformatted, compared, or passed into a later step.
- Discover supported JSON fields by running a supported command with `--json` and no field list.
- Prefer `--jq` for compact extraction and filtering when the next step needs plain values.
- Prefer `--template` when the result should remain human-readable but more structured than the default output.
- Use `gh help formatting` when template functions or field discovery matter.

## Use `gh api` Deliberately

- Use REST paths such as `repos/{owner}/{repo}/issues` for API v3 work.
- Use `graphql` for API v4 queries or when the REST surface is too chatty.
- Use `-f/--raw-field` for string parameters and `-F/--field` for typed values, placeholder expansion, and `@file` input.
- Use `--paginate` for multi-page responses and add `--slurp` when you need one outer JSON array.
- Use `--include` or `--verbose` when debugging headers, rate limits, or request construction.

## Protect The User

- Start with read-only commands before mutations when the target state is uncertain.
- Call out destructive switches such as `--delete`, `--yes`, or `--force` before using them.
- Prefer explicit repository and hostname arguments over implicit inference when a mistake would touch the wrong repo or org.
- Remember the common exit codes: `0` success, `1` failure, `2` cancelled, `4` authentication required. Check command-specific docs when automation depends on exact codes.

## Read References As Needed

- Read [gh-manual-cheatsheet.md](./references/gh-manual-cheatsheet.md) for command-family selection, common workflows, structured output, auth and environment variables, and `gh api` patterns.
