# Racing AI and Race Timeline Systems

> Research compiled: March 18, 2026  
> Sources: GameAIPro, GitHub (SuperTuxKart), academic papers, developer forums

---

## Table of Contents

1. [Core Racing AI Techniques](#1-core-racing-ai-techniques)
2. [AI Difficulty Scaling and Adaptive Difficulty](#2-ai-difficulty-scaling-and-adaptive-difficulty)
3. [Race Timeline Mechanics](#3-race-timeline-mechanics)
4. [Dynamic Events During Races](#4-dynamic-events-during-races)
5. [Ghost Racing Systems and Replay Recording](#5-ghost-racing-systems-and-replay-recording)
6. [AI for Multi-Mode Racing](#6-ai-for-multi-mode-racing)
7. [SuperTuxKart Existing AI System](#7-supertuxkart-existing-ai-system)
8. [AI Architecture Overview](#8-ai-architecture-overview)
9. [Summary and Recommendations](#9-summary-and-recommendations)

---

## 1. Core Racing AI Techniques

### 1.1 Rubber Banding

Rubber banding is a widely-used technique that keeps AI vehicles near the player to maintain excitement. It works by manipulating AI vehicle speed based on distance from the player.

**Mechanism** (from [GameAIPro Chapter 42 by Nic Melder](http://www.gameaipro.com/GameAIPro/GameAIPro_Chapter42_A_Rubber-Banding_System_for_Gameplay_and_Race_Management.pdf)):

| Region | Distance from Player | Effect |
|--------|----------------------|--------|
| a | < -150 (max reverse) | Constant max positive speed multiplier (e.g., 1.4×) |
| b | -150 to -50 | Linear positive increase |
| c | -50 to +50 (dead zone) | No effect — normal behavior |
| d | +50 to +200 | Linear negative decrease |
| e | > +200 (max forward) | Constant max negative speed multiplier (e.g., 0.6×) |

**Two implementation methods**:
1. **Power Modifier**: Adjusts vehicle acceleration and top speed. Applied only after the difficulty multiplier hits its min/max. Forward banding reduces power (0.6× multiplier); reverse banding increases it (1.4×).
2. **Driver Skill Modifier** (preferred): Changes AI skill, causing earlier/later braking and faster/slower cornering. Less visible to the player because the car physics remain unchanged.

**Disable conditions** (to prevent obvious cheating perception):
- Race start — avoids bunching
- AI is at low speed (e.g., post-crash) — prevents underpowered stalling
- AI is being lapped — avoids applying max negative effect to a lapped car

**Multiplayer**: Use the rearmost human player as reference for reverse banding, and the foremost for forward banding. AI between human players gets no banding applied.

**Anti-Bunching**: Set `max_distance = max(defined_max, distance_to_furthest_vehicle)` to spread the rubber-banding effect over a greater distance.

**Special case — first place**: Dedicated banding between 1st and 2nd place (not player) to prevent the leader from running away even when the player is far behind.

**Race Pace System** (long races, e.g., 50 laps):
- Hard push first/last 10 laps
- Relax middle 30 laps
- Hard push lap before/after pit stops

Source: [GameAIPro Chapter 42 — A Rubber-Banding System for Gameplay and Race Management](http://www.gameaipro.com/GameAIPro/GameAIPro_Chapter42_A_Rubber-Banding_System_for_Gameplay_and_Race_Management.pdf)

---

### 1.2 Line-Following and Waypoint Navigation

**Waypoint-based navigation** is the most common and straightforward approach:
- Designers or automated tools place 3D nodes at intervals along the track
- The AI steers toward the current waypoint and switches to the next when within a threshold distance
- Bezier curves between waypoints produce smoother steering vs. discrete point-to-point movement

**Racing line following**:
- A predefined spline defines the ideal racing line through each corner
- The AI uses a **PID controller** to steer toward a look-ahead point on the line
- A **curvature-to-speed function** determines safe corner entry speed
- **Brake maps** define braking distances per corner per vehicle

**Racing line generation methods** (from [GameAIPro Chapter 38](https://www.gameaipro.com/GameAIPro/GameAIPro_Chapter38_An_Architecture_Overview_for_AI_in_Racing_Games.pdf)):
- Offline bisection per node
- Modified genetic algorithm with Monte Carlo random mutations and parental crossover
- Track-knowledge guided optimization (e.g., corner grouping)
- Metric: average lap time
- Fallback: manual editing by designers

**Record-and-replay approach** (from [Reddit Unity3D thread](https://www.reddit.com/r/Unity3D/comments/v8ftcz/simple_and_effective_racing_ai/)):
- Record every player's race in small chunks
- AI selects a recording to follow; switches recordings at segment boundaries
- Adapts difficulty dynamically by choosing faster or slower recordings

---

### 1.3 Neural Network Approaches

**Supervised learning (steering)** (from [Training AI Agents to Race Using Reinforcement Learning, YouTube 2026](https://www.youtube.com/watch?v=2oYJQqepRSM)):
- Collect input-output pairs (sensor inputs → steering angle) from human races
- Train a fully connected neural network (2 hidden layers) offline in Python
- Deploy in-game: each frame, gather inputs → forward propagate → get steering angle
- Conventional AI still controls throttle and brake in the prototype

**Reinforcement learning**:
- Agent receives rewards for speed, checkpoint passing, and (in drift versions) sliding angle
- Uses TensorFlow + NVIDIA GPU for training
- Requires deterministic physics for training stability
- More suitable than supervised learning for long-horizon control decisions

**Forza Motorsport neural network approach** (from [Reddit r/Games discussion](https://www.reddit.com/r/Games/comments/irzu65/how_forza_uses_neural_networks_to_evolve_its/)):
- Uses evolved neural networks to train AI drivers in simulation
- Network inputs include track position, speed, nearby vehicles
- Produces human-like racing behavior at various difficulty levels

**Colin McRae Rally 2.0** was an early adopter of neural networks for training rally AI to handle different turns and bends, as noted in a [neural network AI for racing games paper](http://www.idgt.org/sg/staff/project/TQP104.pdf).

---

### 1.4 Finite State Machine (FSM) Behavioral AI

From [GameAIPro Chapter 38](https://www.gameaipro.com/GameAIPro/GameAIPro_Chapter38_An_Architecture_Overview_for_AI_in_Racing_Games.pdf), the standard approach uses utility-score-based FSMs:

| State | Utility Score | Description |
|-------|--------------|-------------|
| Normal Driving | 500 (baseline) | Follow racing line, conservative spacing |
| Overtake | High when speed advantage exists | More aggressive, closer tolerances |
| Defend/Block | High when faster car behind | Lateral steering moves to match opponent's line |
| Branch | Situational | For pit entry, shortcuts |
| Recover | Rises over time off-track | Return to track at modest speed |

**Hysteresis**: A small additional threshold on the current state's utility prevents rapid state switching.

---

## 2. AI Difficulty Scaling and Adaptive Difficulty

### 2.1 Static Difficulty Levels

**Skill-based approach** (GameAIPro Chapter 38):
- UI difficulty 0–100 maps to internal grip multiplier (e.g., 80–82% for easy, 98–99% for hardest)
- Secondary traits:
  - **Aggression**: Closer driving distance, higher overtake probability
  - **Control**: Throttle application rate
  - **Mistakes**: Random corner errors injected at intervals

**SuperTuxKart difficulty levels** (from [STK source code / command line](https://github.com/supertuxkart/stk-code)):
- `--difficulty=0` → Beginner
- `--difficulty=1` → Intermediate
- `--difficulty=2` → Expert
- `--difficulty=3` → SuperTux (hardest — approximately 10% slower than top human players per [STK Evolution forum](https://forum.supertuxkart.net/thread-10-post-23.html))

### 2.2 Biorhythm System

From [GameAIPro Chapter 38](https://www.gameaipro.com/GameAIPro/GameAIPro_Chapter38_An_Architecture_Overview_for_AI_in_Racing_Games.pdf):
- A sine or square wave modulates the skill level over time
- Example: 100s period; average 99%, dips to 90–95% briefly
- Creates natural-looking performance variations that feel human

### 2.3 Adaptive Difficulty

Dynamically adjusts to player performance during or across sessions:
- **MotoGP 24** and **RaceRoom** implement adaptive AI that adjusts after 3–4 races based on player lap times
- **rFactor 2** uses a fixed-difficulty model but is considered best-in-class for consistency and AI-to-AI interaction
- Dynamic weather adds complexity: AI often struggles more than human players with weather transitions, requiring separate wet/dry skill parameters (see [Fanatec / iRacing discussion](https://www.fanatec.com/us/en/explorer/games/gaming-tips/the-impact-of-weather-conditions-on-sim-racing-physics-and-strategy/))

### 2.4 Race Choreographer

A higher-level system that can script specific in-race events:
- Trigger skill changes at specific lap/time points
- Cause tire blowouts or penalties on scripted AI drivers
- Implement custom behaviors (e.g., "rival driver makes a mistake on lap 3")

---

## 3. Race Timeline Mechanics

### 3.1 Lap-Based Racing

- A **finish line trigger** increments a lap counter on crossing
- **Checkpoints** are mandatory waypoints distributed around the track — a lap only counts if all checkpoints were hit in order
- Prevents cheating (backing over the finish line, cutting corners)
- Standard implementation: ordered list of checkpoint IDs; a new lap requires all N checkpoints visited since the last lap completion

**Anti-cheat logic** (from [Unity racing game checkpoint tutorial](https://www.youtube.com/watch?v=7NehsLWcFIU)):
```
- Store a checkpoint index per player
- Only advance index when the correct sequential checkpoint is hit
- Lap completes only when index reaches checkpoint_count
- Reset index to 0 on lap completion
```

**Position tracking**:
- Compute each kart's track distance (total distance along path + laps × track_length)
- Sort by track distance to determine race positions in real-time

### 3.2 Checkpoint-Based Racing (Point-to-Point / Rally)

- Checkpoints define an ordered route through open terrain
- No lap counter — race completes when all checkpoints are passed in sequence
- Often combined with time penalties for missed checkpoints
- Used in rally, time attack, and open-world races

### 3.3 Time Trial

- Single player races against the clock
- Ghost data from best lap(s) shown as a visual opponent
- No collision with ghost — pure reference
- Leaderboard comparison via uploaded replay data

### 3.4 Follow-the-Leader / Elimination

- One kart is designated the leader; all others must stay within range
- Karts that fall too far behind are eliminated
- SuperTuxKart implements this as game mode `N=4` ([STK main.cpp](https://github.com/supertuxkart/stk-code/blob/master/src/main.cpp))

### 3.5 Grand Prix / Points-Based

- Series of races; points awarded per finishing position
- Cumulative standings determine champion
- SuperTuxKart supports Grand Prix with network AI via `--network-ai` flag

---

## 4. Dynamic Events During Races

### 4.1 Weather Changes

**Dynamic weather systems** add strategic depth:
- **iRacing**: Real-time rain system introduces genuine unpredictability — weather evolves mid-race rather than using scripted presets (from [Fanatec racing guide](https://www.fanatec.com/us/en/explorer/games/gaming-tips/the-impact-of-weather-conditions-on-sim-racing-physics-and-strategy/))
- **Codemasters F1 games**: Localized weather fronts where one sector is wet and another dry
- **Automobilista 2**: Allows manual AI adjustment for wet-weather performance

**AI challenges with dynamic weather**:
- AI often becomes unrealistically fast or slow in wet conditions
- Requires separate skill parameter sets for wet vs. dry
- Proposed solution: real-time AI difficulty adjustments based on weather shift magnitude

**Implementation approach**:
```
weather_state = {dry, light_rain, heavy_rain, storm}
ai_grip_multiplier[state] = {1.0, 0.88, 0.75, 0.60}
ai_braking_distance_multiplier[state] = {1.0, 1.15, 1.35, 1.55}
transition_speed = lerp(current_state, new_state, dt * transition_rate)
```

### 4.2 Track Modifications

- Debris and oil spills placed dynamically after crashes
- Shortcuts opening/closing (gates, barriers)
- Weather-induced puddles accumulating off the racing line
- Dynamic track rubber buildup (grip increases on racing line over a race)

### 4.3 Obstacle Spawns (Kart/Arcade Racing)

- Item boxes scattered on track; AI must decide whether to collect or avoid
- SuperTuxKart `SkiddingAI` uses probability-based item collection (`computeSkill()`) and avoidance steering (`steerToAvoid()`) (from [STK skidding_ai.hpp](https://github.com/supertuxkart/stk-code/blob/master/src/karts/controller/skidding_ai.hpp))
- Items: bombs (target kart ahead), bubblegum (defensive), cake (long-range attack), nitro (speed boost)
- AI uses dedicated targeting methods: `handleBubblegum()`, `handleCake()`, etc.

### 4.4 Nitro and Boost Events

- AI tracks nitro usage with `handleNitroAndZipper()` and priority rules
- Bomb targeting triggers nitro usage: `aim at kart ahead, use nitro`
- "Burster" state (`m_burster`) enables strategic nitro bursts

---

## 5. Ghost Racing Systems and Replay Recording

### 5.1 Core Approaches

Two primary methods for ghost recording (from [Unreal Engine Reddit discussion](https://www.reddit.com/r/unrealengine/comments/1b05myf/how_to_create_a_ghost_recording/) and [Troy Story Games devlog](https://www.troystorygames.com/2025/05/14/record-and-replay-ghost-races/)):

**Method 1 — State Snapshot Recording** (recommended for position/rotation):
- Sample player position, rotation, velocity, and animation state at regular intervals (e.g., 60Hz)
- On playback, interpolate between snapshots using `lerp()`
- Store forward vector instead of rotation angles to avoid -180/+180 sign flip artifacts
- Skip duplicate entries to minimize data size

```gdscript
# Godot 4 example from Troy Story Games
func _physics_process(delta: float) -> void:
    timestamp += delta
    var current_pos = player.global_position
    var entry = GhostRecorderEntry.new()
    # ... fill entry ...
    if last_entry.position == current_pos and ...:
        return  # Don't store duplicates
    ghost_data.append(entry)
```

**Method 2 — Input Recording**:
- Record button/axis inputs each frame
- Replay by feeding inputs to an AI controller
- Requires deterministic physics for accurate reproduction
- More complex but handles physics interactions correctly

**Combined approach** (recommended): Use Method 1 for position/rotation, Method 2 for animation cues (wheel turn, brake lights).

### 5.2 Ghost Playback

```gdscript
# Playback with frame-rate independence
func _physics_process(delta: float) -> void:
    timestamp += delta
    while len(ghost_data):
        next_pos = ghost_data.pop_front()
        if not len(ghost_data) or timestamp <= ghost_data[0].timestamp:
            break
    global_position = lerp(global_position, next_pos.position, 0.90)
```

- `lerp` with 0.90 factor smooths over skipped frames
- Physics should be disabled on the ghost actor
- Ghost is a simplified visual mesh — no collision with active racers

### 5.3 Multiple Ghosts

- Store ghost data with metadata (player name, timestamp, lap time)
- Load best-time ghost or developer ghost from file/server
- Display semi-transparent ghost mesh with distinct color per ghost
- See [Unity Ghost Replay System tutorial (YouTube)](https://www.youtube.com/watch?v=c5G2jv7YCxM) for a reference implementation

### 5.4 Data Storage

- Binary serialization of position arrays is compact
- ~3.6KB per minute at 60Hz with delta compression
- For network leaderboards, compress with zlib before upload

---

## 6. AI for Multi-Mode Racing

### 6.1 Challenge: Mode Transitions

In games combining driving, flying, and jumping (e.g., Jak X, Burnout Paradise, The Crew Motorfest), the AI must handle fundamentally different physics:

| Mode | Primary Axes | Navigation Approach |
|------|-------------|---------------------|
| Driving | X, Z (2D ground) | Waypoint spline + rubber-banding |
| Flying | X, Y, Z (3D) | 3D navigation graph + altitude control |
| Jumping | Ballistic arc | Predictive landing zone targeting |

### 6.2 AI Controller Architecture for Multi-Mode

**Recommended pattern — Mode-Switched Controller**:
```
AbstractRacingAI
├── GroundDrivingAI  (waypoints, PID steering)
├── FlyingAI         (3D navigation mesh, pitch/yaw/throttle)
└── JumpAI           (boost direction, air control)

RacingController {
    current_mode: Mode
    controllers: Map<Mode, AbstractRacingAI>
    
    update() {
        detect_mode_transition()
        controllers[current_mode].update()
    }
}
```

**Mode transition detection**:
- Ground → Air: vehicle leaves ground plane (raycast downward fails)
- Air → Ground: collision with terrain, velocity dampening
- Special zones: trigger volumes force mode switch (ramp entry, hover pad)

### 6.3 Handling Jump AI

- Pre-compute landing zone from launch angle and velocity using projectile equations
- AI aims to land on track segment rather than off-road
- During flight: minimal steering adjustments, primarily boost timing
- SuperTuxKart handles jump sections via `findNonCrashingPoint()` which tests future quad-graph positions

### 6.4 SuperTuxKart Multi-Mode Architecture

SuperTuxKart supports the following game modes (`--mode=N`):
- 0: Normal race
- 1: Time trial
- 2: Battle (arena, free-for-all)
- 3: Soccer
- 4: Follow the Leader
- 5: Capture the Flag

Each mode has distinct AI behavior. The `BattleAI` (referenced in `--battle-ai-stats` debug flag) handles arena combat differently from the `SkiddingAI` used in track racing. The architecture separates `AIBaseLapController` (track racing) from arena-specific controllers.

---

## 7. SuperTuxKart Existing AI System

### 7.1 Source Code Location

- Repository: [https://github.com/supertuxkart/stk-code](https://github.com/supertuxkart/stk-code)
- Primary AI file: `src/karts/controller/skidding_ai.hpp` and `skidding_ai.cpp`
- AI module: `src/karts/controller/` (all kart controllers — human or AI)

### 7.2 SkiddingAI Class

```cpp
class SkiddingAI : public AIBaseLapController
```

**Key algorithms per frame** (from [STK skidding_ai.hpp](https://github.com/supertuxkart/stk-code/blob/master/src/karts/controller/skidding_ai.hpp)):

| Phase | Algorithm |
|-------|-----------|
| Crash Detection | `checkCrashes()` — forward timestep simulation of future positions vs. quad-graph bounds |
| Steering | `findNonCrashingPoint()` (default) or `findNonCrashingPointNew()` |
| Skidding | `canSkid()` with FSM: `NOT_YET / NO_SKID / SKID` — randomized per track section |
| Item Collection | `computeSkill()` probability-based; `steerToAvoid()` for item avoidance |
| Nitro/Zipper | `handleNitroAndZipper()` with priority rules |
| Rescue | `handleRescue()` — triggers when stuck |

**Key member variables**:

| Category | Variable | Purpose |
|----------|----------|---------|
| Kart relations | `m_distance_to_player`, `m_distance_leader` | Rubber-banding inputs |
| Track state | `m_current_track_direction`, `m_current_curve_radius` | Curve analysis |
| Skidding | `m_random_skid`, `m_skid_probability_state` | Randomized skid decisions |
| Items | `m_item_to_collect`, `m_avoid_item_close` | Collection targeting |
| Timing | `m_time_since_stuck`, `m_burster` | Race phases |
| Config | `m_point_selection_algorithm` (PSA_DEFAULT / PSA_NEW) | Algorithm selection |

### 7.3 Difficulty Levels

- The AI plays at approximately 10% below top human speed at SuperTux difficulty
- Network AI is created with: `new NetworkAIController(kart, player_id, new SkiddingAI(kart))`
- Network AI is enabled via: `supertuxkart --connect-now=127.0.0.1 --network-ai=2 --no-graphics`

### 7.4 Extending the STK AI System

**To extend SkiddingAI**:

1. **Subclass** `SkiddingAI` or implement `AIBaseLapController`
2. **Override** `update()` to add custom logic before/after base behavior
3. **Add rubber-banding**: Access `m_distance_to_player`; modify throttle multiplier
4. **Custom item behavior**: Override `handleItems()` for new weapon types
5. **Mode-specific AI**: Create new class extending `AIBaseController` for non-track modes

**For multi-mode (flying/jumping)**:
- Register new controller in `world.cpp` where `ai = new SkiddingAI(new_kart.get())`
- Use `RaceManager::get()->getMinorMode()` to detect current game mode and instantiate appropriate AI class

**For dynamic difficulty**:
- Expose a `setSkillLevel(float)` method
- Adjust `m_distance_to_player` thresholds and steering randomness
- Integrate with a session-level stats tracker

### 7.5 Debug Features

- `AI_DEBUG` macros: visualize aim points, curve data, headings
- `--battle-ai-stats`: verbose logging for battle AI
- `--soccer-ai-stats`: verbose logging for soccer AI
- `--test-ai=n`: use test AI for every n-th AI kart

---

## 8. AI Architecture Overview

From [GameAIPro Chapter 38](https://www.gameaipro.com/GameAIPro/GameAIPro_Chapter38_An_Architecture_Overview_for_AI_in_Racing_Games.pdf), the full 4-layer architecture for production racing AI:

```
┌─────────────────────────────────────────────────────┐
│  CHARACTER / PERSONA LAYER  (long timescale)        │
│  • Driver skill levels (grip multipliers)           │
│  • Biorhythm modulation                             │
│  • Personality traits (aggression, control)         │
├─────────────────────────────────────────────────────┤
│  STRATEGIC LAYER  (1 frame – few seconds)           │
│  • Behavioral state machine (FSM with utility)      │
│  • States: Normal, Overtake, Defend, Branch, Recover│
│  • Evaluates: speed advantage, track position       │
├─────────────────────────────────────────────────────┤
│  TACTICAL LAYER  (every frame)                      │
│  • Resolves competing goals into steering/speed     │
│  • Handles overtaking details, blocking maneuvers   │
├─────────────────────────────────────────────────────┤
│  CONTROL LAYER  (every frame)                       │
│  • Computes actual steering, brake, throttle inputs │
│  • PID controller for racing line tracking          │
└─────────────────────────────────────────────────────┘
           │ Alongside all layers │
┌─────────────────────────────────────────────────────┐
│  COLLISION AVOIDANCE SUBSYSTEM                      │
│  • Long-range: plan routes around obstacles         │
│  • Short-range: rapid adjustments (walls, cars)     │
└─────────────────────────────────────────────────────┘
```

---

## 9. Summary and Recommendations

### Key Takeaways

| Technique | Complexity | Quality | Best Use Case |
|-----------|-----------|---------|---------------|
| Waypoint navigation | Low | Medium | Arcade/kart games |
| Racing line + PID | Medium | High | Simulation/competitive |
| Rubber-banding | Low | High (perceived) | All game types |
| Neural network | High | Very high | AAA titles with RL budget |
| FSM with utility | Medium | High | Most production games |
| Record-and-replay | Low | High | Time trials, indie games |

### For SuperTuxKart Extension

1. **Short term**: Override `SkiddingAI::handleAccelerationAndBraking()` to add rubber-banding
2. **Medium term**: Add a `RaceTimeline` class tracking lap events, dynamic weather states, and trigger zones
3. **Long term**: Implement a `MultiModeAIController` factory that selects `SkiddingAI`, `FlyingAI`, or `JumpAI` based on current kart physics state

### Implementation Priority

```
Phase 1: Checkpoint system + ghost recording (foundational)
Phase 2: Rubber-banding difficulty scaling
Phase 3: FSM behavioral states (overtake, defend)
Phase 4: Dynamic events (weather, items, obstacles)
Phase 5: Neural network augmentation (steering refinement)
```

---

## Sources

| Source | URL |
|--------|-----|
| GameAIPro Ch. 42 — Rubber-Banding | http://www.gameaipro.com/GameAIPro/GameAIPro_Chapter42_A_Rubber-Banding_System_for_Gameplay_and_Race_Management.pdf |
| GameAIPro Ch. 38 — AI Architecture for Racing | https://www.gameaipro.com/GameAIPro/GameAIPro_Chapter38_An_Architecture_Overview_for_AI_in_Racing_Games.pdf |
| SuperTuxKart Source Code | https://github.com/supertuxkart/stk-code |
| STK SkiddingAI Header | https://github.com/supertuxkart/stk-code/blob/master/src/karts/controller/skidding_ai.hpp |
| Neural Network AI for Racing Games (paper) | http://www.idgt.org/sg/staff/project/TQP104.pdf |
| Training AI Agents to Race (RL, YouTube 2026) | https://www.youtube.com/watch?v=2oYJQqepRSM |
| Forza Neural Networks Discussion | https://www.reddit.com/r/Games/comments/irzu65/how_forza_uses_neural_networks_to_evolve_its/ |
| Ghost Recording — Unreal Engine Discussion | https://www.reddit.com/r/unrealengine/comments/1b05myf/how_to_create_a_ghost_recording/ |
| Ghost Races — Troy Story Games | https://www.troystorygames.com/2025/05/14/record-and-replay-ghost-races/ |
| Unity Ghost Replay Tutorial | https://www.youtube.com/watch?v=c5G2jv7YCxM |
| Unity Checkpoint & Lap System | https://www.youtube.com/watch?v=7NehsLWcFIU |
| Dynamic Weather in Sim Racing | https://www.fanatec.com/us/en/explorer/games/gaming-tips/the-impact-of-weather-conditions-on-sim-racing-physics-and-strategy/ |
| STK Network AI Issue | https://github.com/supertuxkart/stk-code/issues/4745 |
| Simple and Effective Racing AI (Reddit) | https://www.reddit.com/r/Unity3D/comments/v8ftcz/simple_and_effective_racing_ai/ |
