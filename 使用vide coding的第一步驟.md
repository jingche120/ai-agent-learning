# 使用 Vibe Coding 的第一步驟

每次要開一個新專案、用 Claude Code 開始 vibe coding 之前，照下面的順序做一輪。重點是把「你的規則」變成檔案存進專案裡，而不是每次開新對話都重新口頭交代一次。

## 0. 前置確認（只需做一次）

- VS Code 版本 1.98 以上
- 已安裝 Claude Code 擴充功能（`Cmd/Ctrl+Shift+X` 搜尋「Claude Code」）
- 已登入 Anthropic 帳號

## 1. 建立專案資料夾並進版控

```bash
mkdir my-project && cd my-project
git init
```

## 2. 在 VS Code 開啟資料夾，叫出 Claude Code

點右上角 Spark 圖示，或 `Cmd/Ctrl+Shift+P` 打「Claude Code」開啟面板。

## 3. 執行 `/init` 產生 CLAUDE.md

在 Claude Code 對話框輸入：

```
/init
```

Claude 會掃過程式碼庫，自動生成一份 `CLAUDE.md`，內容通常包含偵測到的建置/測試指令、目錄結構、它觀察到的程式風格。這份是「Claude 自己看出來的東西」，接下來第 4、5 步要補上「只有你知道，Claude 猜不到」的規則。

跑完之後**建議直接打開 CLAUDE.md 看一下**，但這裡有個概念上容易搞混的地方要先講清楚。

### 重要：CLAUDE.md 不是「AI 的記憶」

CLAUDE.md 比較像是你會去維護的「使用說明書」，不是 Claude 自己持續寫的筆記。`/init` 只是幫你打草稿，跑完之後 Claude 不會再自動回頭改寫這份檔案，每次新對話開始時它只是把這份檔案讀進去當參考。所以這次打開來看的理由是「檢查 AI 對你的程式碼庫的理解對不對、有沒有漏掉只有你知道的事」，不是「看 AI 記了什麼」。

如果你要的是「Claude 自己在過程中學到、記下來的東西」，那是另一個機制，叫 **auto memory**：

| | CLAUDE.md | auto memory |
|---|---|---|
| 誰寫的 | 你（`/init` 先打草稿，之後你自己補充） | Claude 自己根據你糾正過的事、它發現的慣例自動寫的 |
| 存放位置 | 專案根目錄 `CLAUDE.md` | `~/.claude/projects/<project>/memory/`，有一份 `MEMORY.md` 索引 |
| 你該做的事 | 主動打開、檢查、編輯 | 偶爾瀏覽，確認它記的東西沒有記錯 |

想同時看「目前載入了哪些 CLAUDE.md」跟「auto memory 記了什麼」，在對話框輸入：

```
/memory
```

這個指令會列出目前 session 載入的 CLAUDE.md / rules 檔案，也會給你一個連結直接打開 auto memory 資料夾。裡面全部是純文字 markdown，可以直接讀，也可以自己編輯或刪除。

## 4. 建立 `.claude/settings.json`：行為層的硬規則

這個檔案跟 CLAUDE.md 不一樣：CLAUDE.md 是寫給 Claude 看、讓它盡量遵守的「軟性說明」；`settings.json` 是系統層級的設定，會被強制套用，不受 Claude 自由判斷影響。你問的「修改前要我同意」跟「語言設定」都屬於這一層。

在專案根目錄新增 `.claude/settings.json`：

```json
{
  "$schema": "https://json.schemastore.org/claude-code-settings.json",
  "permissions": {
    "defaultMode": "default",
    "deny": [
      "Read(./.env)",
      "Read(./.env.*)",
      "Read(./secrets/**)"
    ]
  },
  "language": "chinese"
}
```

> `defaultMode` 的值要看 repo 性質決定：程式專案用 `"default"`、純知識庫用 `"plan"`。判準見第 7 節「該不該把 `defaultMode` 寫死成 `plan`？」。

逐項說明：

