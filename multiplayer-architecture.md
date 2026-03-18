# Multiplayer Architecture for Racing Games

> Comprehensive research on networking architectures, netcode techniques, lobby systems, anti-cheat strategies, and the SuperTuxKart reference implementation for racing game multiplayer.

---

## Table of Contents

1. [Client-Server vs P2P Architecture for Racing Games](#1-client-server-vs-p2p-architecture-for-racing-games)
2. [Netcode for Racing Games: State Sync, Lag Compensation, Interpolation](#2-netcode-for-racing-games-state-sync-lag-compensation-interpolation)
3. [Lobby Systems and Matchmaking](#3-lobby-systems-and-matchmaking)
4. [Split-Screen Local Multiplayer Implementation](#4-split-screen-local-multiplayer-implementation)
5. [WebSocket vs WebRTC vs WebTransport for Real-Time Game Networking](#5-websocket-vs-webrtc-vs-webtransport-for-real-time-game-networking)
6. [Dedicated Server Architecture vs Player-Hosted](#6-dedicated-server-architecture-vs-player-hosted)
7. [Anti-Cheat Considerations for Racing Games](#7-anti-cheat-considerations-for-racing-games)
8. [SuperTuxKart Networking Code and Protocol](#8-supertuxkart-networking-code-and-protocol)
9. [Synthesis: Recommended Architecture](#9-synthesis-recommended-architecture)

---

## 1. Client-Server vs P2P Architecture for Racing Games

### Architecture Overview

| Dimension | Peer-to-Peer (P2P) | Client-Server |
|-----------|-------------------|---------------|
| **Communication** | Clients talk directly to each other | All traffic routes through server |
| **Authority** | Distributed (trust one peer as host, or lockstep) | Server is the sole authority |
| **Cheating resistance** | Low — any client can manipulate data | High — server validates all actions |
| **DoS vulnerability** | High — peers know each other's IPs | Medium — server IP public, but mitigable |
| **Stability** | Dependent on host's connection and hardware | Dependent on server uptime |
| **Latency** | Low if peers are geographically close | Depends on server proximity |
| **Scalability** | Poor at high player counts | Good with dedicated infrastructure |
| **Cost** | Free (runs on player devices) | Requires server costs |
| **Complexity** | Lower for basic implementation | Higher (requires server management) |

Source: [Hathora Blog — Peer-to-peer vs client-server architecture](https://blog.hathora.dev/peer-to-peer-vs-client-server-architecture/)

### Why Racing Games Favor Client-Server

Racing games have specific characteristics that push toward **client-server**:

1. **Determinism is hard**: Physics simulations diverge rapidly across machines with different CPU speeds, floating-point behavior, and timesteps. A server authority resolves disagreements.
2. **Speed cheating**: P2P allows a malicious peer to report a teleporting car or impossible speed. Server validation constrains this.
3. **Position consistency**: Players need authoritative agreement on positions (especially for lap timing, item hits, and collision resolution at the finish line).
4. **Ghost racing**: For time trials and ghost cars, a server store is needed.

However, **pure P2P with lockstep** is used by some smaller racing games (including early STK experiments) where:
- All clients run identical physics simulations
- Only **inputs** are transmitted (not state), which is bandwidth-efficient
- Any divergence causes a desync and requires correction

P2P lockstep fails gracefully at player counts > ~4 and is highly sensitive to packet loss.

### Hybrid: "P2P with Server Authority" (Best of Both)

A common modern pattern: players connect **directly** for low-latency game data exchange, while a **relay/matchmaking server** handles session management and validation. This is essentially a lightweight client-server where the "server" only maintains game state without handling all real-time packets.

---

## 2. Netcode for Racing Games: State Sync, Lag Compensation, Interpolation

### The Fundamental Latency Problem

With typical internet latency (50–300ms RTT), what a client sees from the server is always stale. A car travelling at 100 km/h moves ~2.8–16.7m during that time — enough to cause visible teleporting without compensation.

### Core Techniques

#### Client-Side Prediction

The local player's input is applied **immediately** on the client, without waiting for server confirmation:

1. Player presses accelerate → car moves instantly on client
2. Input packet is sent to server
3. Server simulates physics, validates, sends back authoritative state
4. If client state ≠ server state → **reconciliation** (snap/blend back to server state)

This is the dominant approach for fast-paced racing. A detailed Unity implementation is documented in [this kart game netcode tutorial](https://www.youtube.com/watch?v=-lGsuCEWkM0) using Netcode for GameObjects:

- **NetworkTimer** class: tick-based (e.g., 60 Hz), frame-rate independent
- **Circular buffers**: client stores input and state snapshots per tick
- **Reconciliation**: if `|client_position - server_position| > threshold`, rewind to last confirmed server state and replay buffered inputs forward

#### Dead Reckoning / Interpolation for Remote Players

Other players' positions must be estimated between received network updates:

| Technique | Description | Error at 200ms / 100 km/h |
|-----------|-------------|--------------------------|
| **Snap** | Jump to new position when update arrives | ~5.6m jump per update |
| **Linear interpolation** | Smooth between last two known positions | Appears behind reality |
| **Dead reckoning** | Extrapolate using last known velocity/acceleration | ~2m average error |
| **Neural DR** (research) | LSTM predicts trajectory; path-tracking controller smooths | ~45% less error vs. classical DR |

Research from [University of Waterloo (2023 IEEE TOG paper)](https://ece.uwaterloo.ca/~sl2smith/papers/2023TOG-Predictive_Dead_Reckoning.pdf) on neural dead reckoning for car racing games:
- Classical DR at 400ms latency results in ~2m average error (roughly half a car length)
- Neural LSTM approach improves prediction accuracy by ~45%
- A path-tracking controller ensures smooth, physically plausible rendered trajectories

For games targeting typical consumer connections (50–100ms), classical dead reckoning is usually sufficient.

Classical approaches from [McGill University (Pantel & Wolf)](https://svn.sable.mcgill.ca/sable/courses/COMP763/oldpapers/pantel-02-suitability.pdf):
- **Scheme 1**: Constant velocity assumption (simplest)
- **Scheme 2/3**: Lagrange polynomial extrapolation (mathematically equivalent, better for acceleration changes)
- Racing games are particularly sensitive to prediction errors because high speeds mean large positional divergence at the finish line

#### Server-Side Rewind (Lag Compensation)

For racing games with **item throwing** or **weapons** (e.g., kart racing):

1. Client fires item at `time T` (seeing the world as it was at T)
2. Server receives the event at `T + RTT/2`
3. Server **rewinds its simulation** to time `T`, validates the hit against the historical position of the target
4. Result: player's aim "hits" what was on their screen, preventing frustrating misses

SuperTuxKart uses this for rubber band item accuracy and kart collision resolution.

### State Frequency and Bandwidth

[SuperTuxKart's documentation](https://github.com/supertuxkart/stk-code/blob/master/NETWORKING.md) confirms:
- Default **state frequency**: 10 updates/second (configurable)
- Higher frequency = more bandwidth + more rewind cost
- A Raspberry Pi 3B+ can host 8 players at ~60% CPU / 60MB RAM on heavy tracks

For a web-based racing game, a **20 Hz state rate** balances bandwidth and visual smoothness, with client prediction filling in between.

---

## 3. Lobby Systems and Matchmaking

### Architecture Components

A racing game lobby/matchmaking system requires three services:

| Service | Responsibility |
|---------|---------------|
| **Lobby service** | Persistent player connection, chat, ready status, player list |
| **Session service** | Store party state, handle mid-session changes, player connections/disconnections |
| **Matchmaking service** | Queue players/parties, group into matches based on rules (skill, region, mode) |

[AccelByte Gaming Services load-tested](https://accelbyte.io/blog/scaling-matchmaking-to-one-million-players) these three services to 1 million concurrent users (CCU) and validated the architecture handles the load.

### Lobby Implementation Patterns

From a [community game lobby implementation on GitHub (TypeScript/React/Node)](https://www.reddit.com/r/gamedev/comments/10td99x/how_do_online_games_implement_lobby_system/):
- Players can join/leave chat rooms, create/join games, select options, ready/unready, and trigger countdowns
- ELO-based matchmaking for ranked queues
- Region selection affects which matchmaking pool to join

**Key implementation concerns**:
- **Region handling**: players switch to the server region closest to the majority of their party
- **Error handling**: what happens if a server holding 20 players in lobbies crashes?
- **Third-party options**: Steam's lobby API (Steam-only), Photon, Nakama, AccelByte

### Nakama Lobby Pattern

[Heroic Labs Nakama](https://forum.heroiclabs.com/t/lobby-matchmaking/1180) supports both relayed (server-assisted P2P) and authoritative multiplayer matches. For a 4-player lobby where the host starts the game when ready:
1. Player creates a lobby (match with no game running)
2. Friends join by code/invite
3. Host initiates match — server spawns game server process
4. All clients connect to game server

This pattern separates **social/lobby infrastructure** (persistent, low-bandwidth) from **game server** (ephemeral, real-time).

---

## 4. Split-Screen Local Multiplayer Implementation

### The Rendering Challenge

Split-screen requires rendering the scene from **multiple camera viewpoints** per frame:

- 2 players: 2 renders per frame (each at ~half screen resolution)
- 4 players: 4 renders per frame (each at ~quarter screen resolution)

Modern performance concern: techniques like **occlusion culling** (don't render what's behind walls) are expensive to run per-view and don't scale linearly.

From the [r/gamedev split-screen discussion](https://www.reddit.com/r/gamedev/comments/ajiciq/split_screen_in_modern_gaming/):
- Simple frustum culling scales well with multiple views
- Occlusion culling systems often assume a single camera — need redesign for split-screen
- Games like Serious Sam handle 4-player split-screen well due to open geometry (less occlusion work)
- Left 4 Dead has hidden split-screen support but with shadow/reflection glitches

### Unreal Engine 5 Implementation

[Unreal Engine 5 split-screen guide](https://www.youtube.com/watch?v=SwRbU2aqiNA) confirms:
1. Enable **Use Split Screen** in Project Settings → Local Multiplayer
2. Configure layouts: 2P (horizontal/vertical), 3P (favor top/bottom/sides), 4P (grid)
3. Each player gets a separate camera and input device (gamepad mapping)
4. No separate render pass setup required — UE5 handles viewport subdivision automatically

### For WebGL/Three.js

In a browser context, split-screen is implemented with multiple `WebGLRenderer` viewports or multiple cameras rendering to different regions of the canvas using `renderer.setViewport()` and `renderer.setScissor()`:

```javascript
// Two player split-screen in Three.js
renderer.setScissorTest(true);

// Player 1 - left half
renderer.setViewport(0, 0, width/2, height);
renderer.setScissor(0, 0, width/2, height);
renderer.render(scene, camera1);

// Player 2 - right half
renderer.setViewport(width/2, 0, width/2, height);
renderer.setScissor(width/2, 0, width/2, height);
renderer.render(scene, camera2);
```

Performance concern: each camera requires a separate draw pass, roughly doubling render cost. Post-processing effects must also be duplicated per view.

---

## 5. WebSocket vs WebRTC vs WebTransport for Real-Time Game Networking

### Protocol Comparison

| Feature | WebSocket | WebRTC (Data Channel) | WebTransport |
|---------|-----------|-----------------------|--------------|
| **Underlying transport** | TCP | UDP (ICE/DTLS/SCTP) | QUIC (UDP-based) |
| **Architecture** | Client-server | Peer-to-peer | Client-server |
| **Latency** | Highest (TCP retransmissions) | Medium | Lowest |
| **Reliability** | Reliable only | Reliable or unreliable | Reliable or unreliable |
| **Encryption** | Optional (TLS/WSS) | Mandatory (DTLS/SRTP) | Mandatory (TLS 1.3) |
| **Scalability** | Simple broadcast | Complex for multi-party | Moderate |
| **Browser support** | Universal | Universal | Chrome 97+, Firefox 114+ |
| **Server complexity** | Low | High (STUN/TURN) | Moderate |

Sources: [videosdk.live comparison (2025)](https://www.videosdk.live/developer-hub/websocket/which-is-better-and-when-to-use-it-webrtc-or-websocket), [NSDI 2025 research paper](https://aaron.gember-jacobson.com/docs/nsdi2025browser-networking.pdf)

### NSDI 2025 Research: Protocol Benchmarks for Real-Time Games

The [NSDI 2025 paper "Evaluating Browser-Based Networking for Real-Time Multiplayer Games"](https://aaron.gember-jacobson.com/docs/nsdi2025browser-networking.pdf) ran a tick-based simulation (120 ticks/sec) measuring client-input-to-server-response latency:

**RTT distribution ranking (lower = better for players):**

| Rank | Protocol | 0% Loss | 0.1% Loss |
|------|----------|---------|-----------|
| 1st (best) | **WebTransport** | Best | Best |
| 2nd | UDP + DTLS | 2nd best | 2nd best |
| 3rd | **WebRTC** | 3rd | 3rd |
| 4th (worst) | **WebSocket** | Worst | Worst (TCP retransmissions most harmful) |

Key findings:
- **WebTransport** wins due to QUIC's BBRv1 congestion control optimized for low latency, plus no retransmission overhead for unreliable datagrams
- **WebRTC** performs better than WebSockets despite UDP semantics, because its complex protocol stack (ICE + DTLS + SCTP) adds processing overhead
- **WebSocket/TCP** retransmissions are "fruitless" for real-time ticks — by the time a retransmit arrives, the tick has already been missed
- **WebSocket** scales best for one-to-many broadcasting due to server infrastructure simplicity

### Recommendation for Racing Games

**For a browser-based racing game:**

| Use Case | Recommended Protocol |
|----------|---------------------|
| Real-time position sync (fast, lossy OK) | **WebTransport** (preferred) or WebRTC |
| Lobby chat, scores, race results (reliable) | **WebSocket** |
| P2P ghost car sync (two-player) | **WebRTC** (DataChannel, unreliable) |
| Server-authoritative game (standard) | **WebSocket** (simpler) or **WebTransport** (lower latency) |

A **hybrid approach** is optimal: WebSocket for lobby/matchmaking/reliable events; WebTransport or WebRTC DataChannel for real-time position updates.

---

## 6. Dedicated Server Architecture vs Player-Hosted

### Feature Comparison

| Feature | Player-Hosted (P2P/Host) | Dedicated Server |
|---------|--------------------------|-----------------|
| **Cost** | Free (runs on player device) | Ongoing server costs (~$5–15/mo for small scale) |
| **Availability** | Only when host is online | 24/7 |
| **Performance impact** | Host's game may run slower | No impact on any player's game |
| **Stability** | Fails if host disconnects | Isolated from player dropouts |
| **Security** | Host has game-state access | Server authority, clients are blind |
| **Customization** | Limited by host's client | Full server-side config, mods |
| **Cheating** | Host can cheat freely | Server enforces rules |

Source: [Servers.com dedicated vs P2P comparison](https://www.servers.com/blog/differences-between-peer-to-peer-and-dedicated-game-server-hosting), [Pine Hosting comparison](https://pinehosting.com/blog/dedicated-server-vs-peer-to-peer/)

### Dedicated Server for Racing Games

Racing-specific advantages of dedicated servers:
1. **Lap time integrity**: No player-hosted server means no host can freeze the timer or manipulate physics for their kart
2. **Anti-cheat**: Server-authoritative physics validation catches speed hacks
3. **Persistence**: Leaderboards, ghost data, and statistics survive across sessions
4. **Spectator mode**: Server can broadcast full game state to spectators without taxing any player's client

[SuperTuxKart's dedicated server](https://github.com/supertuxkart/stk-code/blob/master/NETWORKING.md) can run on a **Raspberry Pi 3B+** with 8 players at ~60% CPU. Compile with `-DSERVER_ONLY=ON` for a GUI-less, low-memory binary.

### Hathora Cloud Pattern

[Hathora Cloud](https://blog.hathora.dev/peer-to-peer-vs-client-server-architecture/) uses **globally distributed edge servers** allocated dynamically based on player geography. This solves the latency problem for client-server architecture: rather than one US server imposing 200ms+ ping for European players, servers are spun up in the nearest region automatically.

For a small racing game:
- **Development**: Player-hosted or small VPS (5 USD/mo) suffices
- **Production**: Managed services (Hathora, AccelByte, AWS GameLift) for auto-scaling and global distribution

---

## 7. Anti-Cheat Considerations for Racing Games

### Racing-Specific Cheat Vectors

| Cheat Type | How It Works | Server Mitigation |
|------------|--------------|------------------|
| **Speed hacking** | Client sends faster-than-possible speeds | Validate: speed ≤ `max_speed × physics_constant` |
| **Position teleporting** | Client reports impossible position jump | Validate: position delta ≤ `max_speed × tick_interval` |
| **Lap time manipulation** | Client reports false lap times | Server tracks lap start/end timestamps independently |
| **Item spoofing** | Client fires item it doesn't have | Server tracks item inventory; reject invalid fire events |
| **Wall hacking** | Client shows other players through walls | Cull non-visible state server-side (not relevant for racing) |
| **Rubber banding abuse** | Exploit catch-up physics | Server controls rubber-banding calculations |

From [Hacker News anti-cheat discussion](https://news.ycombinator.com/item?id=40631223):

> *"Fast-paced games like racing sims will do client-side determination with server-side validation (to ensure that a player's speed never exceeds a certain value, turn speed/friction is inside a certain range, etc.)."*

### Server-Authoritative Validation Pattern

The standard racing game anti-cheat approach:

1. **Client sends inputs** (steering angle, throttle, brake) — not positions
2. **Server simulates physics** from those inputs
3. **Server broadcasts authoritative positions** to all clients
4. Client applies the server's position if it deviates from prediction by more than a threshold

This makes position cheating nearly impossible: the server generates all authoritative positions from physics. Clients cannot inject false positions.

### Implementation Notes

- For kart items: server maintains a **random seed** per race. Item box contents are determined by server, not client
- For rubber banding: catch-up multipliers are server-computed based on actual server-simulated gaps
- Ban lists: SuperTuxKart supports **IP range (CIDR) ban lists** and **online ID bans** via SQLite, manageable without restarting the server
- [Reddit Server Authority discussion (2025)](https://www.reddit.com/r/gamedev/comments/1oo7zfy/sever_authoritative_multiplayer_how_does_it_work/): Server runs the game engine headless (no rendering), validating all physics and game rules

---

## 8. SuperTuxKart Networking Code and Protocol

### Architecture Overview

SuperTuxKart uses a **client-server model** with volunteer-operated public servers and self-hosted options.

**Protocol details** ([NETWORKING.md](https://github.com/supertuxkart/stk-code/blob/master/NETWORKING.md)):

- **Transport**: UDP (ports 2759 for game, 2757 for server discovery)
- **NAT Penetration**: STUN servers handle peer discovery; 24/7 servers require port forwarding
- **IPv6**: Dual-stack IPv4/IPv6 supported
- **State synchronization**: Configurable `state-frequency` (default: 10 Hz); clients rewind and replay for prediction
- **Authentication** (WAN): Players validated against stk-addons server via AES keys (20-second timeout)

### Server Configuration Options

| Parameter | Default | Description |
|-----------|---------|-------------|
| `server-mode` | 0 | 0=GP Race, 1=TT, 3=Race, 4=TT, 6=Soccer, 7=FFA, 8=CTF |
| `server-max-players` | 8 | Max 8 (higher degrades performance) |
| `wan-server` | false | Enable online play (requires STK account) |
| `state-frequency` | 10 | State updates per second sent to clients |
| `ranked` | false | Submit to rankings (forces strict modes) |
| `firewalled-server` | false | Enable STUN/NAT traversal |
| `validating-player` | false | Validate via stk-addons AES key |
| `ping-threshold-ms` | 300 | Kick players over this ping |
| `jitter-threshold-ms` | 100 | Kick players over this jitter |

### Game Modes Supported

- **Grand Prix (GP)**: No mid-game join
- **Linear Race**: Standard race
- **Time Trial**: No mid-game join
- **Soccer, FFA, CTF**: Supports **live join** and **spectate** mid-session

### Network Testing

STK includes a **network AI tester**: connect AI clients with `--connect-now --network-ai --no-graphics` to simulate poor network conditions and load-test the server without human players.

### What Can Be Adapted

For a browser-based racing game inspired by STK:

| STK Pattern | Browser Adaptation |
|-------------|-------------------|
| UDP state sync at 10 Hz | WebTransport datagrams at 20 Hz |
| Configurable `state-frequency` | Adjustable server tick rate via config |
| Rewind + replay client prediction | JavaScript circular buffer + physics rewind |
| STUN NAT traversal | WebRTC ICE (handled by browser automatically) |
| SQLite ban lists | Server-side database (PostgreSQL + admin panel) |
| Volunteer-operated servers | mod.io or open server registration |
| Server-only compile flag | Node.js headless server (no canvas/DOM) |

STK's networking code is open-source at [github.com/supertuxkart/stk-code](https://github.com/supertuxkart/stk-code) under GPL v3.

---

## 9. Synthesis: Recommended Architecture

### For a Browser-Based Multiplayer Racing Game

```
┌──────────────────────────────────────────────────────────┐
│                     CLIENTS (Browser)                     │
│                                                          │
│  Client Prediction (physics sim, 60fps)                  │
│  Input capture (steering, throttle, brake, item use)     │
│  Dead reckoning for remote karts                         │
│  Interpolation / blending                                │
└──────────┬─────────────────────────┬─────────────────────┘
           │ WebSocket (lobby/events) │ WebTransport (real-time state)
           ▼                         ▼
┌──────────────────────────────────────────────────────────┐
│                    GAME SERVER (Node.js)                  │
│                                                          │
│  State authority: runs physics at 20 Hz                  │
│  Input validation: rejects impossible inputs             │
│  Race logic: lap counting, item management, timing       │
│  Matchmaking / lobby coordination                        │
│  Anti-cheat: speed/position validation                   │
│  SQLite / Postgres: leaderboards, ban lists, profiles    │
└──────────────────────────────────────────────────────────┘
```

### Technology Choices Summary

| Component | Recommendation | Notes |
|-----------|---------------|-------|
| Architecture | Client-server, server authoritative | Prevents most cheating; consistent state |
| Real-time protocol | WebTransport (primary) + WebSocket (fallback) | Based on NSDI 2025 benchmarks |
| Tick rate | 20 Hz server state, 60 Hz client prediction | Balance bandwidth and feel |
| Lag compensation | Dead reckoning + interpolation for remote karts | Classical scheme sufficient for 50–100ms |
| Local multiplayer | Scissor/viewport split in Three.js | 2–4 cameras per frame |
| Matchmaking | Nakama or AccelByte (self-hosted) | Or mod.io sessions |
| Dedicated server | Node.js headless + VPS/Hathora Cloud | Can run on Raspberry Pi for dev |
| Anti-cheat | Input-based authority (server simulates physics from inputs) | No direct position trust |
| Ban management | IP + user ID bans in server database | Mirror STK pattern |

---

## Sources

- [SuperTuxKart NETWORKING.md — GitHub](https://github.com/supertuxkart/stk-code/blob/master/NETWORKING.md)
- [SuperTuxKart Networking Looking for Testers — Blog Post (2018)](https://blog.supertuxkart.net/2018/11/supertuxkart-networking-looking-for.html)
- [SuperTuxKart Development Blog April 2019](https://blog.supertuxkart.net/2019/04/)
- [Hathora Blog — Peer-to-Peer vs Client-Server Architecture for Multiplayer Games](https://blog.hathora.dev/peer-to-peer-vs-client-server-architecture/)
- [NSDI 2025 — Evaluating Browser-Based Networking for Real-Time Multiplayer Games (PDF)](https://aaron.gember-jacobson.com/docs/nsdi2025browser-networking.pdf)
- [videosdk.live — Which is Better, WebRTC or WebSocket (2025)](https://www.videosdk.live/developer-hub/websocket/which-is-better-and-when-to-use-it-webrtc-or-websocket)
- [LinkedIn — WebRTC vs WebSocket Comparing 10 Crucial Differences in 2025](https://www.linkedin.com/pulse/webrtc-vs-websocket-comparing-10-crucial-differences-2025-zyz0c)
- [AccelByte — Game Matchmaking Architecture: Scaling to One Million Players](https://accelbyte.io/blog/scaling-matchmaking-to-one-million-players)
- [Heroic Labs Nakama — Lobby Matchmaking Forum](https://forum.heroiclabs.com/t/lobby-matchmaking/1180)
- [How do online games implement lobby systems — Reddit r/gamedev](https://www.reddit.com/r/gamedev/comments/10td99x/how_do_online_games_implement_lobby_system/)
- [Server Authoritative Multiplayer — Reddit r/gamedev (2025)](https://www.reddit.com/r/gamedev/comments/1oo7zfy/sever_authoritative_multiplayer_how_does_it_work/)
- [Netcode Fundamentals for Fast-paced Multiplayer Games — Reddit r/gamedev](https://www.reddit.com/r/gamedev/comments/fc1msz/netcode_fundamentals_for_fastpaced_multiplayer/)
- [Anti-Cheat Expert Discussion — Hacker News](https://news.ycombinator.com/item?id=40631223)
- [Unity Netcode 100% Server Authoritative with Client Prediction — YouTube (git-amend)](https://www.youtube.com/watch?v=-lGsuCEWkM0)
- [Unity Multiplayer Kart — GitHub (adammyhre)](https://github.com/adammyhre/Unity-Multiplayer-Kart)
- [Unity Netcode Lag Compensation using Extrapolation — YouTube](https://www.youtube.com/watch?v=_Kg-D0oRQ9M)
- [Predictive Dead Reckoning for Online Peer-to-Peer Games — University of Waterloo (2023)](https://ece.uwaterloo.ca/~sl2smith/papers/2023TOG-Predictive_Dead_Reckoning.pdf)
- [On the Suitability of Dead Reckoning Schemes for Games — McGill University](https://svn.sable.mcgill.ca/sable/courses/COMP763/oldpapers/pantel-02-suitability.pdf)
- [Split-Screen in Modern Gaming — Reddit r/gamedev](https://www.reddit.com/r/gamedev/comments/ajiciq/split_screen_in_modern_gaming/)
- [Creating Local Multiplayer with Split Screen in Unreal Engine 5 — YouTube](https://www.youtube.com/watch?v=SwRbU2aqiNA)
- [Peer to Peer vs Dedicated Game Server Hosting — Servers.com](https://www.servers.com/blog/differences-between-peer-to-peer-and-dedicated-game-server-hosting)
- [Peer to Peer vs Dedicated Servers — Pine Hosting](https://pinehosting.com/blog/dedicated-server-vs-peer-to-peer/)
- [Tricks and Patterns to Deal with Latency — Unity Netcode for GameObjects Docs](https://docs.unity3d.com/Packages/com.unity.netcode.gameobjects@2.7/manual/learn/dealing-with-latency.html)
- [SuperTuxKart — pybullet.org Physics Forum (early networking discussion)](https://pybullet.org/Bullet/phpBB3/viewtopic.php?t=1774&start=15)
