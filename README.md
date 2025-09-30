# Proxim (Remote Desktop Control)

Proxim is a privacy-focused remote desktop control system that enables secure peer-to-peer connections for controlling a desktop computer, initially between two desktops (PC-to-PC) and later from a mobile device to a desktop. Built with local-first principles, it uses QR codes for session initiation, requiring no accounts or centralized servers.

## Key Features

* Remote desktop control (desktop-to-desktop first, then mobile-to-desktop)
* Secure peer-to-peer connections via WebRTC
* Session-based access with manual approval
* No account or login system required
* Cross-platform core:
  * Desktop and mobile apps in Rust with Tauri (HTML/JS UI)
  * Embedded signaling server in Rust

## Architecture Overview

```
 Mobile App (Tauri)    Signaling (Embedded Rust)    Desktop App (Rust/Tauri)
     |                       |                          |
     |--- Pairing Request -->|                          |
     |                       |--> Notify Rust App --->  |
     |                       |<-- Accept/Reject ------- |
     |<------ WebRTC Negotiation via Signaling ------>  |
     |<-------------------- Direct P2P ---------------->|
```

* **Signaling:** Embedded in the Rust app using crates like `axum` or `warp` for WebSocket communication.
* **Desktop-to-Desktop:** Initial MVP focuses on PC-to-PC control to validate core functionality.
* **Mobile Support:** Added later using Tauri, with HTML/JS frontend for QR scanning and UI, and Rust backend for logic.
* Connections are peer-to-peer after signaling, using WebRTC with STUN for NAT traversal.

## Happy Path Summary

1. User launches the desktop app (host).
2. User clicks "Generate QR Code" and selects session type (one-time, persistent, temporary).
3. A QR code is generated containing UUID, sessionID, and optional IP:port.
4. For desktop-to-desktop: Another PC (client) scans the QR using a webcam or manual entry.
5. For mobile-to-desktop: Mobile app (Tauri) scans the QR.
6. Client sends a pairing request to the embedded signaling server.
7. Host desktop receives the request, displays device info, and awaits manual approval (40 seconds).
8. Upon approval, WebRTC establishes a direct P2P connection for full desktop control.

> See `happy_path.md` for detailed internal flow.

## Session Types

* **One-time**: Expires after the first disconnect.
* **Persistent**: Remains active until manually deleted.
* **Temporary**: Auto-expires after a user-defined duration.

Each session is identified by a `sessionID` and bound to a `UUID` representing the desktop.

## Security Model

* No credentials or passwords are stored or exchanged.
* QR code acts as a one-time authentication token.
* Manual user approval required before each connection.
* QR codes and sessions auto-expire after 15 minutes or if ignored.

## Technologies Used

| Component          | Language / Stack             |
| ------------------ | ---------------------------- |
| Desktop/Mobile App | Rust + Tauri (HTML/JS UI)    |
| Signaling Server   | Rust (embedded, axum/warp)   |
| Signaling Protocol | WebRTC + JSON over WebSocket |

## Development Status

Current phase: **Desktop-to-desktop core functionality**

Next steps:

* Embed signaling server in Rust using `axum` or `warp`.
* Implement PC-to-PC remote control (mouse/keyboard).
* Add Tauri-based mobile support for mobile-to-desktop control.

## Internal Documentation

* [`ideal_flow.md`](./.docs/ideal_flow.md)
* [`system_architecture.md`](./.docs/system_architecture.md)
* [`signaling_protocol.md`](./.docs/signaling_protocol.md)
* [`session_management.md`](./.docs/session_management.md)
* [`webrtc_setup.md`](./.docs/webrtc_setup.md)
* [`tasks.md`](./tasks.md)