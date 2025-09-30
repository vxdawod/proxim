# WebRTC & STUN Setup – Proxim

This document explains how to configure and use WebRTC with STUN for establishing peer-to-peer connections between Proxim desktop and mobile clients, starting with desktop-to-desktop.

---

## Overview

WebRTC creates direct data channels between clients (desktop-to-desktop initially, then mobile-to-desktop). STUN servers help peers discover public IPs/ports behind NAT. No TURN servers are used for privacy and simplicity.

* **STUN**: Session Traversal Utilities for NAT
* **WebRTC**: Handled by `webrtc-rs` crate on both ends
* **Tauri**: Bridges WebRTC for mobile via Rust backend and JS frontend

---

## Basic WebRTC Flow (Simplified)

```
Mobile App (Tauri)      Signaling (Embedded Rust)     Desktop App (Rust/Tauri)
    |                         |                           |
    |--- pair_request ------> |                           |
    |                         |--- pair_prompt ---------> |
    |<---- pair_result ------ |<-- pair_response -------- |
    |-- SDP Offer (signal) -> |-> SDP Offer ------------> |
    |<- SDP Answer (signal) --|<- SDP Answer ------------ |
    |-- ICE Candidates -----> |-> ICE Candidates -------->|
    |<-- ICE Candidates ------|<- ICE Candidates -------- |
    |<-------- WebRTC Peer-to-Peer Connection --------->  |
```

* Desktop-to-desktop flow is similar, with client PC replacing mobile.

---

## STUN Configuration

Use public STUN servers (e.g., Google's) for NAT traversal.

### Recommended STUN server:

```text
stun:stun.l.google.com:19302
```

Optional redundancy:

```text
stun:stun1.l.google.com:19302
stun:stun2.l.google.com:19302
```

---

## Desktop (Rust) – WebRTC Setup

Use `webrtc-rs` crate for desktop.

### Sample ICE config:

```rust
use webrtc::api::APIBuilder;
use webrtc::ice_transport::ice_server::RTCIceServer;

let config = webrtc::peer_connection::configuration::RTCConfiguration {
    ice_servers: vec![RTCIceServer {
        urls: vec!["stun:stun.l.google.com:19302".to_string()],
        ..Default::default()
    }],
    ..Default::default()
};

let api = APIBuilder::new().build();
let peer_connection = api.new_peer_connection(config).await?;
```

* Handle SDP/ICE via async handlers.
* Send/receive signaling messages via embedded `axum`/ `warp` WebSocket.

---

## Mobile (Rust + Tauri) – WebRTC Setup

Use `webrtc-rs` in Rust backend, bridged to Tauri JS frontend.

### Initialize WebRTC:

```rust
// In Tauri Rust backend
#[tauri::command]
async fn init_webrtc() -> Result<(), String> {
    let config = webrtc::peer_connection::configuration::RTCConfiguration {
        ice_servers: vec![RTCIceServer {
            urls: vec!["stun:stun.l.google.com:19302".to_string()],
            ..Default::default()
        }],
        ..Default::default()
    };
    // Setup peer connection
    Ok(())
}
```

### JS Frontend (Tauri):

```javascript
import { invoke } from '@tauri-apps/api/tauri';

async function startWebRTC() {
  await invoke('init_webrtc');
  // Handle SDP/ICE via JS events, send to Rust
}
```

* Use JS to trigger Rust WebRTC logic.
* Handle `onIceCandidate`, `onAddStream` in JS, bridge to Rust.

---

## Summary

* STUN enables P2P across NAT.
* WebRTC handled by `webrtc-rs` on both ends.
* Signaling via embedded Rust server.
* Desktop-to-desktop tested first, then mobile.
* No TURN; connection may fail in rare NAT cases (acceptable for MVP).