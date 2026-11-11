# 怎麼改導線走線大師（給 LIS 同事和你的 AI）

這個 repo 放在 LIS 的 GitHub 組織裡。組織成員都可以改，但改好的東西要先送「PR（合併請求）」，由陳鵬旭（pxchen-89）看過，決定要不要合併。合併後約 1 分鐘，學生用的網址就會更新。

- 學生畫面：https://learninginscience.github.io/chip-ep4-wire/
- 老師投影頁：https://learninginscience.github.io/chip-ep4-wire/teacher.html
- 直接玩版：https://learninginscience.github.io/chip-ep4-wire/play.html

## 用 AI 改（例如 Claude Code）

把下面這段當成給 AI 的第一句話，「我想改什麼」換成你的需求：

```text
請先讀 CLAUDE.md 和 CONTRIBUTING.md。我要改 learninginscience/chip-ep4-wire 這個遊戲：我想改什麼。
請在新的分支上改，改完在本機實際測試，再開 PR 給 pxchen-89 審核，不要推到 main。
```

## 自己動手的流程

1. 下載：`gh repo clone learninginscience/chip-ep4-wire`（已經下載過的話，先 `git pull`）
2. 開新分支：`git switch -c 你的帳號/改什麼`，例如 `ylwu/加大字體`
3. 修改，在本機測試（測試方法寫在 CLAUDE.md）
4. 推上去：`git push -u origin 分支名稱`
5. 開 PR：`gh pr create`，或到 GitHub 網頁按「Compare & pull request」

## PR 要寫什麼

- **改了什麼**：一兩句。
- **為什麼要改**：課堂上遇到什麼問題、學生卡在哪裡。
- **怎麼測的**：用什麼裝置、測了哪些流程；畫面有變的話附截圖。

## 請不要

- 直接推到 main。main 只有陳鵬旭能更新，推了也會被擋。
- 自己改 Firebase 設定或安全規則。規則要到 Firebase 主控台發布，只有專案擁有者（陳鵬旭）能做，要改請先在 PR 裡說明。
- 違反 CLAUDE.md 裡寫的教學設計原則。不確定的話，在 PR 裡提出來討論。
