# Design Science Research (DSR) Framework nach Hevner et al. (2004)

## Primärquelle / Primary Reference

> Hevner, A. R., March, S. T., Park, J., & Ram, S. (2004). **Design Science in Information Systems Research.** *MIS Quarterly*, 28(1), 75–105.
> 
> URL (PDF): https://wise.vub.ac.be/sites/default/files/thesis_info/design_science.pdf  
> ACM Digital Library: https://dl.acm.org/doi/abs/10.5555/2017212.2017217

---

## 1. Überblick: Was ist Design Science Research?

Design Science Research (DSR) ist ein Forschungsparadigma in der Informationssystemforschung, das auf der Schaffung und Evaluation von IT-Artefakten basiert. Hevner et al. (2004) definieren zwei komplementäre Paradigmen der IS-Forschung:

| Paradigma | Ziel | Kernfrage |
|-----------|------|-----------|
| **Behavioral Science** | Wahrheit – Theorien entwickeln und validieren, die menschliches/organisationales Verhalten erklären oder vorhersagen | *Warum und wie funktioniert X?* |
| **Design Science** | Nützlichkeit (Utility) – innovative Artefakte schaffen, die die Grenzen menschlicher und organisatorischer Möglichkeiten erweitern | *Wie soll X gebaut/gestaltet werden?* |

> „Design science is fundamentally a problem-solving paradigm. It seeks to create innovations that define the ideas, practices, technical capabilities, and products through which the analysis, design, implementation, management, and use of information systems can be effectively and efficiently accomplished."  
> — Hevner et al. (2004, S. 76)

**Kernprinzip:** Wissen über ein Designproblem und seine Lösung entsteht durch das *Bauen und Anwenden* eines Artefakts.

---

## 2. IT-Artefakttypen nach March & Smith (1995) / Hevner et al. (2004)

Design-Science-Forschung produziert vier Typen von Artefakten:

| Artefakttyp | Definition | Beispiel im Spielekontext |
|-------------|------------|--------------------------|
| **Konstrukte** | Vokabular und Symbole, die das Problemfeld beschreiben | Spielmechanik-Terminologie, Zustandsdefinitionen |
| **Modelle** | Abstraktionen/Repräsentationen, die Beziehungen zeigen | Entity-Relationship-Diagramm, UML-Klassendiagramm |
| **Methoden** | Algorithmen und Vorgehensweisen | Entwicklungsprozess für Transformations-Animationen |
| **Instanziierungen** | Implementierte und prototypische Systeme | Spielprototyp, lauffähige Engine-Demo |

---

## 3. Die 7 DSR-Leitlinien nach Hevner et al. (2004)

Die sieben Richtlinien sind aus dem Grundprinzip abgeleitet, dass **Wissen und Verständnis über ein Designproblem durch das Bauen und Anwenden eines Artefakts erworben werden**.

---

### Leitlinie 1: Design als Artefakt (Design as an Artifact)

> „Design-science research must produce a viable artifact in the form of a construct, a model, a method, or an instantiation."  
> — Hevner et al. (2004, S. 83)

**Kernaussage:** DSR-Forschung muss ein *lauffähiges, nützliches IT-Artefakt* erzeugen – entweder ein Konstrukt, Modell, Methode oder eine Instanziierung. Das Artefakt ist nicht bloß ein Nebenprodukt, sondern der zentrale Forschungsbeitrag.

**Abgrenzung zu Routinedesign:** Routinedesign wendet bekanntes Wissen auf bekannte Probleme an (Best Practices, Templates). DSR hingegen adressiert Probleme, für die noch keine adäquate Lösung existiert – es braucht Kreativität und explorative Suche.

**Im Spielekontext:** Ein Spielprototyp mit einem neuartigen Transformations-Mechanismus wäre ein Artefakt (Instanziierung). Ein Zustandsmaschinen-Framework für Multi-Modus-Fahrzeuge wäre eine Methode.

---

### Leitlinie 2: Problemrelevanz (Problem Relevance)

> „The objective of design-science research is to develop technology-based solutions to important and relevant business problems."  
> — Hevner et al. (2004, S. 83)

**Kernaussage:** Das zu lösende Problem muss wichtig und relevant sein – für Organisationen, Menschen oder die Gesellschaft. Die Problemdomäne definiert den Suchraum für das Artefakt.

**Komponenten der Problemdomäne (Environment):**
- **Menschen:** Rollen, Fähigkeiten, Charakteristika der Nutzer
- **Organisationen:** Strategien, Strukturen, Prozesse
- **Technologie:** Infrastruktur, Anwendungen, Kommunikationsarchitektur

