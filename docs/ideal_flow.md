# Ideal User Flow – Proxim (Internal Planning)

> This document describes the ideal user flow for initiating a successful session in Proxim, starting with desktop-to-desktop control for initial validation, followed by mobile-to-desktop control. It is intended for internal planning and prototyping purposes only.

---

## 1. Launching the Desktop App (Host)

- The user opens the Proxim desktop application (built in Rust with Tauri or a Rust GUI crate like `iced`/`egui`).
- The app starts an embedded signaling server (Rust, using `axum` or `warp` for WebSocket).
- The UI displays a "Generate QR Code" button.

---

## 2. Choosing a Session Type

- The user clicks "Generate QR Code," and a prompt appears to select the session type:
  - **One-time**: Expires after the first disconnection.
  - **Persistent**: Stays active until manually deleted by the user.
  - **Temporary**: Expires after a user-defined duration (e.g., 1 hour).
- If **Temporary** is selected:
  - The user specifies the session duration.
  - The session will be invalidated after this duration.

---

## 3. QR Code Generation

- After confirming the session type, the app generates a QR code containing:
  - `sessionID` (unique per session)
  - `UUID` (unique desktop identifier)
  - Optional: `IP:port` for signaling server (local or public)
- The QR code is displayed on the screen (using `qrcode` crate for Rust or Tauri web frontend).
- The code is valid for 15 minutes or until manually closed.

---

## 4. Scanning the QR Code

- **Desktop-to-Desktop (Initial MVP):**
  - The client (another PC running Proxim) scans the QR code using a webcam (via `opencv-rust` or `rqrr` for testing) or manually enters the session details.
  - The client app parses the QR to extract `sessionID`, `UUID`, and optional `IP:port`.
- **Mobile-to-Desktop (Later Phase):**
  - The user opens the Proxim mobile app (Tauri with HTML/JS frontend).
  - The app uses a JavaScript library (e.g., `jsQR`) to scan the QR code via the device camera.
  - The mobile app parses the QR data and connects to the signaling server.
- A pairing request is sent to the embedded signaling server.

---

## 5. Pairing Request Prompt on Host Desktop

- The host desktop app receives a `pair_prompt` message via the signaling server.
- The UI displays a prompt showing:
  - Device name (e.g., "Client-PC-123" or "Galaxy S23")
  - Device identifier (e.g., UUID or model)
- The prompt includes:
  - **Approve** button
  - **Reject** button
  - A 40-second countdown timer
- If no action is taken within 40 seconds, the request is automatically rejected.

---

## 6. Final Confirmation and Warning

- If the user clicks **Approve**:
  - A confirmation dialog warns that the client device will gain full control of the system.
  - Upon confirming, the session is marked as `approved`, and WebRTC negotiation begins.
- If the user clicks **Reject** or the timer expires, the session is marked as `rejected` or `expired`, and the client is notified.

---

## 7. Active Session

- After approval, WebRTC establishes a direct peer-to-peer connection (using `webrtc-rs` for Rust on both ends, or Tauri bridge for mobile).
- The client (PC or mobile) gains full control over the host desktop, including:
  - Mouse and keyboard control (via `enigo` crate on desktop or JS events in Tauri).
  - System-level commands (e.g., shutdown, restart, sleep).
- The session behavior depends on the selected session type:
  - **One-time**: Terminates on disconnect.
  - **Persistent**: Continues until manually ended.
  - **Temporary**: Ends at the specified duration.

---

## Notes

- This flow assumes no network or technical errors.
- No account login or authentication is required.
- Desktop-to-desktop is prioritized for MVP to validate signaling, WebRTC, and control.
- Mobile-to-desktop support is added later via Tauri, leveraging the same Rust backend.
- This document is **not for publishing** and captures the ideal user experience for early design.