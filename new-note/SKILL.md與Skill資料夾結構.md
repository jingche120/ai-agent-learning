# SKILL.md 與 Skill 資料夾結構

## 0. 先釐清：`SKILL.md` 是入口，不是整個 Skill

一個 Skill 是一個資料夾，必要入口是 `SKILL.md`，並可視需求加入其他資源：

```text
example-skill/
├── SKILL.md
├── references/
├── scripts/
└── assets/
```

`references/`、`scripts/`、`assets/` 都是選用資料夾。只有 `SKILL.md` 是必要檔案。

因此，只有一個 `SKILL.md` 也能構成完整 Skill，不需要為了符合形式而建立空資料夾。

## 1. Skill 如何被 Agent 使用

Skill 的使用過程可以分成三層：

```text
第一層：發現 Skill
Agent 先取得 name、description，以及 Skill 路徑
        ↓
第二層：載入核心指令
任務符合 description 後，載入完整 SKILL.md
        ↓
第三層：按需求使用資源
依 SKILL.md 的指示讀取 references、執行 scripts 或使用 assets
```

這種設計稱為漸進式載入。Agent 不需要在一開始就讀取所有 Skill 的完整內容，而是先根據簡短的 `description` 判斷哪個 Skill 與任務有關。

這樣可以減少不相關資訊占用 context，也能避免過多內容分散模型注意力。[OpenAI Skills 官方文件](https://learn.chatgpt.com/docs/build-skills)

## 2. `SKILL.md` 負責什麼

`SKILL.md` 是 Agent 使用 Skill 時讀取的核心指令。

它通常包含：

- Skill 的 `name` 與 `description`
- 什麼情況應該觸發
- 什麼情況不應該觸發
- Agent 必須遵循的核心流程
- 重要限制與停止條件
- 什麼時候讀取哪份參考資料
- 什麼時候執行哪個 script
- 什麼時候使用哪個 asset
- 如何驗證結果

基本結構如下：

```markdown
---
name: example-skill
description: 說明這個 Skill 在什麼情況下使用，以及不適用的範圍。
---

# Example Skill

## 流程

1. 確認輸入。
2. 需要詳細規格時，讀取 `references/spec.md`。
3. 執行 `scripts/validate.py` 驗證輸入。
4. 使用 `assets/template.md` 產生輸出。
5. 檢查結果並回報。
```

重要：`SKILL.md` 裡寫的流程是給 Agent 理解與判斷的指令，不是可以直接執行的程式。

## 3. `references/` 負責什麼

`references/` 放置完成任務時可能需要查閱的詳細知識。

適合放入：

- API 文件
- 格式規範
- 業務規則
- 詳細教學
- 長篇背景資料
- 錯誤碼說明
- 特定情況才需要的操作方式

例如：

```text
references/
├── heptabase-api.md
├── zettelkasten規則.md
└── 錯誤處理.md
```

`references/` 的內容不應在每次執行時全部讀取。`SKILL.md` 應指出什麼情況需要哪一份資料，例如：

```markdown
只有需要同步到 Heptabase 時，才讀取
`references/heptabase-api.md`。
```

### 重要：`references/` 不會取代核心流程

如果某條規則每次使用 Skill 都必須遵守，就應留在 `SKILL.md`。

如果內容只在特定情況下才有用，或篇幅很長，才適合移到 `references/`。

## 4. `scripts/` 負責什麼

`scripts/` 放置適合交給程式確定執行的操作。

適合放入：

- 格式轉換
- 資料驗證
- 批次處理
- 重複計算
- 檔案名稱正規化
- 機械式內容檢查

例如：

```text
scripts/
├── convert-links.js
└── validate-card.py
```

如果要把 Markdown 連結：

```markdown
[Agent 架構](./Agent架構.md)
```

轉成 Heptabase 格式：

```text
[[Agent 架構]]
```

這種規則明確、需要重複處理的工作，就適合寫成 script。

| Agent 指令 | Script |
|---|---|
| 需要理解情境與做判斷 | 依固定邏輯執行 |
| 寫在 `SKILL.md` | 放在 `scripts/` |
| 可能依結果調整下一步 | 相同輸入通常產生可預期結果 |
| 例如判斷哪些內容值得做卡片 | 例如批次轉換連結格式 |

Script 不會因為放進資料夾就自動執行。`SKILL.md` 必須說明何時執行、需要哪些輸入，以及如何驗證結果。

## 5. `assets/` 負責什麼

`assets/` 放置執行 Skill 時可以直接使用或複製的資源。

適合放入：

- Markdown 模板
- 文件範本
- 圖片
- 固定格式的輸出骨架
- 範例資料
- 樣式或設定範本

例如：

```text
assets/
└── card-template.md
```

模板內容可能是：

```markdown
# <完整主張>

<概念說明>

**連結**
→ <相關卡片>

**標籤**：#標籤

**來源**：<資料來源>
```

`assets/` 與 `references/` 的差別是：

| `references/` | `assets/` |
|---|---|
| 幫助 Agent 理解與判斷 | 提供可直接使用的資源 |
| 例如 API 文件、業務規則 | 例如模板、圖片、輸出骨架 |
| 內容通常是拿來閱讀 | 內容通常是拿來複製、填寫或產出 |

## 6. 如何判斷內容要放在哪裡

可以依序問：

1. Agent 是否需要靠這段內容判斷要不要觸發 Skill？  
   放入 frontmatter 的 `description`。

2. 每次執行都必須遵守嗎？  
   放入 `SKILL.md`。

3. 只有特定情況才需要查閱嗎？  
   放入 `references/`。

4. 是否是規則固定、可以重複執行的操作？  
   放入 `scripts/`。

5. 是否是要直接使用的模板或資源？  
   放入 `assets/`。

6. 是否有完全不同的觸發條件、目標與輸出？  
   這時才考慮拆成另一個 Skill。

## 7. 套用到目前的 `card` Skill

目前結構是：

```text
.agents/skills/card/
└── SKILL.md
```

這是一個只有核心指令的 Skill，並不是缺少必要結構。

目前的 `SKILL.md` 已包含：

- 觸發條件
- 與 `new-note` 的分工
- 原子卡片規則
- 卡片格式
- 存檔規則
- 使用者確認流程
- Heptabase 的後續處理

如果未來內容繼續增加，可以調整成：

```text
.agents/skills/card/
├── SKILL.md
├── references/
│   ├── zettelkasten規則.md
│   └── heptabase-api.md
├── scripts/
│   └── convert-links.js
└── assets/
    └── card-template.md
```

此時 `SKILL.md` 保留核心流程，並明確說明：

- 需要判斷卡片原子性時讀哪份 reference
- 需要同步 Heptabase 時讀哪份 API 文件
- 需要轉換連結時執行哪個 script
- 產生卡片時使用哪個模板

但目前不需要只為了看起來完整而拆分。當內容變長、只有部分情境才需要，或出現可程式化的重複操作時，再拆分才有實際價值。

## 8. 常見誤解

### 誤解一：一個流程只能屬於四個資料夾之一

不是。

整個 Skill 流程主要寫在 `SKILL.md`；`references/`、`scripts/`、`assets/` 是流程可能使用的輔助資源。

### 誤解二：內容很多就一定要拆成多個 Skill

不一定。

如果目標、觸發條件與輸出仍然相同，通常保留同一個 Skill，再把詳細內容移到輔助資料夾即可。

只有不同任務具有不同觸發條件與產物時，才適合拆成不同 Skill。

### 誤解三：放進 `scripts/` 就會自動執行

不會。

Agent 仍需要依 `SKILL.md` 判斷何時使用 script，並透過可用 Tool 執行與驗證。

### 誤解四：漸進式載入代表所有輔助檔案都會自動載入

不是。

Agent 先透過 metadata 發現 Skill，選中後讀取 `SKILL.md`。其他資料應由 `SKILL.md` 清楚指引，在任務真正需要時再讀取或使用。

## 檢查清單

```markdown
- [ ] Skill 資料夾內有必要的 SKILL.md
- [ ] SKILL.md frontmatter 包含 name 與 description
- [ ] description 清楚說明觸發與不適用範圍
- [ ] 核心流程保留在 SKILL.md
- [ ] 長篇且只在特定情況使用的資料放入 references/
- [ ] 固定、重複、可程式化的操作放入 scripts/
- [ ] 模板與可直接使用的資源放入 assets/
- [ ] SKILL.md 有說明何時讀取或使用輔助檔案
- [ ] 沒有為了形式建立不需要的空資料夾
- [ ] 修改後用正例、反例與故障案例驗證
```
