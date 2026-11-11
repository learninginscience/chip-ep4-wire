# 導線走線大師：給 AI 助手的開發說明

這是 LIS 情境科學教材「晶片系列 EP4：晶片的誕生」的課堂結尾遊戲。修改前請先讀完這份文件。

**閱讀順序**：先讀這份，再讀 `docs/開發紀錄.md`（做過哪些改動、哪些做法已經被否決），以及 `docs/活動說明.md`（課堂流程、時間、計分與正確答案）。

- 學生畫面：https://learninginscience.github.io/chip-ep4-wire/
- 老師投影頁：https://learninginscience.github.io/chip-ep4-wire/teacher.html
- 直接玩版：https://learninginscience.github.io/chip-ep4-wire/play.html （打開就是第一關，不登記名字、沒有房間、不上傳資料）
- 原始碼：https://github.com/learninginscience/chip-ep4-wire （2026-11-11 從 pxchen-89/chip-ep4 搬進 LIS 組織；舊網址 pxchen-89.github.io/chip-ep4 會自動轉到新網址）。組織裡另一個 `learninginscience/chip-ep4` 只是轉址頁，不要在那裡改
- 對象：國小高年級，用平板或手機。
- 教學目標（簡報第 3 頁）：晶片的結構是先在一塊半導體材料上做出所有需要的電子零件，接著在上面一層一層地蓋出讓電子零件彼此連接的導線。
- **定位**：這個活動取代 EP4 影片版簡報第 48 頁的「假設」提問（「晶片的結構會長怎樣？」），玩完接第 49 頁老師統整。不是課後評量，也不是擴充新內容。

## 檔案結構

| 路徑 | 用途 |
|---|---|
| `src/game.src.html` | 學生端原始檔（關卡、接線引擎、組裝晶片、單人與房間模式、暫停／結束畫面） |
| `src/teacher.src.html` | 老師投影頁原始檔（開房間、大廳、即時排行榜、暫停／繼續、頒獎台、各題統計、下載 Excel） |
| `src/play_boot.js` | 直接玩版的啟動程式：組建時取代學生頁最後的 `renderStart();`，進度改存 sessionStorage |
| `src/live_core.js` | 兩頁共用的即時房間核心（Firebase 正式模式＋`?mock=1` 測試模式） |
| `src/firebase_config.json` | Firebase 網頁設定（公開資訊，不是密碼） |
| `src/chars/*.jpg` | 角色圖，取自 EP4 影片畫面 |
| `firestore.rules` | Firestore 安全規則的正本（改了要到 Firebase 主控台重新發布） |
| `build.py` | 組建：把 src 組成根目錄的 `index.html`、`teacher.html`、`play.html` |
| `index.html`、`teacher.html`、`play.html` | **組建產物，不要手改**，GitHub Pages 直接發布這三個檔。`play.html` 和 `index.html` 用同一份 `game.src.html`，改關卡或題目兩邊會一起變 |
| `tools/solver.py` 等 | 關卡驗證用的窮舉程式（最少層數、最短導線） |
| `docs/活動說明.md` | 課堂流程、估計時間、計分與正確答案 |
| `docs/開發紀錄.md` | 歷次改動與原因（避免把被否決的做法做回去） |

## 修改流程

0. **先 `git pull`，看 `git log` 有沒有別人的新 commit。** 曾經發生過：一個 AI 對話用自己手上的舊檔案整包覆蓋，把另一個對話剛推上去的版本蓋掉。
1. 只改 `src/` 裡的檔案（或 `firestore.rules`）。
2. 在專案根目錄執行 `python build.py`。
3. 本機測試：`python -m http.server 8765`，打開 `http://localhost:8765/`（學生）與 `http://localhost:8765/teacher.html`（老師）。
4. 開新分支、推上去、開 PR 給 pxchen-89 審核（步驟見 `CONTRIBUTING.md`）。main 只有 pxchen-89 能更新；合併後 GitHub Pages 約 1 分鐘內自動更新。
5. 如果改了 `firestore.rules`，要請專案擁有者到 Firebase 主控台的 Firestore →「規則」分頁，全選刪掉舊規則、貼上新規則、按「發布」。規則沒發布前，新增的資料欄位會被拒絕。

## 測試方法

- **不連 Firebase 測房間功能**：網址加 `?mock=1`，例如 `teacher.html?mock=1` 與 `index.html?room=1234&mock=1`。資料只存在同一個瀏覽器的分頁之間。
- **連真的 Firebase 測**：同一個網址的所有分頁會共用同一個匿名身分，所以要用不同的 port 模擬不同裝置，例如老師用 `localhost:8765`、學生用 `localhost:8766`、`localhost:8767`。Firebase 的授權網域預設有 `localhost`，沒有 `127.0.0.1`。
- **mock 模式測不到的問題**：登入順序與安全規則。例如學生必須先匿名登入才能讀房間，這只有接真的 Firebase 才測得出來。
- **自動測試要點「畫面上看得見」的按鈕**，並檢查頁面有沒有重複的 id。用 id 直接點，可能點到被隱藏的舊按鈕而漏掉錯誤（曾經發生過）。
- 測完房間要在老師頁按「刪除這場的資料」，不要在正式資料庫留下測試資料。

## 學生端結構（src/game.src.html）

