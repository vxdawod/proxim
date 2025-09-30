# Signaling Protocol – Proxim

This document defines the signaling protocol for establishing WebRTC sessions between desktop (and later mobile) clients, using an embedded Rust server.

---

## Protocol Overview

- **Transport**: WebSocket (via `axum`/ `warp` in Rust)
- **Message Format**: JSON (via `serde`)
- **Communication**: Routed through embedded Rust server
- **Clients**: Desktop-to-desktop (MVP), then mobile-to-desktop

---

## Connection Setup Flow

1. Desktop app starts, embeds signaling server.
2. Host generates `session_id` + `uuid`, creates QR.
3. Client (PC or mobile) scans QR, connects to server.
4. Client sends `pair_request`.
5. Server forwards to host desktop.
6. Host shows approval prompt (40 seconds).
7. If approved, SDP/ICE exchanged via server.
8. WebRTC connection established directly.

---

## Message Format

Messages sent over WebSocket in JSON format, using `serde`.

```json
{
  "type": "pair_request",
  "sessionID": "abc123",
  "deviceInfo": {
    "name": "Client-PC-123",
    "id": "device-xyz"
  }
}
```

---

## Message Types

### 1. `pair_request` (client → server)
```json
{
  "type": "pair_request",
  "sessionID": "abc123",
  "deviceInfo": { "name": "Client-PC-123", "id": "device-xyz" }
}
```

### 2. `pair_prompt` (server → host)
```json
{
  "type": "pair_prompt",
  "sessionID": "abc123",
  "deviceInfo": { "name": "Client-PC-123", "id": "device-xyz" }
}
```

### 3. `pair_response` (host → server)
```json
{
  "type": "pair_response",
  "sessionID": "abc123",
  "approved": true
}
```

### 4. `pair_result` (server → client)
```json
{
  "type": "pair_result",
  "sessionID": "abc123",
  "status": "approved" // or "rejected"
}
```

### 5. `signal` (bidirectional)
```json
{
  "type": "signal",
  "sessionID": "abc123",
  "payload": { "sdp": "..." } // or { "candidate": "..." }
}
```

---

## Timeouts & Cleanup

- Pairing requests expire after 40 seconds.
- Sessions expire after 15 minutes or per session type.
- Expired sessions removed from memory.

---

## Notes

- Connections initiated by clients.
- Server runs in Rust app, no external hosting.
- `sessionID` ensures unique joins.