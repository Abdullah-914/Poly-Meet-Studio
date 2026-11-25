# PolyMeet Studio

> **Desktop Multilingual Collaboration Suite with On-Device Translation and WebRTC Conferencing**

PolyMeet Studio is a cross-platform desktop application (Windows 11 and Linux) that unifies peer-to-peer video conferencing, native screen sharing, host-governed meeting controls, a collaborative vector whiteboard, and local-first multilingual messaging into a single workspace. Core text translation executes locally on user hardware across **English (`en`)**, **Urdu (`ur`)**, **Spanish (`es`)**, and **French (`fr`)**—eliminating context switching and third-party translation API bottlenecks.

---

## Key Features

* **Peer-to-Peer Conferencing & Screen Sharing:** Full-duplex audio/video calling and native desktop display or window capture via WebRTC with dynamic bitrate adjustment, bypassing central media servers for low latency.
* **Meeting Governance & RBAC:** Host vs. attendee Role-Based Access Control enforced over FastAPI WebSockets, supporting waiting room admission, remote mute/video toggle, screen-share locks, participant ejection, and session termination.
* **Collaborative Vector Whiteboard:** Embedded vector canvas serializing stroke paths, colors, and shape primitives into lightweight JSON coordinate deltas broadcast in real time.
* **Edge-First Translation Engine:** Quantized neural translation models executed locally via ONNX Runtime, paired with local idiom and slang lexicon packs for contextual phrasing across `en`, `ur`, `es`, and `fr`.
* **Translated Voice Notes:** Local audio capture, speech-to-text transcription, and cross-lingual translation into each recipient's target language.
* **Local-First Storage & Ephemeral Offline Sync:** Client-side SQLite serves as the single source of truth using deterministic `origin_uuid` deduplication for forwarded messages, backed by a 20-day TTL Firebase Firestore store-and-forward buffer that purges cloud copies immediately upon client sync.

---

## System Architecture

PolyMeet Studio separates media transport, control signaling, and message persistence across a dual-channel design:

1. **Media Plane (WebRTC):** Encrypted peer-to-peer audio, video, and desktop screen-sharing streams flow directly between desktop clients.
2. **Control & Sync Plane (FastAPI + WebSockets + Firestore):** A containerized FastAPI backend handles WebRTC SDP/ICE signaling, RBAC governance frames, and whiteboard delta broadcasts. Online messages route directly over WebSockets; offline recipient payloads queue temporarily in Firebase Firestore with a strict 20-day Time-To-Live (TTL) auto-purge policy.
3. **Client Persistence & Edge AI (Flutter Desktop + SQLite + ONNX Runtime):** The desktop client stores all messages locally in SQLite (indexed by `origin_uuid` to prevent duplicate rows on forwards) and runs quantized ONNX translation models and linguistic asset packs locally.

---

## Tech Stack

| Layer | Technology |
| :--- | :--- |
| **Desktop Client** | Flutter Desktop (Windows 11 / Linux), Dark Mode UI (Maroon & Black accents) |
| **Real-Time Media** | WebRTC (P2P Audio, Video, Native Screen/Window Capture) |
| **Backend & Signaling** | Python, FastAPI, WebSockets, Docker |
| **Local Database** | SQLite (`sqflite_common_ffi`) with `origin_uuid` idempotency |
| **Ephemeral Sync Buffer** | Firebase Firestore (20-day TTL auto-purge for offline delivery) |
| **On-Device AI** | ONNX Runtime (Quantized translation models + local lexicon packs) |

---

## Core Modules & Ownership

| Module | Description | Owner |
| :--- | :--- | :--- |
| **1. Real-Time Conferencing & Screen Sharing** | WebRTC P2P audio/video calls and native display/window screen sharing with adaptive bitrate. | Abdullah Naeem & Muhammad Zaid |
| **2. Audio Message Processing & Translation** | Local voice note capture, speech-to-text transcription, and target-language translation. | Abdullah Naeem & Muhammad Zaid |
| **3. On-Device Translation Engine** | Quantized ONNX Runtime inference pipeline with contextual lexicon packs (`en`, `ur`, `es`, `fr`). | Abdullah Naeem |
| **4. Interactive Vector Whiteboard** | Multi-user vector canvas broadcasting real-time stroke and shape coordinate deltas over WebSockets. | Abdullah Naeem |
| **5. Language Configuration & Asset Management** | Client-side language preferences, dynamic asset retrieval, and runtime loading of quantized models. | Abdullah Naeem |
| **6. Meeting Governance & Control Engine** | WebSocket-coordinated RBAC controls (waiting room, mute, video toggle, screen lock, kick, end call). | Muhammad Zaid |
| **7. Conversation & Local-First Messaging** | SQLite message store with deterministic `origin_uuid` checks to prevent forward duplication. | Muhammad Zaid |
| **8. Backend Orchestration & Ephemeral Sync** | Containerized FastAPI signaling/routing server and 20-day TTL Firestore store-and-forward queue. | Muhammad Zaid |

---

## Repository Structure (Logical Layout)

```text
polym-studio/
├── client/                     # Flutter Desktop app (Windows 11 & Linux)
│   ├── lib/
│   │   ├── core/               # Theme (dark/maroon), SQLite DB & origin_uuid deduplication
│   │   ├── features/
│   │   │   ├── conferencing/   # WebRTC peer connections & native screen capture
│   │   │   ├── governance/     # Host RBAC controls & waiting room state
│   │   │   ├── messaging/      # Local-first chat threads & Firestore sync worker
│   │   │   ├── translation/    # ONNX Runtime inference & lexicon pack loader
│   │   │   ├── voice_notes/    # Audio recording, ASR transcription & translation
│   │   │   └── whiteboard/     # Vector canvas & WebSocket coordinate delta engine
│   └── assets/models/          # Quantized ONNX models & en/ur/es/fr lexicon packs
├── server/                     # Containerized FastAPI backend
│   ├── app/
│   │   ├── api/                # REST endpoints & room management
│   │   ├── ws/                 # WebRTC signaling, RBAC events & whiteboard broadcaster
│   │   └── sync/               # Ephemeral Firestore store-and-forward dispatcher
│   └── Dockerfile
└── docs/                       # SRS, architecture diagrams & evaluation benchmarks
```