**Im Spielekontext:** Das Problem könnte lauten: „Wie können Transformations-Animationen in einem Rennspiel technisch effizient implementiert werden, ohne die 60-fps-Zielmarke zu unterschreiten?"

---

### Leitlinie 3: Designevaluation (Design Evaluation)

> „The utility, quality, and efficacy of a design artifact must be rigorously demonstrated via well-executed evaluation methods."  
> — Hevner et al. (2004, S. 83)

**Kernaussage:** Das Artefakt muss rigoros evaluiert werden. Hevner et al. listen folgende Evaluationsmethoden auf:

| Evaluationsmethode | Beschreibung |
|--------------------|--------------|
| Analytisch | Statische Analyse (Komplexität, Vollständigkeit, Konsistenz) |
| Case Study | Anwendung auf ein reales Problem in der Umgebung |
| Experimentell | Kontrollierter Vergleich im Labor |
| Feldstudie | Beobachtung im natürlichen Einsatzkontext |
| Simulation | Ausführung der Umgebung und des Artefakts |

**Im Spielekontext:** Evaluation eines Spielprototyps durch Playtest-Sessions, Framezeit-Messungen oder Nutzerinterviews.

---

### Leitlinie 4: Forschungsbeiträge (Research Contributions)

> „Effective design-science research must provide clear and verifiable contributions in the areas of the design artifact, design foundations, and/or design methodologies."  
> — Hevner et al. (2004, S. 83)

**Kernaussage:** DSR muss einen *neuen* Beitrag liefern – entweder durch ein neues Artefakt, neue Designgrundlagen oder neue Designmethoden. Gregor & Hevner (2013) strukturieren Beiträge in drei Ebenen:

| Ebene | Beschreibung | Abstraktionsgrad |
|-------|--------------|-----------------|
| Level 1 | Neue konkrete Artefakte | Niedrig |
| Level 2 | Designprinzipien und -regeln | Mittel |
| Level 3 | Designtheorien | Hoch |

---

### Leitlinie 5: Forschungsrigorosität (Research Rigor)

> „Design-science research relies upon the application of rigorous methods in both the construction and evaluation of the design artifact."  
> — Hevner et al. (2004, S. 83)

**Kernaussage:** Rigorosität bedeutet die fundierte Anwendung geeigneter Theorien und Methoden beim Bau *und* bei der Evaluation. Rigorosität kommt aus der Wissensbasis: existierende Theorien, Frameworks, Instrumente.

> „It is contingent on the researchers to thoroughly research and reference the knowledge base in order to guarantee that the designs produced are research contributions and not routine designs based upon the application of well-known processes."  
> — Hevner (2007, S. 89)

---

### Leitlinie 6: Design als Suchprozess (Design as a Search Process)

> „The search for an effective artifact requires utilizing available means to reach desired ends while satisfying laws in the problem environment."  
> — Hevner et al. (2004, S. 83)

**Kernaussage:** Design ist ein Suchprozess im Problemraum. Wie Simon (1996, *The Sciences of the Artificial*) beschrieb: Ein Problem ist die Differenz zwischen einem Zielzustand und dem aktuellen Zustand. Der Suchprozess nutzt verfügbare Mittel, um den Zielzustand zu erreichen – unter Berücksichtigung der Randbedingungen der Problemumgebung.

**Charakteristika von Designproblemen:**
- Instabile Anforderungen und Randbedingungen
- Komplexe Interaktionen zwischen Teilkomponenten
- Malleabilität (sowohl Prozess als auch Artefakt sind formbar)
- Abhängigkeit von menschlichen kognitiven Fähigkeiten (Kreativität)

---

### Leitlinie 7: Kommunikation der Forschung (Communication of Research)

> „Design-science research must be presented effectively both to technology-oriented as well as management-oriented audiences."  
> — Hevner et al. (2004, S. 83)

**Kernaussage:** Forschungsergebnisse müssen für *zwei* Zielgruppen kommuniziert werden:
1. **Technisches Publikum** (Forscher, die das Artefakt erweitern; Praktiker, die es implementieren)
2. **Managementpublikum** (Forscher, die es im Kontext studieren; Entscheider, die über Implementierung entscheiden)

**Im Spielekontext:** Technische Dokumentation (z. B. API-Referenz, Komponentendiagramme) für Entwickler + High-Level-Präsentation des Spielkonzepts für Publisher/Stakeholder.

---

## 4. Das IS-Forschungsframework von Hevner et al. (2004)

