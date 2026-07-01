# 2002 — Virus Field Game

<div align="center">
<img width="741" height="247" alt="image" src="https://github.com/user-attachments/assets/283dfe60-bafc-4852-bdbc-d736dde36891" />
<br>
</div>

<div align="center">

[![C++](https://img.shields.io/badge/C%2B%2B17-Game-00599C?logo=cplusplus&logoColor=white)](#)
[![DigiPen](https://img.shields.io/badge/DigiPen-GAM100F21-orange)](#)
[![Doodle](https://img.shields.io/badge/Framework-Doodle-blueviolet)](#)

</div>

> **Team:** Semicolon
> **Members:** Hyunjun Lee, Geumbi Yeo, Dawoon Jung
> **Course:** GAM100 (DigiPen Institute of Technology)
> **Instructor:** Rudy Castan
> **Language:** C++17
> **Framework:** `doodle` (course-provided framework)
> **Audio:** SFML

[🇰🇷 View README in Korean](./README.ko.md)

---

## Table of Contents

- [Overview](#overview)
- [Gameplay Video](#gameplay-video)
- [Controls](#controls)
- [Feature Specification](#feature-specification)
  - [Scene Flow](#scene-flow)
  - [Player System (HP / MP / Armor)](#player-system-hp--mp--armor)
  - [1. Stage 1 — Ground Floor](#1-stage-1--ground-floor)
  - [2. Stage 2 — Vertical Arena](#2-stage-2--vertical-arena)
  - [3. Stage 3 — Boss Fight](#3-stage-3--boss-fight)
- [Tech Stack](#tech-stack)
- [Project Structure](#project-structure)
- [Build & Run](#build--run)
- [Credits](#credits)
- [Issues & Retrospective](#issues--retrospective)
- [License](#license)

---

## Overview

<div align="center">
<img width="600" height="350" alt="0" src="https://github.com/user-attachments/assets/a6a376f6-a20e-4f82-b5a1-448f2231ca6a" />
<br>
<sub><b>▲ Opening · Main Screen Preview</b></sub>
</div>

**2002** is a 2D side-view action game developed as a term project for DigiPen GAM100 (Fall 2021), with the high concept of a "Virus Field Game." Built using the `doodle` 2D framework, the player fights through three stages — ① Ground Floor → ② Vertical Arena → ③ Boss Fight — battling virus-themed monsters while managing HP, MP, and Armor resources. The game ends in victory when the multi-phase boss is defeated.

> Internally, Stage 2 (Vertical Arena) and Stage 3 (Boss Fight) run sequentially within a single `current_scene == 2`. However, because they represent two clearly distinct gameplay segments, this document describes them separately.

---

## Gameplay Video

<p align="center">
  <a href="https://www.youtube.com/watch?v=CbdT5rrbxoo" target="_blank">
    <img src="https://img.youtube.com/vi/CbdT5rrbxoo/maxresdefault.jpg" width="800" alt="Gameplay Video">
  </a>
</p>

<p align="center">
  <b>▶ Click the image to watch the full playthrough on YouTube.</b>
</p>

---

## Controls

| Key | Action |
|---|---|
| **A** | Move left |
| **D** | Move right |
| **Space** | Jump (gauge-based) |
| **J** (+ A / D) | Melee attack — Sword / Candy; animation varies by direction and Armor state |
| **K** | Context-sensitive: **Alcohol** projectile in Stage 1 (combined with A/D); **Guard** in all stages (reduces damage while inside a monster's warning zone); **Vaccine** skill in Stage 3 |
| **L** | Ranged **Rain** skill (usable only when HP > 300 and MP > 200) |

---

## Feature Specification

### Scene Flow

The game is driven by a single `current_scene` integer in `main.cpp` that controls a top-level switch statement.

| Scene # | Name | Description |
|---|---|---|
| 9 | Logo | DigiPen logo animation plays on launch, then auto-transitions to the Main Menu; Stage 1 BGM begins |
| 0 | Main Menu | Start / How To Play / Credit buttons |
| 1 | Stage 1 | Ground floor action stage |
| 2 | Stage 2 & 3 | Vertical Arena (Stage 2) followed immediately by the Boss Fight (Stage 3) |
| 3 | Game Over | Shown when player HP drops below 0; Replay button resets all state |
| 4 | Victory | Shown when boss HP falls to 1.1 or below; Replay button resets all state |
| 5 | How to Play (1/2) | Tutorial page 1; Next button → Scene 6 |
| 6 | How to Play (2/2) | Tutorial page 2; Next button → Scene 7 |
| 7 | Item Guide | Item description page; Quit button → Main Menu |
| 8 | Credits | Team credits scroll; Back button → Main Menu |

---

### Player System (HP / MP / Armor)

- **HP** (`playerHP_width`, max 360): Regenerates slowly at +0.01 per frame whenever it is below maximum. Reaching 0 triggers Game Over.
- **MP** (`playerMP_width`, max 360): Regenerates at +0.05 per frame while the Rain skill (`L`) is not active. Consumed by the Alcohol, Rain, and Vaccine skills.
- **Armor HP** (`armorHP_width`, range 0–300): Starts at 0. Picking up the armor item on the map sets it to 300 and changes the character sprite to the armored ("warrior") version. While standing inside a monster's warning zone without holding Guard (`K`), Armor HP decreases. Once Armor HP is depleted, HP is damaged directly instead.
- **Jump Gauge** (`Jplayer_width`, Stage 2 & 3 only): Fills up to a maximum of 100 over time. Holding `Space` consumes 4 per frame and pushes the player upward. Because the gauge can refill even while the player is airborne, holding `Space` continuously allows the player to ascend higher and longer than originally intended — a known behavior the team documented as an unintended "cheat." See [Issues & Retrospective](#issues--retrospective).

---

### 1. Stage 1 — Ground Floor

<div align="center">
<table>
<tr>
<td align="center" width="33.3%"><img src="https://github.com/user-attachments/assets/61d03188-c5d8-4a40-90a1-132d0de148cf" width="100%"/><br/><sub><b>Floor 1 Monster</b></sub></td>
<td align="center" width="33.3%"><img src="https://github.com/user-attachments/assets/2b3160fc-167a-436d-ac3b-4579be51f85d" width="100%"/><br/><sub><b>Floor 2 Monster</b></sub></td>
<td align="center" width="33.3%"><img src="https://github.com/user-attachments/assets/f1e36c8a-bec5-4735-a265-95af7671993f" width="100%"/><br/><sub><b>Floor 3 Monster</b></sub></td>
</tr>
</table>
</div>

The stage consists of a three-floor background, four doors connecting the floors, and a fifth door (leading to Stage 2) that unlocks only when the Stage 1 key objective is completed.

**Three floor monster types**, each with distinct behavior:

| Monster | Behavior |
|---|---|
| Floor 1 Monster | Patrols left and right; swings a sword if the player stands directly below it. Drops an MP recovery item on defeat. |
| Floor 2 Monster | Patrols at different speeds depending on direction; damages the player on direct contact. Drops a key item and an HP recovery item on defeat. |
| Floor 3 Monster | Patrols and casts a lightning attack. Protected by two regenerating "shields" that must be depleted first (via sword or alcohol attacks) before the monster's own HP can be reduced. |

**Player attacks:** Melee sword/candy attack (`J` + direction, animation differs based on Armor state) and alcohol projectile (`K` + direction, costs MP). The **Rain skill** (`L`) is available when HP > 300 and MP > 200, and deals damage to all three floor monster types.

**Stage 1 Key Gauge:** Collecting key items decreases the gauge. When the gauge reaches 0 or below, the fifth door (Stage 2 entrance) becomes accessible. Separate HP and MP recovery items fully restore the respective stat on contact.

---

### 2. Stage 2 — Vertical Arena

<div align="center">
<table>
<tr>
<td align="center" width="33.3%"><img src="https://github.com/user-attachments/assets/6a740057-7fdd-4219-8524-5151281b786a" width="100%"/><br/><sub><b>Armor Pickup</b></sub></td>
<td align="center" width="33.3%"><img src="https://github.com/user-attachments/assets/bc21f590-c5a7-4b89-ada3-77d50feaffe4" width="100%"/><br/><sub><b>Monster Defeated</b></sub></td>
<td align="center" width="33.3%"><img src="https://github.com/user-attachments/assets/1599e775-d816-44f0-a4c4-ff2b2dbff3d5" width="100%"/><br/><sub><b>Stage 3 Entry</b></sub></td>
</tr>
</table>
</div>

The arena is enclosed by four fixed walls (bottom / left / right / top) that push the player back on contact. Nine platforms are placed throughout; landing on top vs. hitting from below are handled differently. Two of the platforms passively recover HP or MP while the player stands on them. A cage decoration object is displayed in the center of the room as long as at least one of the four floor monsters is still alive.

**Four floor monsters** (`monster1.h` – `monster4.h`) each have independent HP, bidirectional sword attacks, and a warning zone that drains Armor HP (then HP) unless the player holds Guard (`K`).

**Jump** (`Space`) uses the gauge system described above. Defeating all four monsters removes the cage object and transitions to Stage 3.

---

### 3. Stage 3 — Boss Fight

<div align="center">
<table>
<tr>
<td align="center" width="50%"><img src="https://github.com/user-attachments/assets/e9427051-ca25-4757-b64f-7c8e0dbb0adb" width="100%"/><br/><sub><b>Boss Fight Start</b></sub></td>
<td align="center" width="50%"><img src="https://github.com/user-attachments/assets/0fa6b285-1336-4e3e-a311-43a872a0bcd0" width="100%"/><br/><sub><b>Boss Movement Pattern</b></sub></td>
</tr>
<tr>
<td align="center" width="50%"><img src="https://github.com/user-attachments/assets/4fb34336-f558-4739-b208-e2b942869036" width="100%"/><br/><sub><b>Mini-Boss Phase</b></sub></td>
<td align="center" width="50%"><img src="https://github.com/user-attachments/assets/0729ef53-68fd-4543-b49b-6391c20a547f" width="100%"/><br/><sub><b>Virus Pillars & Wizard</b></sub></td>
</tr>
<tr>
<td align="center" width="50%"><img src="https://github.com/user-attachments/assets/78499344-c52a-44e6-a537-8ae6171fb698" width="100%"/><br/><sub><b>Transition Flight Phase</b></sub></td>
<td align="center" width="50%"><img src="https://github.com/user-attachments/assets/b764af3e-12e5-4000-9d0a-f3156a129b0c" width="100%"/><br/><sub><b>Final Phase</b></sub></td>
</tr>
</table>
</div>

**Vaccine skill** (`K`, hold): Consumes MP and plays the vaccine animation around the player. If MP drops below 50, Armor HP begins to drain — and HP directly after that — creating a resource-management trade-off.

**Environmental hazard:** Once all Stage 2 monsters are defeated, falling virus projectiles ("boss rain") continuously spawn at random positions and deal damage to the player on contact until the boss is defeated.

The boss (`STAGE2_BOSS`) starts with 800 HP, reduced each time a melee sword attack connects within range.

| Boss HP Range | Behavior |
|---|---|
| 620 – 800 | Flies left and right across the top of the arena; melee sword combos deal damage |
| 570 – 620 | Boss briefly stops; 3 mini-bosses spawn and must be fully defeated before the fight resumes |
| 380 – 430 | 8 falling virus pillars spawn across the arena; a "Wizard" monster appears at the side and periodically casts lightning attacks |
| 120 – 380 | Transition flight phase; the player can continue dealing damage |
| 1 – 120 | Final phase — the boss transforms into a small, fast-moving flying enemy; attack when it dives close to the player |
| ≤ 1.1 | Boss defeated → Victory scene |

---

## Tech Stack

- **Language:** C++17
- **Graphics:** `doodle` framework (course-provided 2D engine, `doodle/doodle.hpp`)
- **Audio:** SFML (`sfml-audio-2`, `sfml-system-2`, `openal32`)
- **Build System:** Visual Studio project (`.vcxproj` / `.vcxproj.filters`)
- **Code Style:** LLVM-based `.clang-format` — C++17, 4-space indentation, Allman brace style, 120-column limit

---

## Project Structure

```
gam100-2002-game/
├── Documentation/
│   ├── Final Presentation.pptx
│   └── 2002 play video.mp4
├── Game/                       # Ready-to-run build output
│   ├── asset/
│   ├── sound/
│   │   └── stage1.ogg
│   ├── openal32.dll
│   ├── sfml-audio-2.dll
│   ├── sfml-system-2.dll
│   └── 2002.exe
└── Project/
    └── 2002/                   # Visual Studio project & source code
        ├── asset/
        ├── doodle/              # Course-provided 2D framework
        ├── external/
        │   └── SFML/
        │       ├── include/
        │       ├── license.md
        │       ├── readme.md
        │       └── Version.txt
        ├── sound/
        ├── .clang-format
        ├── alcohol.h
        ├── background.h
        ├── background2.h
        ├── boss_rain.h
        ├── doors.h
        ├── first_floor_monster.h
        ├── Game Menu.h
        ├── main.cpp
        ├── monster1.h … monster4.h
        ├── player.h
        ├── player_key.h
        ├── rain.h
        ├── Screen.h
        ├── second_floor_monster.h
        ├── SmallMini_virus.h
        ├── Stage1_Key.h
        ├── stage2_mini_boss.h
        ├── stage2boss.h
        ├── sword.h
        ├── third_floor_monster.h
        └── vaccine.h
```

---

## Build & Run

- **Run the pre-built game:** Execute `Game/2002.exe`. The DLL files (`openal32.dll`, `sfml-audio-2.dll`, `sfml-system-2.dll`) and the `sound/` folder must be in the same directory — all are already included in `Game/`. Windows only.
- **Build from source:** Open the `Project/2002` folder in Visual Studio. The project references the bundled `doodle` framework and `external/SFML`. Ensure C++17 is enabled, then build and run.

---

## Credits

<img width="360" height="192" alt="credit" src="https://github.com/user-attachments/assets/bf136aa9-b996-4a3e-ae52-ea43c39f0353" />

| Member | Role | Contributions |
|---|---|---|
| **Hyunjun Lee** | Producer & Programmer | Wrote the majority of the game code; handled audio integration |
| **Geumbi Yeo** | Design & Audio Lead | Designed and created all backgrounds and characters; composed the BGM; determined the game's genre direction |
| **Dawoon Jung** | Programming Assistant | Assisted with coding; contributed to overall game systems |
| **Framework** | — | Built on the `doodle` 2D graphics/input library provided for DigiPen's GAM100 course |

---

## Issues & Retrospective

- Floor–player collision handling and the monster "infection zone" damage system function as intended.
- The boss's flight and attack patterns are implemented and behave as designed.
- Jump / gravity is **not** physics-based — it uses a gauge system. Because the gauge can refill while the player is airborne, holding `Space` continuously allows the player to rise higher and longer than originally intended. The team acknowledged this as an unintended "cheat" but did not patch it before submission.

---

## License

© 2021 DigiPen (USA) Corporation. All rights reserved.
This project was created as an academic assignment for DigiPen GAM100F21 (Instructor: Rudy Castan). All game design and assets were produced by the team.
