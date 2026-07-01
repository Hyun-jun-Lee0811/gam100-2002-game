# 2002 — Virus Field Game
<div align="center">
<img width="600" height="350" alt="image" src="https://github.com/user-attachments/assets/af32d291-b82d-47cf-b071-be378a07759c" />
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

[🇺🇸 View README in English](./README.md)

---

## 목차

- [개요](#개요)
- [게임플레이 영상](#게임플레이-영상)
- [조작법](#조작법)
- [기능 명세](#기능-명세)
  - [씬(Scene) 흐름](#씬scene-흐름)
  - [플레이어 시스템](#플레이어-시스템-hp--mp--armor)
  - [1. 스테이지 1 — 지상층](#1-스테이지-1--지상층)
  - [2. 스테이지 2 — 수직 아레나 & 보스](#2-스테이지-2--수직-아레나--보스)
- [기술 스택](#기술-스택)
- [프로젝트 구조](#프로젝트-구조)
- [빌드 및 실행](#빌드-및-실행)
- [크레딧](#크레딧)
- [이슈 / 회고](#이슈--회고)
- [라이선스](#라이선스)

---

## 개요

**2002**는 DigiPen GAM100 과목(2021년 가을학기) 텀 프로젝트로 제작된 2D 사이드뷰 액션 게임이며, 하이 콘셉트는 "바이러스 필드 게임(Virus field game)"입니다. `doodle` 2D 프레임워크로 제작되었고, 플레이어는 총 2개의 스테이지에서 바이러스 컨셉의 몬스터들과 싸우며 HP·MP·Armor 자원을 관리하고, 최종적으로 다단계 패턴을 가진 보스를 처치하면 승리합니다.

---

##  게임플레이 영상

<p align="center">
  <a href="https://www.youtube.com/watch?v=CbdT5rrbxoo" target="_blank">
    <img src="https://img.youtube.com/vi/CbdT5rrbxoo/maxresdefault.jpg" width="800" alt="게임플레이 영상">
  </a>
</p>

<p align="center">
  <b>▶ 이미지를 클릭하면 YouTube에서 전체 플레이 영상을 시청할 수 있습니다.</b>
</p> 

---

## 조작법

| 키 | 동작 |
|---|---|
| **A** | 좌측 이동 |
| **D** | 우측 이동 |
| **Space** | 점프 (게이지 방식) |
| **J** (+ A/D) | 근접 공격 — 소드 / 사탕(캔디), 방향에 따라 다른 모션 |
| **K** | 상황별 기능: 스테이지 1에서는 **알코올** 투사체(A/D와 조합), 두 스테이지 모두에서는 몬스터 경고 구역 내 데미지를 줄이는 **가드**, 스테이지 2에서는 **백신(Vaccine)** 스킬 |
| **L** | 원거리 **레인(Rain)** 스킬 (HP 300 초과, MP 200 초과일 때 사용 가능) |

---

## 기능 명세

### 씬(Scene) 흐름

게임은 `main.cpp`의 단일 `current_scene` 정수 값으로 제어되는 switch문으로 구동됩니다.

| 씬 번호 | 이름 | 설명 |
|---|---|---|
| 9 | 로고 | 시작 시 DigiPen 로고 애니메이션 재생 후 자동으로 메인 메뉴로 전환, 스테이지 1 배경음악 재생 시작 |
| 0 | 메인 메뉴 | Start / How To Play / Credit 버튼 |
| 1 | 스테이지 1 | 지상층 액션 스테이지 |
| 2 | 스테이지 2 | 수직 아레나 + 보스 스테이지 |
| 3 | 게임 오버 | 플레이어 HP가 0 미만이 되면 표시, Replay 버튼으로 전체 상태 초기화 |
| 4 | 승리 | 보스 HP가 1.1 이하가 되면 표시, Replay 버튼으로 전체 상태 초기화 |
| 5 | 게임 방법 (1/2) | 튜토리얼 첫 페이지, Next 버튼 → 씬 6 |
| 6 | 게임 방법 (2/2) | 튜토리얼 두 번째 페이지, Next 버튼 → 씬 7 |
| 7 | 아이템 안내 | 아이템 설명 페이지, Quit 버튼 → 메인 메뉴 |
| 8 | 크레딧 | 팀 크레딧 스크롤, Back 버튼 → 메인 메뉴 |

### 플레이어 시스템 (HP / MP / Armor)

- **HP** (`playerHP_width`, 최대 360): 최대치 미만일 때 프레임당 +0.01씩 서서히 자동 회복됩니다. 0 미만이 되면 게임 오버가 발생합니다.
- **MP** (`playerMP_width`, 최대 360): 레인 스킬(`L`)을 사용하지 않는 동안 프레임당 +0.05씩 자동 회복되며, 알코올·레인·백신 스킬 사용 시 소모됩니다.
- **Armor HP** (`armorHP_width`, 0~300): 초기값 0이며, 맵에 있는 방어구 아이템을 획득하면 300으로 설정되고 캐릭터 스프라이트가 갑옷("warrior") 버전으로 바뀝니다. 몬스터의 "경고 구역(warning zone)"에 들어가면 가드(`K`)를 누르고 있지 않는 한 Armor HP가 감소하며, Armor HP가 소진되면 이후에는 HP가 직접 감소합니다.
- **점프 게이지** (`Jplayer_width`, 스테이지 2 전용): 시간이 지나면 최대 100까지 차오르고, `Space`를 누르고 있는 동안 프레임당 4씩 소모되며 플레이어를 위로 밀어 올립니다. 공중에 떠 있는 상태에서도 게이지가 재충전될 수 있기 때문에, `Space`를 계속 누르고 있으면 원래 의도보다 더 오래 상승할 수 있습니다(팀에서도 의도치 않은 "치트"로 문서화한 부분입니다 — [이슈 / 회고](#이슈--회고) 참고).

---

### 1. 스테이지 1 — 지상층

<div align="center">
<table>
<tr>
<td align="center" width="33.3%"><img src="docs/media/stage1_1.gif" width="100%"/><br/><sub><b>층 몬스터 조우</b></sub></td>
<td align="center" width="33.3%"><img src="docs/media/stage1_2.gif" width="100%"/><br/><sub><b>소드 & 알코올 공격</b></sub></td>
<td align="center" width="33.3%"><img src="docs/media/stage1_3.gif" width="100%"/><br/><sub><b>키 게이지 & 문 개방</b></sub></td>
</tr>
</table>
</div>

3개의 층으로 이루어진 배경과, 층을 연결하는 4개의 문, 그리고 스테이지 1 키 목표를 완료해야 열리는 5번째 문(스테이지 2로 연결)으로 구성됩니다.

**3종류의 층 몬스터**, 각기 다른 패턴을 가집니다:

| 몬스터 | 행동 |
|---|---|
| 1층 몬스터 | 좌우로 순찰하며 플레이어가 자신 아래에 서면 검을 휘두릅니다. 처치 시 MP 회복 아이템을 드롭합니다 |
| 2층 몬스터 | 방향에 따라 다른 속도로 순찰하며, 직접 접촉 시 플레이어에게 데미지를 줍니다. 처치 시 열쇠 아이템과 HP 회복 아이템을 드롭합니다 |
| 3층 몬스터 | 순찰하며 번개 공격을 시전하고, 재생되는 2개의 "방어벽"에 의해 보호받습니다. 이 방어벽을 (검 또는 알코올 공격으로) 먼저 깎아야 몬스터 본체에 데미지를 줄 수 있습니다 |

**플레이어 공격:** 근접 소드/캔디 공격(`J` + 방향키, Armor 상태에 따라 모션이 달라짐)과 알코올 투사체 공격(`K` + 방향키, MP 소모). **원거리 레인 스킬**(`L`)은 HP 300 초과, MP 200 초과일 때 사용 가능하며, 3종류의 층 몬스터 모두에게 데미지를 줍니다.

**스테이지 1 키 게이지:** 열쇠 아이템을 획득하면 게이지가 감소하며, 게이지가 0 이하가 되면 5번째 문(스테이지 2 진입)을 사용할 수 있습니다. 별도의 HP·MP 회복 아이템은 접촉 시 해당 스탯을 완전히 회복시킵니다.

---

### 2. 스테이지 2 — 수직 아레나 & 보스

<div align="center">
<table>
<tr>
<td align="center" width="33.3%"><img src="docs/media/stage2_1.gif" width="100%"/><br/><sub><b>층 몬스터 전투</b></sub></td>
<td align="center" width="33.3%"><img src="docs/media/stage2_2.gif" width="100%"/><br/><sub><b>점프 게이지 & 보스 레인 위협</b></sub></td>
<td align="center" width="33.3%"><img src="docs/media/stage2_3.gif" width="100%"/><br/><sub><b>보스전 — 미니보스 페이즈</b></sub></td>
</tr>
</table>
</div>

4개의 고정 벽(아래/왼쪽/오른쪽/위쪽)으로 둘러싸인 공간이며, 접촉 시 플레이어를 밀어냅니다. 9개의 발판이 배치되어 있으며 접근 방향에 따라 위에서 착지하는 경우와 아래에서 부딪히는 경우가 다르게 처리됩니다. 이 중 2개의 발판은 플레이어가 위에 서 있는 동안 HP 또는 MP를 수동적으로 회복시켜 줍니다. 방(room) 중앙에는 감옥 형태의 장식 오브젝트가 있으며, 4마리의 층 몬스터 중 하나라도 살아있는 동안 표시됩니다.

**4마리의 층 몬스터**(`monster1.h` ~ `monster4.h`)는 각각 독립적인 HP, 양방향 검 공격, 그리고 가드(`K`)를 누르지 않으면 Armor HP(이후 HP)를 감소시키는 "경고 구역"을 가지고 있습니다.

**점프 기능**(`Space`)은 앞서 설명한 게이지 시스템을 사용합니다. **환경 위협 요소:** 4마리의 층 몬스터를 모두 처치하면 낙하하는 바이러스 투사체("보스 레인")가 무작위 위치에서 계속 생성되며, 보스전이 끝날 때까지 접촉 시 플레이어에게 데미지를 줍니다.

**백신 스킬**(`K`, 누르고 있는 동안): MP를 소모하며 플레이어 주변에 백신 애니메이션을 표시합니다. MP가 50 미만으로 떨어지면 Armor HP, 이어서 HP가 감소하기 시작하여 자원 관리의 트레이드오프를 만듭니다.

#### 보스전

보스(`STAGE2_BOSS`)의 HP는 800에서 시작하며, 사거리 내에서 근접 검 공격을 맞을 때마다 감소합니다.

| 보스 HP 구간 | 행동 |
|---|---|
| 620 ~ 800 | 아레나 상단을 좌우로 비행하며 이동하고, 근접 검 콤보로 데미지를 줄 수 있습니다 |
| 570 ~ 620 | 보스가 잠시 멈추고 미니보스 3체가 등장하며, 이들의 HP 합계가 소진될 때까지 전투가 진행되지 않습니다 |
| 380 ~ 430 | 아레나 전역에 8개의 낙하 바이러스 기둥이 생성되고, 옆에서 "위저드" 몬스터가 등장해 주기적으로 번개 공격을 시전합니다 |
| 120 ~ 380 | 전환 비행 페이즈이며, 계속 데미지를 줄 수 있습니다 |
| 1 ~ 120 | 최종 페이즈 — 보스가 작고 빠르게 움직이는 비행형 적으로 변하며, 플레이어 근처로 급강하할 때 공격해야 합니다 |
| 1.1 이하 | 보스 처치 → 승리 씬으로 전환 |

---

## 기술 스택

- **언어:** C++17
- **그래픽스:** `doodle` 프레임워크 (수업에서 제공된 2D 엔진, `doodle/doodle.hpp`)
- **오디오:** SFML (`sfml-audio-2`, `sfml-system-2`, `openal32`)
- **빌드 시스템:** Visual Studio 프로젝트 (`.vcxproj` / `.vcxproj.filters`)
- **코드 스타일:** LLVM 기반 `.clang-format` — C++17, 4칸 들여쓰기, Allman 중괄호 스타일, 120열 제한

---

## 프로젝트 구조

```
gam100-2002-game/
├── Documentation/
│   ├── Final Presentation.pptx
│   └── 2002 play video.mp4
├── Game/                       # 바로 실행 가능한 빌드 결과물
│   ├── asset/
│   ├── sound/
│   │   └── stage1.ogg
│   ├── openal32.dll
│   ├── sfml-audio-2.dll
│   ├── sfml-system-2.dll
│   └── 2002.exe
└── Project/
    └── 2002/                   # Visual Studio 프로젝트 & 소스 코드
        ├── asset/
        ├── doodle/              # 수업에서 제공된 2D 프레임워크
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

## 빌드 및 실행

- **빌드된 게임 바로 실행:** `Game/2002.exe`를 실행하세요. 실행에는 DLL 파일(`openal32.dll`, `sfml-audio-2.dll`, `sfml-system-2.dll`)과 `sound/` 폴더가 같은 경로에 있어야 하며, `Game/` 폴더에 모두 포함되어 있습니다. Windows 전용입니다.
- **소스에서 직접 빌드:** Visual Studio에서 `Project/2002` 폴더를 여세요. 프로젝트는 내부에 포함된 `doodle` 프레임워크와 `external/SFML`을 참조합니다. C++17 언어 표준이 활성화되어 있는지 확인한 뒤 빌드 및 실행하세요.

---

## 크레딧

| 팀원 | 역할 | 담당 내용 |
|---|---|---|
| **이현준 (Hyunjun Lee)** | 프로듀서 & 프로그래머 | 게임 코드 대부분을 작성했으며, 오디오 연동 작업을 담당했습니다 |
| **여금비 (Geumbi Yeo)** | 디자인 & 오디오 리드 | 모든 배경과 캐릭터를 디자인·제작했고, 배경음악을 작곡했으며, 게임의 장르 방향을 결정했습니다 |
| **정다운 (Dawoon Jung)** | 프로그래밍 어시스턴트 | 코딩을 보조했으며, 게임 시스템 전반에 기여했습니다 |
| **프레임워크** | — | DigiPen GAM100 과목에서 제공된 `doodle` 2D 그래픽스/입력 라이브러리를 기반으로 제작 |

---

## 이슈 / 회고

- 바닥-플레이어 충돌 처리와 몬스터 "감염 범위" 데미지 시스템은 의도한 대로 잘 작동합니다.
- 보스의 비행 및 공격 패턴은 구현되어 의도한 대로 작동합니다.
- 점프/중력은 실제 물리 기반 구현이 **아니며**, 게이지 방식으로 되어 있습니다. 공중에서도 게이지가 재충전될 수 있어 `Space`를 계속 누르고 있으면 원래 의도보다 더 높이/오래 상승할 수 있습니다(팀에서 인지하고 있으나 수정하지 않은 "치트"성 동작입니다).

---

## 라이선스

© 2021 DigiPen (USA) Corporation. All rights reserved.
본 프로젝트는 DigiPen GAM100F21 과목(담당 교수: Rudy Castan)을 위해 제작된 학업용 프로젝트입니다. 모든 게임 디자인과 에셋은 팀이 직접 제작했습니다.