- **`permissions.defaultMode: "default"`**：每次 Claude 要修改檔案或執行指令前，都會先跳出來問你同不同意，不會自己默默改完。這其實也是系統的內建預設值，但寫死在這裡的好處是：以後不小心在某次對話裡切換成 `acceptEdits`（自動接受編輯）或 `auto`（自動模式）之後，下次重新開啟專案還是會回到「一定要問你」這個基準，不會卡在你忘記切回來的狀態。如果這個 repo 是**純知識庫**（筆記、文件、卡片盒），這裡建議直接改成 `"plan"`，理由見第 7 節。
- **`permissions.deny`**：順手把 `.env`、`secrets/` 這類敏感檔案排除，避免 Claude 不小心讀到或印出金鑰。這個跟「同意才能修改」不衝突，是另一層防護。
- **`language: "chinese"`**：直接設定 Claude 預設用中文回覆，連帶語音輸入辨識、自動產生的對話標題也會用中文，不用每次對話開頭都交代一次。

這個檔案請存在 `.claude/settings.json`（不是 `.claude/settings.local.json`）。兩者的差別：

| 檔案 | 是否進 git | 用途 |
|---|---|---|
| `.claude/settings.json` | 會 | 跟專案走，換電腦、之後找人合作都還在 |
| `.claude/settings.local.json` | 不會（Claude Code 會自動幫你加進 `.gitignore`） | 只在這台機器、只給你自己用 |

因為你提到「希望這些跟版本一起變動，一個人開發」，意思就是這份規則本身要被當作專案資產來管理，所以用 `.claude/settings.json`，讓它跟程式碼一起被 git 追蹤、一起進 commit history。之後如果你想調整「要不要每次都問我」這種規則，改這個檔案、commit 一次，未來回頭看 git log 就能知道這個規則是什麼時候、為什麼改的。

## 5. 在 CLAUDE.md 寫入專案目標與規則

CLAUDE.md 跟 settings.json 互補：settings.json 管「系統允不允許做這件事」，CLAUDE.md 管「Claude 該怎麼想、該往哪個方向做」。專案目標這種說明文字屬於後者。

打開 `/init` 產生的 `CLAUDE.md`（在專案根目錄，**不是** `CLAUDE.local.md`），補上類似這樣的內容：

```markdown
# 專案說明

## 專案目標
這個專案是要做____，主要使用者是____，核心要解決的問題是____。

## 技術堆疊
- 語言/框架：____
- 資料庫：____
- 部署方式：____

## 一定要遵守的規則
- 一律使用中文回覆與寫註解
- 修改任何檔案前都要先問過我（對應 .claude/settings.json 的 defaultMode）
- 不要動 ____ 資料夾
- commit 前先跑 `____`

## 程式風格
- indentation 用 ____ 個空格
- 命名慣例：____
```

同樣道理：因為你是一人開發，**直接寫在專案根目錄的 `CLAUDE.md`**，不要分流到 `CLAUDE.local.md`。`CLAUDE.local.md` 是設計給「不想進版控的個人偏好」用的（比如你自己本機測試資料庫的網址），但專案目標、規則這種東西你會希望它跟著 git 一起留下歷史紀錄，所以全部寫進會被 commit 的那份 `CLAUDE.md` 就好，不用刻意拆開。

> 補充：CLAUDE.md 裡寫「修改前要先問我」這句話可以加，當作給 Claude 看的提醒，但真正擋下動作的是第 4 步 `settings.json` 裡的 `defaultMode`。兩者一起寫最保險：一個是說明意圖，一個是真的會生效的開關。

## 6.（選用）大專案才需要：`.claude/rules/`

如果規則只適用於特定資料夾（例如只有 API 路由要遵守的規範），不要塞進主要的 `CLAUDE.md`，改放 `.claude/rules/` 底下用 frontmatter 指定 `paths`，這樣只有 Claude 碰到那些檔案時才會載入：

```markdown
---
paths:
  - "src/api/**/*.ts"
---

# API 開發規則
- 所有 endpoint 都要做輸入驗證
```

小專案、一人開發通常用不到這層，全部寫在 CLAUDE.md 就夠了。

## 7. 開始開發時的習慣

### Plan mode 是什麼

Plan mode 是 Claude Code 內建的權限模式之一，跟第 4 步設定的 `defaultMode` 是同一個層級的東西，只是行為不同。Claude Code 總共有六種模式，常用的幾種：

