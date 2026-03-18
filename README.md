# Mariliktux

## SuperTuxKart Transformer Racing Game – DSR Research & Planning

Dieses Repository enthält die Design Science Research (DSR) Dokumentation nach Hevner et al. (2004) für das Projekt **Mariliktux** – ein Transformer-basiertes Rennspiel, aufbauend auf SuperTuxKart.

### Projektübersicht

Mariliktux transformiert SuperTuxKart in ein Transformer-Rennspiel mit:
- **Transformierende Fahrzeuge** – Flug-, Jump- und Drive-Modi
- **Multi-Mode Level Design** – Strecken mit Boden-, Luft- und Sprungpassagen
- **Belohnungs- und Progressionssystem** – XP, Upgrades, Battle Pass
- **Integrierte Renn-AI** – Adaptive KI mit Multi-Mode-Unterstützung
- **Level Design Studio** – Eigene Levels erstellen
- **Scratch-ähnliches Modding** – Low-Code Block-basiertes Modding-System
- **Online Radio & Spotify** – Musik-Integration
- **Sicherer Sprachchat** – WebRTC-basiert mit E2E-Verschlüsselung
- **Single- & Multiplayer** – Lokal und Online
- **Meshy.ai Assets** – KI-generierte 3D-Modelle und Animationen

### Research-Dateien

| Datei | Thema |
|-------|-------|
| `dsr-framework.md` | DSR Framework nach Hevner et al. (2004, 2007) |
| `transformer-animations.md` | Animations- & Erstellungskonzepte für Transformer-Fahrzeuge |
| `level-design.md` | Level Design für Multi-Mode Racing |
| `reward-system.md` | Belohnungs- und Progressionssystem |
| `racing-ai-timeline.md` | Renn-AI und Race Timeline |
| `audio-integration.md` | Online Radio & Spotify Integration |
| `voice-chat.md` | Sicherer Sprachchat |
| `modding-system.md` | Scratch-ähnliches Low-Code Modding |
| `multiplayer-architecture.md` | Multiplayer-Architektur |
| `graphics-performance.md` | Grafik-Performance & Rendering |

### Methodik

Das Projekt folgt dem **Design Science Research (DSR)** Ansatz nach Hevner et al.:
1. **Design as an Artifact** – Das Spiel als IT-Artefakt
2. **Problem Relevance** – Innovative Racing-Game-Mechaniken
3. **Design Evaluation** – Iteratives Testen und Evaluieren
4. **Research Contributions** – Neue Konzepte für Multi-Mode-Racing
5. **Research Rigor** – Fundierte Analyse bestehender Systeme
6. **Design as a Search Process** – Iterative Entwicklung
7. **Communication of Research** – Dokumentation aller Entscheidungen

### Tech-Stack

- **3D Assets**: Meshy.ai (API-basiert)
- **Engine**: Three.js / WebGPU
- **Networking**: WebRTC / WebTransport
- **Audio**: FMOD / Web Audio API
- **Modding**: Google Blockly
- **Voice Chat**: WebRTC + Opus + E2EE

---

*Basiert auf SuperTuxKart (GPL v3) – Transformer-Erweiterung*
