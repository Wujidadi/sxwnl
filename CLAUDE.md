# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## 專案性質

「壽星天文曆」（sxwnl）5.10 — 由許劍偉編寫的純前端（Vanilla JS + HTML）天文曆法工具，可換算公曆／農曆／回曆、計算日月坐標與日月食、行星與恆星星曆等。專案無框架、無套件管理、無模組系統；所有 `.js` 透過 `<script src>` 依序載入到全域作用域。

## 常用指令

開發只有兩件事：直接以瀏覽器開啟原始碼，或在 Windows 下執行打包腳本。

```bat
:: 本機預覽（任何平台）
直接以瀏覽器開啟 src/index.htm （PC）或 src/indexmp.htm （行動版）。
無需建置、無需伺服器。

:: 打包（僅限 Windows，需 cscript / WSH 支援）
convertMarge.bat
::  1. 透過 convertcp.exe 將 src/*.{htm,js,bat} 從 UTF-8 轉為 GBK，輸出到 out/
::  2. 進入 out/ 執行 hebin.bat，內部以 cscript jsZip.js 合併壓縮 JS，
::     並把合併結果嵌入到根目錄的 index.htm 與 indexmp.htm
::  3. 將輸出檔再轉回 UTF-8、刪除 out/ 暫存目錄
```

打包輸出（`index.htm`、`indexmp.htm`）已列入 `.gitignore`，不應提交。

注意事項：

- GitHub 會把 Windows CRLF 自動轉成 LF，導致 `.bat` 在 Windows 上 clone 後直接執行可能失敗；若打包腳本異常，先檢查換行符。
- 非 Windows 環境無法執行打包；對 JS 的修改可直接以瀏覽器開啟 `src/index.htm` 驗證。
- `jsZip.js` 是 WSH 用的合併腳本（不是 npm 的 jszip），打包時 `convertMarge.bat` 會跳過它的編碼轉換、保留原樣。

## 程式碼架構

### 載入順序（即依賴順序）

`src/index.htm`（與 `indexmp.htm`）依序載入下列腳本，後者可使用前者的全域符號：

1. `tools.js` — 通用工具：紀年轉換（`year2Ayear`／`Ayear2year`，公元前以 `B` 開頭或天文紀年負值表示）、`timeStr2hour`、`storageL`（localStorage + cookie 後援）。
2. `eph0.js` — **天文算法核心**：定義 `cs_*` 常數（地球半徑、AU、光速等）、三角函式別名、`rad2str*`／`str2rad`、球面座標轉換、`dt_T`（TD−UT）、`JD` 元件（公曆⇄儒略日）、章動／歲差／視差／大氣折射、`XL0/XL1` VSOP87 行星與月球週期項表、`m_coord` / `e_coord` / `p_coord` / `XL0_calc` / `XL1_calc`。所有後續模組共用此檔的全域符號。
3. `ephB.js` — 太陽系質心（SSB）位置與速度數值表，供恆星章動／光行差計算。
4. `eph.js` — 高階天文物件：`SZJ`（日月升、中天、降）。讀取 `eph0.js` 的座標函式組合輸出結果。
5. `JW.js` — `JWv` 城市經緯度資料庫（自訂壓縮編碼，見檔頭註解）、皇帝／年號紀年表。
6. `lunar.js` — **農曆核心**：`SSQ`（實朔實氣計算器，包含古曆數據）、`Lunar()`（建構日曆物件，回傳含公曆／農曆／干支／節氣／節日的日物件，欄位定義見檔頭 200 餘行註解）、`nianLiHTML` / `nianLi2HTML` 年曆 HTML 產生器。
7. `vml.js` — 日月食繪圖（canvas，物件名 `ht_*`／`HT`）。
8. `help.js` — 各頁面浮動說明 HTML。
9. `page_gj.js` — `src/sm*.htm` 與工具頁面的事件處理（`GJ1_*`、`GJ2_*` …）。

### 頁面結構

- `index.htm`／`indexmp.htm` — PC 與行動版主頁面（同一份程式，不同版面）。`sm1.htm`–`sm9.htm` 是分頁工具（日月食、星曆、節氣、節日、年曆等），均依賴上述 JS 全域物件。
- `readme.htm`、`exphelp1.htm` — 使用者文件，非程式碼。

### 關鍵不變式（修改前務必理解）

- **`eph.js` 的天文算法、`lunar.js` 中古曆部分（`SSQ` 內的朔閏數據與演算法）不得隨意改動**：作者已對 −721 至 1960 年與張培瑜《三千五百年曆日天象》等權威表核對；改動會直接影響萬年曆的正確性（README 末段強調此約束）。如果工作確實需要碰這兩個檔案，先確認任務範圍與用戶意圖。
- 紀年系統有兩套並行：「天文紀年」（含公元 0 年，B.C. n 表為 1−n）與「常規紀年」（無 0 年，公元前以 `B` 字首）。輸入經 `year2Ayear` 正規化，輸出經 `Ayear2year`；新增任何處理年份的程式碼前先確認你拿到的是哪一套。
- 全部時間內部以儒略日（JD）儲存；力學時（TD）與世界時（UT）差為 `dt_T(jd)`（單位：日）。`SZJ` 等高階物件統一回傳力學時 JD。
- `JD.JD(y,m,d)` 中 `d` 可含小數天（=時分秒）。
- 原始版本為 GBK 編碼；本倉庫已統一為 UTF-8，僅在打包過程暫時轉 GBK 以相容原作者的 WSH 工具。修改檔案時請保持 UTF-8。

### 與原版 5.09 的差異

僅兩處公式修改（README「和原版差異」段已詳列）：
- `eph.js` 中日食「直線到太陽中心的最小值」改為迭代尋找最大食分。
- `eph0.js` 月球週期項 `l1` 中 `t3` 係數的符號（`-0.000136` → `+0.000136`）。

## 開發約定

- **語言**：所有回覆、解說、文件，以及工具呼叫的 explanation 欄位，一律使用繁體中文（臺灣），採用臺灣慣用譯名與術語。
- **最小修改原則**：除非明確要求，否則對既有程式碼僅做完成任務所需的最小變更，不順手重構、不擴大影響範圍。
- **Git Commit**：訊息採 Conventional Commits 格式（`feat:`／`fix:`／`docs:` …）。
