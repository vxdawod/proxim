# System Architecture – Proxim

This document describes the high-level architecture of Proxim, starting with desktop-to-desktop control, then extending to mobile-to-desktop, all in Rust with Tauri.

---

## Components Overview

### 1. Desktop Application (Rust + Tauri)
- GUI using `iced`, `egui`, or Tauri web (HTML/JS).
- Generates QR codes, manages session approval.
- Embeds signaling server as Rust module (`axum`/ `warp`).
- Executes remote control commands (mouse, keyboard, system).

### 2. Signaling Server (Embedded in Rust)
- Runs within desktop app as Rust module.
- Handles session registration, pairing, expiration.
- Relays WebRTC signaling messages.

### 3. Mobile Application (Rust + Tauri)
- HTML/JS frontend (via Tauri WebView) for QR scanning, UI.
- Rust backend for WebRTC, signaling logic.
- Connects to embedded signaling server.

---

## Communication Flow

```
Mobile App (Tauri)      Signaling (Embedded Rust)     Desktop App (Rust/Tauri)
    |                         |                           |
    |--- scan QR ----------> |                           |
    |--- connect ----------> |                           |
    |--- pair_request ------>|                           |
    |                         |--- notify ------------->|
    |                         |<-- approval/reject ----|
    |<--- signaling exchange (SDP, ICE) via ----------->|
    |<---------- WebRTC peer-to-peer connection ------->|
```

* Desktop-to-desktop flow replaces mobile with another desktop client.
* Signaling only for setup; WebRTC is P2P.

---

## Session Initiation

1. Host desktop starts, embeds signaling server.
2. User generates QR with `session_id`, `uuid`, optional `IP:port`.
3. Client (PC or mobile) scans QR, connects to server.
4. Session created, pairing begins.

---

## Session Approval and Connection

- Host shows device info, waits 40 seconds for approval.
- If approved, SDP/ICE exchanged, WebRTC connects.
- If rejected/timed out, session invalidated.

---

## Deployment Assumptions

- One session per desktop instance.
- Signaling embedded, not publicly hosted.
- Sessions in memory (no persistent storage).
- WebRTC uses STUN; no TURN.
- NAT traversal via hole punching.

---

## Notes

- Prioritizes simplicity, privacy, local-first.
- Desktop-to-desktop MVP validates core.
- Mobile support via Tauri (Android/iOS).
- Future: Optional external signaling, TURN.