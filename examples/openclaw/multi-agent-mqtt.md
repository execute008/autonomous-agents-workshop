# Multi-Agent via MQTT — Example Setup

## Architecture
```
Agent A (Server)          Agent B (MacBook)
     │                         │
     └──── MQTT Broker ─────────┘
           hive/a/inbox
           hive/b/inbox
           hive/broadcast
```

## 1. Start MQTT Broker (Mosquitto via Podman)
```bash
podman run -d \
  --name mqtt-broker \
  --network=host \
  -v ~/.openclaw/mqtt/config:/mosquitto/config:ro \
  -v ~/.openclaw/mqtt/data:/mosquitto/data \
  eclipse-mosquitto
```

## 2. Mosquitto config (with auth)
```conf
# ~/.openclaw/mqtt/config/mosquitto.conf
listener 1883
allow_anonymous false
password_file /mosquitto/config/passwd
```

## 3. Send a message to another agent
```bash
python3 - << 'EOF'
import paho.mqtt.publish as publish

publish.single(
    topic="hive/agent-b/inbox",
    payload='{"from": "agent-a", "message": "Hello from A!"}',
    hostname="100.x.x.x",  # Tailscale IP
    port=1883,
    auth={"username": "agent-a", "password": "secret"}
)
EOF
```

## 4. Listener daemon (systemd user service)
See: `mqtt-listener.service` — subscribes to inbox topic,
calls `openclaw gateway call chat.inject` to wake the agent session.