```
┌─────────────────────────────────────────────────────────────────────┐
│                           IS Research                               │
│                                                                     │
│  ┌────────────┐  Relevance   ┌──────────────────────────────────┐  │
│  │Environment │ ──────────►  │        IS Research               │  │
│  │            │             │  ┌──────────────┐                  │  │
│  │ People     │             │  │ Develop/Build │                  │  │
│  │ Orgs       │ ◄──────────  │  │ • Theories   │                  │  │
│  │ Technology │  Apply in   │  │ • Artifacts  │                  │  │
│  └────────────┘  Env.       │  └──────┬───────┘                  │  │
│   Business Needs            │         │                           │  │
│                             │  ┌──────▼───────┐                  │  │
│                             │  │Justify/Eval. │                  │  │
│                             │  │ • Analytical │                  │  │
│                             │  │ • Case Study │                  │  │
│                             │  │ • Experiment │                  │  │
│                             │  │ • Field Study│                  │  │
│                             │  │ • Simulation │                  │  │
│                             │  └──────────────┘                  │  │
│                             └──────────────────────────────────┘  │
│                                      │  Rigor                      │
│                                      ▼                              │
│                              ┌──────────────┐                      │
│                              │ Knowledge    │                      │
│                              │ Base         │                      │
│                              │ • Foundations│                      │
│                              │ • Methods    │                      │
│                              └──────────────┘                      │
└─────────────────────────────────────────────────────────────────────┘
```

**Quelle:** Hevner et al. (2004, Figure 2), https://wise.vub.ac.be/sites/default/files/thesis_info/design_science.pdf

### Drei Dimensionen des Frameworks:
1. **Umgebung (Environment):** Definiert Geschäftsbedürfnisse und Bewertungskriterien
2. **IS-Forschung:** Behaviorale und Design-Science-Aktivitäten (Develop/Build + Justify/Evaluate)
3. **Wissensbasis (Knowledge Base):** Theorien, Methoden, existierende Artefakte

---

## 5. Das Drei-Zyklen-Modell (Three Cycle View) nach Hevner (2007)

> Hevner, A. R. (2007). **A Three Cycle View of Design Science Research.** *Scandinavian Journal of Information Systems*, 19(2), 87–92.
>
> URL: https://aisel.aisnet.org/sjis/vol19/iss2/4/  
> PDF: https://www.uni-kassel.de/fb07/index.php?t=f&f=4898&token=930ca01adbdcc1e7f24d1a3d3a1f0a17422df86e

Hevner (2007) ergänzte das 2004er-Framework durch die explizite Darstellung von drei ineinandergreifenden Forschungszyklen. Alle drei Zyklen müssen in einem DSR-Projekt identifizierbar sein.

```
┌──────────────┐     ┌─────────────────────────────────────┐     ┌──────────────────┐
│  Application │     │         Design Science              │     │  Knowledge Base  │
│   Domain     │     │           Research                  │     │                  │
│              │     │                                     │     │ • Scientific     │
│ • People     │     │   ┌─────────────────────────────┐   │     │   Theories &    │
│ • Org.       │◄────┤   │   Build Design Artifacts &  │   │     │   Methods       │
│   Systems    │     │   │   Processes                 │   │     │ • Experience &  │
│ • Technical  │────►│   │                             │   │────►│   Expertise     │
│   Systems    │     │   │   ▼  ▼  ▼  ▼               │   │     │ • Meta-Artifacts│
│ • Problems   │     │   │   Evaluate                  │   │◄────│  (Design Prod.  │
│   &          │     │   └─────────────────────────────┘   │     │  & Processes)   │
│   Opport.    │     │                                     │     │                  │
└──────────────┘     └─────────────────────────────────────┘     └──────────────────┘
        ▲                     Design Cycle                                ▲
        │                                                                │
   Relevance                                                          Rigor
    Cycle                                                             Cycle
```

### 5.1 Relevance Cycle (Relevanzyklus)

> „The Relevance Cycle inputs requirements from the contextual environment into the research and introduces the research artifacts into environmental field testing."  
> — Hevner (2007, S. 87–88)

**Funktion:** Verbindet die Forschungsaktivitäten mit dem Anwendungskontext.

**Inputs aus der Umgebung:**
- Probleme und Chancen (Opportunity/Problem)
- Anforderungen an das Artefakt
- Akzeptanzkriterien für die Evaluation

**Outputs in die Umgebung:**
- Field Testing: Das Artefakt wird im realen Kontext erprobt
- Wenn das Field Testing Defizite zeigt → neuer Iterationsdurchlauf

**Schlüsselfrage:** *Verbessert das Artefakt die Umgebung, und wie kann diese Verbesserung gemessen werden?*

