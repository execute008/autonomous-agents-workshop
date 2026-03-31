# Multi-Agent via MQTT — Setup Guide

Two agents on different machines communicating in real-time over MQTT.

## Architecture

```
Machine A (Server)              Machine B (Laptop/VPS)
  OpenClaw or ZeroClaw    ←──→    OpenClaw or ZeroClaw
         │                               │
         └────── MQTT Broker ────────────┘
                (Mosquitto)
           hive/agent-a/inbox
           hive/agent-b/inbox
           hive/broadcast
```

## 1. Start MQTT Broker

```bash
# Via Podman (rootless)
podman run -d \
  --name mqtt-broker \
  --network=host \
  -v ~/.mqtt/config:/mosquitto/config:ro \
  -v ~/.mqtt/data:/mosquitto/data \
  eclipse-mosquitto

# Via Docker
docker run -d \
  --name mqtt-broker \
  -p 1883:1883 \
  -v ~/.mqtt/config:/mosquitto/config:ro \
  eclipse-mosquitto
```

## 2. Mosquitto Config (with auth)

```conf
# ~/.mqtt/config/mosquitto.conf
listener 1883
allow_anonymous false
password_file /mosquitto/config/passwd
```

```bash
# Create credentials
mosquitto_passwd -c ~/.mqtt/config/passwd agent-a
mosquitto_passwd ~/.mqtt/config/passwd agent-b
```

## 3. Send a message between agents

```python
import paho.mqtt.publish as publish

publish.single(
    topic="hive/agent-b/inbox",
    payload='{"from": "agent-a", "message": "Hello from A!"}',
    hostname="192.168.1.100",  # or Tailscale IP
    port=1883,
    auth={"username": "agent-a", "password": "secret"}
)
```

## 4. OpenClaw — Listener Service (systemd)

The `mqtt-listener` tool subscribes to `hive/<agent>/inbox` and injects messages
directly into the running agent session via WebSocket:

```bash
# Install listener
openclaw gateway call chat.inject '{"message": "..."}'

# Register as systemd user service
systemctl --user enable mqtt-listener
systemctl --user start mqtt-listener
```

## 5. Over Tailscale (recommended for distributed setups)

No port forwarding needed — agents discover each other via Tailscale IPs:

```toml
# Each agent knows the other's Tailscale IP
# Agent A: 100.x.x.1
# Agent B: 100.x.x.2
# Broker runs on Agent A, accessible at 100.x.x.1:1883
```

## Tips

- Use `hive/broadcast` for messages all agents should see
- Keep payloads small — agents process the full text in their context window
- Use timestamps in payloads to avoid replay confusion
