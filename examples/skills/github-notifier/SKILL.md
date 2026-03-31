# github-notifier

**Trigger:** When asked about GitHub notifications, PRs, issues, or CI status. Also runs during heartbeat if `gh` CLI is available.

## Requirements

- `gh` CLI installed and authenticated (`gh auth login`)

## Usage

### Check notifications
```bash
gh api notifications --jq '.[] | {repo: .repository.full_name, title: .subject.title, type: .subject.type}'
```

### Recent PR activity
```bash
gh pr list --repo <owner/repo> --state open --json number,title,author,updatedAt
```

### Failed CI runs
```bash
gh run list --repo <owner/repo> --status failure --limit 5 --json name,conclusion,headBranch,createdAt
```

## Heartbeat Behavior

During heartbeat, check for unread notifications. If any exist:
1. Summarize by repo and type
2. Highlight anything that looks urgent (PR review requested, CI failure on main)
3. Notify user if something needs attention

## Notes

- Rate limit: GitHub API allows 60 unauthenticated / 5000 authenticated requests/hour
- Store last-checked timestamp in `memory/heartbeat-state.json` under key `github`
