# Stoneford — WorldLines 스타터 월드

![Stoneford](./assets/s1bg.png)

> [WorldLines](https://worldlines.gg) 엔진을 위한 잿빛 안개의 북방 강변 항구. 클래식 판타지 TRPG · d20 주사위 · 10-agent 오케스트레이터.

**Language:** [English](./README.md) · [简体中文](./README.zh.md) · [日本語](./README.ja.md) · [한국어](./README.ko.md)

<p align="center">
  <a href="https://youtu.be/QARKAA8bXDk">
    <img src="https://img.youtube.com/vi/QARKAA8bXDk/maxresdefault.jpg" alt="WorldLines · 트레일러" width="640"/>
  </a>
</p>
<p align="center"><em>▶ YouTube에서 트레일러 보기</em></p>

<table align="center">
  <tr>
    <td width="50%" align="center"><img src="./assets/cards/D1%20%E7%B1%B3%EB%9D%BC%EC%99%B8%EC%83%81%20%28deck%20%C2%B7%20KO%29.png" alt="미라 외상 — 스토리 카드" /></td>
    <td width="50%" align="center"><img src="./assets/cards/D2%20%EB%9D%BC%EC%98%A4%EC%9B%A8%EC%9D%B4%20%EC%95%BD%EC%86%8D%20%28deck%20%C2%B7%20KO%29.png" alt="라오웨이 약속 — 스토리 카드" /></td>
  </tr>
  <tr>
    <td width="50%" align="center"><img src="./assets/cards/D3%20Gameplay%20%28deck%20%C2%B7%20KO%29.png" alt="Gameplay" /></td>
    <td width="50%" align="center"><img src="./assets/cards/D4%204%20Worlds%20%28deck%20%C2%B7%20KO%29.png" alt="4 Worlds" /></td>
  </tr>
</table>

> **엔진:** Stoneford는 [WorldLines](https://github.com/LudicDynamics/WorldLines) 엔진 위에서 동작합니다. 엔진 본체 · 런타임 · 전체 문서는 [LudicDynamics/WorldLines](https://github.com/LudicDynamics/WorldLines)를 참조하세요.

> **면책 고지:** Stoneford 는 **nikoloside**, **redoctober**, Claude 가 함께 집필한 오리지널 시나리오로, 동양적 세계관과 서양 중세 마법을 융합한 작품입니다. 유사한 부분이 있다면 수정과 업데이트를 위해 원작자에게 연락해 주십시오.

---

## 빠른 시작

Stoneford 를 실행하는 두 가지 경로 — 이미 가지고 있는 쪽을 선택하세요.

### A. WorldLines 단독으로

```bash
# 1. WorldLines 설치 (엔진 코드명: neonrp)
curl -LsSf https://worldlines.gg/install.sh | sh    # macOS / Linux
# Windows PowerShell:
#   irm https://worldlines.gg/install.ps1 | iex

# 2. 새 디렉토리에 stoneford 스캐폴드
mkdir my-stoneford && cd my-stoneford
neonrp game new --template stoneford

# 3. TUI 를 띄우고 플레이 시작
neonrp tui
```

프롬프트에 `look around` 를 입력하고 Enter. orchestrator 가 턴을 적절한
도메인 agent 로 라우팅하고, 내레이션이 나오고, 새 이벤트가 로그에 기록됩니다.
이것이 첫 에이전틱 턴입니다.

### B. Claude Code 로 바로 열기

**이미 [Claude Code](https://claude.com/claude-code) 가 있다면?** 이 템플릿은
WorldLines 설치 없이 Claude Code 안에서 그대로 돌아가도록 만들어졌습니다.

```bash
# 스타터를 클론 (또는 이 폴더를 어떤 프로젝트에든 복사)
git clone https://github.com/LudicDynamics/stoneford-worldlines.git
cd stoneford-worldlines

# Claude Code 에서 열기
claude
```

그리고 입력:

```
@orchestrator 开始游戏
```

(영어로는: `@orchestrator start the game`.) Claude Code 가
`.claude/agents/*.md` 를 자동으로 읽고 — `orchestrator` 가 도메인 agent 를
오케스트레이트하며 `game/` 아래의 파일을 읽고 쓰고, 장면을 진행합니다.

<p align="center">
  <img src="./assets/gameplay.jpg" alt="Stoneford 플레이 — 안개 속에서 Stone Ford Town에 도착" width="720" />
</p>

---

## Agent 아키텍처 (4 계층 · 10 agent)

```
  L1 ── ROUTER
        orchestrator                (only agent w/ triggers)

  L2 ── DOMAIN  (mutate world files)
        town-agent      dungeon-agent      world-builder
        combat-referee  rules-referee

  L3 ── PERSPECTIVE  (compose narrative)
        npc-mind        story-narrative

  L4 ── WORLD-TIME  (tick & drift)
        clock-keeper    world-evolution

   player ─▶ orchestrator
             │   └─▶ domain      (write files)
             │        ├─▶ perspective (read files → narrative)
             │        └─▶ world-time  (advance clock, drift NPCs)
             │
             ▼
           commit · return text
```

동일한 10-agent 구성이 두 백엔드에서 실행됩니다: **Claude Code** (Claude)
와 **WorldLines 런타임** (Qwen). WorldLines 경로는 Claude Code 경로보다
약 2–3× 빠르며, 내러티브 품질은 동등합니다. 턴 사이에도 세계는 계속
살아 움직입니다.

```
Layer 1  @orchestrator       — 메인 라우터; 모든 플레이어 입력이 먼저 도달
Layer 2  @town-agent         — 도시 상호작용 (NPC, 상점, 길드, 여관)
         @dungeon-agent      — 던전 탐험
         @combat-referee     — d20 전투 판정
         @rules-referee      — 주사위 / 기능 판정 (구조화 JSON, 산문 없음)
         @world-builder      — 월드맵 / lore 업데이트
Layer 3  @npc-mind           — 단일 NPC 의 사적 POV (한 번에 한 명)
         @story-narrative    — 드리프트 폴백 내레이션 (작성된 영역 밖)
Layer 4  @clock-keeper       — 세계 시계 + 예약 이벤트 (JSON 만)
         @world-evolution    — 장기 세계 드리프트 (제안만, 쓰기 안 함)
```

**라우팅 규칙:** 트리거를 가지는 것은 `orchestrator` 뿐입니다. 다른 모든
agent 는 `orchestrator` 의 `task()` 도구를 통해서만 호출됩니다 — 이렇게
해야 장면이 일관되게 유지되고 이벤트 로그도 읽기 쉬워집니다.

| 온도     | Agents                                                                              | 이유              |
|----------|-------------------------------------------------------------------------------------|-------------------|
| 높음     | orchestrator · town-agent · dungeon-agent · npc-mind · story-narrative              | 내러티브 보이스   |
| 낮음(0.3) | combat-referee · rules-referee · clock-keeper · world-evolution · world-builder      | 결정론적 출력     |

---

## 월드 콘텐츠

| 종류      | ID   | 이름                   | 비고                                                          |
|-----------|------|------------------------|---------------------------------------------------------------|
| Town      | T001 | 石津镇 Stoneford        | 시작 지점. 길드 / 잡화점 / 여관 / 예배당 / 부두.              |
| Town      | T004 | 翠穴镇 Jadehollow       | 광산 도시. Deepvein 가문. 길드 타워 · 약초점 · 선술집.        |
| Dungeon   | D001 | 苔墓 Mossbarrow         | 4 개 방. 무언가가 안에서 속삭입니다.                          |
| Rules     | —    | d20 + 능력치 보정 + 상태 이상                                                                             |

플레이어는 Day 1, 가을 황혼, 16:30, Stoneford 북문에서 시작합니다.
Level 1 전사, HP 24/24, MP 8/8, 장비: longsword + 가죽갑옷.

---

## 디렉터리 구조

```
stoneford-worldlines/
├── CLAUDE.md                 # Claude Code 진입점 (@orchestrator 지시서)
├── README.md                 # 이 파일
├── runtime_contract.json     # 엔진 런타임 선언
├── agents/                   # 10 개 WorldLines agent
│   ├── orchestrator/         # Layer 1 — 메인 라우터
│   ├── town-agent/           # Layer 2 — 도메인 agents
│   ├── dungeon-agent/
│   ├── combat-referee/
│   ├── rules-referee/
│   ├── world-builder/
│   ├── npc-mind/             # Layer 3 — 시점 agents
│   ├── story-narrative/
│   ├── clock-keeper/         # Layer 4 — 세계 시간 agents
│   └── world-evolution/
└── game/
    ├── meta/                 # run_state, game-start, schedule, flags
    ├── player/               # profile, stats, inventory, wallet, journal
    ├── towns/                # T001 Stoneford, T004 Jadehollow
    ├── dungeons/D001/        # Mossbarrow
    ├── lore/                 # story, notes, rumors
    ├── rules/                # d20 전투 규칙
    └── world/                # world_map.json
```

**주요 파일:**

| 파일                              | 용도                                             |
|-----------------------------------|--------------------------------------------------|
| `game/meta/run_state.json`        | 월드 상태 (위치, 시간, 마지막 행동)              |
| `game/meta/game-start.json`       | 첫 턴에 재생되는 Day-1 오프닝 장면               |
| `game/player/stats.json`          | HP / MP / 능력치 / 보정                          |
| `agents/orchestrator/system.md`   | 최상위 라우터 프롬프트                            |

---

## 주사위 시스템

모든 불확실한 행동은 `d20 + 능력치 보정 + 상황 보정 ≥ DC` 로 해결됩니다.

| 판정        | 능력치   | 보통 DC    |
|-------------|----------|------------|
| 설득        | CHA      | 12–20      |
| 은신        | DEX      | 14–18      |
| 운동        | STR      | 12–18      |
| 감지        | DEX      | 12–18      |
| 약초학      | INT      | 12–16      |
| 자물쇠 따기 | DEX      | 15–20      |

Natural 20 = 대성공. Natural 1 = 대실패.

---

## 하우스 룰

- 불확실한 행동은 모두 d20 판정. 보이지 않는 벽 없음.
- HP 0 = 사망. 자동 부활 없음.
- NPC 호감도는 영구적. 한 번 어긋나면 기억합니다.
- 모든 퀘스트에는 파벌 동기가 있음. "순수하게 선한 행동" 은 없음.
- 플레이어 자유가 우선 — 첫 번째 옵션은 항상 "자유롭게 행동". 제안은 참고용일 뿐.

---

## Stoneford 확장

1. **도시 추가**: `game/towns/T00X/` 생성 후 `town.json`, `npcs.json`, `quests.json`, `facilities/*.json` 배치.
2. **던전 추가**: `game/dungeons/D00X/` 생성 후 `dungeon.json`, `layout.json`, `rooms/*.json` 배치.
3. **NPC 추가**: `npcs.json` 에 추가. 다음 턴에서 `orchestrator` 가 추론에 반영합니다.
4. **몬스터 추가**: 조우 테이블 확장 또는 `game/world/` 에 `species.md` 추가.
5. **유물 추가**: `game/world/relics.md` 생성 (A → F 등급 — 확장 팩에 완성 예제 있음).

---

## Stoneford 월드 확장 팩 — `stoneford-worldlines-full.zip`

이 폴더는 **스타터** Stoneford 입니다 — 2 도시, 1 던전, 소수의 입문
퀘스트. agent 하네스는 동일하고, 하네스 사용법을 배우기에 딱 맞는 분량입니다.

동봉된 `stoneford-worldlines-full.zip` 은 같은 Stoneford 의
**월드 콘텐츠 확장 팩** 입니다. 엔진도 agent 도 동일하고, 세계만 훨씬
커집니다. Cold Release 예고편에 등장하는 캠페인 규모 세계가 필요하면
압축을 풀어주세요:

```bash
unzip stoneford-worldlines-full.zip
cd stoneford-worldlines-full
neonrp tui        # 또는: claude
```

**스타터 대비 확장 팩이 추가하는 콘텐츠:**

| 콘텐츠     | 스타터 (이 폴더)                        | 풀 (`.zip`)                                                                        |
|------------|------------------------------------------|------------------------------------------------------------------------------------|
| Agent      | 10 agent, 4 계층                         | 동일한 10-agent 레이아웃 (`npc-mind` 프롬프트가 더 풍부)                             |
| 도시       | 2 (T001 Stoneford, T004 Jadehollow)      | **5** — T002 Windrest Keep, T003 Wolfsjaw Pass, T005 Mistmere 추가                 |
| 던전       | 1 (Mossbarrow, 4 방)                     | 1 플래그십 (Mossbarrow) + `dungeons-design/` 시드 스텁                              |
| 퀘스트     | 2 도시 분량의 입문 퀘스트                 | **명명된 28 퀘스트 + 보스 1**(*The Drowned Thing*)                                   |
| 스토리     | 오프닝 장면                              | **교차하는 3 개 백라인** — 5 도시에 걸친 정치 음모                                   |
| 파벌       | —                                        | **5 파벌** — Crown's Hand / The Faith / Free North / The Union / The Fog           |
| 가문       | —                                        | Stoneford 세 House (Blackwater / Ashford / Deepvein)                                |
| Lore       | story / notes / rumors                   | +`foundation/` 신화, +`beyond-the-world.md`                                         |
| World      | `world_map.json`                         | +`cultures/`, +`species.md`, +`relics.md`, +`items.md`                              |
| 톤         | 클래식 판타지 TRPG                       | 로우 판타지 정치 음모 × 음울한 몬스터 헌터 × 동양적 이계 형이상학                    |

agent 레이아웃은 동일합니다 — 확장되는 것은 Stoneford 의 세계 콘텐츠뿐입니다.

---

## 라이선스

AGPL-3.0-only. [LICENSE](./LICENSE) 참조.

Copyright © 2026 Ludic Dynamics.

선행 프로젝트 `llm-rpg-starter` 에서의 드리프트. 내러티브 디자인은
**nikoloside**, **redoctober**. d20 규칙은 오픈 SRD 관례를 따릅니다.

## 링크

- **Template repo**: [LudicDynamics/stoneford-worldlines](https://github.com/LudicDynamics/stoneford-worldlines)
- **Engine**: [worldlines.gg](https://worldlines.gg) · [docs](https://docs.worldlines.gg) · [GitHub](https://github.com/LudicDynamics/WorldLines)
- **Discord**: [discord.gg/HJYWbdqWrE](https://discord.gg/HJYWbdqWrE)
- **Policies**: [SECURITY.md](./SECURITY.md) · [CODE_OF_CONDUCT.md](./CODE_OF_CONDUCT.md)
- **Full Stoneford**: `stoneford-worldlines-full.zip` — 월드 콘텐츠 확장 팩: +3 도시, 28 퀘스트, 3 정치 백라인 (위 섹션 참조)
