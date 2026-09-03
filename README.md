# 動態總覽 — Wei's Dashboard

個人生活儀表板,一頁看完跑步、騎車、AI 學習、英文學習四個領域。
設計出自 Claude Design 畫布的「指標網格」版本,深色玻璃質感 + 流動光暈。

**線上版本**:https://claude.ai/code/artifact/70e2e532-beb4-4dd1-9a26-ed444ca4fd64

## 四個區塊

| 區塊 | 資料來源 | 更新方式 |
|---|---|---|
| RUN 跑步 | Strava | 每日 00:00 自動同步 + 手動上傳截圖 |
| CYCLING 騎車 | Strava | 每日 00:00 自動同步 + 手動上傳截圖 |
| EN 英文 | 手動上傳截圖 | 由 Claude 讀圖擷取數值 |
| AI 學習 | Claude Code session 紀錄 | 每日 00:00 自動重算(估算值) |

每張卡片都有 7 天長條圖,點任一天可看當日明細;右上的「近 7 天 / 近 30 天」可切換統計區間。

## 執行環境

這一頁是為 **Claude Artifact** 環境寫的,依賴兩個 runtime capability:

- `db` — 讀取儀表板資料(`stats/running`、`stats/cycling`、`stats/ai`、`stats/english`、`stats/goals`、`stats/weather`)
- `sample` — 「+」按鈕上傳截圖後,由 Claude 讀圖擷取數值

**直接用瀏覽器開啟這個 `index.html` 只會看到版面與空狀態**,因為 `window.claude` 不存在,取不到資料。視覺、動畫、排版都能正常預覽。

## 資料流

頁面本身的網路被 CSP 封鎖,不能自己打 API。所有外部資料都由排程任務抓取後寫進 artifact 的資料庫,頁面只負責讀取與呈現:

```
Strava API ─┐
Claude Code session ─┼─→ 每日 00:00 排程 ─→ artifact db ─→ 頁面(即時訂閱)
Open-Meteo 天氣 ─┘
截圖 ─→ 瀏覽器轉檔 JPEG ─→ Claude 讀圖 ─→ artifact db
```

天氣在午夜抓當天 24 小時的逐時預報存起來,頁面白天依當下鐘點自動切換顯示,不需再連網。

上傳的截圖只在記憶體中處理(解碼 → 縮圖 → 轉 JPEG → 送出),**不會被儲存**,只有擷取出的數值進資料庫。

## 技術筆記

- 單一 HTML 檔,無建置流程、無框架
- 字體:BBH Bartle(標題)、Noto Sans TC(中文)、IBM Plex Mono(數據)
- 光暈用 `translate` 飄移搭配 `scale` 呼吸,兩組動畫各自獨立週期
- 長條圖與線條的漸層做成首尾同色的無縫磚,單向循環看不出接點
- 外框流光用錐形漸層旋轉(線性漸層在左右兩側會整條同時明滅,不像光在繞行)
- SVG 圖示用 `pathLength="100"` 正規化線長,不同形狀共用同一組畫線動畫
