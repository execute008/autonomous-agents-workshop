# Use Case: Overnight Coding Session

**Scenario:** You want to work on a feature overnight while you sleep. The agent autonomously implements, tests, commits, and reports progress.

## Setup

### 1. Project Structure
```
~/your-project/
├── .planning/         # GSD phases or draht task docs
├── src/              # Your codebase
└── README.md
```

### 2. Spawn Agent
```bash
~/clawd/tools/spawn-draht.sh ~/your-project overnight-build \
  "Complete Phase 5: User Authentication. Implement OAuth flow, session management, \
   and JWT tokens. Test with vitest. Commit atomically per feature."
```

### 3. Track Job
```bash
~/clawd/tools/job-tracker.sh add \
  "overnight-build-$(date +%Y%m%d)" \
  "draht" \
  "tmux has-session -t overnight-build 2>/dev/null" \
  "$(date -v+12H -Iseconds)"
```

## Agent Notification Extension

The agent notifies you after each turn:

```typescript
// ~/clawd/extensions/draht-notify/index.ts
export default {
  name: 'draht-notify',
  agent_end: async (ctx) => {
    if (!process.env.OPENCLAW_SESSION_KEY) return;
    
    const msg = ctx.error 
      ? `❌ Error: ${ctx.error}`
      : `✅ Turn complete (${ctx.stats.tokens}% tokens, $${ctx.stats.cost})`;
    
    exec(`openclaw agent --to ${sessionId} --message "${msg}" --deliver`);
  }
};
```

## HEARTBEAT Monitoring

Add to `HEARTBEAT.md`:

```markdown
## Active Jobs

Check all overnight sessions every heartbeat:
\`\`\`bash
~/clawd/tools/job-tracker.sh check
\`\`\`

For each stuck job (>30min no commits):
1. Check tmux session status
2. Auto-restart if crashed
3. Alert if stuck 3x
```

## Real Example (fr3n overnight)

**What worked:**
- Started: 23:10
- Completed: 02:01
- Delivered: 7 commits, 304+ tests passing
- Cost: $9.39, 13.3% tokens
- Phases 18-25 complete

**Key learnings:**
- Spawn with `--model anthropic/claude-opus-4-6` for complex work
- Use event-driven notifications (no polling)
- Register in job-tracker immediately
- Session crashed but work was committed (verify with git)

## Troubleshooting

**Session stuck in "Working..."**
```bash
tmux send-keys -t overnight-build C-c
# Check last commit:
cd ~/your-project && git log -1
```

**No notifications received**
- Verify `OPENCLAW_SESSION_KEY` is set in tmux
- Check extension is loaded: `--extension ~/clawd/extensions/draht-notify`

**Want to interrupt and resume later**
```bash
tmux detach -t overnight-build  # Keeps running
tmux attach -t overnight-build  # Resume
```
