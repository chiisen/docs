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

**✅ 這篇做什麼：記錄兩個用戶端如何連到同一個遠端 AgentMemory。Grok 使用 stdio wrapper 和 17 個 Skills；Codex 原本就有 MCP 與 session hooks。測試結果分開記，避免把 REST 成功當成 MCP 穩定。**

<!-- more -->

AgentMemory server 已在另一台主機運作。Grok 和 Codex 都連遠端 HTTPS 服務，這兩台用戶端沒有啟動本機記憶 server。公開版以 `<node>.<tailnet>.ts.net` 代替實際 tailnet 主機名，也不列出 secret 值。

# 🐺 Grok：用 wrapper 固定走遠端 MCP

這次使用 Node.js 26.13.0、npm 和 npx。先確認既有的 `~/.agentmemory/.env` 含有 `AGENTMEMORY_URL` 與 `AGENTMEMORY_SECRET`，不輸出變數值，再將檔案權限設為 `600`。

AgentMemory 的 `connect` 沒有 Grok 這個連接器名稱，因此改由 Grok 啟動 `@agentmemory/mcp`。wrapper 腳本放在 `~/.agentmemory/grok-mcp.sh`，權限設為 `700`，啟動時會：

1. 載入 `~/.agentmemory/.env`，檢查網址和 secret 都有值。
2. 只接受設定好的 HTTPS tailnet 網址，可有結尾斜線。
3. 設定 `AGENTMEMORY_FORCE_PROXY=1`，再執行 `npx -y @agentmemory/mcp`。

固定 proxy 有原因。`@agentmemory/mcp` 原本會先探測 `/agentmemory/livez`；探測失敗時，可能改用空的本機 `InMemoryKV`，工具仍可呼叫，資料卻沒有連到遠端。`AGENTMEMORY_FORCE_PROXY=1` 會跳過這次探測並信任設定中的 HTTPS 位址。啟動日誌會留下 `trusting` 和 `proxying` 訊息，方便確認請求走向。

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

`command` 使用絕對路徑，`<user>` 要換成該機器的帳號目錄。設定是透過以下命令加入，再補上 120 秒啟動 timeout：

```bash
grok mcp add --scope user agentmemory -- ~/.agentmemory/grok-mcp.sh
```

Skills 從已下載的套件複製 17 個到 `~/.agentmemory/grok-skills`。沒有放進 `~/.grok/skills`，因為那個目錄是 Git repository。`~/.zshrc` 也會載入同一份 `.env` 給 Codex 和 OpenCode；Grok 從圖形介面或 Orca 啟動時不一定會讀 `.zshrc`，所以 wrapper 仍要自行載入環境變數。

第一次 MCP 握手沒有回應。當時 npx 正在下載 `@agentmemory/mcp`，約 126 秒後仍沒有 MCP 回應；測試結束後還留著 `npm exec @agentmemory/mcp`，已將該程序停止。套件快取完成後再測，MCP 初始化成功。`startup_timeout_sec` 從預設 30 秒調高到 120 秒，讓 npx 冷啟動有較長等待時間；第一次約 126 秒的等待並未通過，不能據此說 120 秒已涵蓋最慢情況。

第二次握手回報 server `agentmemory`、版本 `0.9.33`，並列出 48 個工具，包含 `memory_smart_search`、`memory_recall`、`memory_save` 和 `memory_sessions`。`/agentmemory/livez` 回 HTTP 200，status 為 `ok`；stderr 也確認跳過 livez probe，並 proxy 到設定的遠端 HTTPS 位址。

這台服務使用 BM25。搜尋詞要能對上已存內容；若原文沒有相關字詞，不能期待語意搜尋替你找到它。Grok 也沒有安裝自動捕捉 hooks，對話不會自動寫進 server。查詢要呼叫 `memory_smart_search` 或 `memory_recall`，保存則呼叫 `memory_save`。

# 💻 Codex：設定原本存在，MCP 狀態曾不穩

檢查發現，Codex 連線設定原本就在 `~/.codex/config.toml`，不是這次測試新建的。MCP 命令是 `npx -y @agentmemory/mcp`；`AGENTMEMORY_URL` 和 `AGENTMEMORY_SECRET` 由 `.env` 提供，`~/.codex/hooks.json` 也已有 AgentMemory session hooks。這次沒有重建設定或修改 `.env`。

一開始探測 `localhost:3111` 沒有服務，因為這份設定指向遠端 Tailscale HTTPS 網址。依 `.env` 改測遠端後，唯讀健康檢查回 HTTP 200、status 為 `ok`。沙箱內 DNS 曾失敗，升權網路環境才能完成該次檢查。

MCP 曾成功呼叫：`memory_sessions` 列出一筆 session，`memory_diagnose` 的 sessions 和 memories 檢查也通過。之後重試卻回報 `Transport closed`，所以 MCP 狀態只能判定為不穩定。

另用 REST API 驗證讀寫：測試記憶寫入回 HTTP 201，搜尋回 HTTP 200，並找回唯一測試標記。記憶設定一天後到期。這證明當時遠端 REST 寫入和搜尋可用，不能拿來推論 Codex 的 MCP 傳輸也穩定。雖然 hooks 設定檔裡有 session hooks，本次沒有測它們是否會自動擷取每種事件。

# 🔎 把連線、記憶和自動擷取分開看

這次得到三種不同結果：

| 檢查項目 | 結果 |
| --- | --- |
| 遠端健康狀態 | `/agentmemory/livez` HTTP 200，status `ok` |
| Grok MCP | 第二次握手成功，48 個工具可列出 |
| Codex MCP | 曾成功，之後出現 `Transport closed`，狀態不穩 |
| Codex REST | 寫入 HTTP 201、搜尋 HTTP 200，測試標記命中 |
| 自動擷取 | Grok 未設定；Codex hooks 存在但本次未驗證事件擷取 |

REST、MCP 和 hooks 是不同路徑。REST 成功只證明該次 API 請求可用；MCP 還要看 stdio 行程與握手；hooks 則要另測事件是否真的進到記憶庫。不要把其中一層的通過結果延伸成其他層也正常。

---

# 📚 參考資料

- [rohitg00/agentmemory（GitHub）](https://github.com/rohitg00/agentmemory)
- [AgentMemory 官方文件](https://www.agent-memory.dev/docs)
- [AgentMemory MCP 套件（npm）](https://www.npmjs.com/package/@agentmemory/mcp)

本文記錄 Grok 與 Codex 連接 AgentMemory 0.9.33 的實際設定和測試結果。公開版不含實際 tailnet 主機名、secret 值或本機帳號名稱；不同 MCP client 的啟動與環境變數載入方式請依各自設定確認。