| 模式 | 修改前要不要問你 | 適合場合 |
|---|---|---|
| `default` | 每次都問 | 一般開發，逐步審核 |
| `plan` | 完全不會動程式碼，只研究、最後給你一份完整計畫 | 大改動、架構調整前先看全貌 |
| `acceptEdits` | 自動接受編輯不問 | 已經很信任方向、想加快速度 |
| `auto` | 幾乎不問，靠分類器做安全檢查 | 長任務，需特定模型/方案才能用 |

Plan mode 進入後，Claude 只能讀檔案、跑指令做研究，**不會寫入任何修改**，最後直接呈現一份完整計畫給你看。你可以選擇：

- 核准並切換成自動編輯（`acceptEdits`）
- 核准但每一步編輯仍要你手動確認
- 不滿意，繼續給回饋讓它重新規劃
- 按 `Ctrl+G` 直接在文字編輯器裡修改那份計畫內容

### 怎麼使用

- **CLI**：按 `Shift+Tab` 循環切換模式（`default → acceptEdits → plan`）
- **VS Code**：點對話框最下面的模式指示器，選「Plan mode」
- **單次提問**：在提問前加 `/plan`，只針對這一次生效，不用整個對話都切過去

### 跟 default mode 的差異

`default` mode 其實也會在每次編輯前問你，這點跟「修改前要同意」已經滿足了。Plan mode 多做的是**強迫它先把整個改動想清楚、寫成一份完整計畫，你一次看完全貌再決定要不要放行**，而不是邊做邊一個個編輯跳出來問。

**如果是程式專案**，建議用法是：平時保留 `default` 當基準（已經寫在 `.claude/settings.json` 裡），遇到比較大的功能或架構調整時，自己按 `Shift+Tab` 手動切到 Plan mode 看一下整體規劃，做完這次改動再切回來。這樣不會每次小修改都要多核准一次計畫。

### 該不該把 `defaultMode` 寫死成 `plan`？

看這個 repo 是什麼性質：

| repo 性質 | 建議 `defaultMode` | 為什麼 |
|---|---|---|
| 程式專案（有原始碼、要反覆小修改） | `default` | 開發過程有大量瑣碎編輯，每個都先出一份計畫會卡住節奏。大改動時自己按 `Shift+Tab` 切過去就好 |
| 純知識庫（筆記、文件、卡片盒） | `plan` | 寫入次數本來就少，而且每次寫入都在動你的知識庫內容 |

判準是**寫入的頻率與可逆性**：改一行 code，跑個測試就知道對不對；改一句筆記裡的說法，錯了你三個月後才會發現，而且中間會一直照著錯的做。

換句話說，`plan` 的成本（每次多核准一份計畫）在寫入頻繁的地方很痛，在寫入稀疏的地方幾乎感覺不到 —— 而它擋下的損失剛好相反。所以不要問「plan 是不是比較嚴謹」，要問「這個 repo 一天會被寫幾次」。

- 發現方向跑歪了，盡早糾正，不要等它寫完一大段才處理
- 善用 **checkpoint**：跑歪了可以直接把程式碼跟對話一起回溯到之前的某個點重來
- 用 `/memory` 隨時檢查目前載入了哪些 CLAUDE.md 跟 auto memory 內容，確認 Claude 真的有讀到你寫的規則
- 用 `/status` 可以檢查目前 settings.json 確實被讀到了

## 檢查清單（每次開新專案複製貼上用）

- [ ] `git init`
- [ ] VS Code 開啟資料夾，叫出 Claude Code
- [ ] 跑 `/init` 產生初版 CLAUDE.md
- [ ] 建立 `.claude/settings.json`，依 repo 性質設定 `permissions.defaultMode`（程式專案 `default`／純知識庫 `plan`）與 `language: "chinese"`
- [ ] 在 `CLAUDE.md` 補上專案目標、技術堆疊、一定要遵守的規則
- [ ] 視需要拆 `.claude/rules/` 給特定路徑的規範（小專案可跳過）
- [ ] `git add .claude/ CLAUDE.md && git commit`，讓這些規則跟版本一起走
- [ ] 開始開發，優先用 Plan mode 起手
