# Stoneford — WorldLines スターターワールド

![Stoneford](./assets/s1bg.png)

> [WorldLines](https://worldlines.gg) エンジン向けの灰霧の北方河港。古典ファンタジー TRPG · d20 ダイス · 10-agent オーケストレーター。

**Language:** [English](./README.md) · [简体中文](./README.zh.md) · [日本語](./README.ja.md) · [한국어](./README.ko.md)

> **免責事項:** Stoneford は **nikoloside**、**redoctober**、そして Claude が共同で執筆したオリジナル シナリオで、東洋の世界観と西洋中世の魔法を融合させたものです。類似点がありましたら、修正・更新のため原作者までご連絡ください。

---

## クイックスタート

Stoneford を動かす 2 つの経路 — 手元にある方を選んでください。

### A. WorldLines 単体で

```bash
# 1. WorldLines をインストール（エンジンコード名: neonrp）
curl -LsSf https://worldlines.gg/install.sh | sh    # macOS / Linux
# または Windows PowerShell:
#   irm https://worldlines.gg/install.ps1 | iex

# 2. 新しいディレクトリに stoneford を展開
mkdir my-stoneford && cd my-stoneford
neonrp game new --template stoneford

# 3. TUI を起動してプレイ
neonrp tui
```

プロンプトで `look around` と入力して Enter。orchestrator がターンを
適切なドメイン agent にルーティングし、ナレーションが出て、新しい
イベントがログに記録されます。これが最初のエージェンティック ターンです。

### B. Claude Code で直接開く

**すでに [Claude Code](https://claude.com/claude-code) をお持ちなら？**
このテンプレートは WorldLines のインストール無しで Claude Code 内で
そのまま動くように作られています。

```bash
# スターターをクローン（またはこのフォルダを任意のプロジェクトにコピー）
git clone https://github.com/LudicDynamics/WorldLines-stoneford.git
cd WorldLines-stoneford

# Claude Code で開く
claude
```

そして入力:

```
@orchestrator 开始游戏
```

(英語なら: `@orchestrator start the game`。) Claude Code が
`.claude/agents/*.md` を自動で読み込み — `orchestrator` がドメイン
agent をオーケストレートし、`game/` 下のファイルを読み書きし、
シーンを進めます。

<p align="center">
  <img src="./assets/gameplay.jpg" alt="Stoneford ゲームプレイ — 霧のなかで Stone Ford Town に辿り着く" width="720" />
</p>

---

## Agent アーキテクチャ（4 層 · 10 agent）

```
Layer 1  @orchestrator       — メインルーター; 全プレイヤー入力が最初に到達
Layer 2  @town-agent         — 街の相互作用（NPC・店・ギルド・宿）
         @dungeon-agent      — ダンジョン探索
         @combat-referee     — d20 戦闘裁定
         @rules-referee      — ダイス・技能裁定（構造化 JSON、散文なし）
         @world-builder      — ワールドマップ / lore 更新
Layer 3  @npc-mind           — 単体 NPC の私的 POV（一度に 1 人だけ）
         @story-narrative    — ドリフト時の補完ナレーション（作成済み領域の外）
Layer 4  @clock-keeper       — 世界時計 + スケジュールイベント（JSON のみ）
         @world-evolution    — 長期世界ドリフト（提案のみ、書き込みなし）
```

**ルーティングルール:** トリガーを持つのは `orchestrator` のみ。他の
全 agent は `orchestrator` の `task()` ツール経由でしか呼べません —
これによりシーンの一貫性とイベントログの読みやすさが保たれます。

| 温度     | Agents                                                                              | 理由              |
|----------|-------------------------------------------------------------------------------------|-------------------|
| 高       | orchestrator · town-agent · dungeon-agent · npc-mind · story-narrative              | 物語の声          |
| 低 (0.3) | combat-referee · rules-referee · clock-keeper · world-evolution · world-builder      | 決定論的な出力    |

---

## ワールドコンテンツ

| 種別    | ID   | 名称                   | 備考                                                              |
|---------|------|------------------------|-------------------------------------------------------------------|
| 街      | T001 | 石津镇 Stoneford       | 開始地点。ギルド / 雑貨店 / 宿 / 礼拝堂 / 波止場。                |
| 街      | T004 | 翠穴镇 Jadehollow      | 鉱山街。Deepvein 氏族。ギルドタワー · 薬草店 · 酒場。             |
| ダンジョン | D001 | 苔墓 Mossbarrow      | 4 部屋。中で何かが囁いている。                                    |
| ルール  | —    | d20 + 能力修正 + 状態異常                                                                                 |

プレイヤー開始: Day 1、秋の黄昏、16:30、Stoneford 北門。
Level 1 戦士、HP 24/24、MP 8/8、装備: ロングソード + 革鎧。

---

## ディレクトリ構造

```
stoneford-worldlines/
├── CLAUDE.md                 # Claude Code エントリ（@orchestrator 指示書）
├── README.md                 # このファイル
├── runtime_contract.json     # エンジン ランタイム宣言
├── agents/                   # 10 WorldLines agent
│   ├── orchestrator/         # Layer 1 — メインルーター
│   ├── town-agent/           # Layer 2 — ドメイン agents
│   ├── dungeon-agent/
│   ├── combat-referee/
│   ├── rules-referee/
│   ├── world-builder/
│   ├── npc-mind/             # Layer 3 — 視点 agents
│   ├── story-narrative/
│   ├── clock-keeper/         # Layer 4 — 世界時間 agents
│   └── world-evolution/
└── game/
    ├── meta/                 # run_state / game-start / schedule / flags
    ├── player/               # profile / stats / inventory / wallet / journal
    ├── towns/                # T001 Stoneford、T004 Jadehollow
    ├── dungeons/D001/        # Mossbarrow
    ├── lore/                 # story / notes / rumors
    ├── rules/                # d20 戦闘ルール
    └── world/                # world_map.json
```

**主要ファイル:**

| ファイル                          | 用途                                              |
|-----------------------------------|---------------------------------------------------|
| `game/meta/run_state.json`        | ワールド状態（位置・時間・最後の行動）            |
| `game/meta/game-start.json`       | 最初のターンに再生される Day-1 オープニング       |
| `game/player/stats.json`          | HP / MP / 能力値 / 修正                           |
| `agents/orchestrator/system.md`   | 最上位ルーター プロンプト                          |

---

## ダイスシステム

不確実な行動はすべて `d20 + 能力修正 + 状況修正 ≥ DC` で解決されます。

| 判定      | 能力  | 典型的 DC  |
|-----------|-------|------------|
| 説得      | CHA   | 12–20      |
| 隠密      | DEX   | 14–18      |
| 運動      | STR   | 12–18      |
| 察知      | DEX   | 12–18      |
| 薬草学    | INT   | 12–16      |
| 鍵開け    | DEX   | 15–20      |

ナチュラル 20 = 大成功。ナチュラル 1 = 大失敗。

---

## ハウスルール

- 不確実な行動は全て d20 判定。見えない壁なし。
- HP 0 = 死亡。自動復活なし。
- NPC の好感度は永続。怒らせた相手は覚えている。
- 全クエストに派閥の動機あり。「純粋な善行」は存在しない。
- プレイヤーの自由が最優先 — 第一選択肢は常に「自由に行動」。提案は参考まで。

---

## Stoneford を拡張する

1. **街を追加**: `game/towns/T00X/` を作成し、`town.json`、`npcs.json`、`quests.json`、`facilities/*.json` を配置。
2. **ダンジョンを追加**: `game/dungeons/D00X/` を作成し、`dungeon.json`、`layout.json`、`rooms/*.json` を配置。
3. **NPC を追加**: `npcs.json` に追記。次のターンで `orchestrator` が取り込んで推論します。
4. **モンスターを追加**: 遭遇表を拡張、または `game/world/` に `species.md` を追加。
5. **遺物を追加**: `game/world/relics.md` を作成（A → F 等級 — 拡張パックに完成例あり）。

---

## Stoneford 世界拡張パック — `stoneford-worldlines-full.zip`

このフォルダは **スターター** 版 Stoneford — 2 街、1 ダンジョン、
入門クエスト少数。agent ハーネスは同じで、ハーネスを学ぶのに丁度良い量です。

同梱の `stoneford-worldlines-full.zip` は同じ Stoneford の
**世界コンテンツ拡張パック** です。エンジンも agent も同じで、世界が
ずっと大きくなります。Cold Release トレーラーに登場するキャンペーン
規模の世界が欲しくなったら解凍してください:

```bash
unzip stoneford-worldlines-full.zip
cd stoneford-worldlines-full
neonrp tui        # あるいは: claude
```

**スターターに対して拡張される内容:**

| コンテンツ   | スターター（本フォルダ）                 | フル（`.zip`）                                                                      |
|--------------|------------------------------------------|-------------------------------------------------------------------------------------|
| Agent        | 10 agent、4 層                           | 同じ 10-agent レイアウト（`npc-mind` プロンプトがより豊富）                          |
| 街           | 2 (T001 Stoneford、T004 Jadehollow)      | **5** — T002 Windrest Keep、T003 Wolfsjaw Pass、T005 Mistmere を追加                |
| ダンジョン   | 1 (Mossbarrow、4 部屋)                   | 1 旗艦（Mossbarrow）+ `dungeons-design/` シード stub                                |
| クエスト     | 2 街分の入門クエスト                      | **名前付き 28 クエスト + ボス 1**（*The Drowned Thing*）                              |
| 物語         | オープニング演出                         | **交錯する 3 本のバックライン** — 5 街にまたがる政治陰謀                              |
| 派閥         | —                                        | **5 派閥** — Crown's Hand / The Faith / Free North / The Union / The Fog            |
| 家系         | —                                        | Stoneford 三家（Blackwater / Ashford / Deepvein）                                    |
| Lore         | story / notes / rumors                   | +`foundation/` 神話、+`beyond-the-world.md`                                         |
| World        | `world_map.json`                         | +`cultures/`、+`species.md`、+`relics.md`、+`items.md`                              |
| トーン       | 古典ファンタジー TRPG                    | 低ファンタジー政治陰謀 × 冷厳モンスターハント × 東洋異世界形而上                       |

agent アーキテクチャは同じ — 拡張されるのは Stoneford の世界内容だけです。

---

## ライセンス

AGPL-3.0-only。[LICENSE](./LICENSE) を参照。

Copyright © 2026 Ludic Dynamics.

先行プロジェクト `llm-rpg-starter` からのドリフト。ナラティブ設計は
**nikoloside**、**redoctober**。d20 ルールはオープン SRD 慣習に準拠します。

## 関連リンク

- **Engine**: [worldlines.gg](https://worldlines.gg) · [docs](https://docs.worldlines.gg) · [GitHub](https://github.com/LudicDynamics/WorldLines)
- **Discord**: [discord.gg/HJYWbdqWrE](https://discord.gg/HJYWbdqWrE)
- **Full Stoneford**: `stoneford-worldlines-full.zip` — 世界コンテンツ拡張パック: +3 街、28 クエスト、3 本の政治バックライン（上の節を参照）
