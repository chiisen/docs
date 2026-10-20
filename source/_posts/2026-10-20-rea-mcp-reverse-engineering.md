---
title: 🔍 REA：用一個 MCP 服務，讓 AI Agent 逆向分析你手上的程式
date: 2026-10-20 00:00:00
tags:
  - AI
  - Agent
  - MCP
  - Reverse Engineering
---

## 🔍 REA：用一個 MCP 服務，讓 AI Agent 逆向分析你手上的程式

**✅ 這篇做什麼：介紹 morluto/rea 這個開源專案。它把 Hopper、Ghidra、IDA、JADX 等分析工具包成 MCP 服務，讓 AI Agent 能從應用行為一路追到原生二進位，並且要求每個結論都要附上證據。**

<!-- more -->

看到一個喜歡的功能，想搞清楚它到底怎麼做的？以前這條路的起點是自己打開反編譯器，現在多了一種選擇：把分析工具接給 AI Agent，讓 Agent 去查、去追、去解釋，你負責檢查它的證據夠不夠。

REA 就是為這個流程設計的。專案在 GitHub 上成長很快（撰稿時約 33.9 萬 Star、8.5 萬 Fork），主體以 TypeScript 撰寫，採 MIT 授權。

---

# 🌟 它解決的問題：從「行為」追到「二進位」

多數人對逆向工程的印象停在「打開 IDA，看一堆組語」。但真實的需求通常更具體：

- 這個 App 的搜尋是怎麼實作的？
- 這個 Electron 應用的剪貼簿資料，是經過哪一條 IPC 傳出去的？
- 這段老遊戲的計算公式，能不能還原成可以編譯的 C？

REA 的定位是中間那層橋樑：

1. AI Agent 提出問題（透過 MCP 呼叫 REA）；
2. REA 呼叫本機的分析引擎或解析器，取得程式碼、字串、符號、呼叫關係、網路觀察結果；
3. REA 把發現**連同證據與限制**回傳給 Agent；
4. Agent 據此解釋行為、提出下一個問題，或直接寫出實作並測試。

關鍵在第 3 點。它不是丟一段看起來很合理的推測給你，而是要求回傳可核對的引用與未知項。

---

# 🧩 三個組成部分

| 元件 | 角色 |
| --- | --- |
| **MCP 服務** | `npx rea-agents setup` 會向 AI Agent 註冊 REA 的 MCP 服務，並備份既有設定 |
| **CLI（`rea`）** | 同一套分析能力，讓你在終端機直接跑、可輸出 JSON |
| **Skills / 工作流程指引** | 安裝後提供調查方法論，告訴 Agent 該怎麼問、怎麼驗證，而不是亂槍打鳥 |

同一個分析核心，兩種入口：Agent 用它推理，人用它驗證。

---

# 🚀 快速開始

### 1. 設定 AI Agent

先確認 Node.js 與 npm 可用，然後：

```bash
npx rea-agents setup
```

選擇要接的 Agent（Claude Code、Codex、Cursor、Gemini CLI、Grok Build 等），**檢查它列出的變更計畫再核准**，完成後重新啟動 Agent。

### 2. 直接對 Agent 下指令

```text
弄清 Notes 應用的搜尋功能如何工作，展示證據，並為我的專案實現類似功能。
```

把 `Notes` 換成你的目標，並補上你想理解的行為。原生分析會用到既有的 Hopper、Ghidra 或 IDA；設定流程也能在你同意後代為安裝 Hopper。純靜態的 JavaScript 分析不需要原生引擎。

### 3. 或用終端機

```bash
npx -y rea-agents@latest analyze-javascript-application /absolute/path/to/app --json
```

回傳內容包含模組、匯入關係、Electron 邊界與對應證據。若會常用，裝成全域指令比較順手：

```bash
npm install --global rea-agents
rea --help
```

### 4. 記得更新

這個專案迭代很快，`rea update`（npm 全域安裝）或重跑 `npx rea-agents@latest setup`（npx 方式）都能把 CLI 與 Agent 註冊一起更新。

---

# 🎯 可以分析什麼？

REA 的覆蓋範圍比想像中廣，以下是幾個比較有代表性的目標：

