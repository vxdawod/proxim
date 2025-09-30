# Session Management – Proxim

This document describes how sessions are managed by the Proxim system, using an embedded Rust signaling server for desktop-to-desktop and later mobile-to-desktop connections.

---

## Session Structure

Each session is stored in memory and represented by:

```rust
use serde::{Serialize, Deserialize};
use chrono::{DateTime, Utc};

#[derive(Serialize, Deserialize)]
struct Session {
    session_id: String,
    uuid: String,
    created_at: DateTime<Utc>,
    expires_at: DateTime<Utc>,
    status: String, // "pending", "approved", "rejected", "expired"
    device_info: Option<DeviceInfo>,
}

#[derive(Serialize, Deserialize)]
struct DeviceInfo {
    name: String,
    id: String,
}
```

- `session_id`: Unique per session, generated on QR creation
- `uuid`: Static desktop identifier
- `created_at`: Session creation time
- `expires_at`: Based on session type
- `status`: Session state
- `device_info`: Populated on pairing request

---

## Session Types

| Type        | Description                                   | Expiration Logic               |
|-------------|-----------------------------------------------|---------------------------------|
| One-time    | Expires after first disconnect                | Marked expired on disconnect   |
| Persistent  | Stays active until manually deleted           | No automatic expiration        |
| Temporary   | User defines a custom time limit              | Expires at `expires_at`        |

Session type is selected before generating the QR code.

---

## Lifecycle Flow

1. Rust app generates `session_id` + `uuid` and registers in memory.
2. Session stored with `status = pending`, `expires_at` computed.
3. Client (PC or mobile) sends `pair_request`, attaching `device_info`.
4. Server forwards `pair_prompt` to host desktop.
5. If approved within 40 seconds, session becomes `approved`.
6. If rejected or timed out, session becomes `rejected` or `expired`.
7. Approved sessions enable WebRTC.
8. Cleanup:
   - Manual close
   - Disconnect (one-time)
   - Expiration (temporary)

---

## Cleanup Mechanism

- Background loop (via `tokio::spawn`) checks `expires_at` every 30 seconds.
- Expired/inactive sessions removed from memory.
- One-time sessions removed after disconnect.

---

## Security Notes

- Sessions stored in RAM only (using `tokio::sync::Mutex<HashMap>`).
- No persistent storage or user data logging.
- QR codes are single-use, scoped to one `session_id`.

---

## Future Considerations

- Optional persistent session listing/deletion UI.
- In-memory caching for reconnects.
- External database if needed (e.g., `sled`).