---

### 5.2 Design Cycle (Designzyklus)

> „The central Design Cycle supports a tighter loop of research activity for the construction and evaluation of design artifacts and processes."  
> — Hevner (2007, S. 87)

**Funktion:** Der innerste, schnellste Zyklus – der „Kern" der DSR-Arbeit. Iteriert zwischen Konstruktion und Evaluation.

**Ablauf:**
1. **Build:** Konstruktion/Überarbeitung des Artefakts (mit Inputs aus Relevance Cycle + Rigor Cycle)
2. **Evaluate:** Evaluation in künstlichen Umgebungen (Labor/Gedankenexperiment) *bevor* Field Testing
3. **Feedback:** Verfeinerung auf Basis der Evaluationsergebnisse

> „During the performance of the design cycle it is important to maintain a balance between the efforts spent in constructing and evaluating the evolving design artifact."  
> — Hevner (2007, S. 91)

**Wichtig:** Artefakte sollten rigoros in Labor-/Experimentalsettings getestet werden, *bevor* sie über den Relevance Cycle in den Field Test gehen.

---

### 5.3 Rigor Cycle (Rigorositätszyklus)

> „The Rigor Cycle provides grounding theories and methods along with domain experience and expertise from the foundations knowledge base into the research and adds the new knowledge generated by the research to the growing knowledge base."  
> — Hevner (2007, S. 87)

**Funktion:** Verbindet die Forschungsaktivitäten mit der wissenschaftlichen Wissensbasis.

**Inputs aus der Wissensbasis:**
- Wissenschaftliche Theorien und Methoden
- Erfahrungen und Expertise aus der Anwendungsdomäne
- Existierende Artefakte und Prozesse (Meta-Artefakte)

**Outputs in die Wissensbasis:**
- Erweiterungen bestehender Theorien und Methoden
- Neue Meta-Artefakte (Designprodukte und -prozesse)
- Gesamte Erfahrungen aus Forschung und Field Testing

**Schlüsselfrage:** *Stellt das Artefakt eine echte Forschungsleistung dar (nicht nur Routinedesign)?*

---

### Zusammenfassung: Drei Zyklen im Überblick

| Zyklus | Verbindet | Hauptfunktion | Takt |
|--------|-----------|---------------|------|
| Relevance Cycle | Forschung ↔ Anwendungsdomäne | Probleme einbringen, Field Test | Mittel |
| Design Cycle | Interner Bauloop | Konstruktion + Evaluation | Schnell |
| Rigor Cycle | Forschung ↔ Wissensbasis | Theoretische Fundierung sichern | Langsam |

---

## 6. DSR-Artefakttypen und Evaluationsmethoden im Überblick

| Artefakt | Evaluationsmethoden | Beispiel |
|----------|-------------------|---------|
| Konstrukt | Analytisch, Argumentativ | Ontologie-Überprüfung |
| Modell | Simulation, Analytisch | Formal verification |
| Methode | Case Study, Experiment | A/B-Vergleich zweier Algorithmen |
| Instanziierung | Experiment, Feldstudie | Usability Testing |

---

## 7. DSR im Kontext von Spielentwicklung und Software-Artefakt-Design

### 7.1 Spielentwicklung als DSR-Forschungskontext

DSR wird zunehmend im Bereich der Serious Games und Educational Games eingesetzt. Hevner et al. (2004) positionieren IS-Artefakte an der Schnittstelle von Menschen, Organisationen und Technologie – eine Beschreibung, die auf Spielsysteme direkt zutrifft.

