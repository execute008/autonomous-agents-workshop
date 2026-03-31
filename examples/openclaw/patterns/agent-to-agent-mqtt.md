# Pattern: Agent-to-Agent Communication (MQTT)

**Scenario:** Two agents (Xenia on Mac, Draco on Garuda Linux) coordinate work without polling.

## Architecture

```
┌─────────────┐         MQTT Broker        ┌─────────────┐
│   Xenia     │      (100.116.43.81)      │   Draco     │
│   (Mac)     │◄─────────────────────────►│  (Garuda)   │
│ Coordinator │                            │  Executor   │
└─────────────┘                            └─────────────┘
      │                                            │
      ▼                                            ▼
~/clawd/shared/                          ~/clawd/shared/
(Syncthing sync)                         (Syncthing sync)
```

## MQTT Topics

```
hive/xenia/inbox     → Messages TO Xenia
hive/draco/inbox     → Messages TO Draco
hive/broadcast       → Shared announcements
```

## Setup

### Broker (Draco's machine)
```bash
# Draco runs the broker
sudo systemctl start mosquitto
sudo systemctl enable mosquitto
```

### Xenia (Sender)
```bash
# ~/clawd/tools/mqtt-send.sh
#!/bin/bash
MESSAGE="$1"
mosquitto_pub -h 100.116.43.81 -p 1883 \
  -t "hive/draco/inbox" \
  -m "$MESSAGE"
```

### Xenia (Receiver)
```bash
# LaunchDaemon: com.xenia.mqtt-listener.plist
mosquitto_sub -h 100.116.43.81 -p 1883 \
  -t "hive/xenia/inbox" |
  while read msg; do
    openclaw agent --message "$msg" --deliver
  done
```

### Draco (Both)
```bash
# Send to Xenia
mosquitto_pub -h 100.116.43.81 -t "hive/xenia/inbox" -m "Task complete"

# Receive from Xenia
mosquitto_sub -h 100.116.43.81 -t "hive/draco/inbox"
```

## Usage Patterns

### 1. Task Handoff

**Xenia → Draco:**
```bash
~/clawd/tools/mqtt-send.sh \
  "Run GPU-intensive model training on ~/clawd/shared/dataset.csv"
```

**Draco receives, executes, reports back:**
```bash
# Draco's workflow
mosquitto_sub -t hive/draco/inbox | while read task; do
  echo "$task" > /tmp/current-task
  # Execute...
  mosquitto_pub -t hive/xenia/inbox -m "Task complete: $task"
done
```

### 2. Shared State

**File:** `~/clawd/shared/project-status.json`

**Xenia writes:**
```json
{
  "phase": "implementation",
  "blockers": ["need GPU for training"],
  "next": "draco-execute"
}
```

**Draco reads, updates:**
```json
{
  "phase": "training",
  "blockers": [],
  "next": "xenia-deploy",
  "model_path": "~/clawd/shared/models/trained-v2.pt"
}
```

**Notification via MQTT:**
```bash
mosquitto_pub -t hive/xenia/inbox \
  -m "Model training complete. See ~/clawd/shared/project-status.json"
```

### 3. Broadcast Updates

**Draco (high GPU usage alert):**
```bash
mosquitto_pub -t hive/broadcast \
  -m "GPU at 95% for next 2h (training run)"
```

**Xenia receives, adjusts plans:**
```bash
# Don't spawn GPU-heavy tasks on Draco right now
```

## Real Example: fr3n Parallel Build

**Scenario:** Xenia coordinates, Draco executes heavy compilation.

**Xenia:**
```bash
# Spawn research agents, coordinate
draht --task "Research Shopify API requirements"

# Once research done, hand off to Draco
mqtt-send.sh "Build Shopify integration based on ~/clawd/shared/shopify-spec.md"
```

**Draco:**
```bash
# Receives task via MQTT
draht --task "$(cat /tmp/current-task)" --gpu

# After completion
mqtt-send.sh "Shopify integration built, tests passing. PR ready."
```

**Xenia:**
```bash
# Receives completion notice
# Reviews PR, merges, deploys
```

## Advantages Over Polling

❌ **Polling:**
```bash
while true; do
  check_if_draco_finished
  sleep 30
done
```
- Wastes cycles
- Delayed (30s intervals)
- Misses instant updates

✅ **Event-driven (MQTT):**
```bash
mosquitto_sub -t hive/xenia/inbox | while read msg; do
  handle_message "$msg"
done
```
- Instant notification
- Zero polling overhead
- Clean async coordination

## Security

**MQTT ACLs (mosquitto.conf):**
```
allow_anonymous false

user xenia
topic write hive/draco/inbox
topic read hive/xenia/inbox
topic read hive/broadcast

user draco
topic write hive/xenia/inbox
topic read hive/draco/inbox
topic read hive/broadcast
```

**Auth:**
```bash
mosquitto_passwd -c /etc/mosquitto/passwd xenia
mosquitto_passwd /etc/mosquitto/passwd draco
```

## Debugging

**Monitor all topics:**
```bash
mosquitto_sub -h 100.116.43.81 -v -t 'hive/#'
```

**Check message flow:**
```bash
# Terminal 1 (subscribe)
mosquitto_sub -t hive/xenia/inbox

# Terminal 2 (publish)
mosquitto_pub -t hive/xenia/inbox -m "test message"
```

**Verify LaunchDaemon (Xenia):**
```bash
launchctl list | grep mqtt
tail -f ~/Library/Logs/mqtt-listener.log
```

## Cost

- Broker: Free (self-hosted Mosquitto)
- Bandwidth: Negligible (<1KB per message)
- Latency: ~10-50ms (local network)

## When to Use

✅ **Good for:**
- Task handoffs between machines
- Async event notifications
- Coordinating parallel agents
- Shared state updates

❌ **Overkill for:**
- Single machine (use files + inotify)
- High-frequency data (use shared DB)
- Large payloads (use S3 + notification)
