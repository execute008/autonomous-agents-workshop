# daily-standup

**Trigger:** When asked for a standup, daily summary, or "what did I do yesterday/today".

## What this skill does

Generates a developer standup summary from git history across configured repos.

## Usage

For each repo in `standup.repos` (see config below):

```bash
# Yesterday's commits by current user
git -C <repo_path> log --oneline --since="yesterday 00:00" --until="today 00:00" --author="$(git config user.email)"

# Today so far
git -C <repo_path> log --oneline --since="today 00:00" --author="$(git config user.email)"
```

Then format as:

```
## Daily Standup — <date>

**Yesterday:**
- [repo-name] commit message
- [repo-name] commit message

**Today (so far):**
- [repo-name] commit message

**Blockers:** (ask user)
```

## Config

Add to `TOOLS.md` or `USER.md`:

```markdown
### Standup Repos
- /projects/my-app
- /projects/backend-api
- /projects/mobile
```

## Notes

- If no commits found, say so — don't invent activity
- Offer to post the standup to Slack/Discord/Telegram if channels are configured
