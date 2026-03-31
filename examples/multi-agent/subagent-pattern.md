# Subagent Pattern — Delegating Tasks

The parent agent spawns a child agent for a specific task, waits for results, and incorporates them.

## Use Cases

- Code review (spawn a coding agent on a PR)
- Long-running research (spawn in background, get notified when done)
- Parallel tasks (multiple subagents working simultaneously)
- Isolated execution (subagent can't see parent's private memory)

## OpenClaw — sessions_spawn

```javascript
// One-shot: spawn, run, return result
const result = await sessions_spawn({
  task: "Review this PR diff and list security concerns:\n\n" + diff,
  runtime: "acp",          // ACP = coding agent protocol
  agentId: "claude-code",
  mode: "run",             // one-shot
  streamTo: "parent"       // stream output back to parent session
})

// Persistent: spawn a session that stays alive
const session = await sessions_spawn({
  task: "You are a research assistant. Wait for queries.",
  runtime: "subagent",
  mode: "session",
  thread: true             // bind to current thread (Discord/Telegram)
})
```

## ZeroClaw — WebSocket Gateway

```bash
# Trigger a one-shot agent task via HTTP
curl -X POST http://localhost:3000/api/agent \
  -H "Content-Type: application/json" \
  -d '{"message": "Summarize the last 10 git commits in /projects/myapp"}'

# Connect via WebSocket for streaming
wscat -c ws://localhost:3000/ws/chat
```

## Design Tips

- **Scope the task clearly** — subagents work best with a single, well-defined goal
- **Don't share sensitive memory** — subagents get a fresh workspace by default
- **Set timeouts** — long-running subagents should have a max execution time
- **Stream when possible** — users appreciate seeing progress, not just a final answer