| 分析目標 | 回傳什麼 | 前置需求 |
| --- | --- | --- |
| 原生二進位檔（Mach-O / PE / ELF） | 虛擬碼、組語、字串、符號、呼叫與引用 | Hopper、Ghidra 或 IDA |
| Go 二進位檔 | 內嵌編譯器版本、模組、replace、建置設定 | 內建靜態檢查，免裝 Go |
| Rust 二進位檔 | 內嵌的 cargo-auditable 依賴清單 | 內建 |
| JavaScript / Electron | 模組、匯入、source map、路由、IPC、原生 addon | Node.js 與 npm |
| .NET 組件 | metadata、CIL 指令、宣告的原生依賴 | 純靜態檢查 |
| Android APK | manifest 宣告、類別、反編譯方法與引用 | JADX + 完整 JDK |
| Flutter 應用 | Dart snapshot 雜湊、依 ABI 的程式庫雜湊、引擎建置來源 | 無（靜態解析 APK） |
| WASM 工件 | 驗證結果、section、import/export 的 WAT 形式 | WABT 1.0.42 |
| 網站 / HAR 擷取 | 頁面結構、腳本、網路觀察、截圖 | Chrome 系瀏覽器 |
| 封包擷取（pcap） | 端點、DNS、TLS SNI、協定分佈、HTTP 請求 | tshark、capinfos |
| 韌體映像 | 區域切割、提取結果，再轉交原生分析 | Binwalk / Unblob（Linux） |
| 程序行為 | 終端輸出、互動、退出碼、檔案系統觀察 | Linux / macOS 原生 PTY |

完整的目標清單（還包含 JEB 專案、Android 資源、EVM bytecode、邏輯訊號記錄等）在官方文件的 MCP 工具目錄與各主題指南裡。

---

# 🧪 官方案例：三個不同層次的問題

**🎮 DX-Ball：重建聲像計算**
沿著聲音呼叫追到「依位置計算聲像」的輔助函數，檢查指令，把不完整的虛擬碼補成 C。重建後的函式通過 3,205 個原始 x86 測試案例，並重現了編譯後函式的全部 63 個位元組。

**📝 Notion：追蹤 Electron 剪貼簿橋**
從 renderer 的剪貼簿 API 出發，沿 preload 與 IPC 追到 main process，再回頭檢查富格式剪貼簿資料的處理方式。這是典型的「跨程序邊界」題目，人工追起來很費時。

**🕹️ TH04：恢復 DOS 環形子彈的計算**
處理 16 位元 PC-98 遊戲指令，還原固定角度與瞄準角度的公式，最後把重建的 C++ 與當年的編譯器輸出對照。

三個案例分別代表原生、JavaScript/Electron、以及古早 16 位元平台。共同點是：**結論都要能被驗證**，而不是「AI 說它是這樣」。

---

# ⚠️ 使用前提與風險

這類工具能做的事很強，但邊界要講清楚：

- 📦 **Node 版本有要求**：需要 Node.js 22.x（≥ 22.19）、24.x（≥ 24.11）或 26 以上。
- 💰 **商業工具授權自負**：Hopper、IDA Pro / Hex-Rays、JEB 都需自行取得合法授權，REA 不提供金鑰也不會幫你繞過限制。
- 🏃 **執行階段擷取不是唯讀**：靜態的 JavaScript 與 .NET 檢查只讀檔；但程序擷取、瀏覽器觀察這類功能，會用你的使用者權限實際執行或操作目標。
- ☁️ **分析在本機，結果會離開本機**：REA 本身不上傳你的應用，但工具結果會交給 AI Agent，後續就受模型供應商的資料政策約束。
- ⚖️ **法律邊界**：專案立場是只用於合法研究、教育、互通性、軟體保存，以及你擁有或被授權檢查的軟體。這條線不要自己模糊掉。

---

# 🔍 我看到的意義：逆向工程的「介面」正在改變

過去的逆向工程，門檻在**工具熟練度**：你要會看 control flow graph、會認編譯器最佳化的痕跡、知道怎麼命名變數。

REA 這類專案把門檻往前挪了一格，變成**提問與驗證的能力**：你能不能把「我想知道什麼」講清楚，以及你有沒有能力判斷 Agent 給的證據是紮實還是拼湊。

這件事對防禦方也有意義。當 Agent 可以快速拆解一個應用的行為與通訊協定，靠「客戶端混淆」撐起來的安全假設就更薄了——真正該守住的東西，還是要留在伺服器端。

---

# 📚 參考資料

- [morluto/rea（GitHub）](https://github.com/morluto/rea)
- [README 繁體中文版](https://github.com/morluto/rea/blob/main/docs/readme/README_zh-TW.md)
- [REA 官方網站](https://rea.tools/zh-TW/) ｜ [指南](https://rea.tools/zh-TW/guides/) ｜ [案例](https://rea.tools/zh-TW/showcase/)
- [rea-agents（npm）](https://www.npmjs.com/package/rea-agents)
