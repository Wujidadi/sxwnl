# Copilot 指引

## 專案概觀

這是「壽星天文曆」（sxwnl）5.10.3，純前端 HTML 與原生 JavaScript 專案。沒有套件管理、框架、模組系統或打包器；各 `.js` 檔由 `src/index.htm` 與 `src/indexmp.htm` 透過 `<script src>` 依序載入，並共享全域作用域。

主頁分為 PC 版 `src/index.htm` 與行動版 `src/indexmp.htm`。兩者載入同一組核心腳本，版面不同；`src/sm1.htm` 到 `src/sm9.htm` 是說明與工具頁，`src/readme.htm` 與 `src/exphelp1.htm` 是使用者文件。

## 命令

```text
# 本機預覽（任何平台）
以瀏覽器直接開啟 src/index.htm
以瀏覽器直接開啟 src/indexmp.htm
```

```bat
:: 打包（僅限 Windows，需 cscript / WSH）
convertMarge.bat
```

`convertMarge.bat` 會把 `src/*.{htm,js,bat}` 暫時從 UTF-8 轉為 GBK 到 `out/`，在 `out/` 執行 `hebin.bat`，再由 `cscript jsZip.js` 將 JS 依固定順序合併進根目錄 `index.htm` 與 `indexmp.htm`，最後轉回 UTF-8 並刪除 `out/`。根目錄輸出檔已在 `.gitignore` 中，不要提交。

## 載入順序與架構

`src/index.htm` 與 `src/indexmp.htm` 的腳本順序就是依賴順序：

1. `tools.js`：通用工具，包含 `year2Ayear`、`Ayear2year`、`timeStr2hour`、`storageL`。
2. `eph0.js`：天文演算法基礎，定義常數、角度與時間格式化、儒略日 `JD`、歲差／章動／視差／大氣折射、VSOP87 與月球週期項計算。
3. `ephB.js`：太陽系質心位置與速度資料，供恆星章動與光行差計算。
4. `eph.js`：高階天文計算，包含 `SZJ` 日月升中降、行星天象、日月食等。
5. `JW.js`：城市經緯度壓縮資料庫 `JWv`、時區與年號資料。
6. `lunar.js`：農曆核心，包含 `SSQ` 實朔實氣計算器、`Lunar()` 月曆物件、年曆 HTML 產生器。
7. `vml.js`：日月食與路徑繪圖，使用 canvas。
8. `help.js`：頁面浮動說明內容。
9. `page_gj.js`：工具頁事件處理函式。

## 專案慣例

- 維持 UTF-8 編碼；只有 Windows 打包流程會暫時轉 GBK 以相容原作者的 WSH 合併工具。
- 修改 HTML 入口頁的腳本清單時，PC 版與行動版要同步，且不得破壞既有載入順序。
- `eph.js` 的天文演算法與 `lunar.js` 內 `SSQ` 的古曆朔閏資料、演算法屬高風險區域；除非任務明確要求，避免任意重構或調整公式。
- 年份輸入同時支援天文紀年與傳統紀年：`year2Ayear` 將 `B`、`b`、`*` 開頭或負值輸入正規化為天文紀年，`Ayear2year` 負責顯示用傳統紀年。新增年份處理時先確認使用的是哪套表示法。
- 內部時間多以儒略日（JD）表示；力學時與世界時差由 `dt_T(jd)` 處理。涉及升降、氣朔或星曆計算時，不要混用 TD、UT、北京時間與本地時間。
- 城市經緯度資料在 `JW.js` 中使用自訂壓縮格式；若擴充超出原本範圍，需同步理解並調整解壓邏輯。
- `jsZip.js` 是 Windows Script Host 合併腳本，不是 npm 套件。
- 回覆、說明文件與提交訊息遵循倉庫既有規範：使用繁體中文（臺灣）；Git 提交訊息使用 Conventional Commits 格式。
