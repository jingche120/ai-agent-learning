# Claude Code 與 Codex 的專案設定對照

## 0. 先釐清：這裡的 ChatGPT 端其實是 Codex 端

目前直接操作這個 repository 的 OpenAI coding agent 是 Codex。

因此更精確的對照是：

```text
Claude Code ↔ Codex
```

一般沒有開啟這個 repository 的 ChatGPT 對話，不會因為本機存在 `AGENTS.md` 就自動讀到專案規則。

## 1. 兩邊的核心檔案對照

| 功能 | Claude Code | Codex |
|---|---|---|
| 專案長期規則 | `CLAUDE.md` | `AGENTS.md` |
| 專案 Skills | `.claude/skills/` | `.agents/skills/` |
| 專案設定 | `.claude/settings.json` | `.codex/config.toml`（選用） |
| Skill 核心檔案 | `SKILL.md` | `SKILL.md` |
| 明確呼叫 Skill | `/card` | `$card` |
| 外部工具整合 | MCP | MCP／Connector |
| 平台專用補充 | Claude frontmatter 與設定 | `agents/openai.yaml`（選用） |

重要：正確檔名是 `AGENTS.md`，不是 `agent.md`。

## 2. `CLAUDE.md` 對應 `AGENTS.md` 嗎？

對，兩者在概念上互相對應，都是 AI 開始工作前讀取的專案說明書。

- `CLAUDE.md` 提供 Claude Code 的長期專案指令。
- `AGENTS.md` 提供 Codex 的長期專案指令。

Claude Code 會從 `CLAUDE.md` 載入專案指令；Codex 則會從 repository 根目錄往目前工作目錄尋找 `AGENTS.md`。[Claude Code 設定說明](https://code.claude.com/docs/en/configuration)、[Codex AGENTS.md 說明](https://learn.chatgpt.com/docs/agent-configuration/agents-md)

兩份文件應該對齊核心原則，但不必逐字相同。

### 應該對齊的內容

- 專案目標
- 使用繁體中文
- 使用者親手操作
- 一次只引導一個步驟
- 寫入前先查證
- 文件格式與寫作風格
- Git 操作權責
- 修改後如何驗證

### 應該分開的內容

- Claude 的 Plan mode
- `.claude/settings.json`
- Claude 專用工具或指令
- Codex 的 `$skill` 呼叫方式
- `.codex/config.toml`
- Codex／ChatGPT 專用的 Skill metadata

所以兩者是「目標與原則對齊，平台細節分開」。

## 3. 雙邊都有 Skills 嗎？

有，而且兩邊都使用以 `SKILL.md` 為核心的 Agent Skills 格式。

目前專案實際結構是：

```text
.claude/
├── settings.json
└── skills/
    ├── card/
    │   └── SKILL.md
    └── new-note/
        └── SKILL.md

.agents/
└── skills/
    ├── card/
    │   └── SKILL.md
    └── new-note/
        └── SKILL.md
```

Claude Code 從 `.claude/skills/<名稱>/SKILL.md` 載入專案 Skill，可以由 Claude 自動選擇，也可以使用 `/skill-name` 呼叫。[Claude Code Skills](https://code.claude.com/docs/en/slash-commands)

Codex 從 `.agents/skills/<名稱>/SKILL.md` 載入專案 Skill，可以根據 description 自動選擇，也可以使用 `$skill-name` 明確呼叫。[Codex Skills](https://learn.chatgpt.com/docs/build-skills)

目前兩邊都有：

| 用途 | Claude Code | Codex |
|---|---|---|
| 對話轉卡片 | `/card` | `$card` |
| 新增學習筆記 | `/new-note` | `$new-note` |

## 4. 兩邊的 Skill 會自動同步嗎？

不會。

```text
.claude/skills/card/SKILL.md
```

和：

```text
.agents/skills/card/SKILL.md
```

是兩個獨立檔案。修改其中一份，不會自動修改另一份。

目前兩邊的 Skill 已經有差異：

- Codex 版明確要求使用者確認後才寫檔。
- Codex 版使用 `$card`。
- Claude 版使用 `/card`。
- Codex 版不綁定 Claude 的設定路徑或工具名稱。
- Codex 版 `new-note` 已加入跨平台官方查證流程。

這些差異有些是刻意的，但共同工作流程改變時，仍需檢查另一份是否要同步。

## 5. `settings.json` 對應 `config.toml` 嗎？

概念上相近，但不是完全一對一。

| Claude Code | Codex |
|---|---|
| `.claude/settings.json` | `.codex/config.toml` |
| 設定權限、語言、工具行為等 | 設定 sandbox、模型、MCP、hooks 等 |
| JSON | TOML |

你目前已有：

```text
.claude/settings.json
```

內容設定 Claude 使用 Plan mode 與中文。

目前尚未建立：

```text
.codex/config.toml
```

這不是缺失。現在的核心需求已經由 `AGENTS.md` 表達，不需要為了形式對稱而建立空的 Codex 設定檔。

## 6. 目前專案的載入流程

### Claude Code

```text
開啟 repository
→ 載入 CLAUDE.md
→ 套用 .claude/settings.json
→ 發現 .claude/skills/
→ 根據需求自動使用 Skill，或由你輸入 /card
```

### Codex

```text
開啟 repository
→ 載入 AGENTS.md
→ 發現 .agents/skills/
→ 根據需求自動使用 Skill，或由你輸入 $card
→ 依目前工作階段的 sandbox 與權限執行
```

## 7. 之後修改規則時怎麼判斷？

| 想修改的內容 | 應放的位置 |
|---|---|
| 只適用這次對話 | 直接在對話說明 |
| 每次進入專案都要遵守 | `CLAUDE.md` 與 `AGENTS.md` |
| 可重複執行的特定流程 | 雙邊的 `SKILL.md` |
| Claude 的權限或操作模式 | `.claude/settings.json` |
| Codex 的執行環境設定 | `.codex/config.toml` |
| 外部服務或資料工具 | MCP／Connector |

## 檢查清單

```markdown
- [x] 已建立 CLAUDE.md
- [x] 已建立 AGENTS.md
- [x] Claude Code 已有 card Skill
- [x] Claude Code 已有 new-note Skill
- [x] Codex 已有 card Skill
- [x] Codex 已有 new-note Skill
- [x] Codex Skills 已調整成平台相容版本
- [ ] 確認 Claude Skills 是否需要同步共同規則
- [ ] 建立共用進度與驗收標準
- [ ] 建立第一次 Git commit
- [ ] 設定 remote
- [ ] push 到遠端 repository
```

這個專案目前的核心架構是：`CLAUDE.md` 對應 `AGENTS.md`，雙邊都有 Skills，但檔案位置、呼叫語法與平台設定不同，而且兩份 Skill 不會自動同步。
