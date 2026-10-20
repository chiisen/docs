---
title: 🐑 herdr：住在終端裡的「Agent 復用器」，把多個 AI Agent 收進一個視窗
date: 2026-07-02 00:00:00
tags:
  - AI
  - Agent
  - Rust
  - Terminal
---

## 🐑 herdr：住在終端裡的「Agent 復用器」，把多個 AI Agent 收進一個視窗

**✅ 這篇做什麼：介紹 herdrdev/herdr 這個 Rust 寫的終端工具。它像 tmux 管窗格那樣管 AI Agent：看得到每個 Agent 是 blocked、working 還是 done，斷線後工作仍在背景跑，還能把本機與遠端機器的 Agent 收進同一個列表。**

<!-- more -->

跑 coding agent 跑久了，遲早會遇到同一個困境：三個 Agent 開在三個終端機分頁，你不知道哪一個卡住等你回答、哪一個還在跑、哪一個早就完成了。切來切去查狀態的時間，往往比寫程式還多。

`herdr` 的作者把它定義成 **智能體復用器（agent multiplexer）**，把「多工管理」這件 tmux 已經做過的事，針對 AI Agent 重新設計一次。單一 Rust 二進位檔，沒有 Electron，跑在你原本就在用的終端機裡。

---

# 📦 專案速覽

| 項目 | 內容 |
| --- | --- |
| 專案 | `herdrdev/herdr` |
| 定位 | 終端內的 Agent 多工管理器 |
| 語言 / 授權 | Rust / Apache-2.0 |
| 平台 | macOS、Linux、Windows（另有受端點保護的 Windows 版本） |
| 建立時間 | 2026-03（撰稿時約 4.5 萬 Star、3.6 千 Fork） |
| 官網 / 文件 | [herdr.dev](https://herdr.dev) |

---

# 🌟 四個核心設計

### 1️⃣ 狀態一眼可見，而且是「真的終端」

每個 Agent 會被標成 `blocked`、`working`、`done`。重點在後面那句：它顯示的是真實終端視圖，不是把輸出包成一層摘要再轉述給你。

摘要可能漏掉關鍵錯誤訊息，真實畫面不會。Agent 看到什麼，你就看到什麼。

### 2️⃣ 分離後，工作繼續跑

`ctrl+b q` 分離，關掉筆電或 SSH 斷線都不影響。背景伺服器會保持終端繼續執行，之後 `herdr` 重新連回去。

有個邊界要先講清楚。機器或伺服器重開之後，herdr 會恢復已保存的布局，並可恢復「受支持」的 Agent 工作階段，但原本的行程不會保留。這是終端多工器的共通限制，不是 herdr 的疏漏。

### 3️⃣ 多台機器，一個視窗

本機的工作區與已保存的 SSH 機器可以並排在同一個介面，Agent 列表是彙總的，各條連線獨立重連。對於「本機開發、遠端跑重活」的人，這比開四個 SSH session 乾淨得多。

### 4️⃣ Agent 也能反過來用 herdr

它提供純 socket API：Agent 可以自己建立窗格、讀取其他窗格的輸出、互相等待。你可以讓一個「監督者 Agent」去開分頁、派活、收結果，不必由人類手動分割。

正在開發 Agent 的人，官方也給了[接入指南](https://herdr.dev/zh-cn/docs/add-herdr-support/)。

---

# ⌨️ 操作方式：鍵盤與滑鼠都是主角

很多終端工具會強迫你選邊站：要嘛全鍵盤（tmux 派），要嘛全滑鼠（GUI 派）。herdr 兩者都支援：

- **tmux 風格前綴鍵**：熟 tmux 的人幾乎不用重學。
- **點擊、拖曳、分割**：不確定快捷鍵時，直接用滑鼠比較快。

插件機制可以擴充窗格與工作流，官方有[插件市場](https://herdr.dev/plugins/)。

---

# 🚀 安裝與開始

```bash
# 一行安裝
curl -fsSL https://herdr.dev/install.sh | sh
```

其他管道：

```bash
brew install herdr          # macOS / Homebrew
mise use -g herdr           # mise
```

Windows 用 PowerShell：

```powershell
powershell -ExecutionPolicy Bypass -c "irm https://herdr.dev/install.ps1 | iex"
```

也可以直接抓 [release 二進位檔](https://github.com/herdrdev/herdr/releases)。

裝好後，在你工作的目錄啟動：

```bash
herdr
```

跑你的 Agent、分割窗格，然後安心離開。`ctrl+b q` 分離，`herdr` 重新連線。

---

# 🧭 和 GUI 型 Agent 管理器怎麼選？

同類需求最近出現兩種路線，差別在介面哲學：

| | 終端型（如 herdr） | GUI 型（如桌面 IDE） |
| --- | --- | --- |
| 介面 | 你原本的終端機 | 獨立視窗 |
| 資源 | 單一 Rust 二進位檔 | 通常較重（含瀏覽器引擎） |
| 隔離 | 靠窗格與機器 | 常搭配 git worktree |
| 遠端 | 內建 SSH 機器管理 | 視產品而定 |
| 適合 | 已經活在終端裡的人 | 想要圖形化 diff / 看板的人 |

兩種都有人用。如果你 90% 的時間都在終端機裡，herdr 的摩擦力低一些；需要圖形化 diff 檢視或 issue 看板，GUI 工具還是順手。

---

# 🔍 幾個值得注意的細節

herdr 是單一 Rust 二進位檔，不綁 Electron、不用先裝執行環境，SSH 進遠端機器也能直接跑。對「Agent 跑在遠端」這種用法，差別很明顯。

socket API 是另一個重點。能被 Agent 呼叫的工具，才有機會進入自動化流程；只能給人點的工具，最後都停在自己手上。

狀態標籤也別當萬靈丹。`blocked`、`working`、`done` 屬於啟發式判斷，Agent 在等網路或狂吐日誌時都可能誤判，重要關卡還是自己看一眼畫面。

---

# 📚 參考資料

- [herdrdev/herdr（GitHub）](https://github.com/herdrdev/herdr)
- [簡體中文 README](https://github.com/herdrdev/herdr/blob/master/README.zh-CN.md)
- [官方網站](https://herdr.dev) ｜ [快速開始](https://herdr.dev/zh-cn/docs/quick-start/) ｜ [文件](https://herdr.dev/zh-cn/docs/)
