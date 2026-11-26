---
title: 🧠 Grok 與 Codex 接遠端 AgentMemory：MCP、Skills 和驗證邊界
date: 2026-11-26 00:00:00
tags:
  - AI
  - Agent
  - MCP
  - Grok
  - Codex
---

## 🧠 Grok 與 Codex 接遠端 AgentMemory：MCP、Skills 和驗證邊界

記錄 Grok 與 Codex 連接同一個遠端 AgentMemory 的實測結果。Grok 透過客製 stdio wrapper 搭配 17 個 Skills；Codex 本身已具備 MCP 與 session hooks。實測結果必須分開驗證，不能把 REST 測通就當作 MCP 連線也穩了。

<!-- more -->

AgentMemory 伺服器已經在另一台機器上運作。Grok 和 Codex 這邊都是純用戶端，透過 HTTPS 直連，不另外在本地開 server。公開版以 `<node>.<tailnet>.ts.net` 代替實際主機名，不列出 secret。

# 🐺 Grok：透過 wrapper 固定走遠端 MCP

環境使用 Node.js 26.13.0。先確認 `~/.agentmemory/.env` 包含網址與金鑰，權限鎖 `600`。

AgentMemory 的 `connect` 目前沒有原生支援 Grok，所以直接讓 Grok 啟動 `@agentmemory/mcp`。寫了一支 wrapper 腳本放在 `~/.agentmemory/grok-mcp.sh`（權限設為 `700`），啟動邏輯如下：

1. 載入 `~/.agentmemory/.env`，確認網址與 secret 齊全
2. 檢查是否為設定好的 Tailscale HTTPS 位址
3. 帶上 `AGENTMEMORY_FORCE_PROXY=1`，再執行 `npx -y @agentmemory/mcp`

強制加上 `AGENTMEMORY_FORCE_PROXY=1` 是關鍵。`@agentmemory/mcp` 預設會先 probe `/agentmemory/livez`；要是探測失敗，它會默默退回本地空的 `InMemoryKV`。這時工具清單看似齊全，實際上根本沒連上遠端。加了 force proxy 就會跳過探測，強制導向 HTTPS 位址，啟動日誌也會印出 `trusting` 與 `proxying`，能一眼看出流量跑去哪。

Grok 的 `~/.grok/config.toml` 不放金鑰，只記錄 wrapper 和 Skills 路徑：

```toml
[mcp_servers.agentmemory]
command = "/Users/<user>/.agentmemory/grok-mcp.sh"
args = []
enabled = true
startup_timeout_sec = 120

[skills]
paths = ["~/.agentmemory/grok-skills"]
```

`command` 需使用絕對路徑，`<user>` 替換為實際使用者目錄。加入設定並將啟動 timeout 設為 120 秒：

```bash
grok mcp add --scope user agentmemory -- ~/.agentmemory/grok-mcp.sh
```

Skills 複製了 17 個到 `~/.agentmemory/grok-skills`（避開 `~/.grok/skills` 是因為那是 Git repo）。另外，Grok 若從 GUI 或 Orca 開啟不一定會吃 `~/.zshrc`，所以把 `.env` 的載入寫進 wrapper 最保險。

初次 MCP 握手時踩了個坑：npx 下載套件花了過多時間，等了 126 秒都沒等到回應，背景還卡著殘留行程。手動清掉並等快取就緒後重試才成功。因此在設定中把 `startup_timeout_sec` 從預設 30 秒拉到 120 秒，給冷啟動多點餘裕。

重測後順利辨識出 agentmemory v0.9.33，列出 48 個工具（包含 `memory_smart_search`、`memory_recall`、`memory_save` 和 `memory_sessions` 等），`/agentmemory/livez` 也回傳 200 ok。

遠端目前只開 BM25 關鍵字檢索，查不到就是查不到，不會有向量語意腦補。Grok 這端也沒掛 capture hooks，對話不會自動入庫，想存或查得手動呼叫工具。

# 💻 Codex：設定原本存在，MCP 狀態曾不穩

Codex 本身在 `~/.codex/config.toml` 早就配過 MCP（走 `npx -y @agentmemory/mcp`），`~/.codex/hooks.json` 也有現成的 session hooks，這次不需要重裝。

一開始直覺打了 `localhost:3111` 當然連不上，因為它走的是遠端 Tailscale HTTPS。轉向遠端後健康檢查正常（沙箱內測 DNS 會失敗，需放行網路）。

問題出在穩定度：MCP 第一次呼叫 `memory_sessions` 和 `memory_diagnose` 都過關，但隨後幾次連線卻拋出 `Transport closed`。

改走 REST API 測讀寫則是好的：寫入 201、搜尋 200，測試字串也順利撈回。這說明後端服務本體完全沒問題，卡住的是 Codex 到 MCP 的 stdio 傳輸層。另外，hooks 雖然掛著，這次並未逐一驗證各類事件的自動擷取狀況。

# 🔎 實測結論：別把不同層級混為一談

這次實測整理出三種層次的狀況：

| 檢查項目 | 實測狀況 |
| --- | --- |
| 遠端健康度 | `/agentmemory/livez` 正常回 200，status `ok` |
| Grok MCP | 握手正常，48 個工具可正常列出 |
| Codex MCP | 曾通，但後續偶發 `Transport closed`，穩定度存疑 |
| Codex REST | 讀寫搜尋均正常，成功命中測試標記 |
| 自動捕捉 | Grok 未配；Codex 有 hooks 但尚未驗證事件擷取率 |

REST、MCP、hooks 走的是完全不同的管線。REST 通只代表 API 端點活著；MCP 還牽涉 stdio 行程與 client 端的生命週期；hooks 更是獨立的事件觸發機制。驗證時千萬別看到 API 通了，就推論整條工具鏈都沒問題。

---

# 📚 參考資料

- [rohitg00/agentmemory（GitHub）](https://github.com/rohitg00/agentmemory)
- [AgentMemory 官方文件](https://www.agent-memory.dev/docs)
- [AgentMemory MCP 套件（npm）](https://www.npmjs.com/package/@agentmemory/mcp)

本文記錄 Grok 與 Codex 連接 AgentMemory 0.9.33 的實際設定和測試結果。公開版不含實際 tailnet 主機名、secret 值或本機帳號名稱；不同 MCP client 的啟動與環境變數載入方式請依各自設定確認。
