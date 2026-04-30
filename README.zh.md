# Stoneford — WorldLines 起步世界

![Stoneford](./assets/s1bg.png)

> 为 [WorldLines](https://worldlines.gg) 引擎准备的灰雾北方河港。经典奇幻 TRPG · d20 骰子 · 10-agent 编排。

**Language:** [English](./README.md) · [简体中文](./README.zh.md) · [日本語](./README.ja.md) · [한국어](./README.ko.md)

<p align="center">
  <a href="https://youtu.be/QARKAA8bXDk">
    <img src="https://img.youtube.com/vi/QARKAA8bXDk/maxresdefault.jpg" alt="WorldLines · 预告片" width="640"/>
  </a>
</p>
<p align="center"><em>▶ 在 YouTube 观看预告片</em></p>

<table align="center">
  <tr>
    <td width="50%" align="center"><img src="./assets/cards/D1%20%E7%B1%B3%E6%8B%89%E8%B5%8A%E8%B4%A6%20%28deck%20%C2%B7%20ZH%29.png" alt="米拉赊账 — 故事卡" /></td>
    <td width="50%" align="center"><img src="./assets/cards/D2%20%E8%80%81%E9%9F%A6%E7%BA%A6%E4%BD%A0%20%28deck%20%C2%B7%20ZH%29.png" alt="老韦约你 — 故事卡" /></td>
  </tr>
  <tr>
    <td width="50%" align="center"><img src="./assets/cards/D3%20Gameplay%20%28deck%20%C2%B7%20ZH%29.png" alt="游戏流程" /></td>
    <td width="50%" align="center"><img src="./assets/cards/D4%204%20Worlds%20%28deck%20%C2%B7%20ZH%29.png" alt="四个世界" /></td>
  </tr>
</table>

> **引擎：** Stoneford 跑在 [WorldLines](https://github.com/LudicDynamics/WorldLines) 引擎上。引擎本身、运行时和完整文档 → [LudicDynamics/WorldLines](https://github.com/LudicDynamics/WorldLines)。

> **免责声明：** Stoneford 是由 **nikoloside**、**redoctober** 与 Claude 共同创作的原创剧本，融合了东方世界观与西方中世纪魔法。如有任何相似之处，欢迎联系原作者以便我们修改和更新。

---

## 快速开始

两种方式运行 Stoneford，挑你已经有的那个。

### A. 用 WorldLines（单机）

```bash
# 1. 安装 WorldLines（引擎代号：neonrp）
curl -LsSf https://worldlines.gg/install.sh | sh    # macOS / Linux
# 或 Windows PowerShell：
#   irm https://worldlines.gg/install.ps1 | iex

# 2. 把 stoneford 脚手架到新目录
mkdir my-stoneford && cd my-stoneford
neonrp game new --template stoneford

# 3. 启动 TUI 开始玩
neonrp tui
```

在提示符处输入 `look around` 回车。orchestrator 会把这一回合路由到合适的
domain agent；narration 出现；一个新事件写入日志——这就是你的第一回合。

### B. 用 Claude Code（直接打开）

**已经装了 [Claude Code](https://claude.com/claude-code)？** 本模板不需要
安装 WorldLines 就能在 Claude Code 里直接跑起来。

```bash
# 克隆 starter（或把这个文件夹复制进任何项目）
git clone https://github.com/LudicDynamics/stoneford-worldlines.git
cd stoneford-worldlines

# 用 Claude Code 打开
claude
```

然后输入：

```
@orchestrator 开始游戏
```

Claude Code 会自动读取 `.claude/agents/*.md` —— `orchestrator` 负责编排
domain agents，读写 `game/` 下的文件，推动剧情。

<p align="center">
  <img src="./assets/gameplay.jpg" alt="石津镇游玩画面 —— 雾中抵达石津镇关口" width="720" />
</p>

---

## Agent 架构（4 层 · 10 个 agent）

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

同一套 10-agent 框架可以跑在两个后端：**Claude Code**（Claude）与
**WorldLines 运行时**（Qwen）。WorldLines 路径比 Claude Code 路径快约
2–3×，叙事质量相当。回合之间，世界依然在前进。

```
Layer 1  @orchestrator       — 主路由；所有玩家输入先进这里
Layer 2  @town-agent         — 城镇交互（NPC、商店、公会、旅店）
         @dungeon-agent      — 地下城探索
         @combat-referee     — d20 战斗裁决
         @rules-referee      — 骰子 / 技能裁决（结构化 JSON，不写叙事）
         @world-builder      — 世界地图 / lore 更新
Layer 3  @npc-mind           — 单体 NPC 私人视角（一次只能是一位）
         @story-narrative    — 漂移兜底叙事（玩家走出已写内容时）
Layer 4  @clock-keeper       — 世界时钟 + 定时事件（仅 JSON）
         @world-evolution    — 长期世界演变（只提议不写入）
```

**路由规则：** 只有 `orchestrator` 有 trigger。其它 agent 只能通过
`orchestrator` 的 `task()` 工具调用——这样场景才保持连贯，事件日志才可读。

| 温度     | Agents                                                                              | 原因            |
|----------|-------------------------------------------------------------------------------------|-----------------|
| 高       | orchestrator · town-agent · dungeon-agent · npc-mind · story-narrative              | 叙事声音        |
| 低 (0.3) | combat-referee · rules-referee · clock-keeper · world-evolution · world-builder      | 确定性输出      |

---

## 世界内容

| 类型    | ID   | 名称                   | 备注                                                          |
|---------|------|------------------------|---------------------------------------------------------------|
| 城镇    | T001 | 石津镇 Stoneford        | 起始地点。公会 / 杂货店 / 旅店 / 教堂 / 码头。                |
| 城镇    | T004 | 翠穴镇 Jadehollow       | 矿业城镇。Deepvein 氏族。公会塔 · 草药店 · 酒馆。             |
| 地下城  | D001 | 苔墓 Mossbarrow         | 4 个房间。里面有什么在低语。                                  |
| 规则    | —    | d20 + 能力修正 + 状态效果                                                                             |

玩家起始：Day 1，秋日黄昏，16:30，Stoneford 北门。
Level 1 战士，HP 24/24，MP 8/8，装备：长剑 + 皮甲。

---

## 目录结构

```
stoneford-worldlines/
├── CLAUDE.md                 # Claude Code 入口（@orchestrator 指令）
├── README.md                 # 本文件
├── runtime_contract.json     # 引擎运行时声明
├── agents/                   # 10 个 WorldLines agent
│   ├── orchestrator/         # Layer 1 — 主路由
│   ├── town-agent/           # Layer 2 — domain agents
│   ├── dungeon-agent/
│   ├── combat-referee/
│   ├── rules-referee/
│   ├── world-builder/
│   ├── npc-mind/             # Layer 3 — 视角 agents
│   ├── story-narrative/
│   ├── clock-keeper/         # Layer 4 — 世界时间 agents
│   └── world-evolution/
└── game/
    ├── meta/                 # run_state、game-start、schedule、flags
    ├── player/               # profile、stats、inventory、wallet、journal
    ├── towns/                # T001 Stoneford、T004 Jadehollow
    ├── dungeons/D001/        # Mossbarrow
    ├── lore/                 # story、notes、rumors
    ├── rules/                # d20 战斗规则
    └── world/                # world_map.json
```

**关键文件：**

| 文件                              | 用途                                              |
|-----------------------------------|---------------------------------------------------|
| `game/meta/run_state.json`        | 世界状态（位置、时间、上一次动作）                |
| `game/meta/game-start.json`       | 第一回合播放的 Day-1 开场                         |
| `game/player/stats.json`          | HP / MP / 属性 / 修正                             |
| `agents/orchestrator/system.md`   | 顶层路由 prompt                                   |

---

## 骰子系统

所有不确定的行为都以 `d20 + 属性修正 + 情境修正 ≥ DC` 判定。

| 检定      | 属性  | 常见 DC    |
|-----------|-------|------------|
| 说服      | CHA   | 12–20      |
| 潜行      | DEX   | 14–18      |
| 运动      | STR   | 12–18      |
| 察觉      | DEX   | 12–18      |
| 草药学    | INT   | 12–16      |
| 开锁      | DEX   | 15–20      |

自然 20 = 大成功。自然 1 = 大失败。

---

## 家规

- 所有不确定的行为都以 d20 判定。没有隐形墙。
- HP 0 = 死亡。不自动复活。
- NPC 好感度是永久的。惹到谁，谁就会记住。
- 每个任务都有派系动机。没有"纯粹的善行"。
- 玩家自由优先——第一选项永远是"自由行动"。建议仅供参考。

---

## 扩展 Stoneford

1. **加城镇**：建 `game/towns/T00X/`，放入 `town.json`、`npcs.json`、`quests.json`、`facilities/*.json`。
2. **加地下城**：建 `game/dungeons/D00X/`，放入 `dungeon.json`、`layout.json`、`rooms/*.json`。
3. **加 NPC**：追加到 `npcs.json`。下一回合 `orchestrator` 会开始把他们纳入推理。
4. **加怪物**：扩展遭遇表，或在 `game/world/` 下添加 `species.md`。
5. **加遗物**：创建 `game/world/relics.md`（A → F 分级——参考扩展包里已填好的范例）。

---

## Stoneford 世界扩写包 — `stoneford-worldlines-full.zip`

本文件夹是 **起步版** Stoneford —— 2 座城镇、1 个地下城、少量任务。
agent harness 完全相同，分量刚好够你摸熟。

附带的 `stoneford-worldlines-full.zip` 是同一个 Stoneford 的
**世界内容扩写包**：引擎相同，agent 相同，只是世界规模大得多。
当你想体验 Cold Release 预告片那套战役规模设定时，解压即可：

```bash
unzip stoneford-worldlines-full.zip
cd stoneford-worldlines-full
neonrp tui        # 或：claude
```

**扩写包相较起步版增加的内容：**

| 内容   | 起步版（本文件夹）                    | 完整版（`.zip`）                                                                |
|--------|----------------------------------------|---------------------------------------------------------------------------------|
| Agent  | 10 个 agent，4 层                      | 相同的 10-agent 布局（`npc-mind` prompt 更丰富）                                |
| 城镇   | 2 座（T001 Stoneford、T004 Jadehollow）| **5 座** — 新增 T002 Windrest Keep、T003 Wolfsjaw Pass、T005 Mistmere           |
| 地下城 | 1 座（Mossbarrow, 4 房间）             | 1 座旗舰（Mossbarrow）+ `dungeons-design/` 种子雏形                             |
| 任务   | 2 座城镇的入门任务组                    | **28 个具名任务 + 1 场 Boss**（*The Drowned Thing*）                              |
| 剧情   | 开场演出                               | **3 条交织主线** — 横跨 5 城的政治阴谋                                           |
| 派系   | —                                      | **5 大派系** — 王权之手 / 教团 / 自由北境 / 工会 / 迷雾                           |
| 家族   | —                                      | Stoneford 三大家族（Blackwater / Ashford / Deepvein）                           |
| Lore   | story / notes / rumors                 | +`foundation/` 神话、+`beyond-the-world.md`                                     |
| World  | `world_map.json`                       | +`cultures/`、+`species.md`、+`relics.md`、+`items.md`                           |
| 基调   | 经典奇幻 TRPG                          | 低魔政治阴谋 × 冷峻怪物狩猎 × 东方异世界玄学                                     |

Agent 架构没有变，扩的只是 Stoneford 的世界内容。

---

## 许可

AGPL-3.0-only。详见 [LICENSE](./LICENSE)。

Copyright © 2026 Ludic Dynamics。

漂移自更早的 `llm-rpg-starter` 项目。叙事设计由 **nikoloside**、
**redoctober** 完成。d20 规则参考开放 SRD 惯例。

## 链接

- **模板仓库**：[LudicDynamics/stoneford-worldlines](https://github.com/LudicDynamics/stoneford-worldlines)
- **引擎**：[worldlines.gg](https://worldlines.gg) · [docs](https://docs.worldlines.gg) · [GitHub](https://github.com/LudicDynamics/WorldLines)
- **Discord**：[discord.gg/HJYWbdqWrE](https://discord.gg/HJYWbdqWrE)
- **政策文档**：[SECURITY.md](./SECURITY.md) · [CODE_OF_CONDUCT.md](./CODE_OF_CONDUCT.md)
- **完整版 Stoneford**：`stoneford-worldlines-full.zip` — 内容扩写包：+3 座城镇、28 个任务、3 条政治主线（参见上方章节）
