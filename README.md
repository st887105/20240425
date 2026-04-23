# 20240425
車城國小｜3D列印名牌闖關任務 - Deployed by EZPage
# 車城國小 3D 列印闖關任務 — 完整部署指南

> **架構總覽**
> ```
> 學生瀏覽器 (GitHub Pages)
>      │  fetch POST/GET
>      ▼
> Google Apps Script (Web App)
>      │  讀寫
>      ├──▶ Google Sheets（學習紀錄試算表）
>      │        ├─ 學習暫存紀錄
>      │        ├─ 簽名上傳紀錄
>      │        └─ 最終繳交紀錄
>      └──▶ Google Drive（簽名 PNG 資料夾）
> ```

---

## 第一步：建立 Google 試算表

1. 前往 [Google Sheets](https://sheets.google.com) 新增一份空白試算表。
2. 命名為：`車城國小 3D列印 闖關紀錄`
3. 從網址列複製試算表 ID（網址中 `/d/` 和 `/edit` 之間的長串字）。
   ```
   https://docs.google.com/spreadsheets/d/【這裡是 SHEET_ID】/edit
   ```
4. 工作表的三個分頁（Sheet）**會由 GAS 自動建立**，不需要手動新增。

---

## 第二步：建立 Google Drive 簽名資料夾

1. 前往 [Google Drive](https://drive.google.com)。
2. 在任意位置新增資料夾，命名為：`3D列印簽名檔`（名稱可自訂）。
3. 開啟資料夾，從網址列複製資料夾 ID：
   ```
   https://drive.google.com/drive/folders/【這裡是 FOLDER_ID】
   ```
4. 建議在資料夾上按右鍵 → **共用** → 設為「知道連結的人可以檢視」，方便日後分享。

---

## 第三步：建立 Google Apps Script

1. 前往 [Google Apps Script](https://script.google.com) → **新增專案**。
2. 命名為：`3D列印闖關 API`
3. 將 `Code.gs` 的內容完整貼入（取代預設的空白函式）。
4. **修改開頭的兩個設定值**：
   ```javascript
   const SHEET_ID            = '貼上第一步取得的 SHEET_ID';
   const SIGNATURE_FOLDER_ID = '貼上第二步取得的 FOLDER_ID';
   ```
5. 點擊 **儲存**（Ctrl+S）。

---

## 第四步：部署 GAS 為 Web App

1. 點選右上角 **部署** → **新增部署**。
2. 部署類型選：**網頁應用程式**。
3. 設定如下：

   | 欄位 | 設定值 |
   |------|--------|
   | 說明 | 3D列印闖關 v1 |
   | 以誰的身分執行 | **我自己（你的 Google 帳號）** |
   | 誰可以存取 | **所有人（包含匿名使用者）** |

   > ⚠️ 「所有人」是必要的，因為學生不需要登入 Google 也能使用。

4. 點 **部署** → 出現授權視窗，點 **授權存取**，選你的帳號，點 **進階** → **前往（不安全）** → **允許**。
5. 複製部署完成的 **網頁應用程式網址**，格式如：
   ```
   https://script.google.com/macros/s/XXXXXX/exec
   ```

---

## 第五步：設定前端 HTML

開啟 `index.html`，找到以下這行，貼上剛才複製的網址：

```javascript
const GAS_URL = 'https://script.google.com/macros/s/XXXXXX/exec';
```

---

## 第六步：部署到 GitHub Pages

### 6-1 建立 Repository

1. 前往 [GitHub](https://github.com) → **New repository**。
2. Repository 名稱建議：`3d-print-learning`（全小寫）。
3. 設為 **Public**（GitHub Pages 免費版需要公開）。
4. 勾選 **Add a README file**，點 **Create repository**。

### 6-2 上傳檔案

**方法 A：直接在網頁上傳（最簡單）**

1. 進入 Repository 頁面，點 **Add file** → **Upload files**。
2. 把 `index.html` 拖進去，點 **Commit changes**。

**方法 B：使用 Git 指令**

```bash
git clone https://github.com/你的帳號/3d-print-learning.git
cd 3d-print-learning
cp /你的路徑/index.html .
git add index.html
git commit -m "Add 3D print learning page"
git push origin main
```

### 6-3 開啟 GitHub Pages

1. Repository 頁面 → **Settings** → 左側 **Pages**。
2. Source 選 **Deploy from a branch**。
3. Branch 選 **main**，資料夾選 **/ (root)**。
4. 點 **Save**，等約 1 分鐘後頁面會顯示網址：
   ```
   https://你的帳號.github.io/3d-print-learning/
   ```

---

## 第七步：測試確認

| 測試項目 | 預期結果 |
|---------|---------|
| 輸入座號 `60101` 並按開始 | GAS 查詢無資料，建立新紀錄，進入第一關 |
| 完成第一關測驗 | Sheets「學習暫存紀錄」出現一列，目前關卡=2 |
| 關閉頁面重新輸入同一座號 | 彈出「找到你的進度」，點繼續直接跳到第二關 |
| 第三關畫完簽名後上傳 | Drive 資料夾出現 `60101_簽名檔.png`，Sheets「簽名上傳紀錄」新增一列 |
| 下載按鈕出現 | 上傳成功後才顯示綠色下載區塊 |
| 最終繳交 Tinkercad 連結 | Sheets「最終繳交紀錄」出現一列 |

---

## 試算表欄位說明

### 工作表一：學習暫存紀錄

| 欄位 | 說明 |
|-----|------|
| 座號 | 學生輸入的班級座號（主鍵） |
| 目前關卡 | 上次停在第幾關（0–5） |
| 第一題答案 | A / B / C |
| 第二題答案 | A / B / C |
| Tinkercad連結 | 學生貼上的作品 URL |
| 簽名已上傳 | 是 / 否 |
| 簽名Drive連結 | Drive 上的 PNG 共用連結 |
| 簽名檔名 | `座號_簽名檔.png` |
| 首次建立時間 | YYYY/MM/DD HH:MM:SS |
| 最後更新時間 | YYYY/MM/DD HH:MM:SS |

### 工作表二：簽名上傳紀錄

每次上傳簽名都會在此追加一列，方便查閱歷史（學生可能重複上傳）。

### 工作表三：最終繳交紀錄

| 欄位 | 說明 |
|-----|------|
| 座號 | 學生座號 |
| Tinkercad 連結 | 作品共用 URL |
| 第一題答案 | A / B / C |
| 第二題答案 | A / B / C |
| 簽名是否上傳 | 是 / 否 |
| 繳交時間 | YYYY/MM/DD HH:MM:SS |

---

## 常見問題

**Q：GAS 回傳 403 或 CORS 錯誤？**
A：請確認部署設定「誰可以存取」為「所有人（包含匿名使用者）」，並且重新部署一次（修改設定後需重新部署才生效）。

**Q：修改了 Code.gs 之後沒有生效？**
A：每次修改 GAS 程式碼後，都需要重新點選「部署 → 管理部署 → 編輯 → 版本選新版本 → 部署」，原有的網址才會更新。

**Q：Drive 資料夾看不到上傳的圖片？**
A：確認 `SIGNATURE_FOLDER_ID` 填的是正確的資料夾 ID，且該 Google 帳號有資料夾的寫入權限。

**Q：學生在不同電腦輸入同一座號，進度會同步嗎？**
A：會，只要輸入相同座號，GAS 就會從 Sheets 讀取進度。但當次畫的簽名圖（DataURL）存在記憶體中，跨裝置無法直接下載 PNG；學生可點「下載」按鈕，系統會自動開啟 Drive 的雲端連結。

---

## 檔案清單

```
專案根目錄/
├── index.html        ← 前端頁面（部署到 GitHub Pages）
├── gas/
│   └── Code.gs       ← GAS 後端（貼到 Apps Script）
└── README.md         ← 本部署指南
```
