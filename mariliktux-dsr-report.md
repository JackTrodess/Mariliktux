# Mariliktux — Design Science Research Report

> **Projekt:** Mariliktux — SuperTuxKart Transformer Racing Game  
> **Datum:** 18. März 2026  
> **Methodik:** Design Science Research nach Hevner et al. (2004), Drei-Zyklen-Modell nach Hevner (2007), DSRM-Prozessmodell nach Peffers et al. (2007)

---

## Executive Summary

Mariliktux ist ein browserbasiertes Rennspiel, das auf der Open-Source-Engine SuperTuxKart (STK) aufbaut und ein neuartiges Transformer-Mechanik-System einführt: Fahrzeuge können während des Rennens zwischen Fahr-, Flug-, Sprung- und Mech-Modus wechseln. Das Projekt adressiert eine signifikante Lücke im aktuellen Spielmarkt — kein existierendes browserbasiertes Rennspiel kombiniert flüssige Echtzeit-Transformationsanimationen mit einem vollständigen Multi-Modus-Gameplay-System, kollaborativem Online-Multiplayer, Community-Modding und ethischer Monetarisierung.

Der DSR-Report ist nach dem Drei-Zyklen-Modell strukturiert: Der **Relevance Cycle** analysiert Markt, Spieler und technologische Umgebung. Der **Rigor Cycle** verankert jede Designentscheidung in existierenden Referenzspielen, wissenschaftlichen Erkenntnissen und technischen Standards. Der **Design Cycle** beschreibt zehn miteinander verflochtene Artefakt-Subsysteme — vom Animationssystem über die Multiplayer-Architektur bis zum Low-Code-Modding.

Die Kernartefakte des Projekts sind: (1) ein Hybrid-Animationssystem mit skelettaler Animation und Morph Targets für nahtlose Transformationen, (2) ein Multi-Modus-Level-Design-Framework mit Automatic-Trigger-Architektur, (3) ein ethisches Progressions- und Belohnungssystem nach dem iRacing/Forza-Hybrid-Modell, (4) eine FSM-basierte 4-Layer-Renn-KI mit Rubber-Banding und adaptiver Schwierigkeit, (5) ein WebRTC+SFU-Sprachchat mit Spatial Audio, (6) ein Scratch-ähnliches Blockly-Modding-System mit WASM-Sandboxing, (7) eine WebTransport-Multiplayer-Architektur mit Server-Autorität und (8) ein Three.js/WebGPU-Renderer mit Toon-Shading, der das 60-fps-Ziel durch aggressives Draw-Call-Management erreicht.

Die 12-monatige Roadmap gliedert sich in vier Phasen: Foundation (Monate 1–3), Gameplay (Monate 4–6), Features (Monate 7–9) und Polish (Monate 10–12). Das Projekt ist als Open-Source-Erweiterung des SuperTuxKart-Ökosystems konzipiert und folgt damit dem Rigor-Cycle-Prinzip: Wissen aus der STK-Community und Wissensbasis zurückzuführen.

---

## 1. DSR-Methodik

### 1.1 Die 7 Leitlinien nach Hevner et al. (2004) — angewendet auf Mariliktux

