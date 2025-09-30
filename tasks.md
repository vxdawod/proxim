# Development Tasks – Proxim

This document outlines the development phases and tasks required to implement the Proxim system in a structured and incremental way.

---

> **Note:** All components are implemented in Rust, with Tauri for cross-platform UI (desktop and mobile). The signaling server is embedded as a Rust module using crates like `axum` or `warp`, not a separate subprocess.

---

## Phase 0: Desktop-to-Desktop MVP (PC-PC Remote Control)
> **Info:** Focus on Rust-only implementation for desktop-to-desktop to validate core functionality before adding mobile.

### Task 1: Set up basic Rust desktop app with embedded signaling
- [ ] Initialize Rust project with `cargo new proxim-desktop`
- [ ] Embed WebSocket server using `axum` or `warp` for signaling
- [ ] Handle WebSocket connections and assign roles (host or client)
- [ ] Build message routing based on sessionID
- [ ] Define basic JSON handling with `serde`

### Task 2: Define signaling message types and schema
- [ ] Create Rust structs for message types: `pair_request`, `pair_prompt`, `pair_response`, `pair_result`, `signal`
- [ ] Implement serialization/deserialization with validation
- [ ] Add unit tests for message handling
- [ ] Implement error responses

### Task 3: Implement in-memory session store
- [ ] Define `Session` struct: sessionID, UUID, createdAt, expiresAt, status, deviceInfo
- [ ] Use thread-safe HashMap with `tokio::sync::Mutex`
- [ ] Implement create, get, update, delete functions
- [ ] Add background task with `tokio::spawn` for cleaning expired sessions

### Task 4: Handle pairing requests (PC-PC)
- [ ] On `pair_request`, check sessionID and forward `pair_prompt` to host
- [ ] Attach deviceInfo
- [ ] Enforce single request per sessionID

### Task 5: Auto-expire session and pairing
- [ ] Implement 40-second timeout per session
- [ ] Mark as rejected/expired if no response
- [ ] Log events

### Task 6: Generate UUID and sessionID
- [ ] Generate and save UUID to disk using `uuid` crate
- [ ] Generate sessionID on pairing

### Task 7: Display and scan QR code (for PC-PC testing)
- [ ] Generate QR with `qrcode` crate for host
- [ ] For client PC, implement basic QR scanning using `rqrr` and webcam access (via `opencv-rust` or similar for testing)
- [ ] Render QR in GUI using `iced` or `egui`

### Task 8: Integrate WebRTC for PC-PC
- [ ] Use `webrtc-rs` crate
- [ ] Set up SDP/ICE handling
- [ ] Establish P2P connection after approval

### Task 9: Basic control (mouse/keyboard) for PC-PC
- [ ] Use `enigo` crate for input simulation on host
- [ ] Send inputs via WebRTC data channel from client

---

## Phase 1: Core Infrastructure (Signaling & QR) – Extend to Mobile
> **Info:** Build on Phase 0, adding Tauri for mobile support. Tasks use Rust + Tauri (HTML/JS frontend).

### Task 1: Set up Tauri for cross-platform (desktop + mobile)
- [ ] Add Tauri to Rust project: `cargo install tauri-cli`
- [ ] Configure for desktop and mobile (Android/iOS) builds
- [ ] Use HTML/JS for UI, invoke Rust commands

### Task 2: Mobile QR scanner in Tauri
- [ ] Use JS lib like `jsQR` in WebView for camera access
- [ ] Parse QR data and invoke Rust for connection

### Task 3: Connect from mobile to signaling
- [ ] Establish WebSocket from JS, bridge to Rust backend

### Task 4: Extend pairing for mobile-to-desktop
- [ ] Adapt Phase 0 logic for mobile client role
- [ ] Update UI for mobile approval flow

### Task 5: Auto-expire session and pairing
- [ ] Reuse Phase 0 logic, ensure mobile compatibility
- [ ] Add mobile UI feedback for expiration

### Task 6: Display QR code on desktop
- [ ] Reuse Phase 0 QR generation
- [ ] Optimize for Tauri desktop UI (ensure responsive)

---

## Phase 2: Mobile Pairing & Approval Flow
> **Info:** Fully integrate mobile with Tauri. Tasks in Rust/Tauri.

### Task 1: Implement mobile pairing UI
- [ ] Create HTML/JS UI for scanning and approval feedback
- [ ] Handle pair_prompt, pair_response via Tauri commands

### Task 2: Handle pairing approval on desktop
- [ ] Display deviceInfo in Tauri desktop UI
- [ ] Implement approve/reject buttons with 40-second timeout

### Task 3: Notify mobile of pairing result
- [ ] Send pair_result to mobile via signaling
- [ ] Show success/error on mobile UI

---

## Phase 3: WebRTC Channel Establishment
> **Note:** WebRTC handled by Rust on both ends. Signaling relayed via embedded Rust server.

### Task 1: Integrate WebRTC in Rust (desktop)
- [ ] Reuse Phase 0 WebRTC setup
- [ ] Optimize for mobile compatibility

### Task 2: Integrate WebRTC in Tauri (mobile)
- [ ] Use `webrtc-rs` via Tauri Rust backend
- [ ] Handle SDP/ICE in JS, bridge to Rust

### Task 3: Implement signaling relay
- [ ] Reuse Phase 0 signaling logic
- [ ] Ensure compatibility with mobile WebSocket

### Task 4: Establish working P2P connection
- [ ] Test desktop-to-desktop and mobile-to-desktop
- [ ] Use public STUN servers for NAT traversal
- [ ] Log ICE connection status

---

## Phase 4: Remote Control Functionality
> **Info:** Tasks for desktop and mobile in Rust/Tauri.

### Task 1: Handle mouse input from mobile
- [ ] Capture touch events in Tauri JS
- [ ] Send as WebRTC data message
- [ ] Simulate on desktop with `enigo`

### Task 2: Handle keyboard input from mobile
- [ ] Capture keyboard events in Tauri JS
- [ ] Send via WebRTC
- [ ] Simulate with `enigo`

### Task 3: System-level commands
- [ ] Add buttons in Tauri mobile UI
- [ ] Send commands via WebRTC
- [ ] Execute on desktop with confirmation

### Task 4: Display connection status
- [ ] Show status in Tauri UI (desktop/mobile)
- [ ] Handle reconnects/disconnects

### Task 5: Allow session termination
- [ ] Add "End Session" button in desktop UI
- [ ] Close WebRTC, notify peer

---

## Phase 5: Stability & Polishing
> **Info:** General improvements for Rust/Tauri.

### Task 1: Improve error handling
- [ ] Add retries for WebSocket failures
- [ ] Validate messages rigorously

### Task 2: Add optional session logs
- [ ] Track session events in memory
- [ ] Optional file logging

### Task 3: Optimize startup/shutdown
- [ ] Reduce app launch time
- [ ] Graceful shutdown

### Task 4: Package and distribute
- [ ] Bundle Tauri apps for desktop/mobile
- [ ] Create installers with versioning

### Task 5: UI design polish
- [ ] Unify Tauri UI style
- [ ] Add loading indicators, dialogs

---

## Notes

- Start with Phase 0 for desktop-to-desktop MVP.
- Extend to mobile-to-desktop in Phase 1+.
- Commit after each major step (e.g., `feat: Complete signaling`).
- MVP = Phase 0 + basic WebRTC.