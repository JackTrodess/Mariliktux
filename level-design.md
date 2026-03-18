# Level Design for Multi-Mode Racing (Drive/Fly/Jump)

> Comprehensive research notes for designing racing tracks that support multiple vehicle transformation modes: ground driving, aerial flight, and mech/jump sections.

---

## Table of Contents

1. [Core Level Design Principles for Multi-Mode Racing](#1-core-level-design-principles-for-multi-mode-racing)
2. [Designing Transitions Between Modes](#2-designing-transitions-between-modes)
3. [Track Design Patterns](#3-track-design-patterns)
4. [Procedural Track Generation](#4-procedural-track-generation)
5. [Level Design Tools and Editors](#5-level-design-tools-and-editors)
6. [SuperTuxKart Track Format and Modding](#6-supertuxkart-track-format-and-modding)
7. [Reference Game Analysis](#7-reference-game-analysis)
8. [Level Design Studio Concepts](#8-level-design-studio-concepts)

---

## 1. Core Level Design Principles for Multi-Mode Racing

### Fundamental Framework

According to [Game Design Skills](https://gamedesignskills.com/game-design/racing/), the guiding axiom is: **"Every bend and curve has a reason to be—'why?' is the key word of level design."** Each element must earn its place.

The core principles from [Benoit GOMES's WRC7 postmortem on Game Developer](https://www.gamedeveloper.com/design/racing-level-design-the-rally-case):

- **A track must tell a story, have a scenario.** Narrative context (a mountain road built 100 years ago, an oasis desert route) shapes all subsequent decisions.
- **Reality is only an inspiration.** Use real-world logic as a starting point, but amplify it for gameplay interest.
- **Reference, references, references.** Analyze real-world counterparts before designing—road geometry, terrain, vegetation, historical context.
- **Keep the global vision of all tracks in the game.** Classify tracks by their role (speed-focused vs. technical), their "temper" (the emotional quality they evoke), and how they contrast with each other.

### Multi-Mode Specific Principles

For games with transformation modes (driving → flying → jumping), [Game Design Skills](https://gamedesignskills.com/game-design/racing/) identifies these requirements:

| Principle | Description |
|-----------|-------------|
| **Environment adaptation** | Level must provide contexts where each mode is necessary or rewarding, not merely available |
| **Movement style completeness** | Each mode needs its own interaction language: drifts for land, wave-riding for boats, barrel rolls / 3D movement for air |
| **Physics distinctness** | Modes should feel fundamentally different (e.g., Sonic & All-Stars Racing Transformed simulates true boat and plane physics, not just reskinned car controls) |
| **Control borrowing** | Retain enough shared control vocabulary across modes that players feel competent immediately after transformation |
| **Track response** | The geometry must present opportunities forward, sideways, up, and down across all modes |

### The "Why Each Mode?" Design Test

Before placing any transformation trigger, apply this check:
- Does this mode change create a genuinely different challenge, or just a visual swap?
- Can a skilled player express mastery specifically in this mode (not just survive it)?
- Does the section give players enough time to acclimate to the new physics before the next critical moment?

---

## 2. Designing Transitions Between Modes

### The Transformation Trigger Architecture

Based on analysis of [Sonic & All-Stars Racing Transformed (SASRT)](https://www.adamzwakk.com/games/sonic-racing-transformed/) and [Mario Kart 8 anti-gravity sections](https://www.mariowiki.com/Anti-gravity):

**Ground → Air transition:**
- Use a ramp or launch pad with defined airtime to give players a clear cue
- The vehicle should begin transforming visually *before* the player is fully airborne (animation preview)
- Landing zones should be wide and forgiving on first introduction to that section
- In SASRT, players can earn a "transform boost" by executing a trick *during* the transformation animation window — a high-skill reward that doesn't punish non-execution

**Air → Ground transition:**
- Designate landing zones clearly with track markings, color, or width variation
- Avoid "collision jank" (noted as a flaw in SASRT) where fast-moving aerial vehicles clip invisible geometry ceiling caps
- Anti-gravity sections in Mario Kart 8 use overlapping trigger planes at the start of each angled section; the camera angle adjusts normally to the road surface throughout, preventing disorientation

**Ground → Water transition (boat mode):**
- Waves slow movement but can be treated as ramps by skilled players — design wave patterns to offer both a punishing and a skillful path
- SASRT Crossworlds vs. Transformed comparison: [Reddit community analysis](https://www.reddit.com/r/SonicCrossWorlds/comments/1o8othx/in_ur_opinion_what_are_the_biggest_differences/) notes that Transformed's boat drift mechanics (with wave-ramp interactions) felt significantly more skilled than Crossworlds' simpler implementation; track layouts in Transformed varied more across the water section

### Transition Design Checklist

```
[ ] Is the trigger clearly visible at high speed (>3 seconds of approach)?
[ ] Does the transformation animation have enough duration for players to adjust mentally?
[ ] Is the new mode section long enough to justify the transition overhead (~15–20 seconds minimum)?
[ ] Does the transition section offer a skill-expressive moment (trick, boost window)?
[ ] Is there a forgiving landing/re-entry zone for lower-skill players?
[ ] Does the AI follow the same transition logic as the player?
```

### Anti-Gravity Sections (Mario Kart 8 Model)

From the [Super Mario Wiki](https://www.mariowiki.com/Anti-gravity) and a [YouTube analysis of anti-gravity track design](https://www.youtube.com/watch?v=EH6kqDXbvb4):

- Anti-gravity mode is triggered by driving over specific **Antigravity Panels** (ramps or panels embedded in the track surface)
- The camera aligns to the new gravity normal automatically — no player manual adjustment
- Physics remain essentially unchanged; anti-gravity is primarily a **traversal and visual spectacle mechanic** rather than a fundamental physics shift
- Half-pipes trigger temporary anti-gravity even outside marked zones (Booster Course Pass)
- Tracks transition between anti-gravity and normal sections multiple times per lap, creating varied pacing within a single course

---

## 3. Track Design Patterns

### Core Track Elements

Based on [Game Design Skills](https://gamedesignskills.com/game-design/racing/) and [Innovecs Games analysis](https://www.innovecsgames.com/blog/racing-game-mechanics/):

#### Boost Pads / Zippers
- Place on alternate routes to reward risk-taking
- On straightaways: break monotony without requiring lane changes
- Size and color should communicate power level (small pad = short boost, large platform = long boost)
- In SuperTuxKart, these are called **Zippers** and are fully configurable: `Zipper duration`, `Zipper max speed increase`, `Zipper fade out time`, `Zipper speed gain`, `Zipper minimum speed`
- The minimum speed parameter is critical for **jump ramps** — ensures the kart clears the gap even at low approach speeds ([SuperTuxKart Physics](https://supertuxkart.net/Physics))

#### Ramps and Jump Sections
- Design the **landing area** before the ramp: if the landing is bad, no approach speed is correct
- Space jump sections so landing zones are large enough for the top speed attainable off the ramp
- In SuperTuxKart Evolution (2025), jumps are explicitly identified as underutilized elements the team wants to expand, including "driving on the side" (two-wheel balancing) per the [STK Development Blog 2025](https://blog.supertuxkart.net/2025/12/karting-without-hitch.html)

#### Transformation Triggers (Mode-Change Zones)
- Use texture-based zone designation (consistent color family for each mode: e.g., blue for water, orange for aerial)
- Add approach signage at least 3–4 track-seconds before the trigger
- In SASRT, the track *itself* transforms (e.g., a road section crumbles and becomes a water canal) — this is the gold standard for immersion but requires per-track bespoke asset production

#### Aerial Rings
- Used in SASRT's **Ring Mode** and in standard track layouts to guide aerial sections
- Ring Mode in SASRT routes players through *non-standard track paths* not visible during car racing — inside aircraft carriers, through hidden sections — demonstrating that aerial geometry can be layered over the ground track without overlap
- Design rings in clusters of 3–5 with clear visual connectors between them
- Vary ring orientation (vertical, horizontal, angled) to require 3D maneuvering

#### Landing Zones
- Must be wider than approach sections (1.5–2× width) to account for aerial drift
- Texture differentiation from normal track surface for visual clarity
- In Mario Kart 8, landing from a half-pipe exit retains anti-gravity state if the player lands in an anti-gravity zone, requiring careful zone placement coordination

#### Magnet Sections (Wall/Ceiling Driving)
- SuperTuxKart supports **Magnet Sections** via the `Affect gravity` texture property — any surface with this texture allows karts to drive on it regardless of orientation ([SuperTuxKart Physics](https://supertuxkart.net/Physics))
- Use sparingly; must be foreshadowed visually and spatially (a transition corridor before the full wall)
- The SuperTuxKart STK Evolution blog confirms new wall-riding physics are under development for the next major release

#### Cannon Shots (Forced Aerial Trajectory)
- STK supports **Cannon mechanics**: a start line launches a kart along a Bézier curve path to a defined landing position
- Configurable: `Flight end line`, `Path` (Bézier curve), `Speed`
- Use for dramatic set-pieces — the kart has no input during flight, so arrival at the cannon must be unmistakable and consensual
- Ideal for spectacle moments at track midpoints, not for lap-critical sections

### Track Layout Composition

From the [WRC7 Level Design case study](https://www.gamedeveloper.com/design/racing-level-design-the-rally-case):

**The Landmark Model:**
1. Identify 1–3 memorable landmarks (a crumbling bridge, a volcanic vent, a forest clearing)
2. Design approach and exit roads for each landmark to maximize its impact
3. Connect landmarks with secondary roads that vary in character (smooth vs. rough, wide vs. narrow)
4. The player's mental map follows the sequence: *enter section → reach landmark → exit → anticipate next landmark*

**Elevation Design:**
- Changing height affects both gameplay (lighter vehicles benefit on downhill) and cinematics (camera tilts reveal vistas at crests)
- Combine uphill sections (building tension) with downhill straights (release) for emotional pacing
- For aerial sections: mark altitude boundaries visually (ring gates, colored fog bands) so pilots understand the z-axis envelope

**Track Width Strategy:**

| Width | Effect |
|-------|--------|
| Wide | Encourages multiple racing lines, favors overtaking |
| Narrow | Combative racing, contact more likely, demands precision |
| Variable | Creates strategic choke points and breathing zones |

### Multi-Mode Track Structure Template

```
[Start Gate]
  → Wide ground section: drifting, boost pad rows, item boxes
  → Medium ramp sequence: air transition trigger, 2-3 aerial rings, landing zone
  → Water/aerial section: mode-specific challenge (waves, rings, 3D maneuvering)
  → Mode-reversion zone: vehicle transforms back on designated surface touch
  → Technical ground section: tight turns, elevation change, landmark feature
  → Wide straightaway: speed recovery, item placement
  → Final approach to lap line
[Lap Line]
```

---

## 4. Procedural Track Generation

### Core Algorithms

#### Convex Hull Method (Most Common Approach)
Described in [Radu Angelescu's procedural track generation post](https://raduangelescu.com/post/racetrackproceduralgeneration/) and [Statox PCG notes](https://www.statox.fr/posts/2021/10/race_generator/):

1. Generate random control points in a bounded plane
2. Compute the convex hull (the minimal polygon wrapping all points)
3. Perturb hull vertices randomly (including some very large-radius outliers for dramatic curves)
4. Apply moving-average smoothing (3-point centered) over N iterations
5. Generate track borders by offsetting the centerline perpendicularly by track half-width
6. Build physics representation from border geometry

**Key tuning parameters:**
- `track_size`: Number of control points
- `down_step`: Sampling interval for initial control points
- `radius_curvature`: Min/max range for point offset from center
- `hard_curvature_probability`: Probability of a very large radius (creates dramatic turns)
- `smooth_iterations`: Number of smoothing passes

#### Search-Based PCG with Segment Sequences
From the [academic paper on search-based PCG for race tracks (Semanticscholar PDF)](https://pdfs.semanticscholar.org/ba6e/0e5394fdf6242766f17709619d1f8a80ddb9.pdf):

- Represent a track as a **sequence of parameterized segments** (straight or curved, each with configurable length, angle, banking)
- Use **multi-objective fitness function** evaluating: track validity (closed, continuous, non-intersecting), variety (turn distribution), speed profile compatibility with vehicle specs
- Search algorithms: Tabu Search, Genetic Algorithm (tested in TORCS simulator)
- GA outperforms Tabu Search in generating fun tracks per human evaluation

#### Grid-Based Backtracking (Rapid, ~0.01s per track)
From a [Reddit procedural generation showcase (2024)](https://www.reddit.com/r/proceduralgeneration/comments/1c5kfe8/complete_procedural_race_track_generator/):

1. Define pre-built segment types on a grid (straight, small 90° turn, large 90° turn)
2. Generate a base shape (circle, figure-8, etc.) as a scoring attractor
3. Place segments via backtracking, scoring each by: distance from start, proximity to base shape
4. Terminate early and restart on timeout (very fast with different RNG seed)
5. Post-process: convert to single path, apply smoothing, add banking, elevation changes
6. Convert path to mesh

**Advantages**: Very fast (suitable for real-time generation), produces guaranteed-valid circuits, easily extensible with new segment types.

#### 6-Axis Generation (3D Tracks with Inversions)
From [Reddit 6-axis track generation (2024)](https://www.reddit.com/r/proceduralgeneration/comments/1azbw5l/updated_my_6_axis_race_track_generation_algorithm/):
- Pathfinding through a sparse 3D voxel map
- Generate waypoints, pathfind between them avoiding self-intersection
- Smooth results
- Supports looping, twisting, inverting tracks (roller coaster-style)
- Difficulty tuned at the generation algorithm level, not post-hoc

#### Metaheuristic Comparison
From the [IJAMC comparative study of metaheuristics (2024)](https://www.igi-global.com/article/comparative-analysis-of-metaheuristic-algorithms-for-procedural-race-track-generation-in-games/350330):

| Algorithm | Strengths | Weaknesses |
|-----------|-----------|------------|
| Genetic Algorithm (GA) | Broad exploration, well-studied | Slower convergence |
| Artificial Bee Colony (ABC) | Improved exploration | Less racing-specific validation |
| Particle Swarm Optimization (PSO) | Faster convergence, collective memory | Risk of local optima |

### Multi-Mode Extension: Procedural 3D Tracks

For drive/fly/jump games, extend the 2D track generator to 3D:
- **Aerial segments** are generated as spline arcs departing from and returning to the ground plane
- **Mode transition points** are placed at altitude inflection points (ramp launch, ring gate exit, water entry)
- **Constraint propagation**: aerial sections must be bounded by flight ceiling (maximum altitude), ring placement must respect minimum clearance from track surface
- **Fitness function additions**: penalize transitions that occur too close together (< 5 seconds of mode experience), reward transitions at dramatic geometry moments (hilltops, cliff edges, water bodies)

---

## 5. Level Design Tools and Editors

### TrackMania Track Editor

[TrackMania (2020) uses a block-placement editor](https://www.youtube.com/watch?v=FnT9gvrINvE):

**Core workflow:**
1. Place a Start block → place one or more Finish blocks → track becomes valid
2. Paint blocks with left-click-drag in one direction; editor auto-connects corners
3. Track sections snap to each other on a grid; curves are auto-inserted at perpendicular joins
4. Air Block Mode: enables floating track sections not grounded to the terrain
5. Signs and custom images for track decoration
6. Colored lamp placement
7. Ghost mode and free mode for object placement

**Key design primitives:**
- The editor operates on a **block palette system** — each block is a fixed shape (straight, corner, loop, etc.)
- No node-based scripting; all logic is baked into block types
- The [KatsBits tutorial series](https://www.katsbits.com/site/basic-nations-track-editing/) documents creating circles (perpendicular drag chains), double circles (cross-section overlaps), and crosier shapes

**Community-driven evolution:** The [Reddit suggestion thread (2020)](https://www.reddit.com/r/TrackMania/comments/kbzhll/suggestion_for_a_future_trackmania_game_node/) advocates for a **node-based track editor** in a future TrackMania, allowing:
- Arbitrary rotation and banking per node (not constrained to grid increments)
- Near-roller-coaster design freedom
- This represents an unsatisfied design space that could differentiate a new racing game's editor

### SuperTuxKart Track Editor (Blender-Based)

SuperTuxKart uses Blender as its track creation environment with a custom exporter. All professional-grade tracks in STK are made this way ([STK Making Tracks](https://supertuxkart.net/Making_Tracks)).

**Complete workflow:**
1. Concept art → layout sketch (2D map)
2. Blender: Create road segment plane (10–12 units wide), Bézier curve for path
3. Array modifier (Fit Curve) + Curve modifier on segment → track follows curve automatically
4. Toggle Cyclic on curve → closed loop track
5. Apply modifiers → editable geometry
6. Create **driveline** (series of connected quads marking where AI drives, [Drivelines docs](https://supertuxkart.net/Making_Tracks:_Drivelines_and_Checklines))
7. Create **checklines** (line segments for lap counting and shortcut prevention)
8. Add **items** (nitro, bananas, item boxes — placed as Empty objects with STK type properties)
9. Configure **physics textures** (zippers, magnet sections, slowdown zones, reset zones)
10. Add **scripting** (AngelScript: action triggers, collision callbacks, onStart, onUpdate)
11. Add **special effects** (particles, sound emitters, animated textures, billboards)
12. Export via STK Blender plugin → test in-game

**Supported scripting triggers ([STK Scripting](https://supertuxkart.net/Scripting)):**
- `onStart()` — runs once at track load
- Action Triggers — radius-based point triggers (`createTrigger`, `setTriggerReenableTimeout`)
- `onKartKartCollision(idKart1, idKart2)` — kart-to-kart collision callbacks
- `onKartObjectCollision` — defined per-object in `scene.xml` (`on-kart-collision` property)
- `onItemCollision` — when a powerup hits an object (`on-item-collision` property)

**Physics texture system ([STK Physics](https://supertuxkart.net/Physics)):**

| Feature | Effect | Configuration |
|---------|--------|---------------|
| Zipper | Speed boost | Duration, max speed increase, fade time, speed gain, minimum speed |
| Magnet Section | Drive on walls/ceilings | `Affect gravity` checkbox per texture |
| Slowdown | Reduces speed (off-road) | `Enable Slowdown` per texture |
| High Adhesion | Extra grip | `High tires adhesion` per texture |
| Ghost Material | Visible but passable | `Ignore (ghost material)` |
| Cannon | Launches kart along Bézier path | Start line, End line, Path curve, Speed |
| Reset Player | Returns kart to track on collision | Object interaction type |
| Knock Player | Sends kart airborne | Object interaction type |
| Flatten Player | Flattens kart (flyswatter effect) | Object interaction type |

**Item placement ([STK Items](https://supertuxkart.net/Making_Tracks:_Items)):**
- Use large nitros to reward alternate routes
- Use bananas on straightaways to add decision-making
- Place item rows across track width for equal access from all positions
- Use items as navigation markers where track clarity is low

### Mario Maker-Style Approaches

Mario Maker's approach demonstrates two important editor concepts:
1. **Direct manipulation UI**: Place objects by dragging from a palette to any grid position
2. **Immediate playtesting**: Switch between edit and play mode instantly
3. **Automatic logic inference**: The editor enforces game rules (course must be completable) as part of validation

Applied to racing: a racing level editor could auto-validate that:
- Every track has at least one start and one finish
- Aerial sections have matching launch ramps and landing zones
- Mode transition triggers are bidirectional or explicitly one-way

---

## 6. SuperTuxKart Track Format and Modding

### File Structure

A complete SuperTuxKart track consists of:

```
tracks/
  my_track/
    my_track.blend         # Source Blender file
    my_track.spm           # Exported mesh (SPM = SuperTuxKart Polygon Model)
    scene.xml              # Track metadata, scripted objects, cannon definitions
    scripting.as           # AngelScript logic file
    materials.xml          # Physics/interaction overrides per texture
    track.xml              # Track properties (name, laps, sky, music, etc.)
    textures/              # Track-specific textures
```

### Track Properties (track.xml)

Configurable properties include: track name, groups (category), number of laps, sky model, ambient light, music, reverse mode availability.

### Scaling

- Track width: ~12 Blender units (typical), up to ~18 (wide sections), down to ~8 (narrow corridors) — per [Making Tracks: Notes](https://supertuxkart.net/Making_Tracks:_Notes)
- Polycount limit: ~290,000 tris maximum (Cocoa Temple at 295,058 is the high-water mark)
- Typical track: ~230,000–240,000 tris

### Secondary Drivelines (Alternate Routes/Shortcuts)

Each track has one main driveline (a loop of quads). Secondary drivelines branch off and rejoin, enabling:
- AI pathfinding across shortcuts
- Independent item placement per route
- Multiple racing lines for different modes (a mode-specific path can be a secondary driveline)

### Modding Capabilities

- Full community modding via Blender + the STK Blender plugin ([GameBanana community](https://gamebanana.com/tuts/18871))
- Tracks submitted to the official add-on server
- Scripting allows: dynamic animations, multi-state puzzles, custom challenge modes, interactive objects, transformation-like effects (via teleports + speed triggers)
- [STK Development Blog 2025](https://blog.supertuxkart.net/2025/) confirms STK Evolution (next major release) will include: improved jump physics, wall-riding, improved collision handling, new tracks (Drainage Dash, Freytra Peaks)

### SuperTuxKart Evolution (2025–2026)

Key new mechanics relevant to multi-mode design:
- **Improved jump physics**: Kart stabilization during air, steering continues during impulse
- **Wall-riding**: Kart can balance on side wheels (new physics interaction)
- **Improved collision**: Lateral impulse without unpredictable direction changes
- Donation package: Early access to new STK Evolution tracks for €5+

---

## 7. Reference Game Analysis

### Sonic & All-Stars Racing Transformed (SASRT, 2012)

**Source:** [Sonic Wiki Zone – SASRT](https://sonic.fandom.com/wiki/Sonic_%26_All-Stars_Racing_Transformed), [Adam Z's analysis](https://www.adamzwakk.com/games/sonic-racing-transformed/), [Game Design Skills](https://gamedesignskills.com/game-design/racing/)

**Mode system:**
- Three modes: **Car** (land), **Boat** (water), **Plane** (air)
- Transformation is *automatic* based on track surface — players do not trigger it manually
- All three modes support drifting and boost charging
- **Transform Boost**: timing a trick *during* the transformation animation yields a speed boost — a high-skill reward with zero punishment for missing it

**Land mechanics:**
- Standard drifting; drift charges boost
- Flat and banked corners; obstacles and item boxes

**Boat mechanics:**
- Waves slow movement baseline but function as ramps for skilled players
- Drift available in boat form (different feel from car drift)
- Wide, open water sections allow for multiple lines

**Plane mechanics:**
- Full 3D movement (vertical axis available)
- Multi-directional barrel rolls: reposition + brief speed boost if timed before hitting an obstacle
- **Ring Mode** (side mode): routes players through *hidden aerial geometry* not visible during standard races — inside aircraft carriers, through underwater passages viewed from above, etc.

**Track transformation design:**
- Tracks *physically transform* each lap in many cases (e.g., the road crumbles into a river, the surface shifts from ground to sky)
- Means players see entirely different track geometry on laps 2 and 3
- This requires bespoke asset production per transformation state — high cost but maximum immersion

**Known issues (design lessons):**
- "Collision jank" at high aerial speed: invisible geometry ceiling caps interrupted flight unexpectedly
- Some aerial sections too straight, lacking 3D challenge — contrast with boat sections that made more use of lateral space

### Diddy Kong Racing (1997)

**Source:** [Reddit gamedesign thread on DKR multi-vehicle design](https://www.reddit.com/r/gamedesign/comments/aytq3k/did_diddy_kong_racings_track_design_have_to/)

**Mode system:** Three selectable modes — **Car**, **Hovercraft**, **Plane** — not auto-transformed; player chooses before entering a track

**Balance philosophy:**
- Some vehicles are explicitly superior on some maps — not fully balanced
- The plane circumvents certain track constraints: where a car/hovercraft faces a complex turn, a plane flies through an overhead tunnel, creating a fundamentally different (simpler) racing line
- This design choice trades balance for differentiation: each mode has unique track sections inaccessible or impractical to other modes

**Design lesson:** Forced imbalance can create identity — each vehicle "owns" certain moments rather than all vehicles being approximately equal everywhere. This is a valid alternative to the SASRT approach of balanced multi-mode.

**Multi-path architecture:** The three vehicle types naturally produce 3 distinct paths through the same world space, enabling triple the content per track without triple the asset production.

### Mario Kart 8 Anti-Gravity

**Source:** [Super Mario Wiki – Anti-gravity](https://www.mariowiki.com/Anti-gravity), [YouTube analysis](https://www.youtube.com/watch?v=EH6kqDXbvb4)

**Trigger system:** Antigravity Panels embedded in track surface; driving over them activates anti-gravity mode; driving off the panel/section returns to normal gravity

**Visual indicators:** Kart tires fold sideways and glow blue; certain vehicles show additional glow effects; camera smoothly reorients to new gravity normal

**Gameplay impact:** Primarily visual/spatial; physics essentially unchanged. The value is:
- **Traversal expansion**: Players can drive walls, ceilings, corkscrews
- **Collision bonus**: In anti-gravity, hitting bumpers along the track *boosts* the player (inverts the usual collision penalty)
- **Spectacle**: Tracks become architecture-first designs, not just road geometry

**Track percentage in anti-gravity:** 27 classic courses retrofitted with anti-gravity sections; most new courses include at least one such section; 3 courses (Mute City, Big Blue, Sky-High Sundae) are *entirely* anti-gravity

**Design integration:** MK8 uses anti-gravity sections to pace tracks into distinct acts — an anti-gravity corkscrew section typically follows a normal ground section, creating a transition beat

---

## 8. Level Design Studio Concepts

### Node-Based Track Editors

The gap identified in existing tools: TrackMania uses a block palette (limited rotation), SuperTuxKart uses Blender (full freedom, high barrier). The ideal **node-based editor** concept (from [Reddit TM suggestion](https://www.reddit.com/r/TrackMania/comments/kbzhll/suggestion_for_a_future_trackmania_game_node/)):

**Architecture:**
- Each **node** represents a cross-section of the track at a point in 3D space
- Nodes carry: position, rotation, banking angle, width, height, surface type, mode-trigger flag
- Connecting nodes interpolates the track geometry between them (Catmull-Rom or Bézier interpolation)
- Bank angle per node enables smooth camber variation

**Advantages over block editors:**
- Near-arbitrary shapes without grid constraint
- No invisible geometry "glue" artifacts between blocks
- Track cross-section can vary continuously (narrow entrance to wide landing zone)
- Mode triggers embedded per node segment

**Visual scripting for track logic:**
A track's interactive elements (boost pads, transformation triggers, cannon launch points, item spawn rules) can be expressed as a node graph:
- **Trigger Node**: spatial event (enter zone, collision with object)
- **Condition Node**: checks state (mode == flying, speed > threshold, lap > 1)
- **Effect Node**: applies result (play sound, activate zipper, spawn item, trigger cannon)
- **Sequence Node**: chains multiple effects with delays

This is essentially what STK's AngelScript system implements in code; a visual version would lower the barrier for non-programmers.

### Level Design Workflow Recommendations

From aggregating [WRC7 design process](https://www.gamedeveloper.com/design/racing-level-design-the-rally-case), [Game Design Skills methodology](https://gamedesignskills.com/game-design/racing/), and community practice:

**Phase 1: Concept**
1. Define the track's emotional identity and narrative context
2. Sketch 2D layout in Drawio or on paper
3. Identify 1–3 landmark moments (the aerial canyon, the underwater tunnel, the rooftop highway)
4. Mark mode transition points on the sketch (ground → air, air → water, etc.)

**Phase 2: Greybox**
1. Model track in Blender with no art — pure geometry
2. Define driveline and checklines
3. Add all physics textures (zippers, magnets, cannons, reset zones)
4. Playtest at full speed — evaluate transition timing, mode section length, feel

**Phase 3: Iteration**
1. Adjust mode section lengths based on playtest data
2. Balance item placement across routes and modes
3. Tune zipper parameters to match section speeds
4. Verify AI driveline navigates all modes correctly

**Phase 4: Art**
1. Add geometry detail with art in place (crucial — experience changes radically with environment)
2. Place landmarks (visual and gameplay)
3. Finalize lighting, particles, sounds
4. Final playtest with full art

---

## Sources

| Source | URL |
|--------|-----|
| Game Developer – Racing Level Design: The Rally Case | https://www.gamedeveloper.com/design/racing-level-design-the-rally-case |
| Game Design Skills – Racing Game Design | https://gamedesignskills.com/game-design/racing/ |
| Super Mario Wiki – Anti-gravity | https://www.mariowiki.com/Anti-gravity |
| YouTube – How Anti-Gravity Affects Track Design | https://www.youtube.com/watch?v=EH6kqDXbvb4 |
| Sonic Wiki Zone – Sonic & All-Stars Racing Transformed | https://sonic.fandom.com/wiki/Sonic_%26_All-Stars_Racing_Transformed |
| Adam Z's Notes – SASRT Analysis | https://www.adamzwakk.com/games/sonic-racing-transformed/ |
| Reddit – Sonic Crossworlds vs Transformed | https://www.reddit.com/r/SonicCrossWorlds/comments/1o8othx/in_ur_opinion_what_are_the_biggest_differences/ |
| Reddit – Diddy Kong Racing multi-vehicle design | https://www.reddit.com/r/gamedesign/comments/aytq3k/did_diddy_kong_racings_track_design_have_to/ |
| Innovecs Games – Racing Game Mechanics | https://www.innovecsgames.com/blog/racing-game-mechanics/ |
| Radu Angelescu – Procedural Race Track Generation | https://raduangelescu.com/post/racetrackproceduralgeneration/ |
| Statox – Attempt at Procedural Race Generation | https://www.statox.fr/posts/2021/10/race_generator/ |
| Reddit – 6-Axis Race Track Generation | https://www.reddit.com/r/proceduralgeneration/comments/1azbw5l/updated_my_6_axis_race_track_generation_algorithm/ |
| Reddit – Complete Procedural Race Track Generator | https://www.reddit.com/r/proceduralgeneration/comments/1c5kfe8/complete_procedural_race_track_generator/ |
| Semanticscholar PDF – Search-based PCG for Race Tracks | https://pdfs.semanticscholar.org/ba6e/0e5394fdf6242766f17709619d1f8a80ddb9.pdf |
| IJAMC – Comparative Metaheuristics for PCG | https://www.igi-global.com/article/comparative-analysis-of-metaheuristic-algorithms-for-procedural-race-track-generation-in-games/350330 |
| SuperTuxKart – Making Tracks (index) | https://supertuxkart.net/Making_Tracks |
| SuperTuxKart – Making Tracks: Notes | https://supertuxkart.net/Making_Tracks:_Notes |
| SuperTuxKart – Making Tracks: Modeling | https://supertuxkart.net/Making_Tracks:_Modeling |
| SuperTuxKart – Making Tracks: Drivelines and Checklines | https://supertuxkart.net/Making_Tracks:_Drivelines_and_Checklines |
| SuperTuxKart – Making Tracks: Items | https://supertuxkart.net/Making_Tracks:_Items |
| SuperTuxKart – Physics | https://supertuxkart.net/Physics |
| SuperTuxKart – Special Effects | https://supertuxkart.net/Special_Effects |
| SuperTuxKart – Scripting | https://supertuxkart.net/Scripting |
| SuperTuxKart Development Blog 2025 | https://blog.supertuxkart.net/2025/12/karting-without-hitch.html |
| KatsBits – TrackMania Nations Track Editing Tutorials | https://www.katsbits.com/site/basic-nations-track-editing/ |
| YouTube – TrackMania 2020 Map Editor Tutorial | https://www.youtube.com/watch?v=FnT9gvrINvE |
| Reddit – Node-based TrackMania editor suggestion | https://www.reddit.com/r/TrackMania/comments/kbzhll/suggestion_for_a_future_trackmania_game_node/ |
| Game Developer – Level Design Patterns in 2D Games | https://www.gamedeveloper.com/design/level-design-patterns-in-2d-games |
