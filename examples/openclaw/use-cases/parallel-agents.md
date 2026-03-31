# Use Case: Parallel Agent Orchestration

**Scenario:** You have multiple independent tasks (deploy, test, fix bugs). Spawn parallel agents to work simultaneously.

## Real Example: fr3n Monday Optimization

**Context:** Oskar wanted fr3n optimized (UX, performance, AI features) while spending the day with family.

### Spawned Agents (11:21-20:00)

**Agent 1: fr3n-optimization**
```bash
~/clawd/tools/spawn-draht.sh ~/fr3n/fr3n-mono fr3n-optimization \
  "Optimize fr3n: UX polish, performance (lazy loading, code splitting), \
   AI features working, practical fixes (forms, validation, edge cases). \
   Work through systematically and commit atomically."
```

**Agent 2: fr3n-deploy**
```bash
~/clawd/tools/spawn-draht.sh ~/fr3n/fr3n-mono fr3n-deploy \
  "Push all commits, attempt 'bunx sst deploy --stage dev', \
   document what's working and needs testing."
```

**Agent 3: fr3n-ai-test**
```bash
~/clawd/tools/spawn-draht.sh ~/fr3n/fr3n-mono fr3n-ai-test \
  "Test AI features end-to-end: Agent Executor, MCP Tools, \
   SMM Automation, Vibe-Coding. Document pass/fail + bugs."
```

**Agent 4: fr3n-forms**
```bash
~/clawd/tools/spawn-draht.sh ~/fr3n/fr3n-mono fr3n-forms \
  "Audit all forms: find missing validation, broken submissions, \
   poor error messages. Fix systematically and commit atomically."
```

### Results

| Agent | Duration | Commits | Cost | Outcome |
|-------|----------|---------|------|---------|
| fr3n-optimization | 6h 46m | 15 | $8.94 | ✅ Complete |
| fr3n-deploy | 2h 15m | 0 | $3.23 | ✅ Deployed to dev |
| fr3n-ai-test | 45m | 0 | $1.13 | ✅ Report: 95.8% pass |
| fr3n-forms | 1h 20m | 1 | $2.54 | ✅ 11 bugs fixed |

**Total:** 27 commits, $15.84, all work complete while Oskar enjoyed family time.

## Orchestration Pattern

### 1. Launch All Agents
```bash
for task in optimization deploy ai-test forms; do
  ~/clawd/tools/spawn-draht.sh ~/project project-$task "Task $task..."
  ~/clawd/tools/job-tracker.sh add "project-$task-$(date +%Y%m%d)" ...
done
```

### 2. Event-Driven Notifications
All agents use `~/clawd/extensions/draht-notify` so you get instant updates when they complete turns.

### 3. Coordinator Agent (You)
You wake up after each notification:
- Check for questions
- Answer them
- Let agents continue

### 4. Cleanup
```bash
~/clawd/tools/job-tracker.sh check  # See what's still running
tmux ls | grep project-             # List sessions
```

## When to Use Parallel Agents

✅ **Good for:**
- Independent modules (API + Frontend + Tests)
- Different phases (Research + Implementation + Documentation)
- Deploy + Test + Fix (separate workstreams)

❌ **Bad for:**
- Sequential dependencies (must finish A before B)
- Shared files (merge conflicts)
- Single critical path

## Pro Tips

1. **Use different models per task:**
   - Opus for complex architecture
   - Sonnet for implementation
   - Haiku for tests/docs

2. **Set different timeouts:**
   - Deploy: 2h
   - Research: 4h
   - Implementation: 12h

3. **Spawn in sequence if dependent:**
   ```bash
   spawn agent1 && wait_for_completion && spawn agent2
   ```

4. **Monitor via HEARTBEAT.md:**
   ```bash
   ~/clawd/tools/job-tracker.sh check
   ```
   Auto-restart crashed agents.

## Cost Control

**Budget per agent:**
```bash
# Pass model with token limit
draht --model anthropic/claude-sonnet-4-5 --max-tokens 100000
```

**Track total spend:**
```bash
# Sum across all agents
tmux list-sessions | grep project- | \
  xargs -I {} tmux capture-pane -t {} -p | \
  grep -E "\$[0-9]+\.[0-9]+" | \
  awk '{sum+=$2} END {print sum}'
```