[Hevner et al. (2004)](https://wise.vub.ac.be/sites/default/files/thesis_info/design_science.pdf) definieren Design Science Research als ein problemlösendes Paradigma, das innovative IT-Artefakte schafft, die die Grenzen menschlicher und organisatorischer Möglichkeiten erweitern. Das fundamentale Prinzip lautet: *Wissen und Verständnis über ein Designproblem entstehen durch das Bauen und Anwenden eines Artefakts.* Die folgende Tabelle zeigt, wie jede der sieben Leitlinien auf Mariliktux angewendet wird:

| Leitlinie | Kernaussage (Hevner et al. 2004) | Anwendung auf Mariliktux |
|-----------|----------------------------------|--------------------------|
| **1. Design als Artefakt** | DSR muss ein lauffähiges, nützliches IT-Artefakt erzeugen | Instanziierung: lauffähiger Spielprototyp; Methode: Transformer-Animations-Framework; Modell: Multi-Modus-State-Machine; Konstrukt: Spielmechanik-Terminologie |
| **2. Problemrelevanz** | Das Problem muss wichtig und relevant sein | Browserspiele dominieren den Casual-Gaming-Markt; kein Titel kombiniert bisher Multi-Modus-Transformation mit STK-Basis, ethischem Modding und Online-Multiplayer |
| **3. Designevaluation** | Utility, Qualität und Effizienz müssen rigoros evaluiert werden | Playtest-Sessions, FPS-Benchmarks, Usability-Tests für Level-Editor, A/B-Tests für Reward-System, Netzwerk-Latenz-Tests |
| **4. Forschungsbeiträge** | DSR muss einen neuen, verifizierbaren Beitrag liefern | Level 1: Spielprototyp als neue Instanziierung; Level 2: Designprinzipien für Multi-Modus-Rennspiele; Level 3: Transformations-Gameplay-Theorie |
| **5. Forschungsrigorosität** | Fundierung in geeigneten Theorien und Methoden | Fundierung in: GameAIPro-Algorithmen, WebRTC-Standards, DSR-Spieleforschung (Oliveira et al. 2022, Loch et al. 2023) |
| **6. Design als Suchprozess** | Design ist iterativer Suchprozess im Problemraum | Build-Test-Loop über vier Entwicklungsphasen; Prototyp-Iterationen; Community-Feedback via mod.io |
| **7. Kommunikation** | Ergebnisse müssen für technisches und Management-Publikum kommuniziert werden | Technisch: diese DSR-Dokumentation, API-Referenz, Architektурdiagramme; Management: Spielkonzept-Präsentation, Roadmap-Übersicht |

Das von [Hevner et al. (2004, S. 80)](https://wise.vub.ac.be/sites/default/files/thesis_info/design_science.pdf) formulierte Zieldiktat ist dabei zentral: „The goal of behavioral-science research is truth. The goal of design-science research is **utility**." Für Mariliktux bedeutet Utility: ein spielbares, flüssiges Rennspiel mit echter Transformationsmechanik, das 60 fps im Browser erreicht und eine nachhaltige Community-Plattform aufbaut.

### 1.2 Das Drei-Zyklen-Modell nach Hevner (2007)

[Hevner (2007)](https://aisel.aisnet.org/sjis/vol19/iss2/4/) ergänzte das 2004er-Framework durch drei ineinandergreifende Forschungszyklen, die alle in einem DSR-Projekt identifizierbar sein müssen:

```
┌──────────────────┐     ┌────────────────────────────────────────┐     ┌──────────────────────┐
│  RELEVANCE CYCLE  │     │          DESIGN CYCLE                   │     │   RIGOR CYCLE         │
│                   │     │                                         │     │                       │
│  Umgebung:        │────►│  Build:                                 │────►│  Wissensbasis:        │
│  • Spieler        │     │  • Transformer-Animationssystem         │     │  • Hevner et al. 2004 │
│  • Marktlücke     │     │  • Level-Design-Framework               │     │  • GameAIPro Ch. 38/42│
│  • Technologie    │     │  • AI/Netcode-Architektur               │     │  • WebRTC-Standards   │
│                   │◄────│  • Reward/Modding-System                │◄────│  • STK Source Code    │
│  Field Testing:   │     │                                         │     │  • NSDI 2025 Paper    │
│  • Playtest       │     │  Evaluate:                              │     │  • Referenzspiele     │
│  • Beta-Release   │     │  • FPS-Benchmarks                       │     │                       │
│  • Community      │     │  • Usability-Tests                      │     │  Output: neue         │
│    Feedback       │     │  • A/B-Tests Reward                     │     │  Meta-Artefakte       │
└──────────────────┘     └────────────────────────────────────────┘     └──────────────────────┘
```

**Relevance Cycle:** Die Spielerumgebung (Wunsch nach innovativen Browserspielen, Marktlücke im Transformations-Genre) definiert Anforderungen und Akzeptanzkriterien. Field Testing über Beta-Releases und Community-Feedback schließt den Kreislauf.

**Design Cycle:** Der innerste, schnellste Zyklus iteriert zwischen Build (Konstruktion neuer Artefakte) und Evaluate (Messung in kontrollierten Testumgebungen). [Hevner (2007, S. 91)](https://aisel.aisnet.org/sjis/vol19/iss2/4/) betont: *„During the performance of the design cycle it is important to maintain a balance between the efforts spent in constructing and evaluating."*

**Rigor Cycle:** Die Wissensbasis (Referenzspiele, wissenschaftliche Literatur, existierende Frameworks) fundiert jede Designentscheidung und schützt vor Routinedesign. Neue Erkenntnisse aus Mariliktux fließen als Meta-Artefakte zurück in die Community-Wissensbasis.

| Zyklus | Verbindet | Takt | Hauptoutput für Mariliktux |
|--------|-----------|------|---------------------------|
| Relevance Cycle | Forschung ↔ Spielerumgebung | Mittel (Monate) | Anforderungen → Spielprototyp → Field Test |
| Design Cycle | Internes Build-Test-Loop | Schnell (Wochen) | Iterierte Artefakte: Animation, KI, Netcode |
| Rigor Cycle | Forschung ↔ Wissensbasis | Langsam (initial) | Theoretische Fundierung aller Designentscheide |

### 1.3 DSRM-Prozessmodell nach Peffers et al. (2007)

[Peffers et al. (2007)](https://dl.acm.org/doi/abs/10.2753/MIS0742-1222240302) operationalisieren Hevners Zyklen in sechs Prozessschritten:

```
┌─────────────────────────────────────────────────────────────────────────────────┐
│                         DSRM-PROZESSMODELL FÜR MARILIKTUX                        │
│                                                                                   │
│  1. PROBLEM-          2. LÖSUNGS-         3. DESIGN &          4. DEMONSTRATION  │
│     IDENTIFIKATION       ZIELE               ENTWICKLUNG                          │
│                                                                                   │
│  Kein Browser-        60 fps              10 Artefakt-         Alpha-Prototype    │
│  Transformations-     Transformer-        Subsysteme           mit Core-          │
│  Rennspiel           Rennspiel auf        (Kap. 4.1–4.10)     Transformation     │
│  existiert           STK-Basis                                                    │
│                                                                                   │
│  5. EVALUATION                            6. KOMMUNIKATION                        │
│                                                                                   │
│  Playtest-Methodik                        DSR-Report (dieses  │                  │
│  FPS-Benchmarks                           Dokument)           │                  │
│  Netzwerk-Tests                           Technische Doku     │                  │
│  A/B-Tests Reward                         Publisher-Pitch      │                  │
│                                                                                   │
│  ←────────────── iterative Rückkopplungsschleifen ──────────────→                │
└─────────────────────────────────────────────────────────────────────────────────┘
```

Loch et al. (2023) demonstrierten die Anwendbarkeit dieses Prozessmodells auf interdisziplinäre Spielentwicklungsteams ([European Conference on Games Based Learning, 2023](https://papers.academic-conferences.org/index.php/ecgbl/article/view/1567)), was die Eignung für Mariliktux bestätigt.

---

## 2. Problemdomäne und Relevanz (Relevance Cycle)

### 2.1 Environment: Spieler, Markt, Technologie

#### Spieler (People)

Die primäre Zielgruppe von Mariliktux umfasst drei Segmente:

| Segment | Profil | Hauptbedürfnis |
|---------|--------|----------------|
| **Casual Browser-Gamer** | 16–35 Jahre, spielen 20–60 min/Session, keine Installation | Einsteigerfreundliches, sofort spielbares Spiel |
| **Retro/Transformers-Fans** | Kennen G1-Transformers, War for Cybertron, Devastation | Authentisches Transformations-Gefühl, G1-Ästhetik |
| **STK-Community** | Existing SuperTuxKart-Spieler, Open-Source-Enthusiasten | Erweitertes Gameplay, Modding-Möglichkeiten |

Sekundäre Zielgruppe: **Modder und Level-Designer** (technisch versiert, möchten Custom Content erstellen) sowie **Eltern/Schulen**, die nach browser-basierten Lernspielen suchen (Modding-System als Einführung in Programmierung).

#### Markt

Der browserbasierte Spielemarkt wächst stetig: 2025 spielen über 1,5 Milliarden Nutzer Browser-basierte Spiele weltweit. Gleichzeitig ist das Segment der Transformations-Rennspiele auf Browserbasis nahezu unbesetzt:

| Spiel | Plattform | Transformation | Browser-nativ |
|-------|-----------|----------------|---------------|
| Transformers: Devastation (2015) | PC/Konsole | ✅ | ❌ |
| Transformers: War for Cybertron (2010) | PC/Konsole | ✅ | ❌ |
| Sonic & All-Stars Racing Transformed | PC/Konsole | ✅ (automatisch) | ❌ |
| SuperTuxKart | Browser/Desktop | ❌ | ✅ (WebAssembly) |
| Mario Kart 8 | Konsole | Partiell (Anti-Gravity) | ❌ |
| **Mariliktux** | **Browser** | **✅ Multi-Modus** | **✅** |

**Marktlücke:** Kein aktuelles browserbasiertes Spiel kombiniert Echtzeit-Fahrzeug-Transformationen mit einem vollständigen Rennspiel-Framework. Dies ist die zentrale Relevanz-Begründung im Sinne von [Hevner et al. (2004, S. 83)](https://wise.vub.ac.be/sites/default/files/thesis_info/design_science.pdf): *„The objective of design-science research is to develop technology-based solutions to important and relevant problems."*

#### Technologie

Die technologische Umgebung ist 2026 erstmals reif für ein Projekt dieser Komplexität im Browser:

- **WebGPU:** Seit November 2025 in allen vier großen Browsern standardmäßig aktiv — [WebGPU.com (Dezember 2025)](https://www.webgpu.com/news/webgpu-hits-critical-mass-all-major-browsers/). Speedup von 1,6–8,1× gegenüber WebGL für compute-intensive Workloads ([SitePoint, Februar 2026](https://www.sitepoint.com/webgpu-vs-webgl-inference-benchmarks/)).
- **WebTransport:** QUIC-basiertes Protokoll, das WebSocket in Latenz um signifikante Faktoren schlägt ([NSDI 2025](https://aaron.gember-jacobson.com/docs/nsdi2025browser-networking.pdf)).
- **Three.js r171+:** Production-ready WebGPU-Renderer mit automatischem WebGL-Fallback.
- **Meshy.ai API:** KI-gestützte 3D-Modell-Generierung und Auto-Rigging für humanoidale Robot-Mode-Meshes ([Meshy API Docs](https://docs.meshy.ai/en/api/rigging-and-animation)).
- **LiveKit:** Open-Source SFU-Implementierung für WebRTC-Sprachchat ([LiveKit GitHub](https://github.com/livekit/livekit)).
- **Google Blockly:** 100% client-seitiges Block-Programming-Framework für Low-Code-Modding ([Google Blockly](https://developers.google.com/blockly)).

### 2.2 Problem Statement: Warum ein Transformer-Rennspiel auf STK-Basis?

**Technisches Kernproblem:**
> *Wie kann ein Fahrzeug-Transformations-Mechanismus in einem Echtzeit-Rennspiel so implementiert werden, dass (a) die Transformation flüssig in weniger als 0,5 Sekunden abläuft, (b) alle Spielsysteme (Physik, KI, Kamera, HUD, Netcode) atomisch mit dem Modus wechseln, und (c) die 60-fps-Zielmarke im Browser nicht unterschritten wird?*

Dies ist ein exemplarisches **Wicked Problem** im Sinne von Herbert Simon ([*The Sciences of the Artificial*, 1996](https://gameprogrammingpatterns.com/state.html)): Die Anforderungen sind instabil, die Komponenten komplex miteinander verknüpft, und die Lösung erfordert kreative, explorative Suche — genau das, was [Hevner et al. (2004)](https://wise.vub.ac.be/sites/default/files/thesis_info/design_science.pdf) als Kerndomäne von DSR beschreiben.

**Warum auf STK-Basis?** SuperTuxKart ist das maturste Open-Source-Kart-Rennspiel (GPL v3), mit einer vollständigen Multiplayer-Architektur ([GitHub STK-Code](https://github.com/supertuxkart/stk-code)), einem erprobten AngelScript-Scripting-System ([STK Scripting](https://supertuxkart.net/Scripting)), und einer aktiven Community. Als Basis zu verwenden ist ein optimaler Mittelweg zwischen „von Grund auf neu entwickeln" und „ein proprietäres Spiel modden" — es ist legitimes Forschungsdesign, das dem Rigor-Cycle entspricht.

### 2.3 Anforderungen und Akzeptanzkriterien

**Funktionale Anforderungen:**

| Anforderung | Kriterium | Messmethode |
|-------------|-----------|-------------|
| Flüssige Transformation | < 0,5 Sekunden (15–30 Frames bei 60fps) | Frame-Timing-Profiler |
| Performance | 60 fps im Browser auf Mid-Range-Desktop | `renderer.info.render.calls < 100`; `stats-gl` |
| Multiplayer | 8 gleichzeitige Spieler mit < 100ms Latenz (EU-Server) | Netzwerk-Latenz-Tests |
| KI | Rubber-Banding aktiv; AI ~10% langsamer als Top-Spieler | Vergleichsmessungen |
| Modding | Lauffähiger Blockly-Mod innerhalb von 5 Minuten erstellbar | Usability-Test (Novize) |
| Audio | Online-Radio-Stream, Self-Radio, Spotify-Personal-Playback | Integration-Test |
| Sprachchat | < 100ms Sprachlatenz, DTLS-SRTP-Verschlüsselung | Latenz-Messung |

**Nicht-funktionale Anforderungen:**
- **Ethische Monetarisierung:** Kein Pay-to-Win; alle Gameplay-Features erspielbar
- **Open Source:** Kern-Spielcode unter GPL v3 (STK-Kompatibilität)
- **Datenschutz:** DSGVO/CCPA-konformer Sprachchat; keine biometrischen Daten
- **Barrierefreiheit:** Color-blind-freundliche UI; konfigurierbare Steuerung

---

## 3. Wissensbasis (Rigor Cycle)

### 3.1 Existierende Referenzspiele

Die Wissensbasis wird maßgeblich durch die Analyse von vier Referenzspielen geprägt, die jeweils spezifische Designlektionen liefern:

#### Transformers: War for Cybertron (High Moon Studios, 2010)

Das Referenz-Transformationsspiel: [That Gamers Asylum (2017)](https://thatgamersasylum.wordpress.com/2017/03/01/transforming-in-transformers-war-for-cybertron/) dokumentiert, was War for Cybertron von allen anderen Transformers-Spielen unterscheidet:
- **Fluid und fast:** Transformation während der Bewegung oder in der Luft, kein Stopp erforderlich
- **Das satisfying Whirring-Geräusch:** Explizit als kritisch für die UX identifiziert
- **Bidirektional zu jedem Zeitpunkt:** Fahrzeug↔Roboter mid-Combat, mid-Jump, mid-Air
- **Einfache Controls:** L1 = Boost forward; R2 = Special Maneuver; identisches Schema über alle Fahrzeugtypen

**DSR-Lektion:** Transformationsgeschwindigkeit und Nahtlosigkeit sind wichtiger als Animationskomplexität.

#### Transformers: Devastation (PlatinumGames, 2015)

[Wikipedia: Transformers: Devastation](https://en.wikipedia.org/wiki/Transformers:_Devastation) — G1-Cartoon-Ästhetik, Cel-Shading, Echtzeit-Transformation mid-Combat:
- Fünf spielbare Autobots, jederzeit transformierbar
- G1-proportionen (eckig, angular) erleichterten das skelettale Rigging erheblich
- Fahrzeugmodus-Dash als Kampftempo-Element

**DSR-Lektion:** G1/Toon-Ästhetik (Cel-Shading) reduziert technischen Aufwand für flüssige Transformationsanimationen erheblich — und entspricht dem angestrebten Three.js-`MeshToonMaterial`-Ansatz.

#### Sonic & All-Stars Racing Transformed (SASRT, 2012)

[Sonic Wiki Zone](https://sonic.fandom.com/wiki/Sonic_%26_All-Stars_Racing_Transformed) und [Adam Z's Analyse](https://www.adamzwakk.com/games/sonic-racing-transformed/):
- **Automatische Transformation** basierend auf Streckenoberfläche (kein manueller Trigger)
- **Transform Boost:** Timing eines Tricks während der Transformationsanimation ergibt Speed-Boost — High-Skill-Belohnung ohne Bestrafung für Nicht-Ausführung
- **Ring Mode:** Führt Spieler durch verborgene Streckengeometrie
- **Bekannte Schwäche:** „Collision Jank" bei hoher Luftgeschwindigkeit an unsichtbaren Geometrie-Decken

**DSR-Lektion:** Der Transform Boost ist ein elegantes Skill-Expression-Mechanismus; die SASRT-Kollisionsschwäche warnt vor zu engen Luftsektions-Geometrien.

#### Mario Kart 8 (Nintendo, 2014)

[Super Mario Wiki — Anti-Gravity](https://www.mariowiki.com/Anti-gravity):
- **Anti-Gravity Panels:** Trigger-basierte Gravitations-Änderung; Kamera richtet sich automatisch aus
- **Collision Bonus:** Anti-Gravity invertiert die Kollisionsstrafe zu einem Speed-Bonus
- **Pacing:** Anti-Gravity-Abschnitte als „Beat" innerhalb eines Kurses eingesetzt

**DSR-Lektion:** Der Panel-Trigger-Ansatz (texturbasiert, automatisch) ist effizienter als manueller Spielertrigger für Modus-Wechsel auf Streckenabschnitte.

#### Diddy Kong Racing (Rare, 1997)

[Reddit r/gamedesign Analyse](https://www.reddit.com/r/gamedesign/comments/aytq3k/did_diddy_kong_racings_track_design_have_to/):
- **Selektive Streckenabschnitte:** Einige Fahrzeuge sind auf bestimmten Karten überlegen — dies ist eine bewusste Designentscheidung, die Identität schafft
- **Dreifach-Pfad-Architektur:** Drei Fahrzeugtypen erzeugen drei unterschiedliche Pfade durch denselben Raum

**DSR-Lektion:** „Erzwungene Imbalance" ist valide, wenn sie zur Differenzierung beiträgt: Jeder Modus besitzt spezifische Momente.

### 3.2 Existierende Technologien

| Technologie | Funktion in Mariliktux | Quelle/Referenz |
|-------------|------------------------|-----------------|
| **Three.js r171+ (WebGPU)** | 3D-Rendering, Partikel, Post-Processing | [Utsubo 2026 Best Practices](https://www.utsubo.com/blog/threejs-best-practices-100-tips) |
| **WebGPU (WGSL)** | Compute Shader für Partikel, 1,6–8,1× WebGL-Speedup | [SitePoint Benchmarks 2026](https://www.sitepoint.com/webgpu-vs-webgl-inference-benchmarks/) |
| **TSL (Three Shader Language)** | Write-once-Shader für WebGPU und WebGL | [Three.js r171+ Dokumentation](https://threejs.org/docs/) |
| **Meshy.ai API** | Auto-Rigging humanoidaler Robot-Mode-Meshes | [Meshy API Docs](https://docs.meshy.ai/en/api/rigging-and-animation) |
| **WebRTC + LiveKit SFU** | Multiplayer-Sprachchat, Proximity Voice | [LiveKit GitHub](https://github.com/livekit/livekit) |
| **WebTransport (QUIC)** | Echtzeit-Spielzustand-Synchronisation, < WebSocket-Latenz | [NSDI 2025 Paper](https://aaron.gember-jacobson.com/docs/nsdi2025browser-networking.pdf) |
| **Google Blockly** | Low-Code-Modding-UI, Code-Generierung zu JS/Lua | [Google Blockly](https://developers.google.com/blockly) |
| **WebAssembly (WasmTime)** | Sandboxing für Text-Mode-Mods | [Reddit r/rust 2025](https://www.reddit.com/r/rust/comments/1isoiwa/securesandboxed_game_modding_with_rust/) |
| **mod.io** | Community-Mod-Sharing-Plattform (30M+ MAU) | [mod.io](https://mod.io) |
| **FMOD/Wwise** | Adaptive Musik, Stem-basiertes Audio-System | [The Game Audio Co.](https://www.thegameaudioco.com/wwise-or-fmod-a-guide-to-choosing-the-right-audio-tool-for-every-game-developer) |
| **Icecast/SHOUTcast** | Online-Radio-Streams (Open Source) | [Hostcode Lab Vergleich](https://hostcodelab.com/shoutcast-vs-icecast-streaming-server/) |
| **Opus Codec (RFC 6716)** | Sprachchat (6–510 kbps, ~70ms Latenz) | [Opus-codec.org](http://opus-codec.org) |
| **SuperTuxKart (GPL v3)** | Basis-Engine, KI-Framework, Multiplayer-Protokoll | [STK GitHub](https://github.com/supertuxkart/stk-code) |

### 3.3 Existierende Theorien und Designmuster

**Racing AI:** [GameAIPro Kapitel 38](https://www.gameaipro.com/GameAIPro/GameAIPro_Chapter38_An_Architecture_Overview_for_AI_in_Racing_Games.pdf) definiert die 4-Layer-Architektur (Character/Strategic/Tactical/Control). [GameAIPro Kapitel 42](http://www.gameaipro.com/GameAIPro/GameAIPro_Chapter42_A_Rubber-Banding_System_for_Gameplay_and_Race_Management.pdf) formalisiert das Rubber-Banding-System mit definierten Distanzzonen.

**Game Design Patterns:** [Game Programming Patterns — State](https://gameprogrammingpatterns.com/state.html) liefert die theoretische Grundlage für Finite State Machines als Transformations-Steuerung.

**Reward Systems:** Die Forza-Motorsport-Fallstudie ([Forza Forums 2023](https://forums.forza.net/t/analysis-on-why-the-car-leveling-and-carxp-system-is-driving-people-away-from-forza-motorsport/653361)) zeigt empirisch, welche XP-Design-Entscheidungen Veteranen-Spieler vergraulen — wertvolles negatives Wissen für Mariliktux.

**Netcode:** [University of Waterloo (2023 IEEE TOG)](https://ece.uwaterloo.ca/~sl2smith/papers/2023TOG-Predictive_Dead_Reckoning.pdf) zeigt, dass ein LSTM-basiertes Neural Dead Reckoning die Positionsvorhersage um ~45% verbessert gegenüber klassischen Ansätzen — ein Forschungsbeitrag, der direkt in die Netcode-Architektur einfließt.

**DSR und Spielentwicklung:** Oliveira et al. (2022) nutzen DSR für die Entwicklung eines Lernspiels ([Journal of Interactive Systems](https://games.jmir.org/2024/1/e48099/)). Das JMIR Serious Games Framework (2024) validiert stakeholder-zentriertes DSR mit 29 Praktikern — und bestätigt die Übertragbarkeit der DSR-Methodik auf Spielentwicklung.

---

## 4. Artefakt-Design (Design Cycle)

Der Design Cycle operationalisiert alle zehn Kernkomponenten des Spiels. Jede Designentscheidung wird zurück auf DSR-Leitlinien und Wissensbasis-Referenzen verknüpft.

### 4.1 Transformer-Animationssystem

#### Problembeschreibung im DSR-Kontext

Das Transformations-Animationssystem ist das primäre Differenzierungsmerkmal von Mariliktux. Als DSR-Artefakt ist es eine **Methode** (Algorithmus/Vorgehensweise für die Animationspipeline) und gleichzeitig Teil der **Instanziierung** (lauffähiger Spielcode). Das zentrale Forschungsproblem: *Wie können Fahrzeug-zu-Roboter-Transformationen in < 0,5 Sekunden flüssig und geometrisch plausibel in Echtzeit im Browser gerendert werden?*

#### Hybrid-Ansatz: Skelettale Animation + Morph Targets

Die Forschung zeigt eindeutig: Professionelle Pipelines verwenden beide Techniken kombiniert ([Polycount Forum](https://polycount.com/discussion/35241/morph-vs-skeleton)):

> *„We use both. Morphing for cinematics, and boned for in-game animations."* — Polycount Community

| Eigenschaft | Skelettale Animation | Morph Targets | Mariliktux-Empfehlung |
|-------------|---------------------|---------------|----------------------|
| Speicher/Frame | Nur Knochen-Transforms (gering) | Volle Vertex-Deltas (groß) | **Skelett** für Kern-Transformation |
| LOD-Kompatibilität | Ausgezeichnet (gleicher Skeleton, weniger Vertices) | Schlecht (Vertex-Deltas an Topologie gebunden) | **Skelett** für LOD0–LOD2 |
| Präzision | Gewichtetes Blending (kann Artefakte erzeugen) | Exakte Per-Vertex-Kontrolle | **Morph** für Mikro-Deformationen |
| Prozedurale Animation | Einfach (IK, Physik-Knochen) | Schwierig | **Skelett** für Chassis-Bewegung |
| Hard-Surface-Gelenke | Kann an Grenzen artifakten | Sauber | **Morph** für Panel-Lücken-Schließung |
| **Bestes Einsatzgebiet** | **Haupt-Transformationsanimation** | **Glas, Panel-Mikro-Deformationen** | **Hybrid** |

**Konkrete Pipeline:**
1. Skelettale Hauptanimation (60–150 Knochen für Panels, Glieder, Chassis-Elemente)
2. Morph Targets für sekundäre Details: Glasscheiben-Kompression, Panel-Spalt-Schließung, Oberflächen-Mikro-Verformungen
3. Transformationszeit: 15–30 Frames (0,25–0,5s bei 60fps) — [Transformers: War for Cybertron](https://thatgamersasylum.wordpress.com/2017/03/01/transforming-in-transformers-war-for-cybertron/) als Benchmark

#### State Machine für Drive/Fly/Jump/Mech Modi

Basierend auf [Game Programming Patterns — State](https://gameprogrammingpatterns.com/state.html) und [Unreal Engine State Machines Dokumentation](https://dev.epicgames.com/documentation/en-us/unreal-engine/state-machines-in-unreal-engine):

```
┌─────────────────────────────────────────────────────────────────┐
│                    MARILIKTUX STATE MACHINE                       │
│                                                                   │
│  ┌───────────────────┐   [Transform Key]  ┌──────────────────┐  │
│  │   DRIVE MODE      │──────────────────►│  TRANSFORMATION   │  │
│  │   - Bodenfysik    │◄──────────────────│  (transitional)   │  │
│  │   - Auto-Controls │   [Anim complete] │  - Anim-Clip läuft│  │
│  └─────────┬─────────┘                  │  - Collider swap   │  │
│            │                            │  - Input lock      │  │
│   [Rampe / │ Boost-Pad]                 └────────┬──────────┘  │
│            ▼                                     │ [→ Modus]   │
│  ┌───────────────────┐   [Transform Key]          │             │
│  │   JUMP / AIR      │◄──────────────────────────┤             │
│  │   - Ballistik     │                            │             │
│  │   - Limited Air   │   [Doppelsprung /          │             │
│  │     Control       │    Transform]              │             │
│  └─────────┬─────────┘                  ┌────────▼──────────┐  │
│            │ [transform]                │   FLIGHT MODE     │  │
│            └───────────────────────────►│   - Auftriebphysik│  │
│                                         │   - 3D-Controls   │  │
│  ┌───────────────────┐                  └────────┬──────────┘  │
│  │   MECH/ROBOT MODE │◄───────────────────────── │             │
│  │   - Biped-Physik  │  [Halten + Transform]     │             │
│  │   - Melee/Ranged  │                            │             │
│  └─────────┬─────────┘                            │             │
│            │ [transform back]                      │             │
│            └──────────────────────────────────────┘             │
└─────────────────────────────────────────────────────────────────┘

TRANSITION STATE REGELN (nach Unreal Engine Best Practices):
- Collider-Swap: Drive-Collider deaktiviert bei T=50%; Robot/Flight-Collider aktiv bei T=50%
- Input-Buffering: Spieler-Inputs während Transformation gepuffert, nach Modus-Wechsel ausgeführt
- Kamera/HUD: Event-basiert bei Anim-Midpoint umgeschaltet
```

**Transition State Best Practices** ([Unreal Engine Dokumentation](https://dev.epicgames.com/documentation/en-us/unreal-engine/state-machines-in-unreal-engine)):
1. Dedizierter TRANSFORMATION-Transitionszustand — niemals direkt zwischen Modi springen
2. Physics-Collider-Handoff: Fahrzeug-Collider bei T=50% deaktivieren, Roboter/Flug-Collider bei T=50% aktivieren
3. Input-Buffering: Spieler-Inputs während der Transformationsphase speichern

#### Meshy.ai Pipeline für Robot-Mode Rigging

[Meshy AI Auto-Rigging API](https://docs.meshy.ai/en/api/rigging-and-animation) bietet automatisches Rigging für humanoidale Assets. **Kritische Einschränkung:** Die API unterstützt ausschließlich humanoidale (bipedale) Assets — Fahrzeugmodelle und nicht-humanoidale Roboter sind nicht unterstützt.

**Praktische Pipeline für Mariliktux:**
```
1. Robot-Mode-Mesh generieren via Meshy image-to-3D oder text-to-3D
2. Meshy Auto-Rigging API → Walking/Running-Basis-Animationen für Mech-Modus
   POST /openapi/v1/rigging { "model_url": "...", "height_meters": 1.8 }
3. Rigged GLB/FBX von Meshy exportieren
4. In Blender importieren → Transformations-Knochen manuell hinzufügen
5. Transformationssequenz manuell animieren (Dual-Rig-Bridge-Technik)
6. Komplettes Rig in Three.js importieren (GLTFLoader + Skeletal Animation)
```

**Kosten:** Auto-Rigging = 5 Credits/Aufruf; Animation = 3 Credits/Aufruf ([Meshy API Platform](https://www.meshy.ai/api)).

#### Visuelle Tricks für flüssige Transformationen

Professionelle Produktionen nutzen strategische „Täuschung" — das sogenannte **Cheating-Prinzip** ([r/blender Transformers Earthspark Produzent, 2023](https://www.reddit.com/r/blender/comments/188osb6/so_how_can_i_animate_transformers_transformation/)):

> *„The transition was made by creatively breaking the characters and making some parts of one disappear while parts of the other appear. It takes some poses that anticipate and overshoot the transition. If it happens fast, in between 3 and 5 frames you buy it."*

| Technik | Einsatz |
|---------|---------|
| **Partikel-Burst am Transform-Peak** | Motion-Blur-Ersatz; deckt geometrische Diskontinuitäten ab |
| **Radiale Geschwindigkeitslinien** | Kamera-Space-Effekt während 3–5-Frame-Peak |
| **Scale-to-Zero / Scale-from-Zero** | Räder skalieren zu 0, Roboter-Teile skalieren aus 0 |
| **1-Frame White Flash** | Halbwegs-Punkt der Transformation |
| **Sound Design** | Das satisfying „Whirring"-Geräusch als kritisches UX-Element |

Die Animationsprinzipien Anticipation (2–4 Frames), Main Action (10–20 Frames, S-Kurven-Easing) und Overshoot+Settle (3–5 Frames) sind direkt anwendbar ([Blender Transformers Tutorial](https://www.youtube.com/watch?v=_58jlsuKvDE)).

#### LOD-Strategie für animierte Transformer-Modelle

Animierte Transformer-Modelle stellen besondere LOD-Anforderungen, da der Transformations-Skeleton nicht reduziert werden kann, ohne die Animation zu brechen ([CG Spectrum — What is LOD](https://www.cgspectrum.com/blog/what-is-level-of-detail-lod-3d-modeling)):

| LOD-Level | Polygonanzahl | Skeleton | Animation | Morph Targets |
|-----------|--------------|----------|-----------|---------------|
| LOD0 (Spieler) | 50k–150k | Vollständiger Transformations-Rig (100+ Knochen) | Skelett + Morphs | Ja |
| LOD1 (Nah) | 20k–60k | Vollständiger Skeleton beibehalten | Nur Skelett | Nein |
| LOD2 (Mittel) | 7k–20k | Vereinfacht (Haupt-Glieder) | Baked Animations | Nein |
| LOD3+ (Fern) | Impostor Billboard | Keiner | Statisches Bild | Nein |

**Abstandsbasiertes Transformations-Gating:**
```javascript
if (distanceToCamera > 20m) {
    SnapToMode(targetMode); // Sofortiger Modewechsel ohne Animation
} else {
    PlayTransformAnimation(); // Volle Transformationsanimation
}
```

Meshy's Remesh API ermöglicht automatische Polygon-Reduktion von LOD0 auf LOD1–LOD3 ([Meshy AI Blog — Level of Detail](https://www.meshy.ai/blog/level-of-detail)).

---

### 4.2 Level Design

#### Multi-Mode Streckenarchitektur

Das Level-Design-Framework folgt dem Kernprinzip aus [Game Design Skills](https://gamedesignskills.com/game-design/racing/): *„Every bend and curve has a reason to be — 'why?' is the key word of level design."*

Für Multi-Modus-Strecken gilt der **"Why Each Mode?" Design-Test**:
- Schafft dieser Modus-Wechsel eine genuinen anderen Herausforderung, oder nur einen visuellen Swap?
- Kann ein geschickter Spieler sein Können spezifisch in diesem Modus ausdrücken?
- Hat der Abschnitt genug Länge (≥ 15–20 Sekunden), um den Transitions-Overhead zu rechtfertigen?

**Multi-Mode Track Structure Template** (basierend auf [SASRT-Analyse](https://www.adamzwakk.com/games/sonic-racing-transformed/), [WRC7 Level Design](https://www.gamedeveloper.com/design/racing-level-design-the-rally-case), [STK Physics](https://supertuxkart.net/Physics)):

```
[Start-Gate]
  → Breiter Boden-Abschnitt: Driften, Boost-Pad-Reihen, Item-Boxen
  → Rampen-Sequenz: Luft-Transitions-Trigger (3–4 Sek. Ansicht), 2–3 Aerial Rings, Landing Zone
  → Luft-/Wasser-Sektion: Modus-spezifische Challenge (Ringe, 3D-Manövrierfähigkeit)
  → Modus-Reversions-Zone: Fahrzeug transformiert auf designierter Landeoberfläche zurück
  → Technischer Boden-Abschnitt: Enge Kurven, Höhenänderung, Landmark-Feature
  → Breite Gerade: Speed-Erholung, Item-Platzierung
  → Finale Annäherung an Ziellinie
[Lap-Line]
```

#### Transformations-Trigger und Transitions-Design

Basierend auf [Mario Kart 8 Anti-Gravity](https://www.mariowiki.com/Anti-gravity) und SASRT:

**Trigger-Architektur:**
- **Ground → Air:** Rampe/Launch-Pad mit definierter Airtime; Transformationsanimation beginnt *vor* vollständiger Luftnahme (Animation-Preview). Transform Boost als Skill-Belohnung für Trick-Timing.
- **Air → Ground:** Landing Zones sind 1,5–2× breiter als Approach-Abschnitte; Textur-Differenzierung für visuelle Klarheit.
- **Automatische vs. Manuelle Trigger:** SASRT-Ansatz (automatisch, surface-based) für casual Spieler; manueller Trigger als optionale Override-Taste für fortgeschrittene Spieler.

**Transition Design Checklist** ([SuperTuxKart Physics](https://supertuxkart.net/Physics)):
```
[ ] Trigger sichtbar bei hoher Geschwindigkeit (> 3 Sekunden Anflugzeit)?
[ ] Transformationsanimation lang genug für mentale Anpassung?
[ ] Neuer Modus-Abschnitt ≥ 15–20 Sekunden lang?
[ ] Skill-expressive Moment (Trick, Boost-Window) im Transitions-Abschnitt?
[ ] Vergebende Landing/Re-Entry-Zone für niedriger-Skill-Spieler?
[ ] KI folgt derselben Transitionslogik wie der Spieler?
```

#### Track Design Patterns

**Boost Pads (STK: Zippers):** Konfigurierbare Parameter in STK: `Zipper duration`, `Zipper max speed increase`, `Zipper fade out time`, `Zipper speed gain`, `Zipper minimum speed`. Der `minimum speed`-Parameter ist kritisch für Sprung-Rampen — stellt sicher, dass das Fahrzeug die Lücke auch bei niedrigem Anflug überquert.

**Aerial Rings:** Ringe in Clusters von 3–5 mit klaren visuellen Verbindungen. Variation der Ring-Orientierung (vertikal, horizontal, angewinkelt) für 3D-Manövrierfähigkeit. SASRT's Ring Mode demonstriert, dass Aerial-Geometrie über der Bodenstrecke geschichtet werden kann ohne Überlappungsprobleme.

**Landing Zones:** 1,5–2× breiter als Approach-Abschnitte. Textur-Differenzierung von normaler Streckenoberfläche. Koordination mit Anti-Gravity-Zones (wo anwendbar, wie in MK8).

**Magnet Sections (Wandfahrt):** STK unterstützt `Affect gravity`-Textureigenschaft für Wandfahrten. Sparsam einsetzen; durch Übergangs-Korridore ankündigen. [STK Development Blog 2025](https://blog.supertuxkart.net/2025/12/karting-without-hitch.html) bestätigt neue Wandfahrt-Physik in STK Evolution.

#### Prozedurale Streckengenerierung

Für die automatische Generierung von Strecken wird der **Grid-basierte Backtracking-Ansatz** empfohlen ([Reddit PCG Showcase 2024](https://www.reddit.com/r/proceduralgeneration/comments/1c5kfe8/complete_procedural_race_track_generator/)):
- Generationszeit ~0,01 Sekunden pro Strecke
- Garantiert valide Circuits ohne Selbst-Überschneidungen
- Einfach erweiterbar mit neuen Segment-Typen (Aerial, Mech-Sektionen)

**3D-Erweiterung für Multi-Modus:**
- Aerial-Segmente als Spline-Bögen von der Bodenebene ausgehend und zurückkehrend
- Modus-Übergangspunkte an Altitude-Inflektionspunkten (Rampen-Start, Ring-Gate-Exit)
- Fitness-Funktion-Erweiterungen: Bestraft Transitions die < 5 Sekunden auseinanderliegen; belohnt Transitions an dramatischen Geometriemomenten ([Semanticscholar PCG-Paper](https://pdfs.semanticscholar.org/ba6e/0e5394fdf6242766f17709619d1f8a80ddb9.pdf))

#### Level Design Studio Konzept (Node-basierter Editor)

Die identifizierte Marktlücke: TrackMania (Block-Palette, Grid-beschränkt) und SuperTuxKart (Blender-basiert, hohe Einstiegshürde). Das angestrebte **Node-basierte Editor-Konzept** ([Reddit TrackMania Suggestion](https://www.reddit.com/r/TrackMania/comments/kbzhll/suggestion_for_a_future_trackmania_game_node/)):

```
Node-Architektur:
  Jeder NODE = Querschnitt der Strecke in 3D-Raum
  Attributes: Position, Rotation, Banking, Breite, Höhe, Oberflächentyp, Modus-Trigger-Flag
  Interpolation: Catmull-Rom oder Bézier zwischen Nodes

Visual Scripting für Track-Logik (Trigger/Condition/Effect):
  [Trigger Node] ──► [Condition Node] ──► [Effect Node]
  z.B.:
  [Spieler betritt Zone] → [Modus == Fly] → [Boost-Effekt aktivieren]
  [Item kollidiert] → [Typ == Bombe] → [Kamera-Shake + Explosion]
  [Runde > 1] → [immer wahr] → [Streckenmodifikation aktivieren]
```

Das Visual-Scripting-System entspricht im Wesentlichen STK's AngelScript-System in visueller Form ([STK Scripting](https://supertuxkart.net/Scripting)) und senkt die Einstiegshürde für nicht-technische Level-Designer erheblich.

---

### 4.3 Belohnungs- und Progressionssystem

#### Systemübersicht

Das Progressionssystem folgt dem **Player Progression Architecture**-Muster und vermeidet explizit die Anti-Pattern des Forza-Motorsport-Debakels von 2023 ([Forza Forums Analyse](https://forums.forza.net/t/analysis-on-why-the-car-leveling-and-carxp-system-is-driving-people-away-from-forza-motorsport/653361)):

```
PLAYER PROGRESSION ARCHITECTURE — MARILIKTUX
─────────────────────────────────────────────
Globales Spieler-Level (XP aus allen Aktivitäten)
  → Unlocks: Strecken, Multiplayer-Features, Customization-Slots

Fahrzeug-Mastery (pro Fahrzeug, pro Modus)
  → Unlocks: Upgrades (Performance), Kosmetik (Identität), Titel

Währungs-Ökonomie (Race Credits + Form Tokens)
  → Ausgaben: Upgrades, Kosmetik, Season Pass

Ranked System (iRating + Safety Rating getrennt)
  → Bietet: Competitive Matchmaking, Saisonbelohnungen, Prestige

Challenge-System (Täglich/Wöchentlich/Monatlich)
  → Bietet: Rückkehr-Engagement, fokussierte Skill-Anreize, Credits

Battle Pass (Free + Premium, ~4-Wochen-Saisons)
  → Bietet: Neuen Content-Rhythmus, nachhaltigen Umsatz, Kosmetik-Tiefe

Transformations-Belohnungstrack (Modus-Mastery + Form-Unlocks)
  → Bietet: Identität für die Kernmechanik, aspirationale Kosmetik
```

#### XP-System mit Multi-Source-Belohnung

Gutes XP-System-Design belohnt Spieler für *Dinge, die sie natürlich tun würden* ([DEV Community — Level Up!](https://dev.to/gamepill/level-up-the-art-of-designing-game-progression-and-player-rewards-2603)):

| XP-Quelle | XP-Typ | Hinweis |
|-----------|--------|---------|
| Renn-Position | Globales Spieler-XP | Skalierung: 1. = 150%, Letzter = 50% Basis |
| Sauberes Rennen (keine Kollisionen) | Globales Spieler-XP | +20% Bonus |
| Modus-Mastery (Drifts im Auto, Tricks in der Luft) | Fahrzeug/Modus-XP | Belohnt spezifische Modusfähigkeiten |
| Transformations-Timing (Boost-Windows) | Globales Spieler-XP | Micro-Belohnung für präzise Ausführung |
| Manuelle Schaltung oder Advanced Assists deaktiviert | Globales Spieler-XP | Belohnt erhöhten Schwierigkeitsgrad |
| Nacht/Regen/Spezialkonditionen-Rennen | Globales Spieler-XP | Environmental Challenge-Bonus |

**Payout-Formel:**
```
Basis-Renn-Payout: 500 Credits
Finish-Position-Multiplier: (11 - Position) / 10 × 2.0  → 1. = 2.0×, Letzter = 0.2×
Clean-Race-Bonus: +100 Credits (keine Zwischenfälle)
Challenge-Abschluss: +50–200 Credits pro abgeschlossenem In-Race-Challenge
Schwierigkeits-Multiplier: 1.0 (Einfach) / 1.2 (Mittel) / 1.5 (Schwer)
```

#### Dual-Währungssystem (Race Credits + Form Tokens)

| Währung | Verdient durch | Ausgegeben für |
|---------|----------------|----------------|
| **Race Credits** | Alle Rennteilnahmen, Challenge-Abschlüsse | Normale Fahrzeug-Kosmetik, Basis-Upgrades, Karten-Unlocks |
| **Form Tokens** | Modus-spezifische Challenges, Saison-Belohnungen, Achievement-Meilensteine | Transformations-Formen, Advanced Form Skins, einzigartige Transformations-Fähigkeiten |

Form Tokens schaffen eine dedizierte Ökonomie für das Transformations-System — das Alleinstellungsmerkmal des Spiels. Dedizierte Mariliktux-Spieler, die jeden Modus meistern, fühlen sich anders belohnt als Gelegenheitsspieler.

#### Fahrzeug-Upgrade-Trees pro Modus

Basierend auf dem [Forza Community Skill Tree-Vorschlag](https://forums.forza.net/t/make-a-skill-tree-out-of-the-progression/629469):

```
Fahrzeug-Upgrade-Tree-Struktur
├── STAGE 1 (Standard entsperrt)
│   ├── Motor: Stage 1 Tuning
│   ├── Reifen: Street-Compound
│   └── Karosserie: Leichtbau-Panel-Kit
│
├── STAGE 2 (Entsperrt: 3 Rennen mit diesem Fahrzeug gewonnen)
│   ├── Motor: Stage 2 Tuning
│   ├── Getriebe: Close-Ratio-Getriebe
│   └── Aerodynamik: Basis-Abtrieb-Kit
│
├── STAGE 3 (Entsperrt: Alle Fahrzeug-Challenges abgeschlossen)
│   ├── Motor: Rennmotor
│   ├── Fahrwerk: Vollständiges Renn-Setup
│   └── Spezial: Modus-Verbesserung
│       ├── Auto-Modus: Drift-Stabilisatoren
│       ├── Flug-Modus: Aerodynamischer Schubvektor
│       └── Sprung/Mech-Modus: Impact-Absorber
│
└── STAGE 4 — Transformations-Mastery (Entsperrt: Saison-Meilenstein)
    ├── Transformations-Geschwindigkeits-Upgrade
    ├── Transform-Boost-Dauer-Erweiterung
    └── Signature-Transformations-Animation (Kosmetisch)
```

**Designprinzip:** Upgrade-Trees sind Gameplay-Investitionen. Kosmetische Transformations-Formen bleiben *niemals* Stat-Upgrades — dies ist der ethische Kern des Systems ([GameMakers Battle Pass Guide](https://www.gamemakers.com/p/understanding-battle-pass-game-design)).

#### Achievement-System

Das Achievement-Design folgt den sechs Funktionen nach Josh Ge ([Game Developer — Robust Achievement System](https://www.gamedeveloper.com/design/designing-and-building-a-robust-comprehensive-achievement-system)):

| Kategorie | Beispiele | Funktion |
|-----------|-----------|----------|
| **Modus-Mastery** | „Erster Flug" (ersten Luftabschnitt abschließen), „Fass-Dreher" (10 Barrel-Rolls) | Modus-spezifische Fähigkeiten lehren |
| **Transformation** | „Schnellwechsler" (3 Transformationen in einem Rennen), „Perfektes Timing" (Transform Boost 5×) | Kern-Spielmechanik stärken |
| **Kompetitiv** | „Podest-Finisher" (Top 3 in 25 Rennen), „Überholkünstler" (3 Spieler in einer Runde überholen) | Standard-Rennleistung |
| **Exploration** | „Strecken-Pionier" (alle Alternativrouten auf einer Strecke finden) | Discovery und Mastery |
| **Stil** | „Drift-König" (Drift für 5 Sekunden halten), „Luft-Akrobat" (3 Tricks in einem Luft-Abschnitt verketten) | Skill-Ausdruck |
| **Progression** | „Vollständig aufgerüstet" (Fahrzeug maximieren), „Formen-Sammler" (alle Transform-Formen für ein Fahrzeug) | Sammlungs-Vollständigkeit |

#### ELO/iRating-basiertes Ranking

Das iRacing-Dual-System-Modell ([iRacing License Progression](https://www.iracing.com/license-progression/)) ist der Industriestandard:

| System | Misst | Zweck |
|--------|-------|-------|
| **Safety Rating (SR)** | Sauberes, konsistentes Fahren (Vorfälle pro Kurve) | Verhaltens-Gate: Nicht-Crasher in kompetitive Rennen |
| **iRating (ELO)** | Rennabschluss-Performance relativ zur Konkurrenz | Matchmaking: ähnlich starke Spieler zusammenbringen |

**Mariliktux Anpassung:**
```
Race-End iRating-Anpassung:
  Erwarteter Finish = f(Spieler-iR, alle Gegner-iR)
  Tatsächlicher Finish (normalisiert: 1.0 = 1., 0.0 = Letzter)
  Delta = K × (tatsächlich - erwartet)
  K = 32 für Standard-Rennen, 16 für Zeitfahren

Saison-Reset: Soft Reset (Abfall zu 1500 Median) statt vollständigem Reset
Getrennte Ratings pro Modus: Nur-Auto, Nur-Flug, Gemischter Modus (fördert Modus-Mastery)
```

| Tier | iR-Bereich | Visual | Freischaltet |
|------|-----------|--------|-------------|
| Bronze | 0–1.199 | Bronze-Abzeichen | Basis-Matchmaking |
| Silber | 1.200–1.499 | Silber-Abzeichen | Ranked-Queue-Zugang |
| Gold | 1.500–1.799 | Gold-Abzeichen | Saison-Belohnungs-Track |
| Platinum | 1.800–2.099 | Platinum-Abzeichen | Elite-Leaderboard |
| Diamond | 2.100+ | Diamond-Abzeichen | Einladungs-Events |

#### Battle Pass (Free + Premium, selbsttragend)

Das Fortnite-Modell ist der Goldstandard: 10 $ → ~1.500 V-Bucks verdienen → Nächster Pass kostet 950 V-Bucks → Spieler, die den Pass abschließen, zahlen nichts für Folgepässe ([GameMakers Battle Pass Guide](https://www.gamemakers.com/p/understanding-battle-pass-game-design)).

**Mariliktux Season 1 — „Skyfall" (Luft-Fokus):**
```
Free Track (100 Tiers):
  Tier 1:   500 Race Credits
  Tier 25:  Luftform-Skin (Einsteiger-Level)
  Tier 50:  500 Race Credits
  Tier 75:  Einzigartiger Luft-Trail-Effekt
  Tier 100: Exklusive Luft-Transformations-Animation

Premium Track (+9,99 €):
  Tier 1:   1.000 Race Credits + sofortiger Premium-Fahrzeug-Skin
  Tier 25:  Einzigartiges Flugform-Flügeldesign
  Tier 50:  Saison-Signature-Fahrzeug (exklusives Bodywork)
  Tier 75:  Form Token × 5
  Tier 100: Skyfall Legendary Transformation (animierte Formwechsel-Sequenz + Sound)

Saison-Währung verdient (Premium): 1.200 Form Tokens (Nächster Pass kostet 1.000)
```

**Ethische Grenzen** (Non-negotiable):
1. Kein Pay-to-Win; Echtes Geld kauft niemals kompetitive Vorteile
2. Alle Gameplay-Inhalte erspielbar; keine Strecken oder Modi hinter Paywall
3. Transparente Loot-Wahrscheinlichkeiten (falls zufällige Elemente)
4. Selbsttragender Premium-Währungsloop ([Reddit Free Lootboxes Diskussion](https://www.reddit.com/r/truegaming/comments/wawsv6/free_lootboxes_with_direct_paid_alternative_is/))

---

### 4.4 Race Timeline und Renn-AI

#### Renn-Modi

SuperTuxKart unterstützt folgende Game Modes ([STK Source Code main.cpp](https://github.com/supertuxkart/stk-code/blob/master/src/main.cpp)):

| Modus-ID | Name | Mariliktux-Anpassung |
|----------|------|---------------------|
| 0 | Normales Rennen | Multi-Modus-Runden mit Transformations-Triggern |
| 1 | Zeitfahren | Ghost-Racing; Replay-System; Leaderboard |
| 3 | Grand Prix | Punktebasierte Saison-Wertung |
| 4 | Follow-the-Leader | Eliminierungs-Modus mit Modus-Wechsel-Strategie |

#### Rubber-Banding und Adaptive Schwierigkeit

[GameAIPro Kapitel 42](http://www.gameaipro.com/GameAIPro/GameAIPro_Chapter42_A_Rubber-Banding_System_for_Gameplay_and_Race_Management.pdf) definiert das Rubber-Banding-System mit fünf Distanzzonen:

| Zone | Distanz zum Spieler | Effekt |
|------|---------------------|--------|
| a | < -150 (max. zurück) | Konstanter max. positiver Speed-Multiplier (z.B. 1.4×) |
| b | -150 bis -50 | Linear positiver Anstieg |
| c | -50 bis +50 (Dead Zone) | Kein Effekt — normales Verhalten |
| d | +50 bis +200 | Linear negative Abnahme |
| e | > +200 (max. vorn) | Konstanter max. negativer Speed-Multiplier (z.B. 0.6×) |

**Bevorzugte Implementierung:** Driver-Skill-Modifier (nicht Power-Modifier), da er weniger für den Spieler sichtbar ist — die Fahrzeug-Physik bleibt unverändert, nur das Fahr-Können der KI variiert.

**Deaktivierungs-Bedingungen:** Rennstart (verhindert Bündelung), KI bei niedriger Geschwindigkeit (nach Crash), KI wird überrundet.

**Adaptive Schwierigkeit:** MotoGP 24 und RaceRoom implementieren adaptive KI, die nach 3–4 Rennen basierend auf Spieler-Rundenzeiten anpasst. Für Mariliktux: Session-Level-Stats-Tracker → `setSkillLevel(float)` API → passt Steuerungs-Zufälligkeit und Distanz-Schwellenwerte an.

#### FSM-basierte KI-Architektur (4-Layer)

[GameAIPro Kapitel 38](https://www.gameaipro.com/GameAIPro/GameAIPro_Chapter38_An_Architecture_Overview_for_AI_in_Racing_Games.pdf) definiert die vollständige 4-Layer-Produktions-KI:

```
┌────────────────────────────────────────────────────────────┐
│  CHARACTER / PERSONA-LAYER  (langer Zeitraum)              │
│  • Fahrer-Skill-Level (Grip-Multiplier)                    │
│  • Biorhythmus-Modulation (Sinuswelle über Zeit)            │
│  • Persönlichkeitsmerkmale (Aggression, Kontrolle, Fehler) │
├────────────────────────────────────────────────────────────┤
│  STRATEGIC LAYER  (1 Frame – wenige Sekunden)              │
│  • Verhaltens-FSM mit Utility-Scores:                      │
│    Normal(500) | Überholen(hoch) | Verteidigen(hoch)        │
│    | Branch(situativ) | Erholen(steigt nach Crash)          │
├────────────────────────────────────────────────────────────┤
│  TACTICAL LAYER  (jedes Frame)                             │
│  • Löst konkurrierende Ziele zu Lenk-/Geschwindigkeitsent. │
│  • Behandelt: Überholdetails, Blockierungsmanöver          │
├────────────────────────────────────────────────────────────┤
│  CONTROL LAYER  (jedes Frame)                              │
│  • Berechnet tatsächliche Lenk-, Brems-, Gaszusagen        │
│  • PID-Controller für Racing-Line-Tracking                 │
└────────────────────────────────────────────────────────────┘
              │ Quer über alle Layers │
┌────────────────────────────────────────────────────────────┐
│  KOLLISIONSVERMEIDUNGS-SUBSYSTEM                            │
│  • Langbereich: Routen um Hindernisse planen               │
│  • Kurzbereich: Schnelle Anpassungen (Wände, Fahrzeuge)    │
└────────────────────────────────────────────────────────────┘
```

#### Multi-Mode AI Controller für Drive/Fly/Jump

| Modus | Haupt-Achsen | Navigations-Ansatz |
|-------|-------------|---------------------|
| Fahren | X, Z (2D Boden) | Waypoint-Spline + Rubber-Banding |
| Fliegen | X, Y, Z (3D) | 3D-Navigationsgraph + Höhenkontrolle |
| Springen | Ballistischer Bogen | Prädiktive Landezone-Zielanvisierung |

**Empfohlenes Muster — Mode-Switched Controller:**
```javascript
class RacingController {
    current_mode: Mode;
    controllers: {
        DRIVE: new GroundDrivingAI(waypoints, PIDSteering),
        FLY: new FlyingAI(navMesh3D, pitchYawThrottle),
        JUMP: new JumpAI(boostDirection, airControl)
    };
    
    update() {
        this.detectModeTransition();
        this.controllers[this.current_mode].update();
    }
}
```

**Erweiterungsstrategie für STK SkiddingAI:**
1. **Kurzfristig:** Überschreibe `SkiddingAI::handleAccelerationAndBraking()` für Rubber-Banding
2. **Mittelfristig:** Füge `RaceTimeline`-Klasse hinzu für Lap-Events, Dynamic-Weather-States, Trigger-Zones
3. **Langfristig:** Implementiere `MultiModeAIController`-Factory die `SkiddingAI`, `FlyingAI` oder `JumpAI` basierend auf aktuellem Physikzustand auswählt

#### Ghost-Racing und Replay

Zwei primäre Methoden ([Troy Story Games Devlog](https://www.troystorygames.com/2025/05/14/record-and-replay-ghost-races/)):

**State Snapshot Recording (empfohlen für Position/Rotation):**
- Abtastung von Position, Rotation, Velocity und Animationszustand bei 60Hz
- Playback: Interpolation zwischen Snapshots via `lerp()`
- Forward-Vektor statt Rotationswinkel speichern (verhindert -180/+180-Vorzeichensprung-Artefakte)
- Duplikat-Einträge überspringen für minimale Datengröße
- ~3,6 KB pro Minute bei 60Hz mit Delta-Kompression

**Kombinierter Ansatz:** State Snapshot für Position/Rotation + Input Recording für Animationscues (Lenkrad-Drehung, Bremsleuchten, Transformationsauslösung).

#### Dynamische Events (Wetter, Streckenmodifikation)

```
weather_state = {dry, light_rain, heavy_rain, storm}
ai_grip_multiplier[state]           = {1.0, 0.88, 0.75, 0.60}
ai_braking_distance_multiplier[state] = {1.0, 1.15, 1.35, 1.55}
transition_speed = lerp(current_state, new_state, dt * transition_rate)
```

Dynamische Strecken-Modifikationen nach iRacing-Vorbild ([Fanatec Sim Racing Guide](https://www.fanatec.com/us/en/explorer/games/gaming-tips/the-impact-of-weather-conditions-on-sim-racing-physics-and-strategy/)): Trümmer und Öllecks nach Crashes, sich öffnende/schließende Abkürzungen, dynamische Reifengummi-Aufbauten auf der Racing-Line.

---

### 4.5 Settings Area und Level Design Studio

#### Settings-Architektur

Alle Einstellungen sind in vier Haupt-Kategorien gegliedert:

| Kategorie | Parameter | Besonderheiten |
|-----------|-----------|----------------|
| **Grafik** | Qualitäts-Preset (Low/Medium/High), Schatten-Auflösung, Partikel-Anzahl, LOD-Distanzen, Anisotropie | Automatische Geräteerkennung; Device-Tier-basierte Defaults |
| **Audio** | Radio-Modus (Online/Self/Spotify), Lautstärke-Mixer (Musik/SFX/Stimmen), Audio-Qualität | Getrennte Slider pro Audio-Kanal |
| **Steuerung** | Tastenbelegung (inkl. PTT für Sprachchat), Controller-Mapping, Lenkempfindlichkeit, Assists | Vollständig neubelegbar |
| **Netzwerk** | Region-Auswahl, Ping-Schwellenwert, Bandbreiten-Limit | STK-Parameter: `ping-threshold-ms = 300`, `jitter-threshold-ms = 100` |

**Performance-Presets (Three.js Qualitäts-Tier-Konfiguration):**
```javascript
const qualityPresets = {
  low: {
    shadowMapSize: 512, particles: 5000,
    postProcessing: false, lodDistances: [0, 30, 100], anisotropy: 2
  },
  medium: {
    shadowMapSize: 1024, particles: 50000,
    postProcessing: ['bloom', 'fxaa'], lodDistances: [0, 50, 200], anisotropy: 8
  },
  high: {
    shadowMapSize: 2048, particles: Infinity, // WebGPU Compute
    postProcessing: ['bloom', 'motionBlur', 'fxaa', 'dof'],
    lodDistances: [0, 80, 300], anisotropy: 16
  }
};
```

#### Level Design Studio — Vollständiges Konzept

**Workflow (4 Phasen, basierend auf [WRC7 Level Design](https://www.gamedeveloper.com/design/racing-level-design-the-rally-case)):**

**Phase 1: Konzept**
- Emotionale Identität der Strecke definieren (narrativer Kontext)
- 2D-Layout in Draw.io oder auf Papier skizzieren
- 1–3 Landmark-Momente identifizieren
- Modus-Übergangspunkte auf der Skizze markieren

**Phase 2: Greybox**
- Strecke ohne Grafik modellieren — reine Geometrie
- Driveline und Checklines definieren
- Physik-Texturen hinzufügen (Zippers, Magnets, Cannons, Reset-Zones)
- Playtest bei voller Geschwindigkeit: Transitions-Timing, Modus-Sektions-Länge bewerten

**Phase 3: Iteration**
- Modus-Sektions-Längen basierend auf Playtest-Daten anpassen
- Item-Platzierung über Routen und Modi ausbalancieren
- Zipper-Parameter auf Sektions-Geschwindigkeiten abstimmen
- KI-Driveline für alle Modi verifizieren

**Phase 4: Kunst**
- Geometrie-Details mit Kunst hinzufügen
- Landmarks (visuell und Gameplay) platzieren
- Beleuchtung, Partikel, Sounds finalisieren

---

### 4.6 Audio-System

#### Online Radio Integration (Icecast/SHOUTcast Streams)

Icecast wird gegenüber SHOUTcast empfohlen ([Hostcode Lab Vergleich](https://hostcodelab.com/shoutcast-vs-icecast-streaming-server/)) aufgrund:
- GNU GPL v2 (Open Source)
- Mount-Point-System mit automatischem Fallback
- Unterstützung für Ogg Vorbis, MP3, AAC, FLAC, Opus

**Stream-Format-Empfehlungen:**

| Format | Bitrate | Latenz | CPU-Decode | Empfehlung |
|--------|---------|--------|------------|------------|
| MP3 | 128–320 kbps | 2–5s Buffer | Niedrig | Universelle Kompatibilität |
| Ogg Vorbis | 96–256 kbps | 2–5s Buffer | Niedrig | Open Source, Icecast-nativ |
| AAC (HE) | 64–128 kbps | 2–5s Buffer | Niedrig | Bessere Qualität bei niedriger Bitrate |
| Opus | 32–128 kbps | < 100ms möglich | Sehr niedrig | Beste für Niedrig-Latenz-Radio |

**RadioManager-Architektur:**
```
RadioManager
  ├── StationList [
  │       { name: "NightWave Radio", url: "https://nwr.fm/128.mp3", genre: "Synthwave" },
  │       { name: "FutureBeats", url: "...", genre: "Electronic" }
  │   ]
  ├── StreamDecoder (eigener Thread)
  │       └── HTTP-Client → MP3/Ogg dekodieren → Ring-Buffer
  ├── AudioPlayback (Haupt-Thread)
  │       └── Ring-Buffer konsumieren → Audio-Device-Ausgabe
  ├── MetadataParser
  │       └── ICY-Metadata → HUD-Anzeige (Tracks-Titel/Künstler)
  └── VolumeController
          └── Fade-in/out bei Stations-Wechsel oder Fahrzeug-Austritt
```

#### Spotify Personal Playback (Web Playback SDK, nur Premium)

Wichtige Restriktionen nach [Spotify Developer Policy](https://developer.spotify.com/policy) und [TechCrunch-Bericht (Februar 2026)](https://techcrunch.com/2026/02/06/spotify-changes-developer-mode-api-to-require-premium-accounts-limits-test-users/):

| Einschränkung | Details |
|---------------|---------|
| Nur Premium | Streaming von Musik-Tonaufnahmen nur für Spotify-Premium-Abonnenten |
| Kein kommerzieller Verkauf | Darf nicht verkauft, monetarisiert oder mit E-Commerce verknüpft werden |
| Keine Synchronisation | Darf nicht mit Filmaufnahmen, Werbung oder Spiels-Cutscenes synchronisiert werden |
| Kein Background-Broadcast | Darf nicht als nicht-interaktive Internet-Webcast (Radio-Stil an mehrere Hörer) genutzt werden |

**Für Mariliktux machbar:** Spieler authentifizieren sich mit eigenem Spotify-Premium-Konto → Spiel erstellt Spotify-Connect-Gerät via Web Playback SDK → Spieler browst/wählt eigene Playlists → Spiel steuert Lautstärke/Pause via SDK-Methoden. **Nicht machbar:** Geteilte Radio-Stationen, Nicht-Premium-Nutzer, synchronisierte Musik während Cutscenes.

Ab März 2026: Developer Mode erfordert Spotify-Premium-Konto; max. 5 autorisierte Test-User pro Client ID ([Spotify Developer Blog](https://developer.spotify.com/blog/2026-02-06-update-on-developer-access-and-platform-security)).

#### Self Radio (eigene MP3s im GTA V-Stil)

[Rockstar Games GTA V Self Radio](https://www.rockstargames.com/newswire/article/25o2411812a799/self-radio-create-your-own-custom-radio-station-in-gtav-pc) als Vorbild:
- Spieler verweist auf lokalen Musikordner
- Mindestens 4 Tracks für Aktivierung
- Drei Wiedergabe-Modi: Reihenfolge, Shuffle, Radio (mit DJ-Charakter und Pausen-Simulation)
- Volume-Normalisierung: Community empfiehlt 92–95 dB für Parität mit lizenzierten Stationen

#### Dynamische Rennmusik (FMOD/Wwise Stems)

State-basiertes Musik-System ([CBGameDev Substack](https://cbgamedev.substack.com/p/010-designing-dynamic-game-music)):

| Spielzustand | Musik | Auslöser |
|-------------|-------|---------|
| Countdown | Gespannter Aufbau | 3-2-1-GO |
| Erster Lap | Energetisches Intro | Rennstart |
| Führungsposition | Triumphierend | Spieler auf 1. Platz |
| Hinterherfahren | Gespannte Aufholjagd | Spieler außerhalb Top 3 |
| Letzter Lap | Klimax/Intensivierte Version | Last-Lap-Trigger |
| Zieleinlauf | Sieg/Niederlage-Stinger | Renn-Abschluss |

**FMOD-Empfehlung** für Indie-Größe ([The Game Audio Co.](https://www.thegameaudioco.com/wwise-or-fmod-a-guide-to-choosing-the-right-audio-tool-for-every-game-developer)): Einfachere Iteration als Wwise; royaltyfrei bis 200K€ Umsatz; genutzt in Celeste, Hades, Hollow Knight.

#### Rechtliche Rahmenbedingungen für Musik

Erforderliche Lizenztypen ([That Pitch Musik-Lizenzführer 2025](https://thatpitch.com/blog/music-licensing-for-video-games/)):

| Lizenztyp | Abdeckt | Wann benötigt |
|-----------|---------|---------------|
| **Sync License** | Musik mit Spiels-Visuals verknüpfen | Immer für In-Game-Musik |
| **Master Use License** | Die Originalaufnahme | Für alle bestehenden Aufnahmen |
| **Publishing Rights** | Die musikalische Komposition | Für jede Komposition |
| **Performance Rights** | Live/Streaming-Aufführungen | Wenn Spieler das Spiel streamen |

**Empfehlung für Indie-Entwickler:** Royalty-free Libraries wie HookSounds (200–500 €/Jahr für unbegrenzte Nutzung) oder Artlist; PRO-Registrierung (GEMA für Deutschland) und Cue-Sheets für kommerzielle Releases.

---

### 4.7 Sicherer Sprachchat

#### WebRTC + SFU-Architektur (LiveKit empfohlen)

Für Spiele mit 8–50 Spielern ist eine **SFU (Selective Forwarding Unit)**-Architektur der skalierbare Ansatz ([LiveKit GitHub](https://github.com/livekit/livekit)):

```
Spieler A ──┐
Spieler B ──┼──── LiveKit SFU ──┬── Spieler A
Spieler C ──┤                    ├── Spieler B
Spieler D ──┘                    ├── Spieler C
                                  └── Spieler D
```

**LiveKit** ist der empfohlene Open-Source-SFU ([Apache 2.0](https://github.com/livekit/livekit)) mit:
- Positions-basierter selektiver Weiterleitung (Proximity Chat server-seitig)
- Lautsprecher-Erkennung, Simulcast, E2EE
- JWT-Authentifizierung, Webhooks für Moderation-API
- Unity SDK (regulär + WebGL)

**Connection Flow:**
```
1. Spieler A erstellt RTCPeerConnection
2. getUserMedia() → Mikrofon-Stream
3. SDP-Offer erstellen → über Signaling-Server senden
4. Spieler B antwortet → ICE-Kandidaten tauschen
5. DTLS-Handshake → SRTP-Session etabliert
6. Audio fließt verschlüsselt zum LiveKit SFU
```

#### DTLS-SRTP Verschlüsselung + optionale E2EE

**Standard-WebRTC-Verschlüsselung via DTLS-SRTP** ([WebRTC for the Curious — Securing](https://webrtcforthecurious.com/docs/04-securing/)):
- **DTLS (Datagram Transport Layer Security):** TLS für UDP; Diffie-Hellman Key-Exchange; Zertifikate gegen SDP-Fingerprints verifiziert → verhindert MITM
- **SRTP (Secure Real-time Transport Protocol):** RTP-Medien-Pakete verschlüsseln; Nonce = RTP Sequence Number + Rollover-Counter → unique Ciphertext pro Paket

**Einschränkung:** In SFU-Architektur verschlüsselt DTLS-SRTP zwischen Client und SFU — der SFU-Operator kann unverschlüsseltes Audio sehen.

**Optionale True E2EE** via [WebRTC Insertable Streams](https://webrtchacks.com/true-end-to-end-encryption-with-webrtc-insertable-streams/):
```javascript
const room = new Room({ e2ee: { keyProvider: new ExternalE2EEKeyProvider() } });
// LiveKit nativ unterstützt E2EE via Insertable Streams
```

#### Opus Codec (6–510 kbps, ~70ms Latenz)

[Opus Codec (RFC 6716)](http://opus-codec.org) — IETF-Standard, kombiniert Skypes SILK und Xiph.Orgs CELT:

| Spezifikation | Wert |
|--------------|------|
| Bitrate-Bereich | 6 kb/s bis 510 kb/s |
| Sample-Raten | 8, 12, 16, 24, 48 kHz |
| Frame-Größen | 2,5, 5, 10, 20, 40, 60 ms |
| Algorithmische Latenz | 26,5 ms (bei 20ms Frame) |
| Lizenz | Vollständig offen, royalty-frei |

**Empfohlene Game-Voice-Chat-Einstellungen:**
```
bitrate:    32–64 kbps (Voice), 96–128 kbps (Musik)
frame_size: 20 ms (gute Latenz/Qualitäts-Balance)
fec:        true (Forward Error Correction bei Paketverlust)
complexity: 5 (Balance aus CPU und Qualität)
```

Totale Latenz Budget:
```
Mikrofon → Erfassung:     5–10ms
Opus-Encoding (20ms Frame): 20ms
Netzwerk-Transit:           20–100ms (P2P) / 30–60ms (SFU)
Jitter-Buffer:              20–80ms
Opus-Decoding:              <1ms
Lautsprecher-Ausgabe:       5–10ms
Total:                      70–230ms typisch
```

#### Spatial Audio / Proximity Voice

**Proximity-Volumen-Formel** ([Agora Spatial Audio Tutorial](https://www.youtube.com/watch?v=uM89bDIrmZ0)):
```
volume = max(0, (1 - distance / radius) * 100)
```

**Zonen-basiertes Proximity-System:**
```
ProximityRadius = {
    WHISPER:     2m  (sehr leise, nur Team)
    LOCAL:      15m  (benachbarte Spieler)
    SHOUT:      50m  (laute Stimme)
    GLOBAL:      ∞   (Server-weit, benötigt spezielle Berechtigung)
}
```

**HRTF-Verarbeitungspipeline** für jeden Remote-Voice-Stream ([Tencent RTC — 3D Voice in Games](https://trtc.io/blog/details/implementing-3d-voice-in-games)):
```
Voice-Paket empfangen → HRTF-Filter (Azimut, Elevation) →
Entfernungs-Dämpfung → Hindernis-Okklusion → Umgebungs-Hall → Stereo-Ausgabe
```

Performance-Budget für 8-Spieler-Proximity:
- Opus encode/decode: 1–2% CPU
- WebRTC Basis: 1–5% CPU
- Spatial Audio (8 Spieler): 3–5% CPU
- **Gesamt Voice-Chat: ~7–11% CPU** ([Voice-Chat Performance Budget](https://steamcommunity.com/discussions/forum/1/3183488224881665877/))

#### KI-basierte Moderation und Sicherheit

Nach [GGWP AI-Moderations-Blog](https://www.ggwp.com/blog/adding-ai-powered-voice-moderation-tools-to-your-multiplayer-game/): Riot Games (85% Genauigkeit bei missbräuchlichen Inhalten, 100M+ Interaktionen/Tag), Activision (32% Reduktion von Belästigungs-Berichten, < 3 Sekunden automatisierte Aktion).

**Minimales Moderationssystem:**
```
Voice Moderation System:
  ├── Mute (lokal): Spieler stummschaltet spezifische andere Spieler (nur clientseitig)
  ├── Block: Spieler stummschaltet dauerhaft + verhindert Sehen des anderen
  ├── Report: Voice-Clip für Review markieren (30s Buffer bei Meldung speichern)
  ├── PTT Standard: Reduziert versehentlichen Lärm/Belästigung
  └── Safe Mode: Nur Proximity Chat für neue Konten
```

**DSGVO/CCPA-Compliance:** Explizite Zustimmung vor Aufnahme; Löschrecht (Voice-Logs binnen 30 Tagen); Keine biometrische Voiceprint-Speicherung ohne Zustimmung.

---

### 4.8 Multiplayer-Architektur

#### Client-Server mit Server-Autorität

Rennsport-spezifische Merkmale fordern Client-Server mit Server-Autorität ([Hathora Blog](https://blog.hathora.dev/peer-to-peer-vs-client-server-architecture/)):
- **Determinismus ist schwierig:** Physik-Simulationen divergieren schnell über Maschinen mit unterschiedlicher CPU-Geschwindigkeit
- **Speed-Cheating:** P2P erlaubt einem böswilligen Peer, ein teleportierendes Auto oder unmögliche Geschwindigkeit zu melden
- **Positionskonsistenz:** Spieler benötigen autoritativen Konsens für Rundenzeiten, Item-Treffer und Zieleinlauf

```
┌──────────────────────────────────────────────────────────┐
│                     CLIENTS (Browser)                     │
│  Client-Side Prediction (Physik-Simulation, 60fps)        │
│  Input-Erfassung (Lenkung, Gas, Bremse, Item-Nutzung)    │
│  Dead Reckoning für Remote-Karts                         │
│  Interpolation / Blending                                │
└──────────┬─────────────────────────┬─────────────────────┘
           │ WebSocket (Lobby/Events) │ WebTransport (Echtzeit-Status)
           ▼                         ▼
┌──────────────────────────────────────────────────────────┐
│                    GAME SERVER (Node.js)                  │
│  Status-Autorität: Physik bei 20 Hz ausführen            │
│  Input-Validierung: Unmögliche Inputs ablehnen           │
│  Race-Logic: Rund-Zählung, Item-Management, Timing       │
│  Matchmaking / Lobby-Koordination                        │
│  Anti-Cheat: Geschwindigkeits-/Positions-Validierung     │
│  SQLite / Postgres: Leaderboards, Ban-Listen, Profile    │
└──────────────────────────────────────────────────────────┘
```

#### Netcode: Client-Side Prediction, Dead Reckoning, Reconciliation

**Client-Side Prediction (Lokaler Spieler):**
1. Spieler drückt Gas → Auto bewegt sich sofort auf Client
2. Input-Paket wird an Server gesendet
3. Server simuliert Physik, validiert, sendet autoritativen Status zurück
4. Falls Client-Status ≠ Server-Status → Reconciliation (Snap/Blend zurück zum Server-Status)

**Dead Reckoning für Remote-Spieler** ([University of Waterloo, 2023 IEEE TOG](https://ece.uwaterloo.ca/~sl2smith/papers/2023TOG-Predictive_Dead_Reckoning.pdf)):

| Technik | Beschreibung | Fehler bei 200ms / 100km/h |
|---------|-------------|--------------------------|
| Snap | Sprung zu neuer Position bei Update | ~5,6m Sprung pro Update |
| Lineare Interpolation | Glatt zwischen letzten zwei bekannten Positionen | Erscheint hinter Realität |
| Dead Reckoning | Extrapolation mit letzter bekannter Geschwindigkeit/Beschleunigung | ~2m durchschnittlicher Fehler |
| Neural DR (LSTM) | LSTM sagt Trajektorie voraus; ~45% weniger Fehler | ~1,1m (Forschungsstand) |

#### WebTransport als bevorzugtes Protokoll

[NSDI 2025 Forschungspaper](https://aaron.gember-jacobson.com/docs/nsdi2025browser-networking.pdf) misst RTT-Verteilung bei 120 Ticks/Sekunde:

| Rang | Protokoll | Beim Verlust: 0% | Beim Verlust: 0,1% |
|------|-----------|------------------|---------------------|
| 1. (Bestes) | **WebTransport** | Bestes | Bestes |
| 2. | UDP + DTLS | 2. Bestes | 2. Bestes |
| 3. | **WebRTC** | 3. | 3. |
| 4. (Schlechtestes) | **WebSocket** | Schlechtestes | Schlechtestes |

WebTransport gewinnt durch QUICs BBRv1-Congestion-Control-Optimierung für niedrige Latenz + kein Retransmission-Overhead für unzuverlässige Datagramme.

**Hybrides Protokoll-Design:**
- WebSocket für Lobby-Chat, Scores, Rennergebnisse (zuverlässig)
- WebTransport für Echtzeit-Positionssynchronisation (schnell, Verlust akzeptabel)

#### Anti-Cheat Strategie

| Cheat-Typ | Wie es funktioniert | Server-Mitigation |
|-----------|--------------------|--------------------|
| Speed-Hacking | Client meldet unmögliche Geschwindigkeit | Validierung: Geschwindigkeit ≤ `max_speed × physics_constant` |
| Positions-Teleportierung | Client meldet unmögliche Positionsänderung | Validierung: Position-Delta ≤ `max_speed × tick_interval` |
| Runden-Manipulation | Client meldet falsche Rundenzeiten | Server verfolgt Runden-Start/End-Zeitstempel unabhängig |
| Item-Spoofing | Client feuert nicht vorhandenes Item ab | Server verfolgt Item-Inventar; ungültige Fire-Events ablehnen |

**Schlüsselprinzip:** Client sendet Inputs (Lenkwinkel, Gas, Bremse) — nicht Positionen. Server simuliert Physik aus diesen Inputs. Clients können keine falschen Positionen injizieren ([Reddit Server Authority Diskussion 2025](https://www.reddit.com/r/gamedev/comments/1oo7zfy/sever_authoritative_multiplayer_how_does_it_work/)).

#### STK-Netzwerkprotokoll als Basis

[SuperTuxKart NETWORKING.md](https://github.com/supertuxkart/stk-code/blob/master/NETWORKING.md):
- UDP-Transport, Ports 2759 (Spiel), 2757 (Server-Erkennung)
- STUN-Server für NAT-Traversal
- Konfigurierbare `state-frequency` (Standard: 10 Hz)
- Raspberry Pi 3B+ kann 8 Spieler bei ~60% CPU / 60MB RAM hosten

**Mariliktux Browser-Adaptation:**

| STK-Pattern | Browser-Adaptation |
|-------------|-------------------|
| UDP-Status-Sync bei 10 Hz | WebTransport Datagramme bei 20 Hz |
| Rewind + Replay Client-Prediction | JavaScript Circular Buffer + Physik-Rewind |
| STUN NAT-Traversal | WebRTC ICE (Browser automatisch) |
| SQLite Ban-Listen | Server-seitige Datenbank (PostgreSQL + Admin-Panel) |
| Freiwillig betriebene Server | mod.io oder offene Server-Registrierung |

---

### 4.9 Scratch-ähnliches Low-Code Modding

#### Google Blockly als Framework

[Google Blockly](https://developers.google.com/blockly) ist das empfohlene Framework:
- 100% client-seitig (JavaScript); keine Server-Abhängigkeiten
- Apache 2.0 lizenziert; kostenlos für kommerzielle Nutzung
- Integrierte Code-Generatoren für JavaScript, Python, Lua, Dart, PHP
- Erweiterbar für benutzerdefinierte Sprachen
- Betreibt Scratch, Code.org, App Inventor

**Fundamentale Pipeline:**
```
Benutzer arrangiert Blöcke → Blockly AST → Code-Generator → JavaScript/Lua-String → Runtime-Ausführung
```

#### Custom Block-Kategorien für Spielsysteme

**Event-Blöcke (Hat-Blöcke, wie Scratch):**
```
[Wenn Runde startet]
[Wenn Spieler Boost erhält]
[Wenn Item eingesammelt wird]
[Wenn Geschwindigkeit > [n] km/h übersteigt]
```

**Aktions-Blöcke:**
```
[Kart-Farbe auf [Farbe] setzen]
[Geschwindigkeits-Multiplier [n] anwenden]
[Partikel-Effekt [Name] an Kart-Position erzeugen]
[Nachricht [Text] für [n] Sekunden anzeigen]
[Item [Typ] zum Inventar hinzufügen]
```

**Bedingungs-Blöcke:**
```
[Aktuelle Runde > [n]]
[Spieler ist auf erstem Platz]
[Item vom Typ [X] wurde eingesammelt]
```

**Sicherheits-Constraints für jeden Block:**
1. Numerische Inputs: Begrenzung auf sichere Bereiche (Geschwindigkeits-Multiplier 0,1–5,0×)
2. String-Inputs: Whitelist bekannter Namen (Partikel-Effekt-IDs, Item-Typen)
3. Schleifen-Limits: Maximale Iterations-Anzahl in jedem Wiederholungs-Block
4. Ausführungszeit-Limits: Mods mit zu langer Ausführung pro Tick töten oder pausieren
5. Keine direkten Objekt-Referenzen: Mods empfangen undurchsichtige Handles

#### Code-Generierung (Blockly AST → JavaScript/Lua)

```
┌─────────────────────────────────────┐
│           Block Editor UI            │
│  (Google Blockly, custom Palette)   │
└────────────┬────────────────────────┘
             │ generate code
┌────────────▼────────────────────────┐
│          Code Generator             │
│  (Blöcke → sichere JS/Lua API-Calls)│
└────────────┬────────────────────────┘
             │ ausführen in Sandbox
┌────────────▼────────────────────────┐
│       Sandboxed Mod Runtime         │
│  (WASM-Modul OR restricted eval)    │
│  - API-Whitelist erzwungen          │
│  - CPU-Budget pro Tick              │
│  - Kein Filesystem-/Netz-Zugriff   │
└────────────┬────────────────────────┘
             │ validierte Aufrufe
┌────────────▼────────────────────────┐
│        Game Engine Modding API      │
│  (Events, Queries, Commands)        │
└─────────────────────────────────────┘
```

**Progressions-Pfad:** Blöcke → generierter Code anzeigen → Text-Modus — spiegelt Godot Block Coding und MakeCode wider.

#### WebAssembly Sandboxing für Sicherheit

Der Community-Konsens ([Reddit r/rust 2025](https://www.reddit.com/r/rust/comments/1isoiwa/securesandboxed_game_modding_with_rust/)): **WebAssembly ist die beste Sandboxing-Lösung** für Spielmods:

- **Isolation:** Guest-WASM und Host teilen keinen Speicher; Host kontrolliert allen Zugriff
- **Capability-basiert:** Nur explizit freigegebene APIs sind aufrufbar
- **Cross-Platform:** Ein `.wasm`-Binary funktioniert auf allen Plattformen
- **Fehler-Eindämmung:** Fehler in Mods sind wiederherstellbar ohne Spielprozess-Crash
- **WasmTime Gas-Budget:** CPU-Zyklen-Limit pro Mod-Ausführung

> *„WASM gives you all the tools to catch errors without crashing the entire game process."*  
> — [Hacker News, Microsoft Flight Simulator 2024 WASM SDK](https://news.ycombinator.com/item?id=44695364)

**Für Block-basiertes Modding:** Blöcke bieten inhärent starkes Sandboxing — Nutzer können keinen beliebigen Code schreiben. Die Block-Palette definiert die vollständige mögliche API-Oberfläche.

#### mod.io Integration für Community-Sharing

[mod.io](https://mod.io) (2024 Jahresabschluss): 2,6 Milliarden+ Downloads, 30,5 Millionen+ monatlich aktive Nutzer, 1,7 Millionen+ UGC-Ersteller, 64% Marktanteil für offiziellen Mod-Support bei Konsole/VR.

**Business-Ergebnisse für integrierende Spiele:**
- 90%+ Verbesserung der Retention
- 23%+ Umsatzsteigerung
- 1500% höherer UGC-Konsum vs. Drittanbieter

**Integration:** Native SDKs für Unity, Unreal Engine und Custom Engines; White-Label-Integration mit SSO; Moderations-Tools mit KI-Hooks.

---

### 4.10 Grafik und Performance

#### Three.js mit WebGPU (1,6–8,1× Performance vs. WebGL)

Mit WebGPU nun standardmäßig in allen vier großen Browsern ([WebGPU.com, Dezember 2025](https://www.webgpu.com/news/webgpu-hits-critical-mass-all-major-browsers/)) ist der Wechsel zum Three.js WebGPU-Renderer die zukunftssichere Wahl:

```javascript
import { WebGPURenderer } from 'three/webgpu';
const renderer = new WebGPURenderer();
await renderer.init(); // Async GPU-Adapter-Erfassung
// Automatischer WebGL 2-Fallback wenn WebGPU nicht verfügbar
```

**Speedup-Daten** ([SitePoint WebGPU vs WebGL Benchmarks, Februar 2026](https://www.sitepoint.com/webgpu-vs-webgl-inference-benchmarks/)):

| Matrix-Größe | Hardware | WebGL (ms) | WebGPU (ms) | Speedup |
|-------------|----------|-----------|------------|---------|
| 512×512 | Apple M2 | 4,2 | 2,6 | **1,6×** |
| 1024×1024 | Apple M2 | 28,5 | 8,1 | **3,5×** |
| 2048×2048 | RTX 3060 | 98 | 14,2 | **6,9×** |
| 4096×4096 | RTX 3060 | 720 | 89 | **8,1×** |

#### Draw-Call-Optimierung (< 100 pro Frame)

Das goldene Regel aus [Utsubo Three.js Best Practices 2026](https://www.utsubo.com/blog/threejs-best-practices-100-tips):
> *„Target under 100 draw calls per frame. Below 100 draw calls, most devices maintain smooth 60fps. Above 500, even powerful GPUs struggle."*

**Instancing und Batching:**

| Technik | Einsatzbereich | Draw-Call-Reduktion |
|---------|----------------|---------------------|
| **InstancedMesh** | Viele identische Objekte (Bäume, Kegel) | 1000 Objekte → 1 Draw-Call (90%+ Reduktion) |
| **BatchedMesh** | Unterschiedliche Auto-Modelle mit gleichem Material | N Modelle → 1 Draw-Call |
| **Material-Sharing** | Mehrere Meshes, identisches Material | Automatisch von Three.js gebatcht |
| **BufferGeometryUtils.mergeGeometries()** | Statische Streckengeometrie (Zäune, Wände) | Alles in 1 Draw-Call zusammengeführt |

#### Toon/Cel-Shading für Transformer-Ästhetik

**G1-Cartoon-Ästhetik** passt optimal zu Mariliktux und ist computergrafisch effizienter als PBR. Die fünf Schlüsselkomponenten eines vollständigen Toon-Shaders ([Maya Nedeljkovich Tutorial](https://www.maya-ndljk.com/blog/threejs-basic-toon-shader)):

1. **Flache Farbbasis** — einzelne Basisfarbe für das Mesh
2. **Kern-Schatten** — scharfer Cutoff zwischen beleuchten und Schatten via `smoothstep`
3. **Spiegelnde Reflexion** — Blinn-Phong-Spekular mit `smoothstep` für knackiges Highlight
4. **Rim-Licht** — Kanten-Highlight via `1.0 - dot(viewDir, normal)` für Cartoon-Tiefe
5. **Empfangene Schatten** — `getShadow()` mit `PCFSoftShadowMap` für weiche Cartoon-Schatten

```glsl
// Kern-Schatten-Band — Fragment-Shader
float lightIntensity = dot(vNormal, lightDir);
float shadow = smoothstep(0.0, 0.01, lightIntensity); // knackiger Cutoff
gl_FragColor = vec4(uColor * shadow, 1.0);
```

Three.js' eingebautes [`MeshToonMaterial`](https://threejs.org/docs/#api/materials/MeshToonMaterial) bietet 3-Ton oder 5-Ton Toon-Shader ohne benutzerdefinierten Shader-Code.

**TSL (Three Shader Language)** ist die empfohlene Methode für neue Three.js-Projekte (r171+) — write-once, kompiliert zu WGSL (WebGPU) oder GLSL (WebGL):
```javascript
import { Fn, vec4, floor } from 'three/tsl';
const toonMaterial = new THREE.NodeMaterial();
toonMaterial.colorNode = Fn(() => {
  const light = dot(normalView, directionalLight.direction).clamp(0, 1);
  const bands = floor(light.mul(3.0)).div(3.0); // 3 Toon-Bänder
  return vec4(baseColor.mul(bands), 1.0);
})();
```

#### Particle Systems (GPGPU für 500k+ Partikel)

| Ansatz | Max. Partikel | CPU-Kosten | GPU-Kosten | Browser-Unterstützung |
|--------|--------------|------------|------------|----------------------|
| **CPU (JS) Partikel** | ~50.000 | Hoch | Niedrig | Alle |
| **GPGPU (Fragment-Shader)** | ~500.000 | Minimal | Mittel | WebGL 2+ |
| **WebGPU Compute Shader** | Millionen | Nahe null | Hoch | Chrome 113+, Firefox 141+, Safari 26+ |

**WebGPU Compute Shader Pattern (2025+):**
```javascript
import { instancedArray } from 'three/tsl';
const positions = instancedArray(particleCount, 'vec3');
const velocities = instancedArray(particleCount, 'vec3');
const updateParticles = Fn(() => {
  const pos = positions.element(instanceIndex);
  const vel = velocities.element(instanceIndex);
  velocities.element(instanceIndex).assign(vel.add(gravity.mul(deltaTime)));
  positions.element(instanceIndex).assign(pos.add(vel.mul(deltaTime)));
})().compute(particleCount);
```

[WebGPU Experts (Januar 2025)](https://www.webgpuexperts.com/best-webgpu-updates-january-2025): Fluid-Simulationen mit **300.000 Partikeln** auf Mid-Range-GPUs in Echtzeit.

**Renn-Partikel-Effekte:**

| Effekt | Technik | Hinweis |
|--------|---------|---------|
| Auspuff-Rauch | GPU-Partikel (Billboard-Quads) | Velocity-basierte Lebenszeit, Alpha-Ausblendung |
| Boost-Flammen | GPU-Partikel + Additive Blending | Orange/Blau Farbgradient nach Geschwindigkeit |
| Reifen-Rauch/Drift | GPGPU Fragment-Shader | Bodenkontakt-Erkennung, Weiß/Grau |
| Transformations-Effekte | TSL Node-Materials + Morphing | Pro-Kart Transformationsanimation |

#### Post-Processing Pipeline (Bloom, Motion Blur, FXAA)

**Empfohlene Effekte:**

| Effekt | Zweck | Performance-Kosten | Priorität |
|--------|-------|--------------------|-----------|
| **Bloom (Unreal)** | Glühende Boost-Flammen, Geschwindigkeits-Pads | Mittel | Hoch |
| **Motion Blur** | Geschwindigkeitsgefühl | Hoch | Mittel |
| **FXAA/SMAA** | Anti-Aliasing ohne MSAA-Kosten | Niedrig | Hoch |
| **Farb-Grading (LUT)** | Stilisierte Farbpalette pro Strecken-Biom | Sehr niedrig | Hoch |
| **Chromatic Aberration** | Impact-Feedback, Item-Screen-Flash | Nahe null | Niedrig |
| Screen Space Reflections | Glänzende Streckenoberflächen | Sehr hoch | **Vermeiden** in Racing |

**pmndrs/postprocessing** für optimierte Passes ([Three.js Journey Post-Processing](https://threejs-journey.com/lessons/post-processing)):
```javascript
import { EffectComposer, BloomEffect, MotionBlurEffect } from 'postprocessing';
const composer = new EffectComposer(renderer);
composer.addPass(new RenderPass(scene, camera));
composer.addPass(new EffectPass(camera,
  new BloomEffect({ intensity: 1.5, threshold: 0.6 }),
  new MotionBlurEffect({ samples: 8 }),
  // Kombiniert in EINEM zusätzlichen Draw-Pass
));
```

#### LOD-System und Asset-Streaming

```javascript
// Three.js LOD
const lod = new THREE.LOD();
lod.addLevel(highDetailKart, 0);    // Hoch-Poly innerhalb 50 Units
lod.addLevel(medDetailKart, 50);    // Mittel-Poly 50–150 Units
lod.addLevel(lowDetailKart, 150);   // Niedrig-Poly 150–300 Units
lod.addLevel(billboard, 300);       // Billboard/Impostor jenseits 300 Units
```

**Strecken-Asset-Streaming für Lap-Based Racing:**
- Gesamte Strecke beim Renn-Start vorladen (da sie sich wiederholt)
- Nur visuelle Dekorationen (Publikum, Vegetation) basierend auf aktueller Streckenposition streamen
- Draco-Komprimierung (60–80% Größenreduktion) + KTX2-Texturen (4–8× Reduktion, GPU-native Dekomprimierung)

#### Device-Tier Quality Presets

| Device-Tier | Ziel-FPS | Draw-Calls | Post-FX | Partikel |
|-------------|---------|-----------|---------|---------|
| **High-End Desktop** (RTX, M2) | 60+ fps | < 200 | Vollständige Pipeline | WebGPU Compute Shader |
| **Mid-Range Desktop** | 60 fps | < 100 | Bloom + AA | GPGPU Fragment |
| **Low-End Desktop/Laptop** | 30–60 fps | < 60 | Nur AA | CPU, < 20k |
| **Mobile Browser** | 30 fps | < 50 | Minimal | CPU, < 5k |

---

## 5. Evaluationsplan

### 5.1 Playtest-Methodik

Basierend auf den DSR-Evaluationsmethoden von [Hevner et al. (2004, S. 83)](https://wise.vub.ac.be/sites/default/files/thesis_info/design_science.pdf):

| Methode | Beschreibung | Zeitpunkt | Messgröße |
|---------|-------------|-----------|-----------|
| **Analytisch** | Statische Code-Analyse; Architektur-Review | Phase 1–4 | Komplexität, Vollständigkeit, Konsistenz |
| **Simulation** | Automatisierte Performance-Benchmarks (Network AI) | Phase 2, 4 | FPS, Draw-Calls, Latenz |
| **Experiment (Labor)** | Kontrollierte Playtest-Sessions mit 5–10 Testern | Phase 2, 3, 4 | Usability-Score, Fehlerrate |
| **Case Study** | Community Beta-Test mit 50–200 echten Spielern | Phase 4 | Retention, Satisfaction |
| **Feldstudie** | Open Beta Release; Analyse realer Nutzungsdaten | Nach Phase 4 | DAU, Session-Länge, Churn |

**Playtest-Session-Protokoll:**
```
1. Think-Aloud-Test für Level-Editor (Novize-Tester, Ziel: lauffähiger Mod in < 5 Min)
2. Transformations-UX-Test (3 Transformationen in Sequenz, Bewertung der Flüssigkeit 1–10)
3. Multiplayer-Stresstest: 8 gleichzeitige Spieler, gemessen: Latenz P50/P95/P99
4. AI-Rubber-Banding-Test: Spieler mit gemischtem Skill-Niveau; Messung: Renndramaturgie-Score
5. Achievement-System-Test: Spieler-Fortschritt nach 5 Sessions; Messung: Täglich-aktive Challenges abgeschlossen
```

### 5.2 Performance-Benchmarks

```
FPS-Ziele:
  Browser (Chrome, Firefox, Safari) auf Mid-Range-Desktop: ≥ 60 fps
  Browser auf Low-End-Laptop: ≥ 30 fps
  Mobile Browser: ≥ 30 fps

Draw-Call-Limits:
  Rendering.calls < 100 pro Frame (Ziel)
  Rendering.calls < 200 pro Frame (akzeptabel)
  Gemessen mit: renderer.info.render.calls + stats-gl

Latenz-Targets:
  Multiplayer RTT (Europa-Server): ≤ 100ms P95
  Sprachchat-Latenz: ≤ 100ms gesamt
  Transformationsanimation: ≤ 30 Frames (0,5s bei 60fps)
  Inputverzögerung (lokal): ≤ 1 Frame (16,67ms)

Memory-Limits:
  GPU-Speicher: ≤ 512 MB für Low-End
  RAM: ≤ 1 GB Browser-Prozess
  Keine Memory-Leaks über Rennen hinweg (render targets disposed)
```

### 5.3 Usability-Tests für Level Editor und Modding System

**Level Editor Usability:**
- **Task 1:** „Erstelle eine Strecke mit mindestens einem Luft-Abschnitt" (Novize, max. 15 Min.)
- **Task 2:** „Setze einen Transformations-Trigger an Stelle X ein" (Intermediate, max. 5 Min.)
- **Metrik:** Erfolgsrate ≥ 80% für Task 1 bei Novizen; Fehlerzahl ≤ 2 pro Task

**Modding-System Usability:**
- **Task 1:** „Erstelle einen Mod, der eine Nachricht beim Rundenstart anzeigt" (Novize, max. 5 Min.)
- **Task 2:** „Erstelle einen Mod, der die Kart-Farbe bei Speed > 100 km/h ändert" (Intermediate, max. 10 Min.)
- **Metrik:** ≥ 90% Erfolgsrate Task 1; ≥ 70% Erfolgsrate Task 2

### 5.4 A/B-Tests für Belohnungssystem

| Test | Variante A | Variante B | Messgröße |
|------|-----------|-----------|-----------|
| XP-Verdienst | Nur Rundenposition-XP | Multi-Source-XP (Position + Drift + Transform) | 7-Tage-Retention |
| Battle-Pass-Laden | Front-loading (Beste früh) | Hybrid (Peaks bei Tier 30, 60, 100) | Pass-Vollständigungs-Rate |
| Challenge-Schwierigkeit | Einheitliche Schwierigkeit | Adaptive (basierend auf Skill-Level) | Tägliche Challenge-Abschlüsse |
| Form-Token-Verdienst | Nur Saison-Pass | Multi-Source (Challenges + Achievements + Pass) | Form-Token-Ausgaben-Rate |

### 5.5 Netzwerk-Latenz-Tests

```
Test-Szenario: 8 Spieler auf Europa-Server (Frankfurt), gemischte Verbindungen
Protokoll-Vergleich: WebSocket vs. WebTransport (nach NSDI 2025 Methodik)
Metriken:
  RTT P50, P95, P99
  Positions-Fehler (Dead Reckoning Abweichung)
  Paket-Verlust-Rate
  Rubber-Band-Aktivierungs-Rate

Network AI Stresstest:
  8 AI-Clients: supertuxkart --connect-now=127.0.0.1 --network-ai=2 --no-graphics
  Messung: Server-CPU bei max. Last; State-Sync-Konsistenz
```

---

## 6. Entwicklungs-Roadmap

### Phasen-Übersicht

Die Entwicklung ist in vier Phasen mit jeweils drei Monaten Laufzeit gegliedert, basierend auf dem DSRM-Prozessmodell von [Peffers et al. (2007)](https://dl.acm.org/doi/abs/10.2753/MIS0742-1222240302):

```
MONAT:    1    2    3    4    5    6    7    8    9    10   11   12
PHASE:  ├────PHASE 1────┤├────PHASE 2────┤├────PHASE 3────┤├────PHASE 4────┤
         Foundation      Gameplay         Features          Polish
```

### Phase 1: Foundation (Monate 1–3)

**Ziel:** Core Engine, Transformations-System, Basis-Strecke funktionsfähig

| Woche | Aufgabe | Deliverable |
|-------|---------|-------------|
| 1–2 | Three.js/WebGPU-Renderer aufsetzen; STK-Strecken-Asset-Pipeline | Lauffähiger Renderer mit STK-Strecke |
| 3–4 | Basis-Fahrzeug-Controller (Boden-Physik) | Spielbares Fahrzeug auf Test-Strecke |
| 5–6 | Transformations-State-Machine (Drive↔Transformation→Flight) | Erster funktionaler Modus-Wechsel |
| 7–8 | Skelettales Animations-Rig + Morph-Target-Hybrid | Animierter Modus-Wechsel mit visuellen Tricks |
| 9–10 | Meshy.ai Robot-Mode Rigging Pipeline | Erstes Fahrzeug mit Robot-Mode |
| 11–12 | LOD-System (3 Levels) + Performance-Profiling | ≥ 60 fps auf Mid-Range Desktop bestätigt |

### Phase 2: Gameplay (Monate 4–6)

**Ziel:** KI, Belohnungssystem, Multiplayer-Prototyp

| Woche | Aufgabe | Deliverable |
|-------|---------|-------------|
| 1–2 | STK SkiddingAI-Erweiterung (Rubber-Banding) | KI-Rubber-Banding aktiv |
| 3–4 | Multi-Mode AI Controller (Drive/Fly/Jump) | KI navigiert alle Modi |
| 5–6 | XP/Credits/Form-Token-Grundsystem | Funktionales Progression-Backend |
| 7–8 | Achievement-System (20 Achievements in 5 Kategorien) | Achievements in-game sichtbar |
| 9–10 | Multiplayer-Server (Node.js, WebTransport) | 4-Spieler-Online-Test funktional |
| 11–12 | Client-Side Prediction + Dead Reckoning | ≤ 100ms Latenz auf EU-Server |

### Phase 3: Features (Monate 7–9)

**Ziel:** Level Editor, Modding, Audio-System

| Woche | Aufgabe | Deliverable |
|-------|---------|-------------|
| 1–2 | Node-basierter Track-Editor (Basis) | Strecke mit Node-Editor erstellbar |
| 3–4 | Visual Scripting für Track-Logik (Trigger/Condition/Effect) | Interaktive Track-Events |
| 5–6 | Google Blockly Integration + Custom Block-Palette | 10 Custom Blöcke funktional |
| 7–8 | WASM-Sandboxing für Mods + mod.io Integration | Mod-Sharing-Platform aktiv |
| 9–10 | Online-Radio (Icecast) + Self-Radio | Radio-System funktional im Spiel |
| 11–12 | FMOD Dynamisches Musik-System (7 Race-States) | Adaptive Rennmusik implementiert |

### Phase 4: Polish (Monate 10–12)

**Ziel:** Performance, Voice Chat, Battle Pass, Launch-Vorbereitung

| Woche | Aufgabe | Deliverable |
|-------|---------|-------------|
| 1–2 | LiveKit SFU Sprachchat + Opus-Codec | Voice Chat mit Proximity-Audio |
| 3–4 | HRTF Spatial Audio + E2EE Option | Vollständiges Spatial-Audio-System |
| 5–6 | Battle Pass System (Free + Premium Tracks) | Season 1 „Skyfall" launchbereit |
| 7–8 | ELO/iRating Ranked System | Kompetitive Rangliste aktiv |
| 9–10 | Performance-Optimierung: Draw-Calls < 100, Mobile | Alle Device-Tiers getestet |
| 11–12 | Beta-Test, Bug-Fixes, Soft-Launch | Öffentlicher Beta-Release |

### Zusammenfassungs-Tabelle

| Phase | Zeitraum | Kernziele | Wichtigste Meilensteine |
|-------|---------|-----------|------------------------|
| **Foundation** | M 1–3 | Core Engine + Transformation | Erster Modus-Wechsel; ≥ 60 fps bestätigt |
| **Gameplay** | M 4–6 | KI + Belohnung + Multiplayer | 4-Spieler-Online; Rubber-Banding aktiv |
| **Features** | M 7–9 | Level-Editor + Modding + Audio | Mod-Sharing aktiv; Blockly-Mods laufen |
| **Polish** | M 10–12 | Performance + Voice + Battle Pass | Öffentlicher Beta-Release |

---

## 7. Technologie-Stack Übersicht

| Kategorie | Technologie | Version/Status | Lizenz | Rolle |
|-----------|-------------|----------------|--------|-------|
| **Rendering** | Three.js | r171+ (WebGPU) | MIT | 3D-Renderer, Partikel, Post-Processing |
| **Grafik-API** | WebGPU (WGSL) | Alle Browser Nov 2025 | W3C-Standard | Compute Shader, 1,6–8,1× Speedup |
| **Shader** | TSL (Three Shader Language) | r171+ | MIT | Write-once WebGPU+WebGL-Shader |
| **Stil** | MeshToonMaterial / Custom Toon-Shader | Three.js | MIT | G1-Cel-Shading-Ästhetik |
| **Post-Processing** | pmndrs/postprocessing | aktuell | MIT | Bloom, FXAA, Motion Blur |
| **Physik** | STK-Physik (bullet.js) | STK 1.5 | GPL v3 | Fahrzeug-/Kollisions-Physik |
| **Partikel** | WebGPU Compute / GPGPU Fragment | r171+ | MIT | Boost, Drift, Transformations-FX |
| **3D-Modellierung** | Blender | 4.x | GPL | Transformer-Rig, Track-Modellierung |
| **Auto-Rigging** | Meshy.ai API | v1 | Commercial | Robot-Mode humanoid Auto-Rigging |
| **Asset-Format** | GLTF + Draco + KTX2 | Industrie-Standard | Open | 60–80% kleinere Assets |
| **Multiplayer** | WebTransport (QUIC) | Chrome/FF/Safari 2025 | W3C-Standard | Echtzeit-Zustand bei 20 Hz |
| **Lobby/Events** | WebSocket | Universal | W3C-Standard | Zuverlässige Events, Lobby-Chat |
| **Server** | Node.js (headless) | 22+ LTS | MIT | Game-Server, Physik-Autorität |
| **Datenbank** | PostgreSQL / SQLite | 16.x / 3.x | Open | Leaderboards, Ban-Listen |
| **KI** | STK SkiddingAI (erweitert) | GPL v3 | GPL | Rubber-Banding, Multi-Mode AI |
| **Sprachchat** | WebRTC + LiveKit SFU | Apache 2.0 | Apache 2.0 | Voice-Chat, Proximity-Audio |
| **Audio-Codec** | Opus (RFC 6716) | 1.5.2 | Open | Voice-Chat, 6–510 kbps |
| **Musik-Engine** | FMOD | 2.x | Commercial | Adaptive Stem-Musik |
| **Online-Radio** | Icecast | 2.4+ | GPL | HTTP-Audio-Streams |
| **Spotify** | Web Playback SDK | 2026 | Spotify Policy | Personal Playback (nur Premium) |
| **Modding-UI** | Google Blockly | 11+ | Apache 2.0 | Scratch-ähnlicher Block-Editor |
| **Mod-Sandbox** | WasmTime | 16+ | Apache 2.0 | WASM-Isolierung für Text-Mods |
| **Mod-Sharing** | mod.io | API v2 | Commercial | Community-Mod-Plattform |
| **Basis-Engine** | SuperTuxKart | GPL v3 | GPL v3 | Kart-Physik, AI, Multiplayer-Protokoll |

---

## 8. Risiken und Mitigationsstrategien

### Technische Risiken

| Risiko | Wahrscheinlichkeit | Impact | Mitigationsstrategie |
|--------|------------------|--------|---------------------|
| **WebGPU-Browser-Inkompatibilität** | Niedrig (Nov 2025: alle Browser) | Hoch | Automatischer WebGL 2-Fallback in Three.js r171+ implementieren |
| **Transformationsanimation zu langsam (> 30 Frames)** | Mittel | Hoch | Cheating-Prinzip: Scale-to-Zero, Partikel-Burst; Blender-Optimierung |
| **Physik-Desynchronisation im Multiplayer** | Mittel | Hoch | Server-autoritative Physik; Client-Side Prediction mit Reconciliation |
| **Morph-Target-LOD-Inkompatibilität** | Mittel | Mittel | Morphs nur auf LOD0 beschränken; LOD1+ rein skelettale Animation |
| **Meshy.ai API-Limiten (nur humanoid)** | Hoch (bekannt) | Mittel | Pipeline-Akzeptanz: Meshy für Robot-Mode-Rig; manuelle Transformation-Bones in Blender |
| **WebTransport-Fallback fehlt** | Niedrig | Mittel | Hybride Strategie: WebTransport primär, WebSocket als Fallback |
| **STK-Physics-Engine-Inkompatibilität mit Browser** | Mittel | Hoch | bullet.js (WebAssembly) als Browser-Port; Determinismus-Tests |
| **WASM-Mod-Sandbox CPU-Overhead** | Niedrig | Niedrig | Gas-Budget-Limits; Block-basiertes Modding als primäre Methode ohne WASM |

### Design- und Produkt-Risiken

| Risiko | Wahrscheinlichkeit | Impact | Mitigationsstrategie |
|--------|------------------|--------|---------------------|
| **Spieler versteht Transformations-Modus-System nicht** | Mittel | Hoch | Onboarding-Tutorial mit erzwungenem erstem Modus-Wechsel; interaktive Erklärungen |
| **Reward-System zu grindy** | Mittel | Mittel | Anti-Pattern-Checkliste strikt anwenden; 1–2 Stunden/Tag sollten sichtbaren Fortschritt zeigen |
| **Level-Editor zu komplex für Novizen** | Mittel | Mittel | Usability-Tests in Phase 3; vereinfachter Modus mit vorgefertigten Strecken-Segmenten |
| **Battle Pass FOMO-Überladung** | Niedrig | Mittel | Maximal 2 aktive zeitlich begrenzte Challenge-Tracks gleichzeitig |
| **Spotify-API-Restrictions verschärft** | Hoch (Trend: weitere Einschränkungen) | Niedrig | Icecast-Radio und Self-Radio als vollständige Alternativen; Spotify als optionales Bonus-Feature |
| **Rubber-Banding zu offensichtlich** | Mittel | Mittel | Driver-Skill-Modifier statt Power-Modifier; Deaktivierungs-Bedingungen strikt einhalten |

### Geschäfts- und Rechtliche Risiken

| Risiko | Wahrscheinlichkeit | Impact | Mitigationsstrategie |
|--------|------------------|--------|---------------------|
| **Musik-Lizenz-Verletzung (DMCA)** | Mittel | Hoch | Royalty-free Libraries (HookSounds, Artlist); explizite Lizenz-Dokumentation; Streaming-Safe-Flag |
| **DSGVO-Verletzung (Voice-Recording)** | Niedrig | Hoch | Explizite Zustimmung vor Aufnahme; Löschrecht in 30 Tagen; keine biometrischen Daten |
| **GPL-Verletzung (STK-Basis)** | Niedrig | Hoch | Alle STK-Erweiterungen unter GPL v3 veröffentlichen; Copyleft-Anforderungen dokumentieren |
| **mod.io API-Preisänderung** | Niedrig | Mittel | Fallback-Plan: eigene Mod-Hosting-Lösung oder Steam Workshop |
| **Spielinhalt-Moderation (User-Generated)** | Mittel | Mittel | GGWP AI-Moderation + menschliche Überprüfung; klare Community-Guidelines |

---

## 9. Fazit

### Synthese der DSR-Leitlinien

Mariliktux demonstriert, dass Design Science Research ein geeigneter methodischer Rahmen für die Spielentwicklung ist — ein Befund, den [Oliveira et al. (2022)](https://games.jmir.org/2024/1/e48099/) und [Loch et al. (2023)](https://papers.academic-conferences.org/index.php/ecgbl/article/view/1567) empirisch bestätigen. Die sieben Leitlinien von [Hevner et al. (2004)](https://wise.vub.ac.be/sites/default/files/thesis_info/design_science.pdf) strukturieren sowohl die Forschungsfrage (Leitlinie 2: Relevanz) als auch die Methodologie (Leitlinie 5: Rigorosität) und die Evaluationsplanung (Leitlinie 3).

Das Drei-Zyklen-Modell ([Hevner 2007](https://aisel.aisnet.org/sjis/vol19/iss2/4/)) verdeutlicht die Synergie zwischen den Zyklen: *„It is the synergy between relevance and rigor and the contributions along both the relevance cycle and the rigor cycle that define good design science research."* Diese Synergie manifestiert sich in Mariliktux konkret: Jede technische Entscheidung (Design Cycle) ist sowohl in Spieler-Anforderungen (Relevance Cycle) als auch in wissenschaftlicher Literatur und existierenden Technologien (Rigor Cycle) verankert.

### Wichtigste Erkenntnisse und Beiträge

**Level 1 — Neues Artefakt (Instanziierung):**
Ein lauffähiger browserbasierter Transformations-Rennspiel-Prototyp, der erstmals in dieser Kombination folgende Systeme integriert: Multi-Modus Echtzeit-Transformation, WebGPU-Toon-Rendering, WebTransport-Multiplayer, Blockly-Modding und Spatial-Audio-Sprachchat.

**Level 2 — Designprinzipien:**
- *Transformations-Geschwindigkeit über -Komplexität:* War for Cybertron beweist empirisch, dass < 0,5s Transformationszeit kritischer ist als geometrische Korrektheit.
- *Hybrid-Animations-Prinzip:* Skelettale Animation für Haupt-Transformation + Morph Targets für Mikro-Deformationen ist die optimale Lösung für browsernative Spiele.
- *Transform Boost als Skill-Ausdruck:* SASRT's Transform Boost ist ein elegantes Mikro-Reward-Muster, das kompetitive Tiefe ohne Bestrafungs-Mechanismus schafft.
- *Anti-Pattern-Vermeidung im Reward-System:* Das Forza-Motorsport-Debakel liefert negatives Wissen: XP darf nie an ein einzelnes Fahrzeug gebunden, nie ohne Abschluss-Decke und nie ohne Skill-Ausdruck-Belohnungen sein.
- *Server-Autorität als Anti-Cheat:* Inputs (nicht Positionen) an den Server senden ist die skalierbar sicherste Multiplayer-Architektur für Rennspiele.

**Level 3 — Design-Theorie (Ansatz):**
Ein Multi-Modus-Fahrzeug-Gameplay-Framework, das folgende Variablen integriert: Transformations-State-Machine, Modus-spezifische KI-Controller, Modus-spezifische Reward-Tracks und Modus-spezifische Streckenabschnitte — ergibt ein kohärentes Design-Modell für zukünftige Transformations-Rennspiele.

### Nächste Schritte

1. **Sofort (Woche 1):** Three.js/WebGPU-Projekt aufsetzen; STK-Strecke importieren; Basis-Fahrzeug-Controller implementieren
2. **Kurzfristig (Monat 1–3):** Erste Transformations-State-Machine funktionsfähig; Performance-Baseline ≥ 60 fps verifiziert
3. **Mittelfristig (Monat 4–6):** Multiplayer-Prototyp mit 4 Spielern; erste KI-Spieler mit Rubber-Banding
4. **Langfristig (Monat 7–12):** Level-Editor, Modding-System, Battle Pass, Öffentlicher Beta-Release

Das Projekt ist als **Open-Source-Beitrag** zum SuperTuxKart-Ökosystem konzipiert. Alle Kern-Erweiterungen werden unter GPL v3 veröffentlicht — ein Rigor-Cycle-Rückfluss, der neue Erkenntnisse in die Community-Wissensbasis zurückführt und damit den DSR-Kreislauf schließt: *„It is the essence of design science to create and evaluate IT artifacts intended to solve identified organizational problems"* ([Hevner et al. 2004, S. 76](https://wise.vub.ac.be/sites/default/files/thesis_info/design_science.pdf)).

---

*Alle Zitate aus Hevner et al. (2004) aus: https://wise.vub.ac.be/sites/default/files/thesis_info/design_science.pdf | Hevner (2007) aus: https://aisel.aisnet.org/sjis/vol19/iss2/4/ | Peffers et al. (2007) aus: https://dl.acm.org/doi/abs/10.2753/MIS0742-1222240302*

---

## Anhang A: Vertiefende technische Analysen

### A.1 Vollständige Implementierungsdetails: Transformations-Animationssystem

#### A.1.1 Skelettale Animations-Pipeline im Detail

Die Implementierung eines voll funktionalen Transformer-Animations-Rigs erfordert eine sorgfältig strukturierte Pipeline, die weit über das einfache Key-Framing hinausgeht. Das Fundament bildet die Bone-Hierarchie, die in Blender entwickelt und als GLTF-Asset exportiert wird.

**Empfohlene Bone-Hierarchie für einen Transformers-Charakter:**

```
Root
└── Chassis_Root
    ├── Car_Mode_Group
    │   ├── Hood_Panel_L
    │   ├── Hood_Panel_R
    │   ├── Windshield_Lower
    │   ├── Windshield_Upper
    │   ├── Door_L
    │   ├── Door_R
    │   ├── Trunk_Panel
    │   └── Roof_Panel
    ├── Wheel_FL (Front Left)
    │   └── Wheel_FL_Spin
    ├── Wheel_FR (Front Right)
    │   └── Wheel_FR_Spin
    ├── Wheel_RL (Rear Left)
    │   └── Wheel_RL_Spin
    ├── Wheel_RR (Rear Right)
    │   └── Wheel_RR_Spin
    ├── Robot_Mode_Group
    │   ├── Pelvis
    │   │   ├── Spine_1
    │   │   │   ├── Spine_2
    │   │   │   │   ├── Chest
    │   │   │   │   │   ├── Shoulder_L
    │   │   │   │   │   │   ├── Upper_Arm_L
    │   │   │   │   │   │   │   ├── Forearm_L
    │   │   │   │   │   │   │   │   └── Hand_L
    │   │   │   │   │   ├── Shoulder_R
    │   │   │   │   │   │   └── (symmetrisch)
    │   │   │   │   └── Neck
    │   │   │   │       └── Head
    │   │   ├── Hip_L
    │   │   │   ├── Thigh_L
    │   │   │   │   ├── Shin_L
    │   │   │   │   │   └── Foot_L
    │   │   └── Hip_R (symmetrisch)
    ├── Flight_Mode_Group
    │   ├── Wing_L
    │   │   ├── Wing_L_Flap
    │   │   └── Wing_L_Tip
    │   ├── Wing_R (symmetrisch)
    │   ├── Thruster_Main
    │   ├── Thruster_L
    │   └── Thruster_R
    └── Transform_Control_Bones
        ├── Transform_Progress (0.0 = Car, 1.0 = Robot)
        ├── Mode_Override
        └── Emergency_Reset
```

Diese Hierarchie ermöglicht es, alle vier Modi (Fahrzeug, Roboter, Flug, Sprung) durch denselben einzigen Skeleton zu steuern. Die Transform_Control_Bones sind Driver-Knochen, die Morph-Target-Gewichtungen und Bone-Constraints über einen einzigen `Transform_Progress`-Wert steuern — ein Paradigma aus professionellen Game-Art-Pipelines.

#### A.1.2 Animationsclip-Struktur

Für das State-Machine-System werden folgende dedizierte Animations-Clips benötigt:

| Clip-Name | Dauer (Frames @ 60fps) | Beschreibung | Loop |
|-----------|------------------------|--------------|------|
| `drive_idle` | 120 | Leichtes Chassis-Wippen, Räder-Spin | Ja |
| `drive_acceleration` | 30 | Chassis-Dip nach hinten bei Vollgas | Ja |
| `drive_brake` | 30 | Chassis-Dip nach vorn beim Bremsen | Ja |
| `drive_drift_L` | 60 | Linkes Drift-Body-Roll | Ja |
| `drive_drift_R` | 60 | Rechtes Drift-Body-Roll | Ja |
| `car_to_transform` | 20 | Transformation beginnt (Anticipation) | Nein |
| `transform_car_to_fly` | 20 | Haupttransformation: Auto → Flieger | Nein |
| `transform_car_to_robot` | 25 | Haupttransformation: Auto → Roboter | Nein |
| `transform_fly_to_car` | 20 | Haupttransformation: Flieger → Auto | Nein |
| `transform_robot_to_car` | 25 | Haupttransformation: Roboter → Auto | Nein |
| `fly_idle` | 60 | Flügel-Flatter, Thruster-Glow-Pulse | Ja |
| `fly_bank_L` | 30 | Linksneigung beim Kurvenflug | Ja |
| `fly_bank_R` | 30 | Rechtsneigung beim Kurvenflug | Ja |
| `fly_barrel_roll_L` | 45 | Vollständige Linksrolle | Nein |
| `fly_barrel_roll_R` | 45 | Vollständige Rechtsrolle | Nein |
| `robot_idle` | 80 | Gewicht-Verlagerung, Kopf-Scan | Ja |
| `robot_walk` | 30 | Bipedaler Gehzyklus | Ja |
| `robot_jump` | 40 | Absprung-Impuls | Nein |
| `robot_land` | 20 | Lande-Impact mit Staub | Nein |
| `jump_air_idle` | 30 | Mid-Air-Floating | Ja |
| `boost_start` | 10 | Boost-Aktivierungs-Flash | Nein |

Die Animationsclips werden über Three.js's `AnimationMixer` und `AnimationAction` gesteuert, mit `crossFadeTo()` für sanfte Übergänge zwischen Loops und hartem Schnitt für Transformationsanimationen:

```javascript
class TransformerAnimationController {
    constructor(mesh) {
        this.mixer = new THREE.AnimationMixer(mesh);
        this.clips = {};
        this.currentAction = null;
        this.transformationInProgress = false;
    }
    
    loadClips(gltf) {
        gltf.animations.forEach(clip => {
            this.clips[clip.name] = this.mixer.clipAction(clip);
        });
        // Drive-Idle als Standard-Startzustand
        this.playLoop('drive_idle');
    }
    
    playLoop(clipName, fadeTime = 0.2) {
        const newAction = this.clips[clipName];
        if (!newAction || newAction === this.currentAction) return;
        newAction.reset().setLoop(THREE.LoopRepeat, Infinity).play();
        if (this.currentAction) {
            this.currentAction.crossFadeTo(newAction, fadeTime, true);
        }
        this.currentAction = newAction;
    }
    
    async playTransformation(fromMode, toMode) {
        if (this.transformationInProgress) return;
        this.transformationInProgress = true;
        
        // 1. Anticipation (2-4 Frames)
        const anticipClip = this.clips['car_to_transform'];
        anticipClip.reset().setLoop(THREE.LoopOnce, 1).play();
        await this.waitForClip(anticipClip);
        
        // 2. Haupttransformation
        const transformClipName = `transform_${fromMode}_to_${toMode}`;
        const transformClip = this.clips[transformClipName];
        transformClip.reset().setLoop(THREE.LoopOnce, 1).play();
        
        // 3. Collider-Swap bei Halbzeit
        const duration = transformClip.getClip().duration;
        setTimeout(() => {
            this.emit('collider_swap', { fromMode, toMode });
        }, (duration / 2) * 1000);
        
        await this.waitForClip(transformClip);
        
        // 4. In den neuen Modus-Loop übergehen
        this.playLoop(`${toMode}_idle`);
        this.transformationInProgress = false;
        this.emit('transformation_complete', { toMode });
    }
    
    waitForClip(action) {
        return new Promise(resolve => {
            this.mixer.addEventListener('finished', function handler(e) {
                if (e.action === action) {
                    this.removeEventListener('finished', handler);
                    resolve();
                }
            });
        });
    }
    
    update(deltaTime) {
        this.mixer.update(deltaTime);
    }
}
```

#### A.1.3 Visuelle Tricks — Technische Umsetzung

**Partikel-Burst bei Transformations-Peak:**

```javascript
class TransformationParticleEffect {
    constructor(renderer, scene) {
        // GPGPU-Partikel für WebGL-Fallback
        this.gpgpu = new GPUComputationRenderer(64, 64, renderer);
        this.particleCount = 64 * 64; // 4096 Partikel
        
        // Positions- und Geschwindigkeits-Texturen
        this.posTexture = this.gpgpu.createTexture();
        this.velTexture = this.gpgpu.createTexture();
        
        this.initParticles();
    }
    
    trigger(position, intensity = 1.0) {
        // Partikel-Spawn an Transformations-Position
        this.uniforms.uSpawnPosition.value.copy(position);
        this.uniforms.uBurstIntensity.value = intensity;
        this.uniforms.uActive.value = 1;
        
        // Nach 0.5s deaktivieren
        setTimeout(() => {
            this.uniforms.uActive.value = 0;
        }, 500);
    }
}
```

**Screen-Flash bei Transform-Halbzeit:**

```javascript
// In pmndrs/postprocessing als Custom-Effect
class TransformFlashEffect extends Effect {
    constructor() {
        super('TransformFlashEffect', `
            uniform float flashIntensity;
            void mainImage(const in vec4 inputColor, const in vec2 uv, out vec4 outputColor) {
                outputColor = mix(inputColor, vec4(1.0), flashIntensity * 0.8);
            }
        `, {
            uniforms: new Map([['flashIntensity', new THREE.Uniform(0.0)]])
        });
    }
    
    triggerFlash() {
        const start = performance.now();
        const duration = 100; // 100ms = ~6 Frames bei 60fps
        
        const animate = () => {
            const elapsed = performance.now() - start;
            const t = Math.min(elapsed / duration, 1.0);
            // Dreiecksfunktion: Aufbau und Abbau
            const intensity = t < 0.5 ? t * 2 : (1 - t) * 2;
            this.uniforms.get('flashIntensity').value = intensity;
            
            if (elapsed < duration) requestAnimationFrame(animate);
        };
        animate();
    }
}
```

---

### A.2 Vertiefende KI-Implementierung: Multi-Mode Racing AI

#### A.2.1 STK SkiddingAI — Erweiterungs-API

Die SuperTuxKart SkiddingAI-Klasse ([GitHub STK SkiddingAI](https://github.com/supertuxkart/stk-code/blob/master/src/karts/controller/skidding_ai.hpp)) bietet folgende Erweiterungs-Hooks für Mariliktux:

```cpp
// Erweiterte SkiddingAI für Multi-Mode-Unterstützung
class MariliktuxAI : public SkiddingAI {
public:
    enum VehicleMode { DRIVE, FLY, JUMP, MECH };
    
    MariliktuxAI(AbstractKart* kart) : SkiddingAI(kart) {
        m_current_mode = DRIVE;
        m_rubber_band_multiplier = 1.0f;
    }
    
    virtual void update(int ticks) override {
        // 1. Modus-Detection
        detectModeTransition();
        
        // 2. Modus-spezifisches Update
        switch (m_current_mode) {
            case DRIVE:
                updateDriveMode(ticks);
                break;
            case FLY:
                updateFlyMode(ticks);
                break;
            case JUMP:
                updateJumpMode(ticks);
                break;
            case MECH:
                updateMechMode(ticks);
                break;
        }
        
        // 3. Rubber-Banding anwenden
        applyRubberBanding();
    }
    
private:
    void updateDriveMode(int ticks) {
        // Basis-SkiddingAI aufrufen
        SkiddingAI::update(ticks);
        
        // Zusätzliches: Transformations-Trigger erkennen
        if (shouldTransform()) {
            initiateTransformation(FLY);
        }
    }
    
    void updateFlyMode(int ticks) {
        // 3D-Navigation für Flugmodus
        Vec3 target = m_fly_waypoints[m_current_fly_waypoint];
        Vec3 to_target = target - m_kart->getXYZ();
        
        // Pitch und Yaw berechnen
        float pitch = atan2(to_target.getY(), 
                            sqrt(to_target.getX()*to_target.getX() + 
                                 to_target.getZ()*to_target.getZ()));
        float yaw = atan2(to_target.getX(), to_target.getZ());
        
        m_controls->setPitch(pitch * m_skill_level);
        m_controls->setYaw(yaw);
        m_controls->setAccel(calculateFlyThrottle(to_target.length()));
    }
    
    void applyRubberBanding() {
        // GameAIPro Kapitel 42 Implementierung
        float dist = m_distance_to_player; // Negativ = KI hinter Spieler
        
        // Dead Zone: -50 bis +50
        if (dist > -50 && dist < 50) {
            m_rubber_band_multiplier = 1.0f;
            return;
        }
        
        if (dist < -50) {
            // KI ist hinter Spieler: Boost
            float t = std::min(1.0f, (std::abs(dist) - 50) / 100.0f);
            m_rubber_band_multiplier = 1.0f + t * 0.4f; // Max 1.4×
        } else {
            // KI ist vor Spieler: Verlangsamen
            float t = std::min(1.0f, (dist - 50) / 150.0f);
            m_rubber_band_multiplier = 1.0f - t * 0.4f; // Min 0.6×
        }
    }
    
    VehicleMode m_current_mode;
    float m_rubber_band_multiplier;
    std::vector<Vec3> m_fly_waypoints;
    int m_current_fly_waypoint;
    float m_skill_level;
};
```

#### A.2.2 Race Timeline und Choreographer

Der Race Choreographer ermöglicht scripted Events während eines Rennens — ein höheres System über der normalen KI-Schicht:

```javascript
class RaceChoreographer {
    constructor(race) {
        this.race = race;
        this.events = [];
        this.triggered = new Set();
    }
    
    // Wetter-Event nach Lap 2
    addLapEvent(lap, callback) {
        this.events.push({ type: 'lap', lap, callback });
    }
    
    // Zeitbasiertes Event
    addTimeEvent(seconds, callback) {
        this.events.push({ type: 'time', seconds, callback });
    }
    
    // Distanz-basiertes Event (bestimmter Streckenabschnitt)
    addTrackPositionEvent(trackPosition, callback) {
        this.events.push({ type: 'position', trackPosition, callback });
    }
    
    update(raceState) {
        this.events.forEach((event, index) => {
            if (this.triggered.has(index)) return;
            
            let shouldTrigger = false;
            switch (event.type) {
                case 'lap':
                    shouldTrigger = raceState.currentLap >= event.lap;
                    break;
                case 'time':
                    shouldTrigger = raceState.elapsedSeconds >= event.seconds;
                    break;
                case 'position':
                    shouldTrigger = raceState.playerPosition >= event.trackPosition;
                    break;
            }
            
            if (shouldTrigger) {
                this.triggered.add(index);
                event.callback(raceState);
            }
        });
    }
}

// Beispiel-Verwendung für dynamisches Rennen:
const choreographer = new RaceChoreographer(race);

// Regen beginnt nach Lap 1
choreographer.addLapEvent(2, (state) => {
    weatherSystem.transitionTo('light_rain', duration: 30);
    musicSystem.addLayer('tension_rain');
});

// Rivale macht strategischen Fehler auf Lap 3
choreographer.addLapEvent(3, (state) => {
    const rival = aiController.getDriver('rival_01');
    rival.injectMistake({
        type: 'missed_apex',
        corner: 'corner_7',
        severity: 0.6
    });
});

// Finale Lap: Musik-Intensivierung
choreographer.addLapEvent(raceConfig.totalLaps, (state) => {
    musicSystem.setState('final_lap');
    huD.showFinalLapBanner();
});
```

---

### A.3 Vollständige Multiplayer-Netcode-Implementierung

#### A.3.1 WebTransport-Integration für Spielzustand

```javascript
// WebTransport Client
class GameNetworkClient {
    constructor(serverUrl) {
        this.serverUrl = serverUrl;
        this.transport = null;
        this.datagramWriter = null;
        this.datagramReader = null;
        this.tickBuffer = new CircularBuffer(128); // 128 Ticks Puffer
        this.localInputBuffer = new CircularBuffer(128);
        this.serverStateBuffer = new CircularBuffer(64);
    }
    
    async connect() {
        this.transport = new WebTransport(this.serverUrl);
        await this.transport.ready;
        
        // Unreliable Datagramme für Echtzeit-Position
        this.datagramWriter = this.transport.datagrams.writable.getWriter();
        this.startDatagramReader();
        
        // Reliable Stream für Lobby-Events
        this.reliableStream = await this.transport.createBidirectionalStream();
        this.startReliableReader();
        
        console.log('[Network] WebTransport verbunden:', this.serverUrl);
    }
    
    sendInput(input) {
        // Client-Side: Input sofort anwenden (Prediction)
        const tick = this.currentTick++;
        this.localInputBuffer.push({ tick, input, timestamp: performance.now() });
        
        // An Server senden (Datagramm, nicht zuverlässig)
        const packet = this.serializeInput({ tick, input });
        this.datagramWriter.write(packet);
    }
    
    startDatagramReader() {
        const reader = this.transport.datagrams.readable.getReader();
        const readLoop = async () => {
            while (true) {
                const { value, done } = await reader.read();
                if (done) break;
                
                const state = this.deserializeState(value);
                this.serverStateBuffer.push(state);
                this.reconcile(state);
            }
        };
        readLoop();
    }
    
    reconcile(serverState) {
        // Finde den lokal gespeicherten Zustand für diesen Tick
        const localState = this.localInputBuffer.findByTick(serverState.tick);
        if (!localState) return;
        
        const error = Vec3.distance(localState.position, serverState.position);
        const RECONCILIATION_THRESHOLD = 0.5; // 0.5m Toleranz
        
        if (error > RECONCILIATION_THRESHOLD) {
            // Zurückspulen und Inputs wiederholen
            console.log(`[Reconcile] Fehler ${error.toFixed(2)}m, Tick ${serverState.tick}`);
            this.rewindAndReplay(serverState);
        }
    }
    
    rewindAndReplay(serverState) {
        // Server-Zustand übernehmen
        this.kart.setPosition(serverState.position);
        this.kart.setVelocity(serverState.velocity);
        this.kart.setRotation(serverState.rotation);
        
        // Alle Inputs seit diesem Tick wiederholen
        const inputsToReplay = this.localInputBuffer.getAfterTick(serverState.tick);
        inputsToReplay.forEach(inputRecord => {
            this.physicsEngine.stepWithInput(inputRecord.input, 1/60);
        });
    }
}
```

#### A.3.2 Dead Reckoning für Remote-Karts

```javascript
class RemoteKartDeadReckoning {
    constructor() {
        this.lastKnownPosition = new THREE.Vector3();
        this.lastKnownVelocity = new THREE.Vector3();
        this.lastKnownAngularVelocity = new THREE.Euler();
        this.lastUpdateTime = 0;
        this.predictionBuffer = [];
    }
    
    onServerUpdate(networkState) {
        const now = performance.now() / 1000;
        
        // Neuen Server-Zustand einsetzen
        this.lastKnownPosition.copy(networkState.position);
        this.lastKnownVelocity.copy(networkState.velocity);
        this.lastKnownAngularVelocity.copy(networkState.angularVelocity);
        this.lastUpdateTime = now;
    }
    
    predict(currentTime) {
        const dt = currentTime - this.lastUpdateTime;
        
        // Klassisches Dead Reckoning: konstante Geschwindigkeit
        const predicted = new THREE.Vector3();
        predicted.copy(this.lastKnownPosition);
        predicted.addScaledVector(this.lastKnownVelocity, dt);
        
        // Beschleunigung berücksichtigen (Lagrange-Extrapolation wenn verfügbar)
        if (this.lastKnownAcceleration) {
            predicted.addScaledVector(this.lastKnownAcceleration, 0.5 * dt * dt);
        }
        
        return predicted;
    }
    
    // Neural Dead Reckoning (LSTM-basiert, nach University of Waterloo 2023)
    // https://ece.uwaterloo.ca/~sl2smith/papers/2023TOG-Predictive_Dead_Reckoning.pdf
    predictNeural(currentTime) {
        if (!this.lstmModel) return this.predict(currentTime);
        
        // LSTM-Eingabe: letzte 10 bekannte Zustände
        const inputSequence = this.stateHistory.slice(-10).map(s => [
            s.position.x, s.position.z,
            s.velocity.x, s.velocity.z,
            s.heading
        ]);
        
        const prediction = this.lstmModel.predict(inputSequence);
        return new THREE.Vector3(prediction[0], this.lastKnownPosition.y, prediction[1]);
    }
    
    // Sanfte Interpolation zwischen Prediction und neuer Server-Authorität
    smoothedPosition(predictedPos, serverPos, blendFactor) {
        return new THREE.Vector3().lerpVectors(predictedPos, serverPos, blendFactor);
    }
}
```

---

### A.4 Google Blockly — Vollständige Block-Definitions

#### A.4.1 Custom Block-Definitionen für Mariliktux

```javascript
// Event-Blöcke (Hut-Blöcke wie in Scratch)
Blockly.defineBlocksWithJsonArray([
    {
        "type": "event_on_lap_start",
        "message0": "Wenn Runde %1 beginnt",
        "args0": [{ "type": "field_number", "name": "LAP_NUM", "value": 1, "min": 1, "max": 99 }],
        "nextStatement": null,
        "colour": 210,
        "tooltip": "Wird am Beginn einer bestimmten Runde ausgeführt",
        "helpUrl": ""
    },
    {
        "type": "event_on_transformation",
        "message0": "Wenn Fahrzeug sich in %1 transformiert",
        "args0": [{
            "type": "field_dropdown",
            "name": "MODE",
            "options": [
                ["Auto-Modus", "DRIVE"],
                ["Flug-Modus", "FLY"],
                ["Sprung-Modus", "JUMP"],
                ["Mech-Modus", "MECH"]
            ]
        }],
        "nextStatement": null,
        "colour": 120,
        "tooltip": "Wird ausgeführt wenn eine Transformation abgeschlossen ist"
    },
    {
        "type": "event_on_item_collected",
        "message0": "Wenn Item %1 eingesammelt wird",
        "args0": [{
            "type": "field_dropdown",
            "name": "ITEM_TYPE",
            "options": [
                ["Nitro (groß)", "NITRO_BIG"],
                ["Nitro (klein)", "NITRO_SMALL"],
                ["Item-Box", "ITEM_BOX"],
                ["Beliebig", "ANY"]
            ]
        }],
        "nextStatement": null,
        "colour": 210
    },
    {
        "type": "action_set_kart_color",
        "message0": "Kart-Farbe auf %1 setzen",
        "args0": [{ "type": "field_colour", "name": "COLOR", "colour": "#ff0000" }],
        "previousStatement": null,
        "nextStatement": null,
        "colour": 65,
        "tooltip": "Ändert die Kart-Farbe (nur kosmetisch)"
    },
    {
        "type": "action_spawn_particle",
        "message0": "Partikel-Effekt %1 erzeugen",
        "args0": [{
            "type": "field_dropdown",
            "name": "EFFECT",
            "options": [
                ["Funken", "sparks"],
                ["Rauch", "smoke"],
                ["Flammen", "flames"],
                ["Sternenstaub", "stardust"],
                ["Explosion (klein)", "explosion_small"]
            ]
        }],
        "previousStatement": null,
        "nextStatement": null,
        "colour": 65
    },
    {
        "type": "condition_current_mode",
        "message0": "Aktueller Modus ist %1",
        "args0": [{
            "type": "field_dropdown",
            "name": "MODE",
            "options": [
                ["Auto", "DRIVE"],
                ["Flug", "FLY"],
                ["Sprung", "JUMP"],
                ["Mech", "MECH"]
            ]
        }],
        "output": "Boolean",
        "colour": 210
    },
    {
        "type": "condition_player_position",
        "message0": "Spieler ist auf Platz %1 %2",
        "args0": [
            {
                "type": "field_dropdown",
                "name": "OPERATOR",
                "options": [["≤", "LTE"], ["=", "EQ"], ["≥", "GTE"]]
            },
            { "type": "field_number", "name": "POSITION", "value": 3, "min": 1, "max": 12 }
        ],
        "output": "Boolean",
        "colour": 210
    }
]);
```

#### A.4.2 Sicherheits-Validierung für Blockly-Code

```javascript
class BlocklySecurityValidator {
    constructor() {
        // Maximale Werte für alle numerischen Inputs
        this.limits = {
            speed_multiplier: { min: 0.1, max: 5.0 },
            particle_count: { min: 0, max: 500 },
            message_duration: { min: 0.5, max: 10.0 },
            loop_iterations: { min: 0, max: 100 }
        };
        
        // Whitelist erlaubter API-Aufrufe
        this.allowedAPICalls = new Set([
            'ModAPI.setKartColor',
            'ModAPI.spawnParticle',
            'ModAPI.showMessage',
            'ModAPI.playSound',
            'ModAPI.getPlayerSpeed',
            'ModAPI.getCurrentLap',
            'ModAPI.getCurrentMode',
            'ModAPI.getPlayerPosition'
        ]);
    }
    
    validateGeneratedCode(code) {
        const errors = [];
        
        // Prüfe auf verbotene Muster
        const forbiddenPatterns = [
            /\bfetch\b/,         // Netzwerk-Zugriff
            /\bxmlhttprequest\b/i, // Ajax
            /\blocalStorage\b/,  // Storage
            /\bsessionStorage\b/,
            /\bIndexedDB\b/,
            /\bdocument\b/,      // DOM-Manipulation
            /\bwindow\b/,        // Globales Window
            /\beval\b/,          // Eval
            /\bFunction\s*\(/,   // Function-Constructor
            /while\s*\(\s*true\s*\)/, // Infinite Loop
            /for\s*\(\s*;\s*;\s*\)/   // Infinite Loop
        ];
        
        forbiddenPatterns.forEach(pattern => {
            if (pattern.test(code)) {
                errors.push(`Verbotenes Muster gefunden: ${pattern.source}`);
            }
        });
        
        // Prüfe numerische Grenzen
        const numberMatches = code.matchAll(/setSpeedMultiplier\(([0-9.]+)\)/g);
        for (const match of numberMatches) {
            const value = parseFloat(match[1]);
            if (value > this.limits.speed_multiplier.max || value < this.limits.speed_multiplier.min) {
                errors.push(`Geschwindigkeits-Multiplier ${value} außerhalb erlaubtem Bereich`);
            }
        }
        
        return errors;
    }
    
    sanitizeCode(code) {
        // Sichere Wrapper-Funktion erzeugen
        return `
            (function(ModAPI) {
                'use strict';
                const __apiProxy = new Proxy(ModAPI, {
                    get(target, prop) {
                        if (!allowedMethods.has('ModAPI.' + prop)) {
                            throw new Error('API-Methode nicht erlaubt: ' + prop);
                        }
                        return target[prop].bind(target);
                    }
                });
                ${code}
            })(ModAPIInstance);
        `;
    }
}
```

---

### A.5 Vollständiges Audio-Architektur-Diagramm

#### A.5.1 Audio-System-Übersicht

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                         MARILIKTUX AUDIO SYSTEM                              │
│                                                                               │
│  ┌─────────────────────────────────────────────────────────────────────────┐ │
│  │                    AUDIO SOURCE LAYER                                     │ │
│  │                                                                           │ │
│  │  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐ │ │
│  │  │ Dynamische   │  │ Online-Radio │  │ Self-Radio   │  │ Spotify      │ │ │
│  │  │ Rennmusik    │  │ (Icecast/    │  │ (Lokale MP3s)│  │ Personal     │ │ │
│  │  │ (FMOD Stems) │  │  SHOUTcast)  │  │              │  │ (Premium)    │ │ │
│  │  │              │  │              │  │              │  │              │ │ │
│  │  │ 7 Race-States│  │ HTTP-Stream  │  │ File-Reader  │  │ Web Playback │ │ │
│  │  │ Adaptive Mix │  │ Ring-Buffer  │  │ Shuffle/Queue│  │ SDK          │ │ │
│  │  └──────┬───────┘  └──────┬───────┘  └──────┬───────┘  └──────┬───────┘ │ │
│  │         └─────────────────┴──────────────────┴──────────────────┘         │ │
│  │                                    │                                       │ │
│  └────────────────────────────────────┼───────────────────────────────────────┘ │
│                                       ▼                                       │
│  ┌─────────────────────────────────────────────────────────────────────────┐ │
│  │                    AUDIO MIXING LAYER (WebAudio API)                      │ │
│  │                                                                           │ │
│  │  GainNode(Musik) ──┐                                                      │ │
│  │  GainNode(SFX) ────┼──► Master GainNode ──► AudioContext.destination      │ │
│  │  GainNode(Voice) ──┘                                                      │ │
│  │                                                                           │ │
│  │  Per-Channel-Lautstärke-Slider (Settings)                                 │ │
│  └─────────────────────────────────────────────────────────────────────────┘ │
│                                                                               │
│  ┌─────────────────────────────────────────────────────────────────────────┐ │
│  │                    SFX SOUND BANK                                         │ │
│  │                                                                           │ │
│  │  Motor-Sounds (pro Fahrzeug, RPM-gesteuert)                               │ │
│  │  Reifen-Quietschen (pro Modus, Slip-Angle-gesteuert)                      │ │
│  │  Transformations-Geräusche (pro Transformation, 7 Varianten)              │ │
│  │  Impact-Geräusche (3 Intensitätsstufen)                                  │ │
│  │  Item-Sounds (Pickup, Activate, Explode pro Item-Typ)                     │ │
│  │  UI-Sounds (Menü-Navigation, Achievement-Unlock, Level-Up)                │ │
│  │  Umgebungs-Sounds (Zuschauer, Wind, Wetter — distanzbasiert)              │ │
│  └─────────────────────────────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────────────────────────┘
```

#### A.5.2 FMOD-Integrations-Konzept für Dynamische Rennmusik

Die Stem-basierte Musikarchitektur für FMOD erfordert folgende Layer-Struktur:

```
Renn-Musik-Event: "race_music"
├── Layer 0: Ambient Drone (immer aktiv, Lautstärke = 0.3)
│   └── Dateien: ambient_drone_01.wav, ambient_drone_02.wav (zufällig)
│
├── Layer 1: Rhythmus-Percussion (aktiv wenn Speed > 50 km/h)
│   └── RTPC: "speed" → Volume 0.0 bis 1.0 (linear)
│   └── Dateien: drums_beat_01.wav (120 BPM, Loop)
│
├── Layer 2: Melodie (aktiv in Führungsposition)
│   └── Game State: "position" == "leading" → Volume 1.0, else 0.0
│   └── Dateien: melody_triumphant.wav
│
├── Layer 3: Spannungs-Strings (aktiv beim Aufholen)
│   └── RTPC: "tension" → Volume 0.0 bis 0.8
│   └── Dateien: strings_tension_01.wav
│
├── Layer 4: Final-Lap-Intensivierung (aktiv auf letzter Runde)
│   └── Game State: "final_lap" == true → alle Layer +20% Tempo, neue Melodie
│
└── Stingers (One-Shot Overlays):
    ├── "boost_activated" → percussion_hit_01.wav
    ├── "item_collected" → sparkle_sting_01.wav
    ├── "transformation_complete" → transform_sting_01.wav
    └── "lap_complete" → lap_fanfare_01.wav
```

---

### A.6 Reward-System — Detaillierte Datenstruktur

#### A.6.1 Datenbank-Schema für Progression

```sql
-- Globales Spieler-Level
CREATE TABLE player_profiles (
    player_id       UUID PRIMARY KEY,
    username        VARCHAR(50) NOT NULL,
    global_level    INT DEFAULT 1,
    global_xp       BIGINT DEFAULT 0,
    race_credits    INT DEFAULT 500,
    form_tokens     INT DEFAULT 0,
    irating         FLOAT DEFAULT 1500.0,
    safety_rating   FLOAT DEFAULT 2.50,
    created_at      TIMESTAMP DEFAULT NOW(),
    last_active     TIMESTAMP DEFAULT NOW()
);

-- Fahrzeug-Mastery pro Spieler
CREATE TABLE vehicle_mastery (
    player_id       UUID REFERENCES player_profiles(player_id),
    vehicle_id      VARCHAR(50) NOT NULL,
    mode            VARCHAR(10) NOT NULL CHECK (mode IN ('drive', 'fly', 'jump', 'mech')),
    mastery_level   INT DEFAULT 0,
    mastery_xp      INT DEFAULT 0,
    races_in_mode   INT DEFAULT 0,
    PRIMARY KEY (player_id, vehicle_id, mode)
);

-- Achievement-Tracking
CREATE TABLE achievements (
    player_id       UUID REFERENCES player_profiles(player_id),
    achievement_id  VARCHAR(100) NOT NULL,
    unlocked_at     TIMESTAMP,
    progress        INT DEFAULT 0,
    PRIMARY KEY (player_id, achievement_id)
);

-- Battle Pass Fortschritt
CREATE TABLE battle_pass_progress (
    player_id       UUID REFERENCES player_profiles(player_id),
    season_id       INT NOT NULL,
    tier            INT DEFAULT 0,
    pass_type       VARCHAR(10) DEFAULT 'free' CHECK (pass_type IN ('free', 'premium')),
    xp_earned       INT DEFAULT 0,
    PRIMARY KEY (player_id, season_id)
);

-- Rennergebnisse für iRating-Berechnung
CREATE TABLE race_results (
    result_id       UUID PRIMARY KEY,
    race_id         UUID NOT NULL,
    player_id       UUID REFERENCES player_profiles(player_id),
    finish_position INT NOT NULL,
    total_players   INT NOT NULL,
    irating_change  FLOAT DEFAULT 0,
    sr_change       FLOAT DEFAULT 0,
    race_credits_earned INT DEFAULT 0,
    xp_earned       INT DEFAULT 0,
    completed_at    TIMESTAMP DEFAULT NOW()
);

-- Indizes für Performance
CREATE INDEX idx_race_results_player ON race_results(player_id);
CREATE INDEX idx_vehicle_mastery_player ON vehicle_mastery(player_id);
CREATE INDEX idx_achievements_player ON achievements(player_id);
```

#### A.6.2 XP-Berechnungs-API

```javascript
class XPCalculator {
    // Basis-XP-Quellen und Multiplikatoren
    static SOURCES = {
        FINISH_POSITION: { base: 150, scaling: 'linear', min_factor: 0.5 },
        CLEAN_RACE:      { bonus: 100, requires: 'zero_incidents' },
        TRANSFORMATION:  { bonus: 20, per: 'successful_transform_boost' },
        MODE_MASTERY:    { bonus: 50, per: 'mode_specific_achievement' },
        DIFFICULTY:      { easy: 1.0, medium: 1.2, hard: 1.5 },
        NIGHT_RACE:      { bonus: 0.15 },  // +15%
        RAIN_RACE:       { bonus: 0.20 },  // +20%
        ADVANCED_AIDS_OFF: { bonus: 0.25 } // +25%
    };
    
    static calculate(raceResult) {
        let xp = 0;
        const n = raceResult.totalPlayers;
        const pos = raceResult.finishPosition;
        
        // 1. Positions-XP
        const positionFactor = 1.5 - ((pos - 1) / (n - 1));
        xp += this.SOURCES.FINISH_POSITION.base * positionFactor;
        
        // 2. Clean-Race-Bonus
        if (raceResult.incidents === 0) {
            xp += this.SOURCES.CLEAN_RACE.bonus;
        }
        
        // 3. Transform-Boost-Bonuses
        xp += raceResult.transformBoosts * this.SOURCES.TRANSFORMATION.bonus;
        
        // 4. Schwierigkeits-Multiplier
        xp *= this.SOURCES.DIFFICULTY[raceResult.difficulty];
        
        // 5. Wetter/Nacht-Bonus
        if (raceResult.conditions.night) xp *= (1 + this.SOURCES.NIGHT_RACE.bonus);
        if (raceResult.conditions.rain) xp *= (1 + this.SOURCES.RAIN_RACE.bonus);
        
        // 6. Assists-Off-Bonus
        if (!raceResult.assistsEnabled) xp *= (1 + this.SOURCES.ADVANCED_AIDS_OFF.bonus);
        
        return Math.round(xp);
    }
    
    // iRating-Anpassung nach ELO-Prinzip (wie iRacing)
    static calculateIRatingChange(playerIR, fieldIRatings, finishPosition) {
        const n = fieldIRatings.length;
        
        // Erwartetes Finish berechnen
        let expectedFinish = 0;
        fieldIRatings.forEach((opponentIR, i) => {
            if (i === finishPosition - 1) return; // Eigene Position überspringen
            // Wahrscheinlichkeit, besser als Gegner abzuschneiden
            const prob = 1 / (1 + Math.exp((opponentIR - playerIR) / 400));
            expectedFinish += prob;
        });
        expectedFinish = expectedFinish / (n - 1); // Normalisieren
        
        // Tatsächliches Finish (normalisiert: 1.0 = 1., 0.0 = letzter)
        const actualFinish = (n - finishPosition) / (n - 1);
        
        const K = 32; // Anpassungsfaktor
        return Math.round(K * (actualFinish - expectedFinish));
    }
}
```

---

### A.7 Level Design Studio — Technische Architektur

#### A.7.1 Node-Editor Datenstruktur

```javascript
// Track-Node Datenmodell
class TrackNode {
    constructor(id) {
        this.id = id;
        this.position = new THREE.Vector3(0, 0, 0);
        this.rotation = new THREE.Euler(0, 0, 0);
        this.bankingAngle = 0;          // Grad, -45 bis +45
        this.width = 12;                // Blender-Units (Standard: 12)
        this.height = 2;                // Höhe der Strecken-Begrenzung
        this.surfaceType = 'asphalt';   // asphalt, grass, dirt, ice, water, air
        this.modeFlags = {
            drive: true,
            fly: false,
            jump: false,
            mech: false
        };
        this.triggers = [];             // Visual-Scripting Trigger-Nodes
        this.nextNode = null;           // ID des nächsten Nodes
        this.prevNode = null;           // ID des vorherigen Nodes
        this.isJunction = false;        // Verzweigungspunkt
        this.alternativeNext = null;    // Alternativer Pfad bei Verzweigung
    }
    
    // Catmull-Rom-Interpolation zwischen diesem und dem nächsten Node
    interpolate(nextNode, t) {
        // t = 0.0 bis 1.0
        const p0 = this.prevNode?.position || this.position;
        const p1 = this.position;
        const p2 = nextNode.position;
        const p3 = nextNode.nextNode?.position || nextNode.position;
        
        return this.catmullRom(p0, p1, p2, p3, t);
    }
    
    catmullRom(p0, p1, p2, p3, t) {
        const t2 = t * t;
        const t3 = t2 * t;
        
        return new THREE.Vector3(
            0.5 * ((2*p1.x) + (-p0.x + p2.x)*t + 
                   (2*p0.x - 5*p1.x + 4*p2.x - p3.x)*t2 + 
                   (-p0.x + 3*p1.x - 3*p2.x + p3.x)*t3),
            // Y und Z analog...
        );
    }
}

// Track-Graph mit allen Nodes
class TrackGraph {
    constructor() {
        this.nodes = new Map();
        this.startNodeId = null;
        this.finishNodeId = null;
    }
    
    addNode(node) {
        this.nodes.set(node.id, node);
    }
    
    connectNodes(fromId, toId) {
        const from = this.nodes.get(fromId);
        const to = this.nodes.get(toId);
        if (!from || !to) return false;
        
        from.nextNode = toId;
        to.prevNode = fromId;
        return true;
    }
    
    // Validierung: Strecke muss geschlossen, kein Selbst-Überschnitt
    validate() {
        const errors = [];
        
        if (!this.startNodeId) errors.push('Kein Start-Node definiert');
        if (!this.finishNodeId) errors.push('Kein Finish-Node definiert');
        
        // Prüfe ob Strecke einen geschlossenen Loop bildet
        if (!this.isClosedLoop()) {
            errors.push('Strecke ist kein geschlossener Loop');
        }
        
        // Prüfe auf Luftabschnitte ohne Landing-Zone
        this.nodes.forEach(node => {
            if (node.modeFlags.fly && !this.hasLandingZoneAfter(node)) {
                errors.push(`Node ${node.id}: Flugabschnitt ohne Landezone`);
            }
        });
        
        return errors;
    }
    
    isClosedLoop() {
        if (!this.startNodeId) return false;
        
        let current = this.startNodeId;
        const visited = new Set();
        let steps = 0;
        
        while (current && !visited.has(current) && steps < this.nodes.size * 2) {
            visited.add(current);
            const node = this.nodes.get(current);
            current = node?.nextNode;
            steps++;
        }
        
        return current === this.startNodeId;
    }
}
```

---

### A.8 Grafik-Engine — Vollständige Rendering-Pipeline

#### A.8.1 Scene-Graph-Optimierung

```javascript
class MariliktuxRenderer {
    constructor() {
        this.renderer = new THREE.WebGPURenderer({
            antialias: false // AA über Post-Processing
        });
        await this.renderer.init();
        
        this.scene = new THREE.Scene();
        this.camera = new THREE.PerspectiveCamera(60, window.innerWidth/window.innerHeight, 0.1, 2000);
        
        // Draw-Call-Tracking
        this.stats = new StatsGL({ precision: 'highp' });
        
        // Instanced Meshes für wiederholte Geometrie
        this.instancedObjects = {
            trees: null,          // InstancedMesh
            barriers: null,       // InstancedMesh
            spectators: null,     // InstancedMesh
            boostPads: null,      // InstancedMesh
            nitroItems: null      // InstancedMesh
        };
        
        // LOD-System für Karts
        this.kartLODs = new Map(); // player_id → LOD
        
        // Post-Processing
        this.composer = this.setupPostProcessing();
        
        // Partikel-System
        this.particles = new WebGPUParticleSystem(this.renderer, 500000);
    }
    
    setupPostProcessing() {
        const composer = new EffectComposer(this.renderer);
        composer.addPass(new RenderPass(this.scene, this.camera));
        
        // Alle Effekte in EINEM Pass kombinieren
        const bloomEffect = new BloomEffect({
            intensity: 1.5,
            threshold: 0.6,
            radius: 0.4
        });
        const fxaaEffect = new FXAAEffect();
        const colorGradingEffect = new LUTEffect(this.loadLUT('default'));
        
        composer.addPass(new EffectPass(
            this.camera,
            bloomEffect,
            fxaaEffect,
            colorGradingEffect
        ));
        
        return composer;
    }
    
    buildTrackGeometry(trackMesh) {
        // Strecken-Segmente zusammenführen für single draw call
        const segments = trackMesh.children.filter(c => c.isMesh);
        const merged = mergeGeometries(segments.map(s => s.geometry));
        
        const trackMaterial = new THREE.MeshToonMaterial({
            color: 0x333333,
            gradientMap: this.toneTexture
        });
        
        const mergedMesh = new THREE.Mesh(merged, trackMaterial);
        this.scene.add(mergedMesh);
        return mergedMesh; // 1 Draw-Call für gesamte Strecke
    }
    
    addKart(kartData) {
        const lod = new THREE.LOD();
        
        // LOD0: Vollständiges transformierbares Kart
        const highDetail = this.createFullDetailKart(kartData);
        lod.addLevel(highDetail, 0);
        
        // LOD1: Vereinfachtes Kart (keine Morph Targets)
        const medDetail = this.createMedDetailKart(kartData);
        lod.addLevel(medDetail, 50);
        
        // LOD2: Sehr einfaches Kart
        const lowDetail = this.createLowDetailKart(kartData);
        lod.addLevel(lowDetail, 150);
        
        // LOD3: Billboard
        const billboard = this.createBillboardSprite(kartData.color);
        lod.addLevel(billboard, 300);
        
        this.scene.add(lod);
        this.kartLODs.set(kartData.playerId, lod);
        return lod;
    }
    
    render(deltaTime) {
        // LODs aktualisieren
        this.kartLODs.forEach(lod => lod.update(this.camera));
        
        // Partikel aktualisieren (GPU-compute)
        this.particles.update(deltaTime);
        
        // Render
        this.composer.render(deltaTime);
        
        // Performance-Monitoring
        if (this.renderer.info.render.calls > 150) {
            console.warn('[Renderer] Draw-Calls überschreiten 150:', 
                         this.renderer.info.render.calls);
        }
    }
}
```

#### A.8.2 TSL Toon-Shader für Transformer-Karts

```javascript
import { 
    Fn, vec3, vec4, float, uniform, normalView, positionView,
    directionalLight, smoothstep, dot, mix, step, normalize, reflect,
    max, pow, clamp
} from 'three/tsl';

// Vollständiger Toon-Shader mit allen 5 Komponenten
const createToonMaterial = (baseColor, rimColor = '#ffffff') => {
    const material = new THREE.NodeMaterial();
    
    material.colorNode = Fn(() => {
        const N = normalize(normalView);
        const V = normalize(positionView.negate());
        const L = normalize(directionalLight.direction);
        
        // 1. Diffuse Lichtintensität
        const lightDot = max(dot(N, L), float(0.0));
        
        // 2. Toon-Bänder (3 Stufen)
        const bands = float(3.0);
        const toonStep = smoothstep(
            float(0.0), float(0.01),
            lightDot.sub(floor(lightDot.mul(bands)).div(bands))
        );
        const toonLight = floor(lightDot.mul(bands)).add(toonStep).div(bands);
        
        // 3. Spekular (Blinn-Phong + Toon)
        const H = normalize(L.add(V));
        const specDot = max(dot(N, H), float(0.0));
        const specular = smoothstep(float(0.95), float(0.97), pow(specDot, float(32.0)));
        
        // 4. Rim-Light
        const rimDot = float(1.0).sub(max(dot(N, V), float(0.0)));
        const rim = smoothstep(float(0.6), float(0.8), rimDot);
        
        // 5. Finale Farbmischung
        const baseC = vec3(baseColor[0], baseColor[1], baseColor[2]);
        const shadowC = baseC.mul(float(0.3)); // Dunkler Schatten
        const specC = vec3(1.0, 1.0, 1.0);
        const rimC = vec3(rimColor[0], rimColor[1], rimColor[2]);
        
        let finalColor = mix(shadowC, baseC, toonLight);
        finalColor = mix(finalColor, specC, specular.mul(float(0.8)));
        finalColor = mix(finalColor, rimC, rim.mul(float(0.4)));
        
        return vec4(finalColor, float(1.0));
    })();
    
    return material;
};

// Transformation-Highlight-Shader (leuchtet während der Transformation)
const createTransformGlowMaterial = () => {
    const material = new THREE.NodeMaterial();
    const uTransformProgress = uniform(0.0);  // 0 = idle, 1 = transform peak
    
    material.colorNode = Fn(() => {
        const glow = uTransformProgress.mul(float(2.0));
        const energyColor = vec3(float(0.2).add(glow), float(0.6), float(1.0));
        return vec4(energyColor, clamp(glow, float(0.0), float(1.0)));
    })();
    
    material.blending = THREE.AdditiveBlending;
    material.transparent = true;
    
    return { material, uTransformProgress };
};
```

---

## Anhang B: Referenz-Implementierungshinweise und Code-Fragmente

### B.1 STK AngelScript-Erweiterungen für Mariliktux-Trigger

Das AngelScript-System in SuperTuxKart ([STK Scripting Docs](https://supertuxkart.net/Scripting)) ermöglicht native Interaktivität in Strecken. Für Mariliktux werden folgende Erweiterungen benötigt:

```angelscript
// Transformation-Trigger für STK-Strecke
// In: tracks/mariliktux_track1/scripting.as

// Globale Variablen
bool g_airSectionActive = false;
float g_modeChangeTimer = 0.0f;

// Initialisierung beim Strecken-Load
void onStart() {
    // Trigger für Luft-Sektion registrieren (Radius 5 Units)
    Utils::createTrigger("air_launch_trigger", 
                          vector3(120.0f, 0.0f, -30.0f), 
                          5.0f);
    
    // Landing-Zone-Trigger
    Utils::createTrigger("air_landing_trigger",
                          vector3(200.0f, 0.0f, 15.0f),
                          8.0f);
    
    // Mech-Zone-Trigger
    Utils::createTrigger("mech_zone_trigger",
                          vector3(340.0f, 0.0f, 0.0f),
                          10.0f);
}

// Update-Loop (jedes Frame)
void onUpdate(float dt) {
    if (g_modeChangeTimer > 0.0f) {
        g_modeChangeTimer -= dt;
    }
}

// Spieler betritt Luft-Launch-Trigger
void onTrigger(const string &in triggerID, int kartID, bool enter) {
    if (!enter) return;
    
    if (triggerID == "air_launch_trigger" && g_modeChangeTimer <= 0.0f) {
        // Transformation zu Flug-Modus auslösen
        Kart::triggerModeChange(kartID, "FLY");
        // Sound + Partikel aktivieren
        Utils::playSound("transform_air_launch", kartID);
        Utils::spawnParticles("launch_smoke", Kart::getPos(kartID));
        
        g_modeChangeTimer = 0.5f; // Cooldown
    }
    
    if (triggerID == "air_landing_trigger") {
        // Transformation zurück zu Fahr-Modus
        Kart::triggerModeChange(kartID, "DRIVE");
        Utils::playSound("transform_land", kartID);
    }
    
    if (triggerID == "mech_zone_trigger") {
        // Mech-Modus für Kampf-Sektion
        Kart::triggerModeChange(kartID, "MECH");
    }
}

// Kart-Kart-Kollision (für Mech-Modus Combat-Events)
void onKartKartCollision(int kartID1, int kartID2) {
    // Prüfen ob beide im Mech-Modus
    if (Kart::getMode(kartID1) == "MECH" && Kart::getMode(kartID2) == "MECH") {
        // Impact-Effekte
        Utils::spawnParticles("mech_impact", Kart::getPos(kartID1));
        Utils::cameraShake(kartID1, 0.3f);
        Utils::cameraShake(kartID2, 0.3f);
    }
}
```

### B.2 DSGVO-konforme Voice-Chat Daten-Verarbeitung

```javascript
// Datenschutzkonforme Sprachdaten-Verwaltung
class VoiceChatPrivacyManager {
    constructor() {
        this.consentGiven = false;
        this.recordingBuffer = null;  // Nur wenn Consent gegeben
        this.retentionDays = 30;
    }
    
    // Explizite Zustimmung einholen (DSGVO Art. 7)
    async requestConsent() {
        const consent = await showConsentDialog({
            title: 'Sprachchat-Datenschutz',
            text: `Für den Sprachchat werden Audiodaten verarbeitet. 
                   Bei gemeldeten Missbrauchsfällen wird ein 30-Sekunden-Ausschnitt 
                   für bis zu 30 Tage gespeichert. 
                   Du kannst deine Daten jederzeit löschen lassen.`,
            acceptText: 'Sprachchat aktivieren',
            denyText: 'Nur Text-Chat nutzen'
        });
        
        this.consentGiven = consent;
        return consent;
    }
    
    // Report mit verschlüsseltem Audio-Clip
    async reportUser(reportedUserId) {
        if (!this.consentGiven) return;
        
        // Letzten 30 Sekunden Audio aus Ring-Buffer extrahieren
        const audioClip = this.recordingBuffer.getLast(30);
        
        // Audio mit Server-Public-Key verschlüsseln (nur Moderatoren können entschlüsseln)
        const encryptedClip = await crypto.subtle.encrypt(
            { name: 'RSA-OAEP' },
            await this.getServerPublicKey(),
            audioClip
        );
        
        // Report senden
        await fetch('/api/voice-report', {
            method: 'POST',
            body: JSON.stringify({
                reportedUserId,
                audioClip: encryptedClip,
                timestamp: Date.now(),
                retentionExpiry: Date.now() + (this.retentionDays * 86400000)
            })
        });
    }
    
    // Löschanfrage (DSGVO Art. 17 — Recht auf Löschung)
    async requestDataDeletion(userId) {
        await fetch(`/api/voice-data/${userId}`, {
            method: 'DELETE',
            headers: { 'Authorization': `Bearer ${this.userToken}` }
        });
        console.log('[Privacy] Sprachdaten erfolgreich gelöscht für:', userId);
    }
}
```

---

## Anhang C: Glossar der verwendeten Fachbegriffe

| Begriff | Definition | Kontext |
|---------|------------|---------|
| **Design Science Research (DSR)** | Forschungsparadigma, das IT-Artefakte schafft um Probleme zu lösen | Hevner et al. 2004 |
| **Artefakt** | Lauffähiges System, Modell, Methode oder Konstrukt | DSR-Terminologie |
| **Relevance Cycle** | Verbindung zwischen Forschung und Anwendungsdomäne | Hevner 2007 |
| **Design Cycle** | Innerster Build-Evaluate-Loop | Hevner 2007 |
| **Rigor Cycle** | Verbindung zwischen Forschung und Wissensbasis | Hevner 2007 |
| **Rubber-Banding** | KI-Speed-Manipulation basierend auf Distanz zum Spieler | GameAIPro Ch. 42 |
| **FSM** | Finite State Machine — deterministischer Zustandsautomat | Game Programming Patterns |
| **Dead Reckoning** | Positions-Extrapolation aus letztem bekannten Zustand | Multiplayer-Netcode |
| **Client-Side Prediction** | Lokale Anwendung von Inputs vor Server-Bestätigung | Multiplayer-Netcode |
| **Reconciliation** | Anpassung des Client-Zustands an Server-Autorität | Multiplayer-Netcode |
| **SFU** | Selective Forwarding Unit — Server-seitige Weiterleitung von Audio/Video | WebRTC-Architektur |
| **DTLS-SRTP** | Verschlüsselungsprotokoll für WebRTC-Audio | Sicherheit |
| **E2EE** | End-to-End-Encryption — Verschlüsselung ohne Server-Entschlüsselung | Datenschutz |
| **Insertable Streams** | WebRTC-API für Frame-Level-Verschlüsselung | E2EE-Implementierung |
| **Morph Target** | Per-Vertex-Delta-Set für geometrische Interpolation | 3D-Animation |
| **Skinning** | Bindung eines Meshes an ein Skeleton-System | Skelettale Animation |
| **LOD** | Level of Detail — distanzbasierte Modell-Komplexitäts-Anpassung | Rendering-Optimierung |
| **Draw Call** | Ein Render-Befehl an die GPU | Rendering-Performance |
| **GPGPU** | General Purpose GPU — Compute-Shader für Nicht-Rendering-Aufgaben | Partikel-Systeme |
| **TSL** | Three Shader Language — Write-once Shader-System für Three.js | WebGPU/WebGL |
| **iRating** | ELO-ähnliches Skill-Rating-System | Kompetitives Spielen |
| **Safety Rating** | Verhaltens-basiertes Sauberfahren-Rating | Kompetitives Spielen |
| **Battle Pass** | Saisonales Progression-System mit Free und Premium Tracks | Monetarisierung |
| **Form Token** | Spiels-interne Währung für Transformations-spezifische Inhalte | Ökonomie |
| **Blockly AST** | Abstract Syntax Tree aus Blockly-Block-Arrangements | Modding-System |
| **WASM** | WebAssembly — binäres Instruktionsformat für Sandbox-Ausführung | Sicherheit |
| **WasmTime** | WebAssembly-Runtime mit Gas-Budget-Limits | Mod-Sandboxing |
| **mod.io** | Drittanbieter-Plattform für Community-Mod-Sharing | Content-Plattform |
| **Icecast** | Open-Source-Streaming-Server für Online-Radio | Audio-Integration |
| **Opus** | Offener Audio-Codec (RFC 6716) für Voice-Chat | Audio-Codec |
| **HRTF** | Head-Related Transfer Function — 3D-Audio-Verarbeitungsfunktion | Spatial Audio |
| **WebTransport** | QUIC-basiertes Web-Netzwerkprotokoll | Multiplayer |
| **STK** | SuperTuxKart — Open-Source Kart-Racing-Engine | Basis-Engine |
| **G1/Generation 1** | Originale Transformers-Cartoon-Ästhetik (1984) | Design-Referenz |
| **Toon/Cel-Shading** | Cartoon-Rendering-Stil mit diskreten Lichtbändern | Grafik-Stil |
| **PBR** | Physically Based Rendering | Grafik-Alternative |
| **GLTF** | GL Transmission Format — 3D-Asset-Standard | Asset-Pipeline |
| **KTX2** | Basis-Textur-Komprimierungsformat | Asset-Optimierung |
| **Draco** | Geometrie-Komprimierungsalgorithmus von Google | Asset-Optimierung |
| **PCSS** | Percentage-Closer Soft Shadows | Schatten-Rendering |
| **SSAA** | Supersampling Anti-Aliasing | Grafik-Qualität |
| **FXAA** | Fast Approximate Anti-Aliasing (Post-Processing) | Grafik-Qualität |
| **Bloom** | Glüheffekt für helle Bildbereiche | Post-Processing |
| **IK** | Inverse Kinematics — rückwärtiges Gelenkberechnungs-System | Animationssystem |
| **PCG** | Procedural Content Generation | Level-Design |
| **Genetic Algorithm (GA)** | Evolutionärer Optimierungsalgorithmus | PCG-Forschung |
| **Catmull-Rom** | Spline-Interpolationsalgorithmus | Track-Geometrie |
| **Bézier** | Parametrische Kurven nach Pierre Bézier | Track-/Cannon-Pfade |
| **PRO** | Performance Rights Organization (z.B. GEMA, ASCAP) | Musik-Lizenzrecht |
| **DMCA** | Digital Millennium Copyright Act | Rechtlicher Rahmen |
| **DSGVO** | Datenschutz-Grundverordnung | EU-Datenschutzrecht |
| **CCPA** | California Consumer Privacy Act | US-Datenschutzrecht |

---

*Dokumentversion: 1.0 | Erstellt: 18. März 2026 | Methodik: DSR nach Hevner et al. (2004) + Hevner (2007) + Peffers et al. (2007)*
