# Secure Voice Chat for Multiplayer Games

> Research compiled: March 18, 2026  
> Sources: WebRTC for the Curious, Ant Media Server, Mumble, LiveKit, Agora, academic papers

---

## Table of Contents

1. [WebRTC for In-Game Voice Chat](#1-webrtc-for-in-game-voice-chat)
2. [End-to-End Encryption for Game Voice Chat](#2-end-to-end-encryption-for-game-voice-chat)
3. [Open-Source Voice Chat Solutions](#3-open-source-voice-chat-solutions)
4. [Spatial Audio and Proximity Voice Chat](#4-spatial-audio-and-proximity-voice-chat)
5. [Push-to-Talk vs. Voice Activation](#5-push-to-talk-vs-voice-activation)
6. [Moderation and Safety Features](#6-moderation-and-safety-features)
7. [Performance Impact of Voice Chat](#7-performance-impact-of-voice-chat)
8. [Summary and Architecture Recommendations](#8-summary-and-architecture-recommendations)

---

## 1. WebRTC for In-Game Voice Chat

### 1.1 What is WebRTC

WebRTC (Web Real-Time Communication) is a browser/application framework for peer-to-peer audio, video, and data communication. It is the underlying technology in Discord, Google Meet, and increasingly in game voice chat systems.

**Key properties**:
- Every WebRTC connection is **authenticated and encrypted** (mandatory, not optional)
- Works over UDP (low-latency), with fallback to TCP via TURN servers
- Built-in congestion control, jitter buffering, and echo cancellation
- Supported natively in browsers (Chrome, Firefox, Safari, Edge); native SDKs available for C++, Go, Rust, Unity

### 1.2 WebRTC Architecture

```
Player A ──────────────── STUN Server ──────────────── Player B
    │                  (ICE candidate                     │
    │                   discovery)                        │
    │                                                     │
    └─────────── TURN Server (relay fallback) ────────────┘
                                                          
Signaling Server (separate, e.g., WebSocket)
    └── Exchanges SDP offers/answers and ICE candidates
```

**Components**:
- **ICE (Interactive Connectivity Establishment)**: Discovers the best network path between peers
- **STUN (Session Traversal Utilities for NAT)**: Helps peers discover their public IP/port; Google provides free STUN: `stun:stun.l.google.com:19302`
- **TURN (Traversal Using Relays around NAT)**: Relay server for cases where P2P fails (symmetric NAT); higher latency but guaranteed connectivity
- **SDP (Session Description Protocol)**: Describes media capabilities; exchanged via signaling server

### 1.3 Connection Flow

From [WebRTC for the Curious — Securing](https://webrtcforthecurious.com/docs/04-securing/) and [Infobip WebRTC Guide](https://www.infobip.com/blog/webrtc-guide):

```
1. Player A creates RTCPeerConnection
2. Player A calls getUserMedia() → gets microphone stream
3. Player A creates SDP offer → sends via signaling server (WebSocket)
4. Player B receives offer → creates answer → sends back
5. ICE candidates exchanged → best path selected
6. DTLS handshake → SRTP session established
7. Audio flows encrypted P2P (or via TURN relay)
```

**Minimal JavaScript implementation** (from [DEV Community WebRTC tutorial](https://dev.to/eyitayoitalt/develop-a-video-chat-app-with-webrtc-socketio-express-and-react-3jc4)):
```javascript
const configuration = {
    iceServers: [
        { urls: ["stun:stun1.l.google.com:19302", "stun:stun2.l.google.com:19302"] }
    ],
    iceCandidatePoolSize: 10,
};

// Caller
const pc = new RTCPeerConnection(configuration);
localStream.getTracks().forEach(track => pc.addTrack(track, localStream));
const offer = await pc.createOffer();
await pc.setLocalDescription(offer);
socket.emit("message", { type: "offer", sdp: offer.sdp });

// Callee
pc.ontrack = (e) => remoteAudio.srcObject = e.streams[0];
await pc.setRemoteDescription(offer);
const answer = await pc.createAnswer();
await pc.setLocalDescription(answer);
socket.emit("message", { type: "answer", sdp: answer.sdp });
```

### 1.4 Scalable Multi-Player: SFU Architecture

For games with 4–100+ players, peer-to-peer doesn't scale (each player would need N-1 connections). The solution is an **SFU (Selective Forwarding Unit)**:

```
Player A ──┐
Player B ──┼──── SFU Server ──┬── Player A
Player C ──┤                   ├── Player B
Player D ──┘                   ├── Player C
                               └── Player D
```

- Each player sends one stream to the SFU
- SFU forwards relevant streams to each subscriber
- Server can selectively forward based on proximity (proximity chat)
- **LiveKit** is the leading open-source SFU implementation

**[LiveKit](https://livekit.com)** (from [LiveKit GitHub](https://github.com/livekit/livekit)):
- Open source, written in Go using Pion WebRTC
- Features: speaker detection, simulcast, selective subscription, end-to-end encryption, webhooks, multi-region
- SDKs: Unity, Unity WebGL, iOS, Android, React Native, Rust, Node.js, Python, Flutter
- Supports JWT authentication
- Can be self-hosted or used as managed service

---

## 2. End-to-End Encryption for Game Voice Chat

### 2.1 Default WebRTC Encryption: DTLS-SRTP

WebRTC mandates encryption via two protocols (from [WebRTC for the Curious](https://webrtcforthecurious.com/docs/04-securing/) and [Ant Media WebRTC Security Guide](https://antmedia.io/webrtc-security/)):

**DTLS (Datagram Transport Layer Security)**:
- TLS for UDP — same algorithms as HTTPS but over unreliable transport
- Handles the key exchange between peers
- Handshake phases: ClientHello → ServerHello → Certificate exchange → Diffie-Hellman key generation → ChangeCipherSpec → Finished
- Generates the Pre-Master Secret via Diffie-Hellman
- Derives Master Secret via PRF(Pre-Master Secret + client/server randoms)
- Certificates verified against fingerprints in the SDP — prevents MITM

**SRTP (Secure Real-time Transport Protocol)**:
- Specifically designed for encrypting RTP media packets
- No independent handshake — initialized with keys exported from DTLS (RFC 5705)
- Nonce = RTP Sequence Number + rollover counter → unique ciphertext per packet
- Provides: confidentiality, integrity (MAC), anti-replay

```
Flow:
ICE → DTLS Handshake → keys exported → SRTP session initialized → encrypted audio
```

**Limitation**: In SFU architecture, DTLS-SRTP encrypts between client and SFU — the SFU decrypts and re-encrypts. The SFU operator can access unencrypted audio. Acceptable for most games but not for maximum privacy.

### 2.2 True End-to-End Encryption: WebRTC Insertable Streams

From [WebRTCHacks — True E2EE with WebRTC Insertable Streams](https://webrtchacks.com/true-end-to-end-encryption-with-webrtc-insertable-streams/):

**Insertable Streams** (also called Encoded Transform) allows JavaScript to intercept, encrypt, and decrypt audio frames before/after the WebRTC stack:

```javascript
// Sender: encrypt before sending
const sender = pc.getSenders()[0];
const {readable, writable} = sender.createEncodedStreams();
readable.pipeThrough(new TransformStream({
    transform: (encodedFrame, controller) => {
        // Apply additional encryption layer
        encryptFrame(encodedFrame, e2eKey);
        controller.enqueue(encodedFrame);
    }
})).pipeTo(writable);
```

This creates true E2EE even through an SFU — the SFU forwards encrypted frames it cannot decrypt.

**LiveKit implementation**: LiveKit natively supports E2EE via Insertable Streams. Enable with:
```javascript
const room = new Room({ e2ee: { keyProvider: new ExternalE2EEKeyProvider() } });
```

### 2.3 Key Exchange for E2EE

For game voice chat, the key exchange model depends on trust:

| Model | Key Storage | Trust | Complexity |
|-------|-------------|-------|-----------|
| Server-managed | Server holds keys | Trust the server | Low |
| ECDH per-session | Keys never leave clients | Zero server trust | Medium |
| Pre-shared key (lobby) | Lobby creator distributes | Trust lobby host | Low |
| Signal Protocol | Ratcheting, forward secrecy | Full E2EE | High |

**Recommended for games**: Per-session ECDH with lobby-based key distribution:
1. Each player generates an ephemeral ECDH key pair on session join
2. Public keys exchanged via secure game server (signed with player's long-term key)
3. Symmetric session key derived via ECDH → used as E2EE key for Insertable Streams

---

## 3. Open-Source Voice Chat Solutions

### 3.1 Mumble Protocol

**[Mumble](https://www.mumble.info)** — Free, open source, low latency, high quality voice chat:

**Key features**:
- Low latency — was the first VoIP app to establish true low-latency voice communication
- Always encrypted (TLS/DTLS + SRTP)
- Public/private key authentication by default
- Positional audio (hear players from their in-game location)
- Extensive ACL (Access Control List) permission system
- Extensible via Ice protocols, web interfaces, and authenticators
- In-game overlay showing who is talking, FPS, current time

**Protocol**:
- Transport: TLS over TCP for control, DTLS over UDP for voice
- Codec: Originally Speex; modern versions use **Opus**
- Server software: `murmur` (also called `mumble-server`)
- Latest stable release: **v1.5.857** (October 10, 2025) from [Mumble Downloads](https://www.mumble.info/downloads/)

**Game integration options**:
- Link plugin: reads player position from game memory → sends to Mumble → triggers positional audio
- Positional audio plugins available for many games
- Open source → can embed Mumble server into a game server executable

**Community adoption**: Used by Eve Online for 100+ simultaneous voice participants, competitive TF2 community, and various enterprise environments.

### 3.2 Opus Codec

From [Opus codec official site](http://opus-codec.org) and [Vodlix Opus guide](https://vodlix.com/blog/opus-audio-codec):

**Opus** is the IETF standard audio codec (RFC 6716), combining Skype's SILK codec and Xiph.Org's CELT codec:

| Specification | Value |
|--------------|-------|
| Bitrate range | 6 kb/s to 510 kb/s |
| Sample rates | 8, 12, 16, 24, 48 kHz |
| Frame sizes | 2.5, 5, 10, 20, 40, 60 ms |
| Algorithmic latency | 26.5 ms (at 20ms frame) |
| License | Totally open, royalty-free |
| IETF standard | RFC 6716 |
| Current version | 1.5.2 (with ML-based features) |

**Use cases**: VoIP, videoconferencing, in-game chat, remote live music, streaming

**Game voice chat settings**:
```
Recommended for game voice:
    bitrate: 32–64 kbps (voice), 96–128 kbps (music)
    frame_size: 20 ms (good latency/quality balance)
    fec: true (Forward Error Correction for packet loss)
    complexity: 5 (balance of CPU and quality)
```

**Used in**: Discord, WhatsApp, Zoom, WebRTC (browser default), Unreal Engine 4/5 (preferred codec per [Reddit UE4 discussion](https://www.reddit.com/r/paragon/comments/4em8t5/opus_is_a_codec_used_for_voice_communication_its/))

### 3.3 LiveKit (Open Source WebRTC SFU)

From [LiveKit GitHub](https://github.com/livekit/livekit):

```
Server: Written in Go (Pion WebRTC)
License: Apache 2.0
Deployment: Single binary, Docker, Kubernetes

Features:
├── Scalable SFU (Selective Forwarding Unit)
├── Built-in speaker detection
├── Simulcast support
├── End-to-end encryption
├── Selective subscription (proximity-based)
├── Moderation APIs
├── JWT authentication
├── Webhooks for server events
└── Multi-region support

Unity SDK: Available (regular + WebGL)
```

**Why LiveKit for games**: The selective subscription feature lets you implement proximity chat server-side — only forward audio streams between players within range. This reduces bandwidth and CPU for large lobbies.

### 3.4 Other Options

| Solution | Type | License | Game Engine Support |
|---------|------|---------|---------------------|
| **Agora Voice SDK** | Managed service | Commercial | Unity, Unreal, Web |
| **Vivox** (Unity Gaming Services) | Managed service | Commercial | Unity-native |
| **Photon Voice** | Managed service | Commercial | Unity |
| **Discord GameSDK** (deprecated) | Managed service | Free-tier | Multiple |
| **Mumble** | Self-hosted | GPL v2 | Via plugin/integration |
| **LiveKit** | Self-hosted or cloud | Apache 2.0 | Unity SDK available |

---

## 4. Spatial Audio and Proximity Voice Chat

### 4.1 What Is Spatial Audio in Voice Chat

Spatial (3D) audio applies **HRTF (Head-Related Transfer Function)** processing to make voices appear to come from specific 3D positions relative to the listener's head, matching the speaker's in-game position.

From [Tencent RTC — Implementing 3D Voice in Games](https://trtc.io/blog/details/implementing-3d-voice-in-games):

**Processing pipeline for each remote voice stream**:
```
Receive voice packet (N-1 streams for N players)
    ↓
Extract voice + position data (received every ~20ms)
    ↓
HRTF filtering based on relative position (azimuth, elevation)
    ↓
Distance attenuation model (volume ∝ 1/distance² or custom curve)
    ↓
Obstacle occlusion (optional — muffles voice through walls)
    ↓
Environmental reverb (indoor/outdoor acoustic profile)
    ↓
Mix to stereo/binaural output
```

**Performance considerations**: HRTF computation is CPU-intensive. For N players, each frame requires processing (N-1) voice streams. This can strain low/mid-end devices. Solutions:
- Progressive HRTF: full quality for nearest 5 players, simplified for others
- LOD (Level of Detail) for audio: simplified panning (left/right only) at large distances

### 4.2 Proximity Voice Chat Design

**Proximity-only**: Players only hear others within a configurable radius.

**Volume formula** (from [Agora spatial audio tutorial — YouTube](https://www.youtube.com/watch?v=uM89bDIrmZ0)):
```
volume = max(0, (1 - distance / radius) * 100)

// At distance = 0:      volume = 100% (full volume)
// At distance = radius: volume = 0%   (inaudible)
// Beyond radius:         volume = 0%   (not transmitted)
```

**Selective forwarding for proximity** (LiveKit approach):
- Publish player position updates to the SFU
- SFU only forwards voice streams between players within proximity radius
- Reduces bandwidth: 100-player server only routes ~6–8 streams per player instead of 99

**Zone-based proximity** (common in large games):
```
ProximityRadius = {
    WHISPER:     2m  (very quiet, team-only)
    LOCAL:      15m  (nearby players)
    SHOUT:      50m  (loud voice / megaphone)
    GLOBAL:     ∞   (server-wide, requires special permission)
}
```

### 4.3 Games With Best Proximity Chat

From [HowToGeek list](https://www.howtogeek.com/games-with-the-best-proximity-chat/) and [Steam curator](https://store.steampowered.com/curator/45899036-Games-With-Proximity-Voice-Chat/):

| Game | Implementation Notes |
|------|---------------------|
| **Hell Let Loose** | Layered: squad proximity + command channel + party chat |
| **Lethal Company** | Core gameplay mechanic — warnings based on proximity |
| **Phasmophobia** | Local (proximity) + radio (unlimited range) + ghost can hear |
| **Content Warning** | Voice chat recorded in-game as video content |
| **Sea of Thieves** | Nautical realism — across-ship communication |
| **Call of Duty Warzone** | Cross-team proximity (enemies can hear each other) |
| **Among Us** | Mod-based; affects strategy when impostor is nearby |

### 4.4 Cross-Team Proximity Chat

Some games allow enemies to hear each other's proximity chat (Warzone, Insurgency). This requires:
- No channel separation in the proximity radius
- Potentially offensive — needs clear player expectation-setting
- Can be toggled per-server or per-game mode

---

## 5. Push-to-Talk vs. Voice Activation

### 5.1 Comparison

| Aspect | Push-to-Talk (PTT) | Voice Activation (VAD) |
|--------|--------------------|----------------------|
| Privacy | Better — no accidental transmission | Lower — background noise may transmit |
| Usability | Requires button binding | Hands-free |
| Bandwidth | Lower — only transmits when pressed | Higher — may transmit continuously |
| Noise pollution | None when not pressed | Background noise can disrupt others |
| Gameplay impact | Requires key binding (can conflict with WASD/mouse) | None (but hands-free advantage) |
| Common use | Competitive games, FPS | Casual games, tabletop RPGs |

### 5.2 Recommended Defaults

- **Default: Push-to-Talk** for competitive games (from [HackerNews community consensus](https://news.ycombinator.com/item?id=17953765): "at worst it's actively disrupting people's gameplay and team communication")
- **Voice activation** acceptable for casual/social games with good noise gate tuning

### 5.3 Implementation

```
PTT Key Binding:
    - Store configurable key/button binding
    - On press: open microphone, set tx_active = true
    - On release: close microphone, set tx_active = false
    - Send audio only when tx_active = true

VAD Implementation (Voice Activity Detection):
    - Short-time energy threshold (adjustable sensitivity)
    - Or WebRTC built-in VAD: RTCPeerConnection with voiceActivityDetection: true
    - Hold-time: keep microphone open 0.3s after voice stops (prevents clipping)
```

**PTT key binding recommendations**:
- Default: `Left Alt`, `Caps Lock`, or mouse side button (doesn't conflict with WASD)
- Allow remapping in settings
- Visual indicator in HUD when PTT is active (microphone icon)

### 5.4 Noise Suppression

Both modes benefit from pre-processing:
- **Echo Cancellation (AEC)**: Remove speaker output from microphone input
- **Noise Suppression**: Filter background noise (fans, keyboards, ambient)
- **Automatic Gain Control (AGC)**: Normalize volume across players

WebRTC provides these natively:
```javascript
navigator.mediaDevices.getUserMedia({
    audio: {
        echoCancellation: true,
        noiseSuppression: true,
        autoGainControl: true
    }
});
```

Disabling these features reduces CPU usage (see [Fatshark Forums voice stutter report](https://forums.fatsharkgames.com/t/game-stutters-whenever-i-push-to-talk-someone-joins-voice-chat/48658)) but may reduce audio quality.

---

## 6. Moderation and Safety Features

### 6.1 AI-Powered Voice Moderation (2025–2026)

From [GGWP voice moderation blog](https://www.ggwp.com/blog/adding-ai-powered-voice-moderation-tools-to-your-multiplayer-game/) and [cryo.com gaming toxicity prevention](https://www.cryo.com/gaming-toxicity-prevention-news-today-ai-powered-moderation-tools-reshape-digital-security/):

Modern voice moderation uses AI to monitor voice chat in real-time:
- **Speech-to-text transcription**: Convert voice to text for NLP analysis
- **Sentiment analysis**: Detect escalating disputes
- **Tone analysis**: Identify aggressive or threatening speech patterns
- **Pattern matching**: Flag hate speech, slurs, threats using trained classifiers

**Reported accuracy metrics**:
- Riot Games: 85% accuracy identifying abusive content before human review; processes 100M+ interactions/day across LoL and Valorant
- Activision: 32% reduction in harassment reports; processes ~5M voice chat minutes/day; automated action in under 3 seconds
- EA: 8% false positive rate; context-aware (distinguishes friendly banter from genuine harassment)

**Games with 40–60% reduction in harassment** after implementing AI voice moderation, with 45% better new user retention and 52% longer average play times (from [cryo.com](https://www.cryo.com/gaming-toxicity-prevention-news-today-ai-powered-moderation-tools-reshape-digital-security/)).

### 6.2 Tools Available to Developers

| Tool | Type | Integration |
|------|------|-------------|
| **GGWP** | Managed AI moderation | SDK/API, real-time |
| **Vbox (mentioned in YouTube demo)** | AI voice moderation | Mute/ban actions out of box |
| **Speechify / Hive** | Speech-to-text + classifier | API-based |
| **Riot Vanguard** | Proprietary (not licensable) | — |
| **Microsoft Azure AI Content Safety** | Cloud API | Real-time speech analysis |

From [ACM Digital Library — Towards Ethical AI Moderation in Multiplayer Games](https://dl.acm.org/doi/10.1145/3677109): AI moderation research notes ethical concerns around:
- Privacy implications of always-on voice monitoring
- False positive rates and unfair penalties
- Cultural/linguistic bias in training data
- Balance between automation and human review

### 6.3 Basic Moderation Features to Implement

**Minimum viable moderation set**:

```
Voice Moderation System:
    ├── Mute (local): Player mutes specific other players (client-side only)
    ├── Block: Player permanently mutes + prevents seeing another player
    ├── Report: Flag voice clip for review (save 30s buffer on report)
    ├── PTT default: Reduces incidental noise/harassment
    ├── Voice Volume: Per-player volume sliders
    └── Safe Mode: Proximity chat only (no global channels for new accounts)
```

**Server-side tools**:
- Admin mute/ban from voice channels
- Time-limited communication bans
- Voice chat logs (hash/encrypt for privacy; retain for appeals)
- Channel permissions (separate channels for team vs. all-chat)

### 6.4 Privacy Compliance

**GDPR/CCPA considerations** for voice recording:
- Explicit consent required before recording voice data
- Right to deletion: purge voice logs within 30 days (or on user request)
- Data minimization: only retain what's needed for moderation
- No biometric voiceprint storage without explicit consent

---

## 7. Performance Impact of Voice Chat

### 7.1 CPU Usage

Voice processing components and their CPU costs:

| Component | CPU Impact | Notes |
|-----------|-----------|-------|
| Opus encode/decode | Low (< 1%) | Highly optimized; hardware-accelerated |
| WebRTC base | Low–Medium (1–5%) | Depends on N connections |
| Echo cancellation (AEC) | Medium (2–5%) | Can be disabled to save CPU |
| Noise suppression | Medium (3–8%) | Most expensive processing step |
| HRTF spatial audio | Medium–High (5–15% per stream) | Scales with number of players |
| Speech-to-text (server) | High (server-side) | Cloud-based, no client impact |

**Agora SDK**: Claims 20% lower CPU/storage/battery than industry average (from [Agora gaming page](https://www.agora.io/en/solutions/gaming/))

**Steam voice chat high CPU issue** (from [Steam Community discussion](https://steamcommunity.com/discussions/forum/1/3183488224881665877/)): Some users report 50% CPU spikes from Steam's voice chat due to UI animation rendering conflicts. Root cause is typically: WebHelper processes rendering animated UI elements while audio processing occurs simultaneously. Fix: disable animated avatars/frames in Steam settings.

### 7.2 Network Bandwidth

| Configuration | Bandwidth per player |
|--------------|---------------------|
| Opus voice (32 kbps) | ~4 KB/s |
| Opus voice (64 kbps) | ~8 KB/s |
| WebRTC overhead | +1–2 KB/s per stream |
| 10-player proximity | ~40–80 KB/s receive |
| 50-player global | ~200–400 KB/s receive |

**Optimization**: SFU selective forwarding reduces receive bandwidth to only nearby players. A 100-player lobby with 10-player proximity radius reduces receive bandwidth from 990 KB/s to ~40–80 KB/s.

### 7.3 Latency

Voice chat latency budget:

```
Microphone → Capture:        5–10ms
Opus encoding (20ms frame):  20ms
Network transit:             20–100ms (P2P) / 30–60ms (SFU)
Jitter buffer:               20–80ms
Opus decoding:               <1ms
Speaker output:              5–10ms

Total:                       70–230ms typical
```

Mumble achieves < 50ms total latency under ideal conditions, making it the benchmark for game voice chat.

### 7.4 Thread Architecture Recommendation

To avoid audio stutters:

```
Main Game Thread:       Rendering, physics, game logic
    ↕ lock-free ring buffer
Audio Capture Thread:   Microphone → Opus encode → send buffer
    ↕ separate thread
Network Thread:         Send/receive UDP packets
    ↕ lock-free ring buffer
Audio Playback Thread:  Receive buffer → Opus decode → HRTF → output
```

**Key rule**: Never block the audio thread with locks held by the main game thread. This is the root cause of most "push-to-talk stutter" bugs.

---

## 8. Summary and Architecture Recommendations

### 8.1 Recommended Stack by Game Type

| Game Type | Recommended Solution | Notes |
|-----------|---------------------|-------|
| Small indie (2–8 players) | WebRTC P2P + Opus | Simple, low-cost, good quality |
| Medium multiplayer (8–50) | LiveKit (self-hosted) + Opus | Free, open source, proximity support |
| Large multiplayer (50–1000) | LiveKit / Agora + SFU | SFU required at scale |
| Mobile-focused | Agora Voice SDK | Optimized for mobile battery/CPU |
| Maximum privacy | LiveKit + E2EE Insertable Streams | True E2EE through server |
| Moderated community | Agora / Vivox + GGWP | Managed moderation pipeline |

### 8.2 Reference Architecture for a Multiplayer Racing Game

```
┌─────────────────────────────────────────────────────────────┐
│  VOICE CHAT SYSTEM                                          │
│                                                             │
│  Player Client                                              │
│  ├── Microphone → Opus Encoder (20ms frames, 32 kbps)       │
│  ├── PTT Key Handler (default: Left Alt)                    │
│  ├── VAD fallback (threshold-based)                         │
│  ├── WebRTC Connection Manager → LiveKit SFU               │
│  ├── Spatial Audio Engine                                   │
│  │       ├── HRTF processor (top N nearest players)         │
│  │       ├── Distance attenuation (0→radius)                │
│  │       └── Obstacle occlusion (optional)                  │
│  └── Voice Moderation                                       │
│          ├── Local mute/block list                          │
│          └── Report button → server                        │
│                                                             │
│  LiveKit SFU Server                                         │
│  ├── Position-based selective forwarding                    │
│  ├── Channel management (team vs. all-chat)                │
│  ├── JWT authentication                                     │
│  └── Webhook → moderation API                              │
│                                                             │
│  Security: DTLS-SRTP (default) or Insertable Streams E2EE  │
│  Codec: Opus, 32–64 kbps, 20ms frames, FEC enabled         │
└─────────────────────────────────────────────────────────────┘
```

### 8.3 Implementation Priority

```
Phase 1: Basic P2P voice (WebRTC + Opus, PTT, DTLS-SRTP)
Phase 2: SFU (LiveKit) for >8 players
Phase 3: Proximity/spatial audio (distance attenuation + selective forwarding)
Phase 4: HRTF 3D spatial audio
Phase 5: E2EE via Insertable Streams
Phase 6: AI voice moderation integration
```

### 8.4 Performance Budget

For a racing game targeting 60 FPS:

| Component | CPU Budget |
|-----------|-----------|
| Voice encode/decode | 1–2% |
| Network I/O thread | 0.5% |
| Echo cancellation | 2–3% |
| Spatial audio (8 players) | 3–5% |
| **Total voice chat budget** | **~7–11%** |

Keep voice processing off the main game thread. Use lock-free ring buffers between threads. Monitor with profiler — the most common issue is noise suppression overhead when enabled.

---

## Sources

| Source | URL |
|--------|-----|
| WebRTC for the Curious — Securing | https://webrtcforthecurious.com/docs/04-securing/ |
| Ant Media WebRTC Security Guide | https://antmedia.io/webrtc-security/ |
| True E2EE with WebRTC Insertable Streams — WebRTCHacks | https://webrtchacks.com/true-end-to-end-encryption-with-webrtc-insertable-streams/ |
| DEV Community WebRTC Tutorial | https://dev.to/eyitayoitalt/develop-a-video-chat-app-with-webrtc-socketio-express-and-react-3jc4 |
| Infobip WebRTC Guide | https://www.infobip.com/blog/webrtc-guide |
| LiveKit — Open Source WebRTC SFU | https://livekit.com |
| LiveKit GitHub Repository | https://github.com/livekit/livekit |
| Mumble Official Website | https://www.mumble.info |
| Mumble Downloads | https://www.mumble.info/downloads/ |
| Opus Codec Official Site | http://opus-codec.org |
| Opus Codec — Vodlix Guide | https://vodlix.com/blog/opus-audio-codec |
| Agora Gaming Solutions | https://www.agora.io/en/solutions/gaming/ |
| Agora Community Building Blog | https://www.agora.io/en/blog/building-community-around-single-player-games/ |
| HowToGeek — Games With Best Proximity Chat | https://www.howtogeek.com/games-with-the-best-proximity-chat/ |
| Steam — Games With Proximity Voice Chat | https://store.steampowered.com/curator/45899036-Games-With-Proximity-Voice-Chat/ |
| Tencent RTC — 3D Voice in Games | https://trtc.io/blog/details/implementing-3d-voice-in-games |
| Agora Spatial Audio Unity Tutorial — YouTube | https://www.youtube.com/watch?v=uM89bDIrmZ0 |
| GGWP AI Voice Moderation Blog | https://www.ggwp.com/blog/adding-ai-powered-voice-moderation-tools-to-your-multiplayer-game/ |
| Gaming Toxicity Prevention — cryo.com (2026) | https://www.cryo.com/gaming-toxicity-prevention-news-today-ai-powered-moderation-tools-reshape-digital-security/ |
| ACM — Ethical AI Moderation in Multiplayer Games | https://dl.acm.org/doi/10.1145/3677109 |
| Fatshark Forums — Voice Chat Stutter | https://forums.fatsharkgames.com/t/game-stutters-whenever-i-push-to-talk-someone-joins-voice-chat/48658 |
| Steam Community — Voice Chat High CPU | https://steamcommunity.com/discussions/forum/1/3183488224881665877/ |
| Opus codec — Unreal Engine 4 (Reddit) | https://www.reddit.com/r/paragon/comments/4em8t5/opus_is_a_codec_used_for_voice_communication_its/ |
| HackerNews — PTT vs Voice Activation | https://news.ycombinator.com/item?id=17953765 |
