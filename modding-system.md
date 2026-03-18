# Scratch-like Low-Code Modding System for Games

> Comprehensive research on visual/block-based programming for game modding, modding APIs, sandboxing, and community platforms.

---

## Table of Contents

1. [Visual / Block-Based Programming Systems](#1-visual--block-based-programming-systems)
2. [Google Blockly — Integration in Game Engines](#2-google-blockly--integration-in-game-engines)
3. [How Games Implement Visual Scripting for Modders](#3-how-games-implement-visual-scripting-for-modders)
4. [Node-Based Editors for Game Logic](#4-node-based-editors-for-game-logic)
5. [Sandboxing and Security for User-Created Mods](#5-sandboxing-and-security-for-user-created-mods)
6. [Mod Sharing and Community Platforms](#6-mod-sharing-and-community-platforms)
7. [How to Design a Modding API That Exposes Game Systems Safely](#7-how-to-design-a-modding-api-that-exposes-game-systems-safely)
8. [Synthesis: Design Recommendations](#8-synthesis-design-recommendations)

---

## 1. Visual / Block-Based Programming Systems

### The Scratch / Blockly Paradigm

Visual block-based editors represent code as interlocking, puzzle-piece blocks — variables, loops, conditionals, and events — eliminating syntax errors and reducing the barrier for non-programmers. The two dominant open-source foundations are:

| System | Maintained By | License | Output Languages | Primary Use |
|--------|--------------|---------|-----------------|-------------|
| **Scratch** | MIT Media Lab | BSD | Scratch bytecode | Educational game creation |
| **Google Blockly** | Google | Apache 2.0 | JavaScript, Python, Lua, Dart, PHP + custom | Embedding in any web/mobile app |
| **MakeCode** | Microsoft | MIT | JavaScript, Python, MicroPython | Educational coding in Minecraft, micro:bit |
| **Stencyl** | Stencyl LLC | Commercial (free tier) | ActionScript/bytecode | 2D game development |

[Google Blockly](https://developers.google.com/blockly) is a **100% client-side JavaScript library** that renders a drag-and-drop block editor and compiles blocks to any target language. It powers Scratch, Code.org, App Inventor, and CS First, and is the foundation for most game modding systems. Developers integrate it as a library, define custom blocks, and wire up a code generator that emits the target scripting language.

### How Blocks Map to Game Code

The fundamental pipeline for a block-based modding system is:

```
User arranges blocks → Blockly AST → Code generator → Target language string → Runtime exec
```

A Roblox community developer described the transpiler pattern precisely:

> *"You write a transpiler which translates blocks to Lua code. The simplest approach is to have each block carry a string representation of some Lua code and concatenate them together into a string that can be run by `loadstring`."*  
> — [Roblox Developer Forum](https://devforum.roblox.com/t/how-would-i-make-a-block-based-coding-system-like-scratch-on-roblox/2908042)

The more robust pattern uses an **object tree approach**: each block is a runtime object with an `execute()` function, and the tree is walked at runtime — more OOP-friendly but harder to author well.

---

## 2. Google Blockly — Integration in Game Engines

### Features and Capabilities

[Google Blockly](https://developers.google.com/blockly) provides:

- **Visual programming editor** with drag-and-drop interlocking graphical blocks for all common programming constructs (variables, logical expressions, loops, functions, etc.)
- **100% client-side** — no server dependencies; works in all major browsers (Chrome, Firefox, Safari, Opera, Edge)
- **Cross-platform** — web and mobile
- **Plugin system** — add custom fields, themes, renderers
- **Built-in code generators** for JavaScript, Python, PHP, Lua, and Dart; extensible for custom languages
- Powers major platforms: [Scratch](https://scratch.mit.edu), Code.org, App Inventor

### Integration Pattern for a Game Engine

For a browser-based or web-wrapped game, the integration pattern is:

1. **Embed the Blockly editor** in a panel/modal within the game UI (iframe or React component)
2. **Define custom block categories** mapping to game API calls (e.g., `Set Car Speed`, `On Lap Complete`, `Add Checkpoint`)
3. **Implement a custom code generator** that emits the game's scripting language (JavaScript, Lua, GDScript)
4. **Execute generated code** in a sandboxed runtime (see Section 5)

A real-world example: [CopperCube](https://ambiera.com/forum.php?t=11889) — a community developer injected the CopperCube JavaScript API into Blockly, wiring Blockly's visual editor to the engine's action/behavior system and showing generated JS in real time. The main advantage: users see the generated code immediately, aiding learning.

### Blockly vs Node-Based (Blueprint-Style)

| Dimension | Blockly / Scratch-style | Node/Wire (Unreal Blueprints) |
|-----------|------------------------|-------------------------------|
| Beginner accessibility | Very high — linear, top-down reading | Medium — spatial, non-linear |
| Complexity ceiling | Medium — large scripts get unwieldy | High — suitable for professional logic |
| Visual clarity at scale | Degrades (tall stacks) | Also degrades (spaghetti wires) |
| Transpilation simplicity | Easy — blocks = code fragments | Harder — graph traversal required |
| Best for | Educational modding, simple game rules | Pro modders, complex logic flows |

A key GitHub issue in the Godot Visual Scripting repo (2022) proposed [a hybrid](https://github.com/godotengine/godot-visual-script/issues/29): Scratch-style puzzle-blocks for local logic, connected by Blueprint-style wires for flow — combining vertical readability with horizontal flexibility.

---

## 3. How Games Implement Visual Scripting for Modders

### Roblox — BlockLua Plugin

[BlockLua](https://devforum.roblox.com/t/blocklua-roblox-visual-scripting-like-scratch/2129958) is a community-made Roblox Studio plugin that provides Scratch-like block coding targeting the Roblox Lua environment:

- Blocks map directly to Lua constructs and Roblox APIs
- Uses Script Injection permissions to write generated Lua into Roblox scripts
- Blocks include: `touched by character`, `kill`, variable setters/getters, event handlers
- Completely free; the previous plugin (EventBlocks) was discontinued
- Generates clean Lua from block trees, allowing users to graduate from blocks to real Lua

### Minecraft Education Edition — Microsoft MakeCode

[Microsoft MakeCode for Minecraft](https://minecraft.makecode.com) provides a full in-game coding application with three modes:

- **Blocks** — Scratch-like drag-and-drop for beginners
- **JavaScript** — Text mode with full MakeCode API
- **Python** — Python mode for CS curriculum compatibility

MakeCode exposes Minecraft game systems through categorized block palettes (Builders, Players, Mobs, World events). A 2025 update added [File Read/Write with MakeCode](https://edusupport.minecraft.net/hc/en-us/articles/38490275582100) so students can do real I/O with `.txt` and `.csv` files — bridging creative play and computer science education.

### Dreams (PS4/PS5) — Media Molecule

[Dreams](https://en.wikipedia.org/wiki/Dreams_(video_game)) (Media Molecule, 2020) is the most advanced consumer-facing creation system. Its visual logic system is **Gadget-based and node-wired** rather than block-based, but offers important lessons:

- **Imp cursor** = interaction metaphor (like a mouse in 3D space)
- **Gadgets** = functional logic units (sensors, emitters, logic gates, timers)
- **Wires** link gadgets to objects and to each other, forming data/event flow graphs
- Creations are organized into: Dreams → Scenes → Elements → Collections
- **Dreamiverse** = community sharing platform with optional remix licensing
- The system is closed — users cannot export to other engines — and [Dreams is no longer receiving updates](https://www.reddit.com/r/playstation/comments/1qgzqcf/) as of early 2026

Dreams demonstrates that a **wire-and-gadget model** can be made approachable for consumers without exposing raw programming syntax. The key design choice: every game concept (physics body, audio trigger, AI controller) is a draggable gadget.

### Godot Block Coding Plugin (2024)

The [Godot Block Coding plugin](https://github.com/endlessm/godot-block-coding) (Endless OS Foundation, released 2024) is the most directly relevant open-source implementation:

- Targets **Godot 4.3+**
- Available via Godot AssetLib as "Block Coding"
- Generates and executes **GDScript** from block trees — users can view the generated GDScript
- Blocks include: position/rotation/scale for Node3D, event handlers, conditionals, loops, variable assignments, physics checks (e.g., `is_on_floor()`)
- December 2024 release added **Vector3 support**, inspector drag-to-block, context menus
- Experimental status but actively developed; Pong example game ships with it

This plugin is the closest open-source reference for implementing Scratch-like block coding in a custom game engine.

### Stencyl and Construct 3 — Event Sheets

[Stencyl](https://www.stencyl.com) and Construct 3 use **event sheet editors** rather than blocks or nodes: logic is organized as rows and columns of trigger–condition–action rules. This paradigm has the highest track record for shipped commercial games among visual-only programmers. Construct 3 has a free tier with limited events.

---

## 4. Node-Based Editors for Game Logic

### Unreal Engine Blueprints — The Industry Standard

Unreal Engine's [Blueprint Visual Scripting](https://dev.epicgames.com/documentation/en-us/unreal-engine/exposing-gameplay-elements-to-blueprints-visual-scripting-in-unreal-engine) is the reference implementation for node-based game logic editing:

- C++ classes are marked `UCLASS(Blueprintable)` to expose them to Blueprint
- Properties are exposed with `UPROPERTY(BlueprintReadWrite)` macros
- Functions exposed with `UFUNCTION(BlueprintCallable)` macros
- Blueprint nodes are visual representations of these exposed C++ calls
- Sea of Thieves used Blueprint for treasure maps and ship control
- Allows designers/modders to build game logic without writing C++

**Adapting for modding:** The key insight is that Blueprint's architecture is a **controlled API surface** — the developer explicitly chooses which systems to expose. This is the pattern to replicate: define a modding API by annotating or wrapping game functions, then let the visual editor expose only those annotated functions as blocks/nodes.

### Other Node-Based Systems

| System | Engine | Notes |
|--------|--------|-------|
| **Shader Graph** | Unity | Node-based material authoring |
| **Visual Effect Graph** | Unity | Node-based particle/VFX |
| **Flowgraph** | CryEngine 3 | Entity logic, triggers |
| **Nodons** | Nintendo Game Builder Garage | Color-coded I/O nodes for Switch |
| **Event Sheets** | Construct 3, Clickteam Fusion | Row/column trigger-action; most games shipped |

According to community analysis in the [Godot Visual Scripting GitHub repo](https://github.com/godotengine/godot-visual-script/issues/29), event sheet systems (Construct, Clickteam) dominate the number of commercially shipped visual-scripting games on Steam over both Scratch-style blocks and Blueprint-style node wires.

---

## 5. Sandboxing and Security for User-Created Mods

### The Core Problem

User-created mod code is inherently untrusted. It can:
- Access the file system, network, or OS APIs
- Run infinite loops, hogging CPU
- Crash the host game process
- Install malware or mine cryptocurrency on players' machines

### WebAssembly (WASM) — The Recommended Solution

The current consensus in the game development community ([Reddit r/rust, 2025](https://www.reddit.com/r/rust/comments/1isoiwa/securesandboxed_game_modding_with_rust/)) is that **WebAssembly is the best sandboxing solution** for game mods:

- **Isolation**: Guest WASM and host do not share memory; host controls all access
- **Capability-based**: Only APIs explicitly exposed by the host are callable by mods
- **Cross-platform**: A single `.wasm` binary works on all platforms; no `.dll`/`.so`/`.dylib` variants needed
- **Error containment**: Errors in mods are recoverable without crashing the game process
- **Performance**: WasmTime allows a "gas budget" (CPU cycle limit) per mod execution
- **Language agnostic**: Mods can be written in Rust, C, C++, Go, AssemblyScript, etc.

Example runtimes: **WasmTime**, **Wasmer**, **wasm3**

> *"WASM gives you all the tools to catch errors without crashing the entire game process. The tricky part is to design the plugin runtime system so such errors are easily recoverable without restarting the session."*  
> — [Hacker News discussion on Microsoft Flight Simulator 2024 WASM SDK](https://news.ycombinator.com/item?id=44695364)

**Microsoft Flight Simulator 2024** migrated its entire addon SDK to WebAssembly, noting that WASM provides a secure sandbox with predictable performance while enabling compatibility if the game is ported to Xbox.

### Alternative: Lua Scripting (LuaJIT)

For games targeting desktop-first (not browser), **LuaJIT** in a restricted sandbox is the classic approach used by:
- Roblox (pre-Luau)
- Garry's Mod
- World of Warcraft (limited Lua API)
- Factorio

LuaJIT can achieve near-native C performance, but sandboxing requires manually removing dangerous API access (`os`, `io`, `package`, `debug` libraries).

### Security Architecture Patterns

1. **Capability model**: Mods receive only handles/interfaces to game objects, never raw memory pointers. The game engine validates all interactions server-side.
2. **Isolated processes**: Run each mod in a separate OS process; communicate via IPC (sockets or shared memory). Provides OS-level separation at the cost of overhead.
3. **Time-slicing**: Limit mod execution time per tick (via WASM gas budgets or cooperative yields).
4. **API allowlist**: Only expose a declared set of game functions. Block filesystem, network, and OS access entirely.
5. **Content signing**: Mods distributed through an official platform can be signed and content-scanned before distribution (mod.io supports this).

### For Visual/Block-Based Systems Specifically

A block-based system inherently provides strong sandboxing: **users cannot write arbitrary code**. The block palette defines the entire possible API surface. If blocks only emit calls to a safe, validated game API, no additional WASM sandboxing is required. This is the key advantage of block-based modding over text-based scripting.

---

## 6. Mod Sharing and Community Platforms

### mod.io — The Leading Cross-Platform Solution

[mod.io](https://mod.io) is the dominant third-party UGC middleware platform as of 2024–2025:

**Scale (2024 year-end stats):**
- **2.6 billion+** total downloads
- **30.5 million+** monthly active users
- **1.7 million+** UGC creators
- **341 live games** (with 260+ games using the platform in 2024)
- **281,000 creators** submitted content in 2024

**Platform reach:**
- Cross-platform: PC, console (PlayStation, Xbox, Switch), mobile, VR
- Console accounts for **65%+ of all downloads** in 2024
- mod.io holds **64% market share** of official mod support launches on console/VR
- On PC, Steam Workshop holds 85% share; mod.io ~10%

**Features:**
- White-label integration with custom SSO and embedded UGC hub
- End-to-end content management via dashboards, SDKs, and plugins
- Native SDKs/plugins for Unity, Unreal Engine, and custom engines
- Moderation tools: four levels of checks, per-platform rules engine, AI hooks
- Premium UGC monetization: cross-platform marketplace, revenue share, creator self-onboarding
- Community tools: in-game discovery, events, contests, insider invites for early access creators
- Advanced metrics dashboard: acquisition, retention, health, trends, top creators

**Business outcomes:**
- 90%+ retention improvement for games using mod.io
- 23%+ sales lift
- 1500% higher UGC consumption vs. third-party
- Case studies: Baldur's Gate 3 (white-label cross-platform), SnowRunner (lifecycle extension via mods), Crypt of the Necrodancer (multiplayer mods on consoles/mobile)

### Steam Workshop

Steam Workshop remains the dominant PC platform. For games targeting Steam, it offers:
- Integrated mod upload/download in the Steam client
- Automatic mod updates for subscribers
- Workshop item voting and ratings
- API for querying and downloading items from within the game

### Other Platforms and Approaches

| Platform | Notes |
|----------|-------|
| **Nexus Mods** | Largest PC modding community by content volume; no in-game integration API |
| **CurseForge** | Strong in Minecraft/WoW ecosystem; has launcher integration |
| **GameBanana** | Popular for fighting games and Valve titles |
| **Thunderstore** | Dominant for Unity roguelikes (Lethal Company, Risk of Rain 2) |
| **Custom portals** | Some studios (e.g., Paradox, Bohemia) run their own mod hubs |

### The "Dreamiverse" Pattern

Dreams' [Dreamiverse](https://en.wikipedia.org/wiki/Dreams_(video_game)#Gameplay) built community sharing directly into the creation toolset with:
- **Remix licensing**: creators optionally allow others to build on/modify their content
- **Rating system**: recommended to friends, surfaced through discovery algorithms
- **Collections**: curated groups of Dreams, Scenes, and Elements

This tightly integrated platform-as-creation-community is a model worth emulating for a game with a strong modding focus.

---

## 7. How to Design a Modding API That Exposes Game Systems Safely

### Core Principle: Explicit Surface Area

The Unreal Blueprint model teaches the right approach: **the API surface is explicitly annotated by developers, not inferred**. Modders can only access what is deliberately exposed. This applies equally to block-based modding:

> Define blocks ↔ Define API surface. Every block is a gate into the game's systems.

### API Design Layers

A well-designed modding API should expose systems in layers of abstraction:

| Layer | Examples | Who Can Use |
|-------|----------|-------------|
| **Events** | `on_lap_complete`, `on_boost_activated`, `on_item_collected` | All modders |
| **Queries** | `get_player_speed()`, `get_position()`, `get_lap_count()` | All modders |
| **Commands** | `set_player_speed(n)`, `add_item(type)`, `trigger_effect(id)` | Trusted mods |
| **Hooks** | Override physics, AI behavior, UI rendering | Advanced modders |
| **Engine internals** | Raw memory, render pipeline, asset loading | Game developers only |

### Block Palette Design

For a racing game modding system, a block palette might include:

**Event blocks (hat blocks, like Scratch):**
- `When lap starts`
- `When player gets boost`
- `When item is collected`
- `When speed exceeds [n] km/h`

**Action blocks:**
- `Set kart color to [color]`
- `Apply speed multiplier [n]`
- `Spawn particle effect [name] at kart position`
- `Show message [text] for [n] seconds`
- `Add item [type] to inventory`

**Condition blocks:**
- `Current lap > [n]`
- `Player is in first place`
- `Item of type [X] is collected`

**Variable blocks:**
- `Set [variable] to [value]`
- `Change [variable] by [n]`

### Safety Constraints for Each Block

1. **Numeric inputs**: Clamp to safe ranges (speed multiplier 0.1–5.0x, not 1000x)
2. **String inputs**: Validate against a whitelist of known names (particle effect IDs, item types)
3. **Loop limits**: Enforce maximum iteration counts in any repeat block
4. **Execution time limits**: Kill or pause mod scripts that run longer than a per-tick budget
5. **No direct object references**: Mods receive opaque handles, not raw object pointers

### WASM for Text-Based Mods (Advanced)

For more powerful mod access beyond blocks, use WebAssembly with:
- A **host ABI** that exposes safe, typed function signatures
- **WasmTime** with per-invocation fuel (computational budget)
- **Separate WASM module** per mod — failures are isolated
- **Signed manifests** declaring required capabilities, reviewable before install

### Reference Architecture

```
┌─────────────────────────────────────────────────┐
│                  Block Editor UI                  │
│  (Google Blockly embedded, custom block palette) │
└────────────────────┬────────────────────────────┘
                     │ generate code
┌────────────────────▼────────────────────────────┐
│              Code Generator                       │
│  (Blocks → safe JS/Lua API call strings)         │
└────────────────────┬────────────────────────────┘
                     │ execute in sandbox
┌────────────────────▼────────────────────────────┐
│           Sandboxed Mod Runtime                   │
│  (WASM module OR restricted eval scope)          │
│  - API allowlist enforced                        │
│  - CPU budget per tick                           │
│  - No filesystem/network access                  │
└────────────────────┬────────────────────────────┘
                     │ validated calls
┌────────────────────▼────────────────────────────┐
│             Game Engine Modding API               │
│  (Events, Queries, Commands — explicitly exposed)│
└─────────────────────────────────────────────────┘
```

---

## 8. Synthesis: Design Recommendations

For a browser-based racing game aiming for a Scratch-like modding system:

### Recommended Stack

1. **Editor**: [Google Blockly](https://developers.google.com/blockly) — 100% client-side, Apache 2 licensed, code generators for JS/Lua, well-documented
2. **Block execution**: Generated JavaScript executed in a restricted scope (`new Function()` with a safe global object, or a WASM sandbox)
3. **API design**: Expose via an explicit `ModAPI` object with racing-specific events/commands only
4. **Progression path**: Blocks → view generated code → graduate to text editing — mirrors Godot Block Coding and MakeCode
5. **Community platform**: [mod.io](https://mod.io) SDK for mod upload/download/moderation, or Steam Workshop if Steam-targeted
6. **Security baseline**: Blocks provide strong inherent sandboxing; add WASM/isolated eval for text-mode advanced mods

### Key Design Decisions

| Decision | Recommendation |
|----------|---------------|
| Block system | Google Blockly |
| Target language | JavaScript (native in browser) |
| Sharing platform | mod.io (cross-platform, 30M+ MAU) |
| Sandboxing | Restricted JS scope for blocks; WASM for advanced text mods |
| API style | Event-driven + command pattern, no raw engine access |
| Progression | Block → view code → text mode |

---

## Sources

- [Google Blockly Developer Documentation](https://developers.google.com/blockly)
- [BlockLua — Roblox Visual Scripting Plugin](https://devforum.roblox.com/t/blocklua-roblox-visual-scripting-like-scratch/2129958)
- [How to make a block-based system like Scratch on Roblox](https://devforum.roblox.com/t/how-would-i-make-a-block-based-coding-system-like-scratch-on-roblox/2908042)
- [Godot Block Coding Plugin — Endless OS Foundation (GitHub)](https://github.com/endlessm/godot-block-coding)
- [Godot Block Coding Plugin releases](https://github.com/endlessm/godot-block-coding/releases)
- [Godot Block Coding Forum Thread](https://forum.godotengine.org/t/block-coding-high-level-block-based-visual-programming/68941)
- [New Godot Visual Scripting Language — Block Coding (YouTube)](https://www.youtube.com/watch?v=U7n7gJIkbk0)
- [Re-implement VisualScript using a 'block and wire' system (Godot GitHub)](https://github.com/godotengine/godot-visual-script/issues/29)
- [Microsoft MakeCode for Minecraft](https://minecraft.makecode.com)
- [File Read/Write with MakeCode — Minecraft Education](https://edusupport.minecraft.net/hc/en-us/articles/38490275582100-File-Read-Write-with-MakeCode)
- [Dreams (video game) — Wikipedia](https://en.wikipedia.org/wiki/Dreams_(video_game))
- [Dreams — PlayStation Store](https://www.playstation.com/en-us/games/dreams/)
- [Secure/Sandboxed Game Modding with Rust — Reddit r/rust (2025)](https://www.reddit.com/r/rust/comments/1isoiwa/securesandboxed_game_modding_with_rust/)
- [Using WASM as a mod system for a game built in Rust — Wasmer GitHub Discussions](https://github.com/wasmerio/wasmer/discussions/2627)
- [Microsoft Flight Simulator 2024 WebAssembly SDK — Hacker News](https://news.ycombinator.com/item?id=44695364)
- [mod.io — Cross Platform Mod Support for Games](https://mod.io)
- [mod.io 2024 Year in Review](https://blog.mod.io/2024-year-in-review-f06139966528)
- [Exposing Gameplay Elements to Blueprints — Epic Games Developer Documentation](https://dev.epicgames.com/documentation/en-us/unreal-engine/exposing-gameplay-elements-to-blueprints-visual-scripting-in-unreal-engine)
- [CopperCube Blockly Visual Scripting Discussion](https://ambiera.com/forum.php?t=11889)
- [Is there any FREE block coding game engines aside from Scratch — Reddit r/GameDevelopment](https://www.reddit.com/r/GameDevelopment/comments/1e7w7vh/is_there_any_free_block_coding_game_engines_aside/)
- [The legal and creative battle over who gets to modify beloved games — Boing Boing (March 2026)](https://boingboing.net/2026/03/16/game-modding-makes-virtual-worlds-go-around.html)
