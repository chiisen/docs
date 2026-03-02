---
title: 🦀 mini_bot.rs
date: 2026-03-02 00:00:00
tags:
  - AI
  - Agent
  - Rust
---

## mini_bot.rs

**🦀 MiniBot.rs - 100% Rust 實作的高效能、極致輕量 AI Agent 執行框架**

繼 Python 與 Go 版本後，最強大的極致效能版本。專注於極低資源消耗（⚡ RAM < 5MB）與快速啟動（冷啟動 < 10ms），提供安全隔離的工具呼叫環境。

<!-- more -->

# 🌟 核心特色
- 🦀 **純 Rust 實作**：利用 Rust 的記憶體安全與高效能特性，打造極簡內核。
- ⚡ **極致效能**：執行時期記憶體消耗小於 5MB，啟動時間小於 10ms。
- 🔧 **高度模組化**：採用 Trait 驅動架構，所有元件（Provider, Tools, Memory）皆可輕鬆擴展與替換。
- 🛡️ **安全預設**：嚴格的沙箱隔離機制，確保 AI 執行的 Shell 指令與檔案操作在安全範圍內。
- 🌍 **跨平台支援**：完美支援 ARM (樹莓派)、x86 以及 RISC-V 架構。

# 🚀 快速開始

### 1. 📦 建置專案
請確保您的系統已安裝 Rust 與 Cargo。

```bash
git clone https://github.com/chiisen/mini_bot.rs.git
cd mini_bot.rs

# 建置發佈版本
cargo build --release
```

### 2. 🔥 使用方式
```bash
# 啟動互動式對話
./target/release/mini_bot.rs agent

# 或是直接傳送訊息
./target/release/mini_bot.rs agent --message "你好，請幫我分析目前的專案目錄"

# 啟動 Webhook 閘道器
./target/release/mini_bot.rs gateway --port 3000
```

# 🛠️ 核心工具集 (MVP)
- **Shell Tool**：安全執行系統命令。
- **File Tool**：檔案系統讀寫操作。
- **Web Fetch Tool**：(研發中) 擷取並解析網頁內容。

# 🏗️ 專案宗旨
`mini_bot.rs` 是對「Minimalist AI Agent」概念的終極體現。它不只是一個專案，更是一個展示如何用最少的代碼、最低的資源消耗，實現最強大生產力工具的範例。

### 我的 Github 專案

[🔗 我的 Github 專案: mini_bot.rs](https://github.com/chiisen/mini_bot.rs)  
✅最小可執行的 Rust AI Agent，體驗極致效能，歡迎 Star 🌟！

---
