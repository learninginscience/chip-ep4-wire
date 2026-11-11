# 導線走線大師

LIS 情境科學教材「晶片系列 EP4：晶片的誕生」課堂結尾活動。

取代 EP4 影片版簡報第 48 頁的「假設」提問：學生用基爾比和諾伊斯的想法，親手接導線、往上蓋立體電路（含三井三屋），最後把整顆晶片由下往上組裝起來。

## 上課使用

- **學生自己玩**：https://learninginscience.github.io/chip-ep4-wire/ ，掃 QR code 或打開網址就能開始，不用登入、不用安裝。這個模式不記錄任何資料。
- **直接玩版**：https://learninginscience.github.io/chip-ep4-wire/play.html ，打開就是第一關，連名字都不用輸入，適合沒有要比賽、只想讓學生直接體驗的時候。
- **全班即時競賽**：老師打開 https://learninginscience.github.io/chip-ep4-wire/teacher.html ，按「開新的一場」，投影房間代碼與 QR code。學生加入後等老師按開始，投影幕即時顯示排行榜，老師可以暫停（全班學生畫面一起停住）、繼續，按結束時學生畫面停在結束頁、投影幕出現頒獎台。可以看各題第一次答對的情況，也可以下載 Excel 成績。

## 想修改遊戲

1. 改 `src/` 裡的原始檔，不要直接改根目錄的 `index.html`、`teacher.html`、`play.html`。
2. 在專案根目錄執行 `python build.py`，組出新的 `index.html`、`teacher.html`、`play.html`。
3. 推上 GitHub，網站約 1 分鐘內自動更新。

完整的檔案說明、測試方法、要同步修改的地方與教學設計原則，寫在 [CLAUDE.md](CLAUDE.md)。用 Claude Code 打開這個專案時，它會自動讀這份說明。

## 資料與安全

- 即時競賽用 Firebase（匿名登入＋Firestore）暫存學生輸入的名字、班級、遊戲進度與每題第一次的作答。老師可以在頒獎台按「刪除這場的資料」清除。
- 安全規則見 `firestore.rules`：學生只能改自己的資料，作答紀錄只有開房的老師讀得到，只有開房的老師能開始、結束或刪除房間。
