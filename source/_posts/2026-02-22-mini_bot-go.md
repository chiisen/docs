---
title: 🐹 mini_bot.go
date: 2026-02-22 00:00:00
tags:
  - AI
  - Agent
  - Go
  - Telegram
---

## mini_bot.go

**🐹 MiniBot.go - 極致輕量、無依賴的 Go 實作 Personal AI Agent**

專注於低資源消耗（📦 單一執行檔 < 15MB、⚡ RAM < 10MB）提供強大的本地工具呼叫與遠端通訊功能。

<!-- more -->

# 🌟 核心特色
- 🐹 **純 Go 實作**：除了標準函式庫外，零第三方肥大框架依賴。
- 🔧 **本地工具呼叫**：內建 Sandbox 沙箱機制，允許 AI 安全地讀寫檔案、執行指令。
- 🔍 **內建網頁搜尋**：透過 DuckDuckGo API，即使不設定額外的 Search API Key 也能讓 AI 上網。
- 🤝 **多供應商支援**：採用 OpenAI 協定標準，對接 OpenAI、Ollama、MiniMax、Groq 等。
- 🚀 **跨平台與容器化**：支援 Linux / Windows / macOS 交叉編譯，並提供極小型 Alpine Dockerfile。
- 📲 **Telegram 整合**：內建 Long Polling 接收器與白名單過濾機制。

# 🚀 快速開始

### 1. 📦 安裝與編譯
請確保您的系統已安裝 Go 1.21+。

```bash
git clone https://github.com/chiisen/mini_bot.go.git
cd mini_bot.go

# 編譯
go build -o minibot ./cmd/app/
```

### 2. 🛠️ 環境初始化 (Onboard)
執行 onboard 指令來建立工作區與設定檔：

```bash
./minibot onboard
```

此指令將建立 `~/.minibot.go/` 資料夾並初始化 API Key 與 Agent 人格模板。

# 🎮 使用方式
- 🎯 **單次指令問答**：`./minibot agent -m "你好"`
- 💬 **本地互動式對話**：`./minibot chat`
- 🌐 **啟動對外伺服器**：`./minibot gateway`
- 📱 **啟動 Telegram Bot**：`./minibot telegram`

# 🏗️ 專案宗旨
`mini_bot.go` 代表了對極致效能與簡約代碼的追求。適合需要將 AI Agent 部署在低配筆電、樹莓派或極簡容器環境中的開發者。

### 我的 Github 專案

[🔗 我的 Github 專案: mini_bot.go](https://github.com/chiisen/mini_bot.go)  
✅最小可執行的 Go AI Agent，歡迎 Star 🌟 與 Fork 🍴！

---