**Anwendungsbeispiele:**
- **Serious Games:** Oliveira et al. (2022) nutzen DSR für die Entwicklung eines Lernspiels (*Design Science Research: Balancing Science and Art in Building a Game Applied to Health*, Journal of Interactive Systems). Die drei Zyklen strukturierten Diagnose, Entwurf, Implementierung und Weiterentwicklung des Spielartifakts.
- **Gamified Decision-Making:** Loch et al. (2023) wenden den DSR-Rahmen von Peffers et al. auf ein interdisziplinäres Team zur Spielentwicklung an (fünf Phasen: Problemidentifikation, Lösungsziele, Artefaktdesign/-entwicklung, Demonstration, Evaluation). (*Applying design research for cross-disciplinary collaboration*, European Conference on Games Based Learning, 2023, https://papers.academic-conferences.org/index.php/ecgbl/article/view/1567)
- **Framework-Entwicklung:** Serious-Game-Design-Framework via DSR (JMIR Serious Games, 2024): Stakeholder-zentriertes Framework, validiert durch 29 Praktiker. https://games.jmir.org/2024/1/e48099/

### 7.2 Mapping der DSR-Leitlinien auf Spielentwicklung

| DSR-Leitlinie | Entsprechung in der Spielentwicklung |
|---------------|-------------------------------------|
| Design als Artefakt | Spielprototyp, Engine-Komponente, Framework |
| Problemrelevanz | Spielerlebnis-Hypothese, Marktlücke |
| Designevaluation | Playtest, Framezeit-Messung, Usability-Test |
| Forschungsbeiträge | Neues Mechaniksystem, innovativer Algorithmus |
| Forschungsrigorosität | Fundierung in bestehender Spielforschung |
| Design als Suchprozess | Iterativer Prototyp-Build-Test-Loop |
| Kommunikation | Technische Doku + Publisher-Pitch |

### 7.3 DSR und Software-Artefakt-Design

DSR ist besonders gut für Software-Systeme geeignet, die:
- **Wicked Problems** adressieren (komplexe, schlecht definierte Probleme)
- **Iterative Entwicklung** erfordern (mehrere Design-Evaluate-Loops)
- **Theorieanwendung und Schöpfung** kombinieren

Herbert Simon (*The Sciences of the Artificial*, 1996) legte die philosophische Grundlage: Natürliche Wissenschaften beschreiben, wie Dinge *sind*; Designwissenschaften beschreiben, wie Dinge *sein sollen*.

Im Spielekontext: Ein Rennspiel-System, das Transformations-Animationen implementiert, ist ein IT-Artefakt (Instanziierung). Die Forschungsfrage lautet: *Wie soll das System gebaut werden, damit Transformation in Echtzeit flüssig und performant ist?*

---

## 8. DSRM Prozessmodell (Peffers et al. 2007) – Ergänzung zu Hevner

> Peffers, K., Tuunanen, T., Rothenberger, M. A., & Chatterjee, S. (2007). A Design Science Research Methodology for Information Systems Research. *Journal of Management Information Systems*, 24(3), 45–77.
> https://dl.acm.org/doi/abs/10.2753/MIS0742-1222240302

Der DSRM-Prozess besteht aus sechs Schritten, die Hevners Zyklen operationalisieren:

```
1. Problem identification & motivation
         ↓
2. Definition of objectives for a solution
         ↓
3. Design & development
         ↓
4. Demonstration
         ↓
5. Evaluation
         ↓
6. Communication
         ↑ (iterativ zurück zu beliebigem Schritt)
```

---

## 9. Weiterführende Literatur und Ressourcen

| Quelle | Beitrag | URL |
|--------|---------|-----|
| Hevner et al. (2004) | Sieben Leitlinien, IS-Framework | https://wise.vub.ac.be/sites/default/files/thesis_info/design_science.pdf |
| Hevner (2007) | Drei-Zyklen-Modell | https://aisel.aisnet.org/sjis/vol19/iss2/4/ |
| Gregor & Hevner (2013) | Design Theory, drei Artefakt-Ebenen | https://doi.org/10.25300/MISQ/2013/37.2.06 |
| Peffers et al. (2007) | DSRM-Prozessmodell (6 Schritte) | https://dl.acm.org/doi/abs/10.2753/MIS0742-1222240302 |
| Design-Science-Research.de | Übersichtsressource (deutsch/englisch) | https://design-science-research.de/en/course/dsr-questions/artefakte-knowledge/ |
| Van der Merwe et al. (2019) | Leitlinien für DSR-Novizen | http://www.cair.org.za/sites/default/files/2020-02/SACLA2019.pdf |
| Wikipedia DSR | Übersicht, Artefakttypen | https://en.wikipedia.org/wiki/Design_science_(methodology) |

---

## 10. Schlüsselzitate

> „The fundamental principle of design-science research from which our seven guidelines are derived is that **knowledge and understanding of a design problem and its solution are acquired in the building and application of an artifact**."  
> — Hevner et al. (2004, S. 82)

> „The goal of behavioral-science research is truth. The goal of design-science research is **utility**."  
> — Hevner et al. (2004, S. 80)

> „Design is both a process (set of activities) and a product (artifact) – a verb and a noun."  
> — Hevner et al. (2004)

> „It is the synergy between relevance and rigor and the contributions along both the relevance cycle and the rigor cycle that define good design science research."  
> — Hevner (2007, S. 92)

> „The essence of Information Systems as design science lies in the **scientific evaluation of artifacts**."  
> — Iivari (2007), zitiert in Hevner (2007, S. 91)