- `STEPS`：遊戲步驟順序，目前是「阿寶的晶片（chip）→ 三井三屋（k33big）→ 組裝晶片 → 工程師證」；`STOPS`：進度列的名稱；每個 step 的 `s` 是它屬於第幾站。
- `LV` 裡的 practice、detour、wall、cross3 與 `SC` 的問答已經移出主線（簡報第 43～47 頁已教過，或是簡報沒有的新推理），定義還留著，彩蛋或日後可能再用。
- `LV`：接線關卡（格子大小、電晶體位置、要接的導線、能不能蓋層 `layers:'no'|'gated'|'yes'`）。
- `SC`：角色對話題目（目前主線沒有用到）。每題第一次的選擇寫進 `A[id]`，之後可以重選，但成績只算第一次。
- `ITEMS`：計分項目（id、名稱、學到什麼、配分）。稱號依得分比例。
- `PARTS`：組裝晶片的材料卡；`scoreStack()`：組裝晶片的判分。沿路收集零件的背包已停用（`#bagBox` 隱藏）。
- `SOLUTIONS`：三井三屋兩種關卡的最佳解（由 `tools/` 的窮舉程式算出）。
- 房間模式：`liveReport()` 回報進度到 `rooms/{房間代碼}/players/{uid}`，每題第一次作答回報到 `rooms/{房間代碼}/answers/{uid}`。
- 老師暫停或結束：`liveAttach()` 監看房間狀態，`livePause()` 在畫面最上層蓋 `#hold` 擋住操作，`liveEnded()` 停在結束頁不能再操作。`liveState` 記錄學生在哪個階段（等待／遊戲中／被取消／已結束），在大廳等待時由 `waitForStart()` 處理老師取消。

## 改的時候一定要同步的地方

- **改題目、選項文字或新增題目**：老師頁的 `QMETA`（畫面統計）與 `QBANK`、`QIDS`（Excel）要一起改，選項的 `k` 要和學生端一致。
- **改 STEPS 或 STOPS**：老師頁的 `STOPS` 也要改（進度欄數會自動跟著 `STOPS.length`）；學生端 `PKEY` 要換新版本號，避免舊的接續進度對到錯的步驟。
- **學生回報或房間新增欄位、新增房間狀態**：`firestore.rules` 的 `hasOnly([...])` 或 `status in [...]` 要加上，並重新發布規則。目前規則限制分數 0 到 2000、作答紀錄最多 10 題。
- **改關卡格子或導線**：用 `tools/` 的窮舉程式重新驗證最少層數與最短長度，再更新完成訊息與 `SOLUTIONS`。

## 教學設計原則（專案擁有者確認過的）

- **取代一頁簡報，不擴增新邏輯，也不重複簡報已教的內容。**（主管嚴天浩的回饋）簡報第 47 頁已經講過「用不導電材料隔開，做成立體電路」，所以「鋪一層不導電材料」的按鈕一開始就在，不要讓學生重新發現。

- **只考簡報有教的內容。** 影片裡才出現的細節不能當必答。例如組裝晶片時，第 1 層導線底下的不導電材料，簡報沒教，所以可放可不放。
- **干擾選項不能和課程裡的正確說法撞在一起。** 例如「保護外殼」放在最上面（把整顆晶片包起來，簡報第 24 頁「再封起來」）算對，夾在中間才算錯。
- **答錯不打叉。** 由角色解釋後讓學生重選；成績只記第一次，而且要在任何提示或回饋出現之前記錄。
- **學生畫面的操作提示要放在不用捲動就看得到的位置。** 例如蓋新一層的粉紅色「＋」按鈕放在棋盤正上方。
- **學生畫面不放老師入口。** 老師一律用 `teacher.html`。
- 介面文字用繁體中文，中文情境用全形標點。

## 已驗證的關卡數字

| 關卡 | 格子 | 最少層數 | 最短導線 | 目前用途 |
|---|---|---|---|---|
| 阿寶的晶片（chip，一層一定無解） | 5×5 | 2 | 8 | 主線第 1 關 |
| 阿寶的晶片 · 三井三屋（k33big） | 7×7 | 2 | 56 | 主線第 2 關 |
| 彩蛋：再縮小一點（k33small） | 5×5 | 3 | 52 | 破關後彩蛋 |
| 訓練場（practice） | 5×5 | 1 | 8 | 已移出主線 |
| 1959 走廊（detour，要繞路） | 7×7 | 1 | 14 | 已移出主線 |
| 諾伊斯（wall，和 chip 同一塊板） | 5×5 | 2 | 8 | 已移出主線 |
| 晶片工廠：三條兩兩交叉（cross3） | 5×5 | 3 | 16 | 已移出主線 |

## Firebase

- 專案 `chip-ep4`（Spark 免費方案），匿名登入、Firestore（asia-east1）。
- 資料：`rooms/{房間代碼}`（老師開房；`status` 是 lobby／playing／paused／ended，`pausedAt`、`pausedMs` 用來讓計時器扣掉暫停時間）、`players/{uid}`（排行榜進度）、`answers/{uid}`（每題第一次作答，只有開房老師讀得到）。
- Firestore 推送在部分網路會被暫存，所以強制使用長輪詢；學生等待開始時另外每 2 秒主動查一次。
