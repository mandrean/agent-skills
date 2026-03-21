# GH CLI Manual Cheatsheet

Use this file as a compact map of the official manual. Prefer `gh help reference` and `gh <command> --help` on the current machine when exact flags matter.

## Contents

1. Command Families
2. Establish Context
3. Common Workflows
4. Structured Output
5. API Patterns
6. Auth And Environment
7. Exit Codes

## Command Families

- Core repository work: `gh repo`, `gh browse`, `gh search`, `gh status`
- Collaboration: `gh pr`, `gh issue`, `gh label`, `gh project`
- Delivery: `gh release`, `gh workflow`, `gh run`, `gh cache`
- Environment and access: `gh auth`, `gh secret`, `gh variable`, `gh ruleset`, `gh gpg-key`, `gh ssh-key`
- Platform features: `gh codespace`, `gh gist`, `gh extension`, `gh alias`
- Escape hatch: `gh api`

## Establish Context

- Check the installed CLI version with `gh --version`.
- Check auth with `gh auth status`.
- Inspect the current default repository with `gh repo set-default -v` when repo inference may be unclear.
- Set `GH_REPO=[HOST/]OWNER/REPO` to target a repo outside the current checkout.
- Set `GH_HOST` or pass `--hostname` for GitHub Enterprise Server or multi-host setups.
- Prefer `GH_TOKEN` or `GH_ENTERPRISE_TOKEN` for headless and CI usage.

## Common Workflows

- Inspect pull requests:
  `gh pr list --json number,title,headRefName,reviewDecision,updatedAt`
- Inspect issues:
  `gh issue list --json number,title,state,labels,assignees`
- View workflow runs:
  `gh run list --json databaseId,displayTitle,headBranch,status,conclusion,updatedAt`
- Open a browser quickly:
  `gh pr view 123 --web`
- Work outside a clone:
  `GH_REPO=OWNER/REPO gh issue view 123`

## Structured Output

- Use `--json` on commands that support it.
- Omit the field list once to discover available JSON fields:
  `gh pr list --json`
- Use `--jq` for extraction:
  `gh pr list --json number,title --jq '.[] | {number, title}'`
- Use `--template` for formatted terminal views:
  `gh pr list --json number,title,updatedAt --template '{{range .}}{{tablerow (printf "#%v" .number) .title (timeago .updatedAt)}}{{end}}{{tablerender}}'`
- Helpful template functions from `gh help formatting`: `tablerow`, `tablerender`, `timeago`, `timefmt`, `join`, `pluck`, `truncate`, `hyperlink`, `autocolor`

## API Patterns

- Use REST:
  `gh api repos/{owner}/{repo}/issues`
- Post a comment:
  `gh api repos/{owner}/{repo}/issues/123/comments -f body='Hi from gh'`
- Use typed fields and file input:
  `gh api gists -F 'files[note.txt][content]=@note.txt'`
- Force query-string parameters on GET:
  `gh api -X GET search/issues -f q='repo:cli/cli is:open'`
- Use GraphQL:
  `gh api graphql -F owner='{owner}' -F name='{repo}' -f query='query($owner:String!,$name:String!){repository(owner:$owner,name:$name){nameWithOwner}}'`
- Paginate when needed:
  `gh api graphql --paginate --slurp ...`

## Auth And Environment

- Interactive login:
  `gh auth login`
- Browser login:
  `gh auth login --web --clipboard`
- Token login from stdin:
  `gh auth login --with-token < mytoken.txt`
- High-value environment variables:
  `GH_TOKEN`, `GITHUB_TOKEN`, `GH_ENTERPRISE_TOKEN`, `GITHUB_ENTERPRISE_TOKEN`
- Target selection:
  `GH_HOST`, `GH_REPO`
- Non-interactive behavior and debugging:
  `GH_PROMPT_DISABLED`, `GH_DEBUG`, `GH_FORCE_TTY`, `GH_CONFIG_DIR`
- Rendering and output control:
  `GH_PAGER`, `NO_COLOR`, `CLICOLOR`, `CLICOLOR_FORCE`, `GLAMOUR_STYLE`

## Exit Codes

- `0`: success
- `1`: failure
- `2`: cancelled
- `4`: authentication required

Check command-specific help if automation depends on more granular exit behavior.
