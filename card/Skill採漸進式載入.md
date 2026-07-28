# Skill 採漸進式載入，讓 Agent 只在需要時取得細節

Agent 會先看 `name` 與 `description` 判斷任務是否符合，再讀取完整的 `SKILL.md`。<br>
只有任務真的需要時，才繼續載入 reference、執行 script 或使用 asset，藉此減少無關資訊占用 context。

**連結**
→ [〈SKILL.md 是 Skill 的入口，不是整個 Skill〉](./SKILL.md是入口.md)
→ [〈條件式知識應放進 references，避免每次都載入〉](./條件式知識放references.md)
→ [〈SKILL.md 與 Skill 資料夾結構〉](../new-note/SKILL.md與Skill資料夾結構.md)

**標籤**：#Skill #漸進式載入 #Context

**來源**：2026-07-28〈SKILL.md 與 Skill 資料夾結構〉學習筆記
