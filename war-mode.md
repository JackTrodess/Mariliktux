# WAR Mode Research: Vehicular Combat Game Design for Mariliktux
*Research compiled: March 18, 2026*

---

## TABLE OF CONTENTS

1. [Vehicular Combat Weapon Systems](#1-vehicular-combat-weapon-systems)
   - 1.1 Twisted Metal Weapon Design
   - 1.2 Mario Kart Battle Mode Weapon Balance
   - 1.3 Destruction AllStars Combat Mechanics
   - 1.4 Crossout / War Thunder Arcade Vehicle Combat
   - 1.5 Weapon Categories Summary
   - 1.6 Weapon Balance for Transformer Vehicles (Tri-Mode)
   - 1.7 Ammo Systems vs. Cooldown Systems
   - 1.8 How STK Already Handles Items
2. [PvPvE Design](#2-pvpve-design)
   - 2.1 PvPvE Mechanics in Fast-Paced Games
   - 2.2 NPC Enemies as Quick-Point Targets
   - 2.3 NPC Wave Spawning in PvP Contexts
   - 2.4 Balancing PvE Difficulty Against PvP
   - 2.5 Respawn Mechanics for Fast-Paced Vehicular Combat
3. [Game Mode Designs](#3-game-mode-designs)
   - 3.1 SKAT Mode (1v3) — Asymmetric Multiplayer
   - 3.2 REVERSI / Tag Mode — Escalating Infection
   - 3.3 TEAM DEATHMATCH
   - 3.4 BOWL Mode — Free-for-All / Battle Royale Lite
4. [Arena Design for Combat](#4-arena-design-for-combat)
   - 4.1 Arena vs. Track Design Differences
   - 4.2 Multi-Level Arenas for Transformer Vehicles
   - 4.3 Destructible Environments
   - 4.4 Power-Up Placement Strategy
   - 4.5 Spawn Point Design to Prevent Spawn Camping
5. [Scoring and Progression in WAR Mode](#5-scoring-and-progression-in-war-mode)
   - 5.1 Kill/Death/Assist Scoring
   - 5.2 Streak Rewards and Comeback Mechanics
   - 5.3 WAR Mode Rankings and Leaderboards
   - 5.4 Integration with Existing Reward Systems

---

## 1. VEHICULAR COMBAT WEAPON SYSTEMS

### 1.1 Twisted Metal Weapon Design

Twisted Metal is the definitive reference for vehicle-specific combat weapons. Its design philosophy is well-documented by the [Twisted Metal Wiki](https://twistedmetal.fandom.com/wiki/Special_Weapon) and [IGN's guide](https://www.ign.com/wikis/twisted-metal/Special_Weapons).

**Core Principles:**

- **Vehicle Identity through Weapons**: Every vehicle has one or two Special Weapons tied directly to the character's personality and vehicle design. Sweet Tooth (the clown ice cream truck) fires a Napalm Cone and can transform into Sweet Bot. The Reaper (motorcycle with scythe) uses a Chainsaw for close combat or RPG for range. This identity lock-in makes vehicle selection a strategic choice, not just cosmetic.

- **Weapon Categories in Twisted Metal**:
  | Category | Example | Behavior |
  |----------|---------|----------|
  | Projectile | Mr. Grimm's Screaming Soul | Homing or direct-fire missile/energy bolt |
  | Energy Beam | Thumper's Megafire | Sustained damage flamethrower/laser |
  | Ram/Melee | Darkside's Darkside Slam | Vehicle mass charge, uses momentum |
  | Manipulation | Mr. Slam's Grab n' Slam | Grabs enemy, slams into ground |
  | Shockwave | Axel's Crowd Controller | AoE blast, knocks back nearby enemies |
  | Trap/Vortex | Twister's Tornado Spin | Creates area hazard pulling enemies in |
  | Remote Detonation | Junkyard Dog's Death Ball | Launch + remote explode |
  | Piloted Drone | Vermin's Piloted Rat Rocket | Player-guided sub-projectile |
  | Copy | Mime's ability | Replicates any enemy's special |

- **Ammo System**: Infinite uses, but each use triggers a **recharge cooldown** (duration varies per vehicle). Some specials require button-mashing to charge up to full power; overcharging can backfire. In *Twisted Metal (2012)*, you can hold up to **three stored specials** — once at three, recharging stops, creating an inventory-style buffer.

- **Upgrade System** (*Head-On*): After 5 kills, collect 6 upgrade icons to unlock an enhanced version of your special (e.g., faster fire rate, charge mechanics). This skill-reward loop keeps invested players engaged.

- **Balance Design**: Effectiveness deliberately varies. Some specials one-shot (Calypso's Nuke), others whittle down health. Range flaws and tracking inaccuracies counterbalance high-damage specials. Multiplayer variants add team mechanics (e.g., Junkyard Dog's Taxi Bomb becomes a Team Health Drop).

**Takeaway for Mariliktux WAR Mode**: Each transformer chassis should have a signature weapon tied to its visual form. A sports car gets high-speed missiles; a heavy truck gets a ram/slam; a fighter jet gets guided bombs; a mech gets a melee crusher. The weapon identity deepens the choice of transformer form.

---

### 1.2 Mario Kart Battle Mode Weapon Balance

Mario Kart's battle design philosophy is intentionally asymmetric — it rewards aggressive positioning, not just aim, according to the [Polygon Battle Mode guide](https://www.polygon.com/mario-kart-8-deluxe-guide/2017/4/28/15473910/battle-mode-tips-balloon-battle-bob-omb-blastrenegade-roundupcoin-runnersshine-thief/) and analysis by [YouTube Design Delve](https://www.youtube.com/watch?v=p4Mz6rKQ8Hs).

**Key Principles:**

- **Positional Item Distribution**: Players in last place receive powerful offensive items (Blue Shell equivalent), players in first place receive only defensive items. In first place, offensive items convert to defensive forms — this is a structural comeback mechanic baked into item distribution itself.

- **Two-Item Slots** (MK8 Deluxe): Players can hold two items simultaneously. This prevents defensive camping (you can't trail a shell AND use a mushroom) but increases tactical depth.

- **Item Usage Skill**: Items can be held behind the kart to deflect incoming attacks. This creates a risk/reward: use it now for offense, or trail it defensively? Feathers allow jumping over opponents to pop balloons — melee-range item usage adds spatial skill.

- **Bob-omb Blast Mode**: All players get only Bob-ombs (AoE explosion items). This single-weapon mode creates very different dynamics — everyone is equally armed, forcing positional/chase skills over item luck. Highly regarded as the "fairest" sub-mode by the community ([Reddit discussion](https://www.reddit.com/r/MarioKart8Deluxe/comments/12eiqah/is_battle_mode_balanced/)).

- **Balloon Battle Scoring Evolution**: Original MK64 used elimination (lose all balloons = eliminated). MK8 Deluxe uses timed point-based scoring with a penalty: losing all 5 balloons halves your current score, then you respawn with 3 balloons. This keeps everyone in the game (no spectating) but creates risk around protecting your last balloon, as noted by the [Super Mario Wiki](https://www.mariowiki.com/Balloon_Battle).

**Takeaway for Mariliktux WAR Mode**: Weapon distribution should vary by player health/score — players behind get better attack items, leading players get shields/evasion tools. This creates natural tension without rubber-banding AI.

---

### 1.3 Destruction AllStars Combat Mechanics

Destruction AllStars (PlayStation 5, 2021) is the most recent major vehicular combat arena game. Details from [TheGamer's guide](https://www.thegamer.com/destruction-allstars-breaker-mode-explained/) and [TheSixthAxis tips](https://www.thesixthaxis.com/2021/02/06/destruction-allstars-guide-11-essential-tips-tricks/).

**Core Design:**

- **Two-Phase Combat**: Players alternate between **in-vehicle combat** (ramming, vehicle specials) and **on-foot parkour** (collecting Gems to charge Breaker abilities). This hybrid design means total destruction of your vehicle isn't terminal — it's a tactical pivot to collect power-up gems.

- **Vehicle Classes**: Three base classes (Light, Medium, Heavy) plus unique **Hero Vehicles**. Light = high speed/control; Heavy = high durability. Each handles similarly to lower the skill floor.

- **Breaker System** (power specials):
  - **Character Breakers** (on-foot): Unique per character. Examples: Jian leaves stun/damage mines; Lupita leaves fire trails; Shift turns nearly invisible. Charged by combat damage OR collecting Gems scattered across the arena.
  - **Vehicle Breakers**: Activate Hero Vehicle — cannot be hijacked while active. Each has unique special moves (e.g., Jian's Morningstar covers in spikes to impale cars; Lupita's Wildfire leaves stronger fire trail).
  - **Charge Sources**: Red Gems (slight charge), Red Gem Clusters (full Character Breaker), Blue Gem Clusters (full Vehicle Breaker). Only on-foot players can grab gems — creating incentive to exit vehicles strategically.

- **Key Design Innovation**: The **on-foot/vehicle toggle** prevents turtling. A destroyed vehicle equals an opportunity to switch gameplay mode, not a death sentence. This is excellent for transformer vehicles.

- **Gridfall Mode**: Arena sections fall away over time (marked red). Creates natural shrinking zone pressure without an artificial storm mechanic — elegant for vehicular combat.

**Takeaway for Mariliktux WAR Mode**: The Breaker charging system is a clean model for transformer mode-specific power unlocks. Collect energy while in car mode, unlock aerial or mech special when charged. The vehicle/on-foot toggle maps directly to transformer form transitions.

---

### 1.4 Crossout / War Thunder Arcade Vehicle Combat

**Crossout** weapon design (from [Crossout Wiki](https://crossout-archive.fandom.com/wiki/Weapons) and [Reddit guide](https://www.reddit.com/r/Crossout/comments/6gb75x/vehicle_constructiondamage_basics_guide/)):

Crossout uses a modular construction system where players physically attach weapons to their vehicles. This is less relevant for a kart game but the **weapon taxonomy is extremely useful**:

| Weapon Type | Damage Type | Range | Best Use |
|-------------|-------------|-------|----------|
| Machine Guns | Bullet | Close-Medium | Strip enemy parts, sustained damage |
| Miniguns | Bullet | Close-Medium | Devastating DPS, needs spin-up time |
| Shotguns | Bullet | Close | Hit-and-run, burst damage |
| Autocannons | Bullet | Medium-Long | Accurate, automatic fire |
| Cannons | Blast | Medium-Long | High burst, limited ammo |
| Melee (Saw/Drill) | Contact | Touch | Brawling, high damage on contact |
| Unguided Rockets | Blast | Short-Medium | Saturation fire, splash damage |
| Homing Missiles | Blast | Long | Tracking, can be shot down |
| Grenade Launchers | Blast | Short-Medium | Arc trajectory, AoE on impact |
| Flamethrowers | Fire | Close | DoT (damage over time), part softening |
| Minelayers | Blast | Deployed | Defense, area denial |
| Flying Drones | Various | Auto-target | Support/distraction |
| Artillery | Blast | Very Long | Support fire behind front line |
| Energy Weapons | Energy | Medium | Accurate, delayed shot release |

Crossout's **cabin/part damage split** is notable: you must hit the Cabin to kill the vehicle (hit points), but destroying weapons/wheels disables function first. This "ablative armor" model works well for vehicular combat games.

**War Thunder Arcade Mode** ([War Thunder Wiki](https://wiki.warthunder.com/gamemode/arcade_battles)) key mechanics:
- **Boosted mobility and weapons characteristics**: Vehicles move faster, turn faster, aim faster than simulation modes. Creates fast, arcade-feel combat.
- **Aim Assist**: Crosshair shows penetration chance (green/yellow/red) and lead marker for moving targets. Lowers skill floor without removing skill ceiling.
- **Multiple Respawns**: Up to 3 ground vehicles per battle + aircraft earned from kills/assists. Creates economy around risk: dying wastes a slot.
- **Spotted Enemy Markers**: Enemies visible to any ally get floating name/vehicle tags. Reduces "fog of war" confusion in fast matches.
- **Mixed Nations**: Teams mix nationalities, avoiding faction-based imbalance.
- **PvE Layer**: AI ground units in strike missions, AI ships in naval. Provides targets for players who aren't engaging other players.

---

### 1.5 Weapon Categories Summary

For **Mariliktux WAR Mode**, the following weapon categories should be designed:

| Category | Examples | Role | Balance Lever |
|----------|---------|------|---------------|
| **Projectile (Homing)** | Homing missiles, guided rockets | All-range offensive | Countered by speed/evasion, shield |
| **Projectile (Direct)** | Cannon shells, energy beams | Skill-shot offense | High damage, requires aim |
| **Mines / Traps** | Dropped mines, bubblegum patches | Area denial | Time-delayed, avoidable with awareness |
| **Melee / Ram** | Blade bumpers, spike rams, drill | Close-range high damage | Requires approach (high risk) |
| **AoE / Shockwave** | EMP blast, explosion pulse | Multi-target hit | Short range, high cooldown |
| **Shield / Defensive** | Energy shield, reflective armor | Damage mitigation | Blocks one hit, then needs recharge |
| **Drone / Turret** | Auto-targeting gun, deployable | Passive offense | Limited charges, destroyable |

---

### 1.6 Weapon Balance for Transformer Vehicles (Tri-Mode)

Transformer vehicles have three modes: **Car** (ground), **Fly** (aerial), **Mech** (biped). Each mode should enable or restrict certain weapons:

**Car Mode:**
- Full access to ground mines and traps
- Ram damage bonus (higher mass = more damage on collision)
- Best horizontal mobility for dodge-roll evasion
- Weapons: Machine guns, rockets, cannon, mines
- Weakness: Cannot reach aerial opponents easily; slow arc weapons

**Fly Mode:**
- Access to aerial bombs and guided air-to-ground missiles
- Higher speed, but reduced armor (trade-off)
- Can strafe ground targets; evasion via altitude and speed changes
- Weapons: Air-to-ground bombs, speed missiles, EMP altitude blast
- Weakness: Mines/traps irrelevant; Mech mode players can still shoot upward

**Mech Mode:**
- Access to melee weapons (fist slam, blade) dealing highest per-hit damage
- Slowest speed but highest armor
- Can grab/throw objects (including other vehicles if mechanic allows)
- Weapons: Melee slams, short-range shotgun blast, deployable turrets
- Weakness: Slowest movement; most vulnerable to homing missiles

**Mode-Transition Weapons**: Consider a "Transform Attack" — the transition animation itself deals damage to any nearby vehicle. This discourages spam-transforming while making the timing of transformation tactical.

**Weapon Behavior Matrix:**

| Weapon | Car Mode | Fly Mode | Mech Mode |
|--------|----------|----------|-----------|
| Homing Missile | Full damage | Reduced (too fast to aim lock) | Full damage |
| Mine | Drop behind | Cannot deploy | Can throw forward |
| Ram | Full bonus | No bonus (no mass impact) | Double bonus (heavier) |
| EMP Blast | Full | Half radius | Full |
| Drone | Full | Cannot deploy | Full |
| Aerial Bomb | Cannot use | Full | Cannot use |

---

### 1.7 Ammo Systems vs. Cooldown Systems

**Ammo Systems:**
- Examples: Twisted Metal shared pickup pool, Crossout limited cannon ammo
- Design implications:
  - Forces pickup hunting — creates natural movement incentives
  - Scarcity tension: save ammo for high-priority targets
  - Risk: Players may turtle until ammo refills
  - Best for: Single-weapon loadouts, slower-paced arenas

**Cooldown Systems:**
- Examples: Twisted Metal special weapons (infinite ammo, recharge timer), Destruction AllStars Breakers
- Design implications:
  - Never "run out" — keeps pace high
  - Timing skill becomes primary (when to use, not whether)
  - Can stack multiple cooldowns for burst windows
  - Best for: Fast-paced arcade modes, multi-weapon designs

**Hybrid Recommendation for WAR Mode:**
- **Signature Special Weapon** (per vehicle/form): Cooldown-based, always available. Short cooldown (8-15 seconds). This is the "identity weapon."
- **Pickup Weapons** (from arena boxes): Ammo-based, powerful but limited (2-5 shots). Incentivizes arena traversal and pickup control.
- **Ram/Melee**: No ammo/cooldown — always available on contact. Makes brawling viable even when empty.
- This hybrid creates distinct tactical layers: identity weapon rhythm, pickup management, always-available melee fallback.

---

### 1.8 How STK Already Handles Items

SuperTuxKart's item system is well-documented in its [pystk API](https://pystk.readthedocs.io/_/downloads/en/latest/pdf/), [Speedrun guide](https://www.speedrun.com/stk/guides/hpig5), and [development blog](https://blog.supertuxkart.net/2011/):

**Existing STK Powerup Types:**
| Item | Type | Mechanic |
|------|------|----------|
| **Cake** | Projectile | Auto-aims, compensates for speed differential, forward throw |
| **Bowling Ball** | Projectile | Bouncing shot; can boomerang back; targets leading kart |
| **Rubber Ball** | Chasing Projectile | Bounces along track, targets lead kart, slows on hit |
| **Bubblegum** | Trap | Drop behind; also blocks one incoming item (used as shield); on ground creates slowing trap |
| **Bubblegum Shield** | Defensive | Wraps kart in bubble shield; flashes before expiring |
| **Plunger** | Debuff | Shot forward: latches to target; shot backward: blinds pursuer |
| **Bomb** | Explosive | Sticky bomb passed by contact; ticking timer; explodes when timer runs out |
| **Swatter** | Melee | Mounted arm; hits nearby karts (squashes, halves top speed); removes parachutes/bombs from own kart |
| **Nitro** | Speed Boost | Speed boost; stackable; race and battle modes |
| **Zipper** | Speed Boost | Track-specific boost strip |
| **Parachute** | Debuff | Attached to target, dramatically reduces speed |
| **Switch** | Utility | Converts bananas↔boxes, gum↔big nitro tanks |
| **Anvil** | Attachment Penalty | Heavy weight attached to kart; slows |

**STK Battle Mode Specifics:**
- **3 Strikes Battle**: Each kart has 3 lives (represented as spare tires). Lose all 3 = eliminated.
- **Free for All**: Points for eliminations. 
- **Capture the Flag**: Online mode, team-based flag capture.
- Items regenerate in gift boxes scattered around arenas.
- Random spawn point in local battle mode (since recent update).
- Navmesh required for arena AI navigation.

**Gaps in STK for WAR Mode:**
1. No aerial-specific weapons (all items assume ground plane)
2. No mode-transition weaponry
3. No vehicle-specific identity weapons (all characters share same item pool)
4. No shield/absorb mechanics beyond Bubblegum Shield
5. No homing missile class (Rubber Ball is semi-homing but not weaponized)

---

## 2. PvPvE DESIGN

### 2.1 PvPvE Mechanics in Fast-Paced Games

PvPvE (Player vs. Player vs. Environment) mixes active human opponents with AI enemies in the same arena. In **arcade** contexts (unlike the slow extraction-shooter PvPvE of Escape from Tarkov), the design goal is:
- NPCs are **quick-point targets**, not main threats
- NPCs **disrupt** the PvP equilibrium without dominating it
- NPCs create **movement incentives** (go to NPC spawn zones to earn points)

Key design principles from game design research ([Reddit gamedesign thread](https://www.reddit.com/r/gamedesign/comments/vtgkbu/tips_on_enemy_spawn_logic_and_enemy_game_balance/)):
- NPCs should have **predictable attack patterns** — fast-paced players can dispatch them in 1-3 seconds
- NPC health should be **significantly lower** than player health (20-30% of player HP)
- NPC damage should **tickle** rather than threaten (perhaps 5-10% of a player's max HP per hit)
- NPC value: Easy points that become **harder to collect** as more players contest the zone

**Arcade PvPvE Reference — War Thunder's AI units**: [War Thunder's Arcade mode](https://wiki.warthunder.com/gamemode/arcade_battles) places AI cargo ships and AI ground units on the same battlefield as PvP players. Destroying AI ships depletes the enemy team's "tickets" (shared HP pool). This creates a dual imperative: fight players AND destroy AI targets for team score.

---

### 2.2 NPC Enemies as Quick-Point Targets

**Design Framework for WAR Mode NPCs:**

| NPC Tier | Health | Speed | Point Value | Spawn Location |
|----------|--------|-------|-------------|----------------|
| **Drone** (easiest) | 1 hit | Stationary/slow patrol | 5 pts | Edges of arena |
| **Patrol Bot** | 2-3 hits | Moving route | 15 pts | Mid-arena corridors |
| **Mini-Boss Bot** | 8-10 hits | Aggressive toward nearest player | 50 pts | Arena center, timed spawn |
| **Boss Vehicle** | High HP | Full combat AI | 150 pts | Special event spawn, 1 per round |

**NPC Mechanics to Consider:**
- **Contested NPCs**: When multiple players engage the same NPC, the killing blow claims the points. This creates a "KS culture" that's actually desirable — it pulls players toward each other.
- **NPC Drops**: Defeated NPCs drop pickup weapons (ammo-based). This incentivizes NPC hunting but also creates ambush opportunities.
- **NPC Aggro**: NPCs should **prioritize the player leading the scoreboard**. This creates natural "boss on the leader" handicapping pressure.

---

### 2.3 NPC Wave Spawning in PvP Contexts

Wave-based NPC spawning must be calibrated to avoid overwhelming PvP with PvE noise. Per [Unreal Engine wave spawning research](https://www.youtube.com/watch?v=OWV4S_HI1xk) and game design principles ([Reddit](https://www.reddit.com/r/gamedesign/comments/vtgkbu/tips_on_enemy_spawn_logic_and_enemy_game_balance/)):

**Wave Timing Design:**
- **Spawn waves every 30-45 seconds** (not continuous) — gives PvP breathing room
- **Small group sizes**: 2-4 NPCs per spawn point; never more than 10% of total player count at once
- **Wave ramp-up**: Early waves have only drones; mid-game adds patrol bots; late-game spawns a mini-boss
- **Respawn throttle**: If player count in arena is below 3, suppress NPC spawns entirely — PvP should dominate when numbers are small
- **Visual/audio tells**: Announce NPC wave with siren/alarm — gives players the choice to disengage from PvP and hunt NPCs

**Spawn Location Strategy:**
- Spawn NPCs at **edge zones**, never at center
- NPCs **move toward center** over time — creates pressure convergence
- NPCs never spawn within 50 meters of a player (prevents ambush from behind by AI)

---

### 2.4 Balancing PvE Difficulty So NPCs Don't Overshadow PvP

The core risk of PvPvE is NPCs becoming too powerful or too numerous, reducing the match to a PvE grind. Key balance levers from [game design discussions](https://www.reddit.com/r/gamedesign/comments/11atvrt/how_do_you_balance_pvp_combat_in_predominantly/):

**Rule of Thumb: NPCs should never do more than 25% of any player's maximum HP per engagement.**

**Behavioral Constraints:**
- NPCs never chase a player beyond a set zone radius (50m). They don't follow you across the arena.
- NPCs never fire at airborne (Fly mode) targets — ground-only AI. This makes aerial form a NPC escape valve.
- NPCs have no respawn system. Each NPC wave is finite; points scale to wave difficulty.

**Economic Balance:**
- NPC kills should be worth **50-70% of a player kill**. Player combat remains primary.
- Too low: NPCs ignored entirely (wasted design effort)
- Too high: Players avoid each other to farm NPCs

---

### 2.5 Respawn Mechanics for Fast-Paced Vehicular Combat

Respawn systems significantly affect combat pacing. Research from [multiplayer level design analysis](https://games.themindstudios.com/post/multiplayer-level-design-techniques/) and game mode comparisons:

**Respawn System Types:**

| System | How It Works | Best For |
|--------|-------------|---------|
| **Instant Respawn** | Respawn immediately at random safe spawn | Deathmatch, high-lethality modes |
| **Timer Respawn** | Wait 2-5 seconds, then respawn | Balanced modes; creates brief penalty |
| **Wave Respawn** | All dead players respawn simultaneously every N seconds | Team modes; creates synchronized surges |
| **Conditional Respawn** | Respawn requires teammate action or point cost | Tactical/squad modes |
| **Elimination** | No respawn; spectate until round ends | Battle royale, last-stand modes |

**Recommended for WAR Mode:**

- **SKAT Mode**: Instant respawn for the 3 regular players (penalty: lose 50% current score); the solo "SKAT" player has 1 respawn (very long 15-second cooldown to emphasize their advantage risk)
- **REVERSI Mode**: No respawn — once caught, you switch sides. Match ends when all become catchers.
- **TEAM DEATHMATCH**: Timer respawn (3 seconds) + wave override (all team members respawn simultaneously every 10 seconds if majority are dead)
- **BOWL Mode**: Timer respawn (5 seconds) in early game; last 3 players have no respawn (elimination phase)

**Anti-Spawn-Camping Techniques** ([Mind Studios guide](https://games.themindstudios.com/post/multiplayer-level-design-techniques/)):
- Spawn location selected dynamically based on **distance from enemies** (furthest safe point)
- 2-3 seconds of **spawn invincibility** (invulnerable but cannot attack)
- Spawn zones rotate — no fixed spawn camping point
- Spawn effects (flash, sound) alert nearby players to not camp; attacking spawning players triggers a brief debuff on the attacker

---

## 3. GAME MODE DESIGNS

### 3.1 SKAT Mode (1v3) — Asymmetric Multiplayer

**Concept**: One player (the "Alleinspieler" / SKAT player) is dramatically overpowered and must survive against — and defeat — three regular players. Named after the German card game Skat, where the Alleinspieler (solo player) plays alone against two defenders.

**Research: Asymmetric Multiplayer Design**

The [Complete History of Asymmetrical Multiplayer](https://www.youtube.com/watch?v=cMf2CWmjTfc) and [Evolve's post-mortem](https://www.reddit.com/r/Games/comments/8p2s2m/evolve_dev_says_4v1_caused_more_problems_than_we/) reveal key lessons:

**Evolve's Failures (4v1 Monster game, 2015):**
- If the 4-player team coordinated perfectly, the monster was helpless
- If hunters failed to coordinate, even a weak monster won
- **Lesson**: Asymmetric balance must work at ALL skill levels, not just coordinated play
- **Lesson**: The solo role needs "floor skills" — minimum power even against excellent coordination

**Dead by Daylight's Design** ([BHVR forums](https://forums.bhvr.com/dead-by-daylight/discussion/comment/3757363)):
- Target 57-60% kill rate for the killer (1 player vs 4 survivors)
- The solo role is intentionally "overpowered" to compensate for number disadvantage
- **Lesson**: 1v3 (Mariliktux SKAT) needs ~60-65% win rate for the SKAT player when all are equal skill

**Pac-Man Vs. as Model** ([TV Tropes](https://tvtropes.org/pmwiki/pmwiki.php/VideoGame/PacManVs/) and [Racketboy review](https://racketboy.com/retro/review-pac-man-vs-gamecube)):
- 1 vs 3: Pac-Man (privileged role) vs 3 Ghosts
- The 1 player has **full map visibility** (via separate screen) while the 3 have only local vision
- Information asymmetry IS the power asymmetry — not just stats
- **Lesson**: SKAT player can have different information access (radar/minimap advantage) in addition to raw power

**SKAT Mode Design Proposal:**

**The SKAT Player (Alleinspieler):**
- **3x health** vs regular players
- **+50% weapon damage** 
- **Radar advantage**: Full minimap view, can see all regular players at all times
- **Special SKAT Weapon**: Unique powerful weapon unavailable to regular players (e.g., area-effect EMP that stuns multiple players simultaneously)
- **Victory condition**: Reach a target score (e.g., 200 points) OR eliminate all 3 opponents
- **Respawn**: 1 long respawn (15 seconds) with massive point penalty (−100 pts)

**The Regular Players (Gegenspieler):**
- Standard stats
- Cannot see SKAT player on minimap (only visual detection)
- **Coordination bonus**: Standing within 30m of another regular player grants +15% damage vs SKAT player — incentivizes grouping without mandating it
- **Victory condition**: Collectively accumulate 300 points (shared pool), or reduce SKAT player to 0 respawns
- **Respawn**: Instant (but −50% current score)

**Power Asymmetry Design — the "Skat Card Game" Parallel:**
In the actual Skat card game ([Wikipedia](https://en.wikipedia.org/wiki/Skat_(card_game))), the Alleinspieler needs 61 of 120 card points to win. They have access to the **Skat** (2 bonus cards) and choose trumps. The solo player has INFORMATION ADVANTAGE (knows more cards) and STRUCTURAL ADVANTAGE (trump selection), but not raw "more cards."

Apply to Mariliktux:
- SKAT player has **form-switching advantage**: Can switch transformer modes in 1 second (regular players: 3 seconds). This information/reaction advantage mirrors the Alleinspieler's trump selection.
- SKAT player has access to **all 3 form types simultaneously** (weapons from all modes available), while regular players can only use weapons appropriate to their current form.

**Potential Problem**: If SKAT player turtles, this is boring. Solution: **Active Score Requirement** — SKAT player must score at least 20 points per 60-second window or begins taking passive damage (like Skat's time pressure mechanic).

---

### 3.2 REVERSI / Tag Mode — Escalating Infection

**Concept**: One "Catcher" player tags others; tagged players become new Catchers who must each tag TWO more to escape (or become permanent catchers if they can't). Creates a cascade: 1 → 2 → 4 potential catchers with escalating pressure.

**Research: Tag/Infection Game Modes**

**Halo Infection Mode** ([Halopedia](https://www.halopedia.org/Infection)):
- Survivors killed by Zombies switch to Zombie team. Starts with Alpha Zombies.
- Zombie advantages: 200% health, higher speed, 1-hit melee kill
- Survivor advantages: Ranged weapons, shields, teamwork
- **Last Man Standing**: Final survivor gets revealed by waypoint — extreme pressure
- **Key Design**: Exponential team growth. First zombie is manageable; 3-4 zombies overwhelm remaining survivors.

**Pac-Man Vs. Cascade Mechanic** ([Fandom page](https://pacman.fandom.com/wiki/Pac-Man_Vs.)):
- Ghost who catches Pac-Man becomes the new Pac-Man (privileged role transfers)
- The "infected" player gets a reward (role upgrade), not a penalty — this is a **positive transfer mechanic**
- **Key Design**: Catching someone is desirable because you become the powerful role, not the weak one

**REVERSI Tag Mode Design Proposal:**

The game's name (REVERSI) references the reversal of roles and the "flip" dynamic.

**Starting State:**
- 1 Catcher (special vehicle visual — glowing outline, distinct sound)
- 3-7 Runners (depending on lobby size)

**Catch Mechanic:**
- Catcher tags Runner by ramming (contact for 2+ seconds) OR hitting with a net/grab weapon
- Tagged Runner becomes a **Secondary Catcher** 
- Secondary Catchers must tag **2 people** to "escape" back to Runner status
- If a Secondary Catcher tags only 1 person and time runs out, they remain a Catcher permanently for that round

**Escalation Table:**
| Wave | Catcher Count | Situation |
|------|---------------|-----------|
| Start | 1 | Easy chase, many runners |
| Wave 2 | 2-3 (if early catches) | Runners must coordinate escape routes |
| Wave 3 | 3-5 | Arena feels crowded with catchers |
| End | Potentially all | Last runner wins if surviving |

**Catcher vs. Runner Asymmetry:**
- **Catchers**: Higher speed (+20%), visible on all minimaps, longer tagging contact window
- **Catchers**: Cannot use offensive weapons — tagging is their only mechanic (prevents projectile-tagging at range)
- **Runners**: Full weapon access to slow/stun Catchers temporarily
- **Runners**: Evasion items (bubblegum traps, smoke screens) give them counterplay

**The "REVERSI" Twist**: At 60 seconds remaining, a **Role Reversal** occurs — all Catchers temporarily become Runners (inverted), and the last surviving Runner gets to be Catcher for a final 60-second bonus round. This prevents Catcher-dominant snowballs from ending the game too early.

**Scoring:**
- Runners: +1 point per 5 seconds of surviving uncaught
- Catchers: +10 points per successful tag
- Final Runner: +100 bonus points
- Secondary Catchers who escape: +25 points

---

### 3.3 TEAM DEATHMATCH

**Concept**: Two teams of transformer vehicles fight for eliminations. Standard team combat, adapted for transformer vehicles.

**Research: Team-Based Vehicular Combat**

**Rocket League Team Scoring** ([Rocket League Wiki](https://rocketleague.fandom.com/wiki/Points)):
Rocket League's scoring recognizes non-kill contributions:
- Goals: 100 pts; Assists: 50 pts; Saves: 50 pts; Epic Saves: 75 pts
- Team-supportive actions score significantly — not just the scorer
- MVP goes to the highest-scoring player on the winning team

Key insight: **Assists and defensive actions should score visibly and significantly.** Players who feel rewarded for support continue supporting.

**World of Tanks Team Design** ([Reddit respawn discussion](https://www.reddit.com/r/WorldofTanks/comments/1d3rowr/opinion_game_modes_with_respawn_vs_no_respawn/)):
- Modes with infinite respawns (Frontline, Mars events) feel more fast-paced and engaging
- Limited-life modes create higher stakes per engagement
- **Lesson**: Use respawns for TEAM DEATHMATCH (keeps everyone active) but limit in late-game scenarios

**TEAM DEATHMATCH Design Proposal:**

**Match Format:**
- 2 teams of 2-4 players each
- 5-minute match, first team to 100 kills wins OR most kills at time end
- Respawn: 3-second timer, wave respawn every 10 seconds if multiple teammates dead simultaneously

**Scoring Actions:**
| Action | Points | Rationale |
|--------|--------|-----------|
| Kill | +10 | Primary objective |
| Assist (dealt 30%+ damage) | +5 | Rewards support |
| Destroyed enemy vehicle while under 20% HP | +3 (Clutch Kill) | Rewards skill |
| Protected teammate (block shot) | +3 | Rewards defensive play |
| NPC Kill | +6 | Equal incentive to fight NPCs |
| First Blood | +5 | Momentum starter |
| Kill Streak x3 | +5 bonus | Streak reward |
| Kill Streak Ender | +8 (Streak Break) | Comeback incentive — kill the leader |

**Team Composition:**
- No required roles — all transformer vehicles can do everything
- Emergent roles: tanks (heavy, high HP Mech-default builds) vs scouts (fast Car-default builds) vs aces (Fly-default)
- Team bonus: +15% damage vs enemy team if all living teammates are within 40m of each other (wolf-pack bonus)

**Transformer Mode Team Tactics:**
- Mixed teams should include at least one Fly mode player for aerial perspective/control
- Mech mode players naturally become "point anchors" (slow, high-damage, objective holders)
- Car mode players are primary scouts/rushers

---

### 3.4 BOWL Mode — Free-for-All / Battle Royale Lite

**Concept**: All-vs-all chaos. Named "BOWL" (like a bowling alley or battle bowl). Shrinking arena, survival points, last-transformer-standing wins. Hybrid of point accumulation and elimination.

**Research: FFA in Kart Games and Shrinking Arena**

**Mario Kart Balloon Battle FFA Analysis** ([Super Mario Wiki](https://www.mariowiki.com/Balloon_Battle) and [Reddit](https://www.reddit.com/r/mariokart/comments/1i58e85/classic_survivalstyle_balloon_battles_need_to/)):

The community has been debating Mario Kart's FFA scoring since MK Wii:

- Original MK64 style: **Elimination** — last player standing wins. Small arenas feel claustrophobic; pure survival skill.
- MK8 Deluxe: **Timed Point System** — pop balloons for points; losing last balloon halves current score, respawn with 3 (not 5).
- Community preference: The original elimination format produces higher tension, but the point system keeps everyone playing.

**Hybrid Approach (Reddit suggestion)**: 24-player drop into large arena with 5 balloons (life points). Arena shrinks at timed intervals (like a storm eye). Zone elimination adds urgency without immediately reducing player count.

**Destruction AllStars Gridfall Mode**: Sections of the arena floor fall away (marked red in advance). Players must move or die. Naturally concentrates survivors into smaller spaces — elegant shrinking mechanic for vehicular combat.

**BOWL Mode Design Proposal:**

**Phase 1 — Grand Melee (Minutes 0-2):**
- All players in full arena
- Point-based scoring (kills = points, survival time = points)
- NPCs actively present at edges as quick points
- Respawn: 5-second timer; only 2 respawns allowed

**Phase 2 — Shrinking Arena (Minutes 2-3):**
- Arena begins shrinking (floor sections fall, walls close, lava rises — environment-specific)
- Respawns reduced to 1
- Points per kill increase (+50%)
- NPC spawns stop

**Phase 3 — Death Bowl (Final 60 seconds):**
- Arena is ~30% original size
- No respawns
- Elimination only
- Massive bonus for each kill (+25 pts)
- All players visible on minimap
- Last survivor gets bonus 200 pts

**Scoring in BOWL:**

| Action | Phase 1 | Phase 2 | Phase 3 |
|--------|---------|---------|---------|
| Kill | +10 | +15 | +25 |
| Survival (per 10s) | +2 | +3 | +5 |
| NPC Kill | +6 | — | — |
| Last Survivor | — | — | +200 |
| Most Kills (at match end) | +50 | — | — |

**Transformer Mode in BOWL:**
- Fly mode players have natural advantage in Phase 3 (can avoid ground hazards)
- Balance: Aerial hits score +50% in Phase 3 because they're harder to execute
- Mech mode players can survive longer per engagement but are slow to escape shrinking zones
- Car mode is best for Phase 2 rapid repositioning

---

## 4. ARENA DESIGN FOR COMBAT

### 4.1 Arena vs. Track Design Differences

From [SuperTuxKart's arena design guide](https://supertuxkart.net/Making_Tracks:_Appendix_D:_Soccer_and_Battle_Modes) and [Mind Studios level design](https://games.themindstudios.com/post/multiplayer-level-design-techniques/):

| Dimension | Race Track | Battle Arena |
|-----------|-----------|--------------|
| **Layout** | Linear with defined start/end | Open, non-directional |
| **Goals** | Lap completion, placement | Elimination, score accumulation |
| **Width** | Constrained (2-4 vehicle widths) | Open (many vehicle widths) |
| **Verticality** | Optional, cosmetic | Strategic, affects combat |
| **Flow** | Forced (you must follow track) | Player-driven (choose your path) |
| **Density** | Designed for regular traffic flow | Designed for unpredictable chaos |
| **Obstacles** | Decorative, hazard-only | Tactical cover, flanking enablers |
| **Respawn** | Track-specific (put back on road) | Zone-based (safe spawn areas) |

Key arena design principles from the SuperTuxKart documentation:
- Arenas must have **ramps, obstacles, tunnels, and ridges** for dynamism
- Multiple paths are essential for battle (flanking, escape routes)
- Soccer arenas should not be too path-heavy (clarity needed for ball tracking)
- **Navmesh** (AI navigation layer) must cover all drivable surfaces

---

### 4.2 Multi-Level Arenas for Transformer Vehicles

For Mariliktux specifically, arenas must support three combat planes: **Ground** (Car mode), **Air** (Fly mode), **Platform** (Mech mode). This is a unique design challenge.

**Vertical Level Design Framework:**

**Ground Level (Car Mode Primary):**
- Wide open floors with ramps and obstacles
- Tunnel systems (low ceiling = Fly mode can't enter, creates Car/Mech zones)
- Pickup items concentrated here (incentivizes ground play)
- Spawn points exclusively at ground level

**Aerial Layer (Fly Mode Primary):**
- Open air above the arena; ceiling height determines aerial viability
- **Aerial platforms** (floating islands, raised structures) — landing zones for Fly mode to transition to Mech
- Aerial-specific pickups (bomb caches, fuel cells) at height
- Downward sight lines give Fly mode information advantage — balance with limited ceiling height

**Elevated Platform Layer (Mech Mode Primary):**
- Raised platforms at 10-15m height — accessible by ramp (Car slow), jump (Mech), or landing (Fly)
- Defensible choke points for Mech melee builds
- Siege lines with clear downward shooting angles

**Multi-Level Arena Reference: The "Layers" Design:**
```
Layer 3: Ceiling/High Air (Fly mode dominates)
           ↑ Bomb drops, missile strikes, aerial dogfights
Layer 2: Platforms (10-15m) (Mech/Fly transition zone)
           ↑ Ramps, jump pads, landing pads
Layer 1: Ground (0-2m) (Car mode dominates)
           ↑ Full road surface, obstacles, tunnels
```

**Balance Considerations:**
- Fly mode players should not be immune to ground attacks — introduce **anti-air weapons** (or let all weapons hit aerial targets with a height penalty of -20% damage)
- Mech platform players should not be unreachable — ramp/zip line access must exist
- Ground players need "Fly mode safe havens" (low-ceiling tunnels) where Fly mode cannot follow

---

### 4.3 Destructible Environments in Arenas

Destructible environments add dynamism to combat. Design lessons from vehicular combat games:

**Types of Destructible Elements:**
- **Breakable obstacles** (crates, pillars, walls): Destroyed by weapons/ram; reveal pickup items
- **Collapsing platforms**: Platforms that degrade after heavy combat in their zone; forces Mech mode players to relocate
- **Dynamic hazards**: Explosive barrels that detonate on impact; fires that spread
- **Structural demolition**: Destroying certain points changes the arena shape (closes a route, opens a new one)

**Balance Warning**: Full destructible arenas can create "camping depressions" where sustained fire creates defensible craters. Limit structural changes to **predetermined zones** and ensure arena geometry doesn't collapse into unplayable shapes.

**Regenerating Destructibles**: After 45-60 seconds, destroyed elements respawn. This prevents permanent depletion and ensures arenas remain dynamic throughout longer matches.

---

### 4.4 Power-Up Placement Strategy

From [Mind Studios level design guide](https://games.themindstudios.com/post/multiplayer-level-design-techniques/), [SuperTuxKart arena design](https://supertuxkart.net/Making_Tracks:_Appendix_D:_Soccer_and_Battle_Modes), and Mario Kart Battle Mode analysis:

**Power-Up Placement Principles:**

1. **Central Concentration** (contested): Most powerful pickups (Boss weapon, extra lives) at arena center. Forces players toward each other and creates fight zones.

2. **Edge Placement** (safe but weak): Weaker pickups at edges. Rewards safe play but not overly — should never be as rewarding as center pickups.

3. **Height-Layered Pickups**: Ground = standard weapons; Platform/aerial = aerial-specific weapons (bombs, air missiles). Reinforces mode-appropriate reward loops.

4. **Timed Announce Pickups**: High-value pickups (Boss Weapon, Extra Life) announced via icon/sound 10 seconds before spawning. Creates **pickup rushes** where multiple players converge — guaranteed fight encounters.

5. **Scarcity by Design**: Never so many pickup boxes that players always have a weapon. Resource scarcity forces engagements and decision-making. Recommended: 1 pickup box per 2 players, refreshing every 20-30 seconds.

6. **Anti-Camping Boxes**: Place a pickup box in notoriously campable spots to draw players there and deny indefinite hiding.

---

### 4.5 Spawn Point Design to Prevent Spawn Camping

From [Mind Studios level design guide](https://games.themindstudios.com/post/multiplayer-level-design-techniques/) and [community discussions](https://steamcommunity.com/app/251950/discussions/0/687493774987976692/):

**Core Spawn Design Rules:**
1. **Distribute evenly**: At minimum 4 spawn points, well-distributed across the full arena. Never cluster spawns in one area.
2. **Algorithmic selection**: On respawn, the system selects the spawn point **furthest from any enemy** automatically.
3. **Spawn invincibility**: 2-3 seconds of invincibility upon spawning (cannot deal damage, cannot receive damage). Shown visually with a unique effect.
4. **Cover near spawns**: Every spawn point should have at least one piece of cover within 5m — gives players time to orient.
5. **Spawn visibility warning**: Spawning player is visible to nearby players (special flash effect) as a warning NOT to attack — attacking a spawning player triggers a brief "aggressor" debuff reducing attacker damage by 15%.
6. **Avoid high-traffic areas**: Spawn points must be >30m from any pickup item box and >50m from objective points.
7. **Rotating spawn lock**: Once a spawn is used, it's locked for 10 seconds — spreads respawn distribution automatically.

**Multi-Level Spawn Considerations:**
- Spawn points on each level (ground, platform, aerial)
- Players should spawn on the **level they were last playing** (if possible) to prevent mode-context disorientation
- Fly mode players respawn in mid-air at a safe hovering altitude

---

## 5. SCORING AND PROGRESSION IN WAR MODE

### 5.1 Kill/Death/Assist Scoring

**Research: Killstreak Systems and Their Problems** ([Reddit Games discussion](https://www.reddit.com/r/Games/comments/5f38d7/is_the_killstreak_method_of_rewards_actually_a/)):

The community broadly agrees that **pure killstreak rewards are problematic** because:
- They give winning players more tools, widening the gap further
- They "lock out" comebacks for losing players
- They frustrate good players who die just before earning their streak reward

Better alternatives:
- **Assist rewards equal to kill rewards** — prevent "kill-stealing" frustration
- **Defensive action rewards** (block shots, protect teammates)
- **Scoring by time alive** in addition to kills

**Recommended WAR Mode Kill/Death/Assist Matrix:**

| Action | Points | Notes |
|--------|--------|-------|
| Kill | +10 | Base kill value |
| Assist (30%+ damage dealt) | +6 | Rewards coordination |
| First Blood (first kill of match) | +5 | Opening game momentum |
| Comeback Kill (kill while at <25% HP) | +5 bonus | Rewards aggression when low |
| Kill Streak x3 | +5 | Modest streak bonus |
| Kill Streak x5 | +10 | Escalates |
| Kill Streak x7+ | +20 | Significant — but defeatable |
| Streak Ender (kill player on x3+ streak) | +15 | Comeback mechanic |
| Death | 0 (no penalty to score) | Penalize by respawn timer, not score |
| Survival (per 30s alive) | +2 | Passive reward for staying in match |

**Why No Score Penalty on Death?**: Death is already punished by respawn timer. Score penalties on death create "quit incentives" — players leave if they're having a bad match rather than adapting.

---

### 5.2 Streak Rewards and Comeback Mechanics

**Streak Rewards** (inspired by Call of Duty and modified to avoid feedback loops):

| Streak | Reward | Duration/Nature |
|--------|--------|-----------------|
| x3 Kill Streak | +5 bonus points + visual crown | Non-combat reward |
| x5 Kill Streak | +10 bonus + special weapon drop in your lane | One-time pickup |
| x7 Kill Streak | +20 bonus + 30s speed boost | Temporary power |
| x10 Kill Streak | +50 bonus + player becomes a visible "Target" | Score bounty — kills are worth +30 to anyone who gets you |

**The x10 "Target" mechanic is key**: At 10+ kills, you become marked on everyone's minimap with a bounty. This is a natural comeback/rubber-banding mechanic that doesn't feel artificial — skilled players choose to press for the streak, accepting the increased pressure.

**Comeback Mechanics:**

1. **Deficit Bonus**: If a player or team is 30+ points behind, all kills earn +3 bonus. Subtle, non-visible to other players. ([Game AI Pro rubber-banding research](http://www.gameaipro.com/GameAIPro/GameAIPro_Chapter42_A_Rubber-Banding_System_for_Gameplay_and_Race_Management.pdf))

2. **Item Scaling**: Lower-ranked players receive better pickup items from boxes. (Mario Kart Battle Mode equivalent — last-place gets Blue Shell tier)

3. **Revenge Multiplier**: Killing the person who last killed you grants +5 "Revenge" bonus. Soft comeback without changing balance.

4. **Catch-Up Spawn**: Players who have died 3+ times in a row respawn with a free "Combat Boost" (brief invincibility for 5s + +25% damage for 10s) to get back into contention.

---

### 5.3 WAR Mode Specific Rankings and Leaderboards

**In-Match Leaderboard Design:**
- Visible during match in corner; shows top 4 players (or all, scroll-able)
- Shows: Player name, current score, kill/death count, current form (car/fly/mech icon)
- Leader visually distinct (crown icon) — makes them a target

**Post-Match Detailed Breakdown:**

```
┌─────────────────────────────────────────────────┐
│ MATCH RESULT: TEAM A WINS (102 - 87 points)     │
├────────────┬──────┬──────┬──────┬──────┬────────┤
│ Player     │ Pts  │ Kills│ Asst.│Deaths│ Form   │
├────────────┼──────┼──────┼──────┼──────┼────────┤
│ ★ Player1  │  45  │  5   │  2   │  1   │ Fly    │
│ Player2    │  32  │  3   │  4   │  2   │ Car    │
│ Player3    │  25  │  2   │  3   │  3   │ Mech   │
└────────────┴──────┴──────┴──────┴──────┴────────┘
Performance Medals:
  🏆 Most Kills: Player1 (5 kills)
  🛡️ Most Assists: Player2 (4 assists)  
  ⚡ Fastest Kill: Player3 (kill at 0:04)
  💥 Highest Streak: Player1 (streak of 4)
  🔄 Most Transformations: Player2 (11 transforms)
```

**Seasonal Leaderboard:**

| Metric | Description |
|--------|-------------|
| WAR Rating | ELO-style rating; adjusts per match based on opponent strength |
| Win Rate by Mode | Separate win rates for SKAT, REVERSI, TDM, BOWL |
| Form Mastery | Kills in each transformer form (Car/Fly/Mech) — tracks playstyle |
| Mode Specialist Rank | Separate rank per game mode |

**Rank Tiers (borrowing from competitive games):**
- Bronze → Silver → Gold → Platinum → Diamond → Transformer Elite
- Rank resets seasonally with soft reset (placed 2 tiers below your peak)
- Leaderboard shows weekly top 10 + your personal rank position

---

### 5.4 Integration with Existing Reward System

**Race Credits and Form Tokens** should function in WAR Mode as follows:

**Race Credits (General Currency):**
- Earned per WAR Mode match: Base 50 credits per match completion
- Bonus credits by performance: +10 per kill, +5 per assist, +100 for MVP award
- Credits allow cosmetic purchases, vehicle upgrades applicable across all modes
- WAR Mode credit earn rate: 80% of Race Mode equivalent (WAR matches are shorter)

**Form Tokens (Transformer Form Currency):**
- WAR Mode-specific Form Token bonus: +2 Form Tokens per kill **in a non-default form**
- Incentivizes transformer form diversity — players who only drive in Car mode earn fewer Form Tokens
- Special Form Token bonus for winning while using a non-natural form (e.g., winning BOWL in Mech mode grants 2x Form Tokens for that round)

**WAR Mode Exclusive Unlocks:**
- Certain cosmetics (weapon skins, vehicle war paint, kill effect animations) only unlockable through WAR Mode
- "WAR Badge" profile icon for reaching Gold WAR Rating
- Unique "Alleinspieler" badge for winning SKAT mode as the solo player 3 times
- "Last Standing" trophy icon for winning BOWL mode 10 times

**Weekly/Daily WAR Challenges:**
- Examples:
  - "Win 1 SKAT match as the Alleinspieler" → 200 Race Credits
  - "Get 5 assists in a single TEAM DEATHMATCH" → 1 Form Token
  - "Survive Phase 3 of BOWL 3 times in a row" → Exclusive cosmetic
- Challenges rotate daily (3 daily, 1 weekly), driving retention without requiring grinding

---

## DESIGN RECOMMENDATIONS SUMMARY

**For the WAR Mode development of Mariliktux, prioritize:**

1. **Tri-mode weapon design first**: The most unique aspect of Mariliktux is the Car/Fly/Mech transition. Every WAR mode weapon must be designed with all three modes in mind. Start with the mode-transition attack (the Transform Attack) — it immediately teaches new players that form transitions are tactical.

2. **SKAT mode as the signature mode**: The Skat card game inspiration is brilliant and culturally distinct (German game, perfect for a German-developed game). The Alleinspieler design with information asymmetry (radar advantage) + power asymmetry (3x HP) + flexibility advantage (instant form switch) avoids the Evolve failure mode of requiring perfect coordination from the 3 players to win.

3. **REVERSI cascade logic**: The catch-TWO escalation creates unpredictable, chaotic late-game scenarios that feel earned. The 60-second Role Reversal twist prevents stalemates and adds a climactic final act.

4. **BOWL as casual entry point**: BOWL Mode (FFA with shrinking arena) should be the mode shown first to new players. It's the most immediately understood, requires no coordination, and the shrinking arena creates guaranteed climactic final moments.

5. **Arena design: build vertical first**: The multi-level (ground/platform/air) design is what separates Mariliktux arenas from every other kart game arena. Every official WAR arena should have all three layers accessible from ground level.

6. **Hybrid ammo+cooldown system**: Signature Special (cooldown) + Pickup Weapons (ammo) + Ram (always available) covers all engagement types and matches the STK pickup tradition while adding WAR Mode depth.

---

## SOURCES

- [Twisted Metal Special Weapons Wiki](https://twistedmetal.fandom.com/wiki/Special_Weapon)
- [Twisted Metal Special Weapons — IGN Guide](https://www.ign.com/wikis/twisted-metal/Special_Weapons)
- [Mario Kart 8 Deluxe Battle Mode Guide — Polygon](https://www.polygon.com/mario-kart-8-deluxe-guide/2017/4/28/15473910/battle-mode-tips-balloon-battle-bob-omb-blastrenegade-roundupcoin-runnersshine-thief/)
- [Mario Kart Design Delve — YouTube](https://www.youtube.com/watch?v=p4Mz6rKQ8Hs)
- [Balloon Battle — Super Mario Wiki](https://www.mariowiki.com/Balloon_Battle)
- [Destruction AllStars: Breaker Mode Explained — TheGamer](https://www.thegamer.com/destruction-allstars-breaker-mode-explained/)
- [Destruction AllStars Tips — TheSixthAxis](https://www.thesixthaxis.com/2021/02/06/destruction-allstars-guide-11-essential-tips-tricks/)
- [Crossout Weapons Wiki](https://crossout-archive.fandom.com/wiki/Weapons)
- [Crossout Vehicle Construction Guide — Reddit](https://www.reddit.com/r/Crossout/comments/6gb75x/vehicle_constructiondamage_basics_guide/)
- [War Thunder Arcade Battles Wiki](https://wiki.warthunder.com/gamemode/arcade_battles)
- [Halo Infection Mode — Halopedia](https://www.halopedia.org/Infection)
- [Pac-Man Vs. — TV Tropes](https://tvtropes.org/pmwiki/pmwiki.php/VideoGame/PacManVs)
- [Pac-Man Vs. Fandom Wiki](https://pacman.fandom.com/wiki/Pac-Man_Vs.)
- [Pac-Man Vs. Review — Racketboy](https://racketboy.com/retro/review-pac-man-vs-gamecube)
- [Asymmetric Multiplayer History — YouTube](https://www.youtube.com/watch?v=cMf2CWmjTfc)
- [Evolve 4v1 Post-Mortem — Reddit](https://www.reddit.com/r/Games/comments/8p2s2m/evolve_dev_says_4v1_caused_more_problems_than_we/)
- [Dead by Daylight Killer Balance — BHVR Forums](https://forums.bhvr.com/dead-by-daylight/discussion/comment/3757363)
- [Rocket League Points System — Fandom](https://rocketleague.fandom.com/wiki/Points)
- [SuperTuxKart Arena Design Guide](https://supertuxkart.net/Making_Tracks:_Appendix_D:_Soccer_and_Battle_Modes)
- [SuperTuxKart STK Changelog — GitHub](https://github.com/supertuxkart/stk-code/blob/master/CHANGELOG.md)
- [SuperTuxKart Powerups — pystk API Docs](https://pystk.readthedocs.io/_/downloads/en/latest/pdf/)
- [SuperTuxKart Items Guide — Speedrun.com](https://www.speedrun.com/stk/guides/hpig5)
- [SuperTuxKart Dev Blog 2011](https://blog.supertuxkart.net/2011/)
- [Skat Card Game Rules — Pagat](https://www.pagat.com/schafkopf/skat.html)
- [Skat — Wikipedia](https://en.wikipedia.org/wiki/Skat_(card_game))
- [Enemy Spawn Design Tips — Reddit](https://www.reddit.com/r/gamedesign/comments/vtgkbu/tips_on_enemy_spawn_logic_and_enemy_game_balance/)
- [Multiplayer Level Design Techniques — Mind Studios](https://games.themindstudios.com/post/multiplayer-level-design-techniques/)
- [Killstreak Design Analysis — Reddit](https://www.reddit.com/r/Games/comments/5f38d7/is_the_killstreak_method_of_rewards_actually_a/)
- [Rubber-Banding in Racing Games — Game AI Pro PDF](http://www.gameaipro.com/GameAIPro/GameAIPro_Chapter42_A_Rubber-Banding_System_for_Gameplay_and_Race_Management.pdf)
- [PvP/PvE Balance Discussion — Reddit](https://www.reddit.com/r/gamedesign/comments/11atvrt/how_do_you_balance_pvp_combat_in_predominantly/)
- [Shrinking Arena Battle Royale Design — Reddit](https://www.reddit.com/r/mariokart/comments/1i58e85/classic_survivalstyle_balloon_battles_need_to/)
- [War Thunder Balanced Battles Discussion — Forum](https://forum.warthunder.com/t/balanced-battles-a-new-game-mode-idea/266071)
- [Halo Infection Design Talk — Reddit](https://www.reddit.com/r/halo/comments/3hrys3/infection_design_talk_infected_choice_mechanics/)
- [Game Balance Guide — Game Design Skills](https://gamedesignskills.com/game-design/game-balance/)
