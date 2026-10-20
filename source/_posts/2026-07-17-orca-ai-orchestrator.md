---
title: 🐳 Orca：把一整隊 AI Agent 排進同一張工作台的開源編排器
date: 2026-07-17 00:00:00
tags:
  - AI
  - Agent
  - Tool
  - Opensource
---

## 🐳 Orca：把一整隊 AI Agent 排進同一張工作台的開源編排器

**✅ 這篇做什麼：介紹 stablyai/orca 這個開源桌面應用。它不自己做 Agent，而是讓你同時跑 Codex、Claude Code、OpenCode、Pi 等 CLI Agent，每一個都待在獨立的 git worktree 裡，再用手機接手監控。**

<!-- more -->

當 AI 寫程式的工具從「一個聊天框」長成「一整排終端機」之後，新的問題就出現了：你要同時盯五個 Agent、五個分支、五份 diff，還要記得哪一個跑到哪一步。

Orca 想解決的就是這件事。它的自我定位很明確——**面向 100x 構建者的 AI 編排器（AI orchestrator）**，不是再訓練一個更強的模型，而是把「同時操作多個 Agent」這件事變成一套可以管理的介面。

---

# 📦 專案速覽

| 項目 | 內容 |
| --- | --- |
| 專案 | `stablyai/orca` |
| 定位 | 多 Agent 編排的桌面 IDE |
| 語言 / 授權 | TypeScript / MIT（自由且開源） |
| 平台 | macOS、Windows、Linux（另有 iOS、Android companion app） |
| 建立時間 | 2026-03（撰稿時約 9.8 萬 Star、6.2 千 Fork） |
| 官網 / 文件 | [onorca.dev](https://www.onorca.dev/) |

它自己不做推論。你用它既有的訂閱跑自己的 Agent——Claude Code、Codex、Grok、Cursor、Copilot、OpenCode 都可以，只要能在終端機裡跑就行。

---

# 🧠 核心概念：Agent 的調度層，不是另一個 Agent

傳統作法是「一個 repo、一個終端機、一個 Agent 輪流做」。Orca 換了一個假設：

> **同一個提示，同時丟給五個 Agent，各自在自己的 worktree 裡做，做完再比結果。**

這在工程上不是新想法（`git worktree` 本來就是為此而生），但把它做進 GUI，並且處理掉 worktree 建立、分支切換、diff 檢視、合併比較這些瑣事，才是 Orca 的價值所在。隔離帶來的好處很實際：五個 Agent 亂改檔案也互不干擾，你只需要挑一個留下來。

---

# 🛠️ 主要功能

| 功能 | 說明 |
| --- | --- |
| **並行 Worktree** | 一個提示分發給多個 Agent，各自在隔離的 worktree 執行，再比較、合併最佳方案 |
| **終端分屏** | Ghostty 等級的終端，WebGL 渲染、無限分割，重啟後保留滾動歷史 |
| **設計模式** | 在內建 Chromium 裡點選任一 UI 元素，把 HTML、CSS 與裁切截圖直接塞進 Agent 的 prompt |
| **標註 AI Diff** | 直接在 diff 的任一行留言並回傳給 Agent，評審、修改、commit 不用離開應用 |
| **SSH Worktree** | 把 Agent 丟到遠端高效能機器跑，檔案編輯、git、終端全支援，含自動重連與連接埠轉發 |
| **GitHub / Linear 整合** | 應用內瀏覽 PR、issue 與專案看板，從任務直接開 worktree |
| **拖檔案給 Agent** | 檔案或圖片直接拖進 prompt，處處自動儲存 |
| **Orca CLI** | `orca worktree create`、`snapshot`、`click`、`fill`，把工作流腳本化 |
| **行動 Companion App** | 手機監控與指揮 Agent，完成時收到通知，隨時補一句後續指令 |

還有幾個比較容易被忽略但實用的：

- 🔎 **快速開啟**：在 worktree、檔案、Agent、指令之間跳轉，不用離開鍵盤流。
- 📊 **用量追蹤**：查看 Claude / Codex 的用量與額度重置時間，並熱切換帳號。
- 🖥️ **Computer Use**：需要真實互動時，讓 Agent 操作桌面應用與可見 UI。
- 🔔 **通知與未讀**：Agent 完成或需要你時立即知道，也能標記未讀稍後再處理。

---

# 🤖 支援的 Agent

官方列出的清單很長，但規則只有一條：**能在終端機跑，就能在 Orca 跑。**

Claude Code、Codex、Grok、Cursor、GitHub Copilot、OpenCode、Amp、Kimi、Kiro、Qwen Code、Goose、Cline、Continue、Droid、Mistral Vibe、Rovo Dev、Charm…族繁不及備載。

這件事的意義在於**不綁定供應商**。今天 Claude Code 順手就用它，明天想換 Codex 比較同一題的答案，介面不用重學。

---

# 🚀 安裝

### 桌面端

直接從官網下載，或抓安裝包：

```bash
# macOS（Homebrew）
brew install --cask stablyai/orca/orca

# Arch Linux（AUR）
yay -S stably-orca-bin
```

Windows 用 `.exe`、Linux 用 AppImage，也可以在 [releases](https://github.com/stablyai/orca/releases/latest) 頁面找全部建置。

### 行動端

iOS 從 App Store 安裝，Android 直接下載 APK，與桌面端配對後就能遠端盯場。

---

# 🔍 三個值得注意的設計選擇

### 1️⃣ 用 worktree 當隔離單位，而不是容器

比起開五個 VM 或五個 container，worktree 便宜得多：共享同一個 repo 的物件庫，切換只是換目錄。代價是隔離只在檔案層級，Agent 執行的指令仍然共用同一台機器——所以對「跑來路不明的程式」沒有防護力，這是使用時要自己拿捏的邊界。

### 2️⃣ 讓 Agent 也能驅動 Orca

`orca worktree create`、`snapshot`、`click`、`fill` 這組 CLI 意味著反向操作：不只是你用 Orca 管 Agent，Agent 也能呼叫 Orca 去開工作區、截圖、點按鈕。這讓「Agent 操作 GUI」變成可腳本化的流程，也讓 Computer Use 這類功能有了比較穩的落點。

### 3️⃣ 把「離開電腦」當成正式場景

行動 companion app 不是附贈的遙測面板，而是補上 AI 開發最實際的痛點：Agent 跑完通常不是在你盯著螢幕的時候。通知加上「隨手回一句」，等於把等待時間換成可用的時間。

---

# ⚠️ 使用前想想這幾件事

- 🧾 **它不提供模型**：你要自備訂閱或 API Key，Orca 只負責編排。用量與費用仍由各供應商計算。
- 🔐 **本機信任邊界**：Agent 在你的機器上執行真實指令，worktree 只隔離檔案，不隔離權限。
- 📡 **遙測**：官方文件有說明收集哪些匿名使用資料與退出方式，在意的人先讀 [telemetry 文件](https://www.onorca.dev/docs/telemetry)。
- 🔄 **迭代極快**：專案自稱每天發版，README 的功能清單永遠落後，實際能力以[更新日誌](https://github.com/stablyai/orca/releases)為準。

---

# 📚 參考資料

- [stablyai/orca（GitHub）](https://github.com/stablyai/orca)
- [簡體中文 README](https://github.com/stablyai/orca/blob/main/docs/readme/README.zh-CN.md)
- [Orca 官方網站](https://www.onorca.dev/) ｜ [文件](https://www.onorca.dev/docs/) ｜ [下載](https://onorca.dev/download)
