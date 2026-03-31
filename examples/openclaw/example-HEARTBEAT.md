# HEARTBEAT.md — Developer Server Agent

## Every heartbeat (~30 min)
- Check for new GitHub notifications if `gh` CLI is configured
- Any mentions in Slack/Discord?

## Daily (once, morning)
- [ ] Summarize yesterday's git activity across active projects
- [ ] Check for dependency updates: `npm outdated` / `cargo outdated`
- [ ] Any failing CI runs? (`gh run list --status failure`)

## Weekly
- [ ] Review `memory/` files — distill anything worth keeping into `MEMORY.md`
- [ ] Run `npx clawhub update --all` to update skills

## State tracking
Keep last-check timestamps in `memory/heartbeat-state.json` to avoid redundant checks.
