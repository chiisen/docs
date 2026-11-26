---
title: 🧠 agentmemory：讓 AI Coding Agent 接著上次的工作記憶
date: 2026-11-26 00:00:00
tags:
  - AI
  - Agent
  - Memory
  - MCP
---

## 🧠 agentmemory：讓 AI Coding Agent 接著上次的工作記憶

**✅ 這篇做什麼：介紹 [rohitg00/agentmemory](https://github.com/rohitg00/agentmemory)。它把 coding session 裡的觀察、決策和教訓存起來，讓不同的 AI Agent 下次能搜尋並取回。**

<!-- more -->

換一個對話，AI 可能就不知道上一輪查到哪裡、為什麼選這個套件，或還有哪些測試不能漏。把事情寫進 `AGENTS.md` 或 `MEMORY.md` 有用，但內容得自己整理；如果每次都把整份檔案放進 context，也會佔掉 token。

[agentmemory](https://github.com/rohitg00/agentmemory) 提供一個本機記憶服務，讓多種 Agent 透過 plugin、hook 或 MCP 收集和查詢工作歷史。它把舊的決策和調查紀錄變成可搜尋的資料。

# 🧭 記憶怎麼進來，又怎麼回去

有支援 hook 的整合會走這個流程：

1. Agent 開始工作或呼叫工具時，整合把事件送進 agentmemory。
2. 服務先過濾密鑰等敏感資料，再保存觀察紀錄。
3. 新工作階段開始時，依專案和問題搜尋舊紀錄，把符合的內容交還給 Agent。

預設搜尋使用 BM25 關鍵字檢索，不需要模型 API key。設定 embedding provider 後，才會啟用向量語意搜尋；如果已建立知識圖譜資料，smart search 也會參考圖上的關係。搜尋結果會合併排序；未設定 embedding 時，仍是關鍵字搜尋。

官方文件把資料分成幾種用途：

| 類型 | 適合留什麼 |
| --- | --- |
| Observation | 一次 session 發生了什麼，例如工具呼叫及結果 |
| Memory | 之後還會用到的決策或事實 |
| Lesson | 從錯誤中整理出的做事規則 |
| Crystal | 完成一段工作後留下的精簡摘要 |

例如，工具輸出可以留作 observation；反覆適用的修正方式則整理成 lesson，之後比較容易找回來。

# 🔌 不同 Agent 共用同一份記憶

agentmemory 提供 REST API 和 MCP server，部分 Agent 另有原生 plugin 或 hook。各整合能擷取的事件不同：有些會記錄工具呼叫前後，有些主要讓 Agent 透過 MCP 主動儲存與搜尋。使用前要查對應整合的說明，確認它實際會收集哪些事件。

一般情況下，服務和資料都留在本機。macOS 的預設資料位置是 `~/Library/Application Support/agentmemory`；API 預設綁在 `localhost:3111`，記憶檢視器在 `localhost:3113`。若要讓遠端 Agent 或團隊共用，應依部署文件設定認證和 TLS，並限制 API 與檢視器的網路存取。

想試的話，先開一個終端機啟動服務：

```bash
npx -y @agentmemory/agentmemory@latest
```

再開另一個終端機連接 Agent，例如 Codex：

```bash
npx -y @agentmemory/agentmemory@latest connect codex
```

Node.js 需求是 20 以上。第一次可以用 `demo` 建立範例資料，再到 `http://localhost:3113` 看服務記錄了什麼、搜尋結果是否符合預期。實際安裝選項和 hook 支援仍要看所用 Agent 的整合方式。

# 🌐 實際部署：Tailscale Serve 接遠端 Agent

這台 Mac 的服務用 `agentmemory 0.9.33`，執行檔是 `/opt/homebrew/bin/agentmemory`。初始化時執行：

```bash
CI=1 npx -y @agentmemory/agentmemory@latest init
```

沒有填 LLM 金鑰，也沒有指定 `AGENTMEMORY_DATA_DIR`，所以資料留在 macOS 預設位置：`~/Library/Application Support/agentmemory/state_store.db`。第一次啟動時，agentmemory 下載 iii 0.25.1，以及 worker state 0.22.22、http 0.22.8。REST、viewer 和引擎都只聽 `127.0.0.1`；當時 `/agentmemory/livez` 和 status 都回報 healthy，記憶筆數是 0。

接著再次執行 `npx -y @agentmemory/agentmemory@latest`，在提示中同意全域安裝，之後新 shell 就能直接使用 `agentmemory` 指令。服務資料放在 `state_store.db`；同目錄的 `compose-state` 約 46 MB，包含 worker state 和日誌。金鑰放在 `~/.agentmemory/secret`，權限是 `0600`，資料目錄還有一份同內容的副本。文章不列出金鑰值。

這份設定維持 **keyless + BM25-only**：壓縮、合併、知識圖譜和對話注入都關閉。從 `/tmp/agentmemory-demo` 跑過 `demo` 後，資料庫裡留下 3 個工作階段、6 筆 observation。關鍵字 `jwt auth middleware` 找到 3 筆，`rate limiting` 找到 1 筆；`database performance optimization` 找到 0 筆。這是關鍵字索引的實測結果，沒有開啟向量搜尋。

遠端入口用 Tailscale Serve。REST 和 viewer 繼續綁在本機，Tailscale 只把 HTTPS 的 `/` 轉到 `http://127.0.0.1:3111`；viewer `3113` 和引擎 `49134` 都沒有對外轉發。第一次執行 `tailscale serve --bg 3111` 時，tailnet 尚未允許 Serve，因此被拒。管理員在 Tailscale 授權 Serve 後再執行，`tailscale serve status` 才顯示 tailnet-only 的 HTTPS 入口。**只開 Serve，不開 Funnel**，區網也連不到這個服務。

公開文章不放實際 tailnet 主機名和 IP，以下用佔位符表示：

```bash
export AGENTMEMORY_URL="https://<node>.<tailnet>.ts.net"
read -r -s AGENTMEMORY_SECRET
export AGENTMEMORY_SECRET
npx -y @agentmemory/agentmemory@latest connect codex
unset AGENTMEMORY_SECRET
```

另一台 Mac 或其他主機只要 Node.js 20 以上，並設定服務網址和 secret 即可；**不要在客戶端再啟動一份 agentmemory server**。在 `read` 提示時，貼上服務主機 `~/.agentmemory/secret` 的內容再按 Enter；HTTPS 網址不會自動讀取那個檔案。`connect` 的 agent 名稱可換成 `claude-code`、`codex` 或 `cursor`。服務所在的這台 Mac，自己的 Agent 繼續用 `http://127.0.0.1:3111`。

日常查看和重啟：

```bash
agentmemory status
agentmemory stop
agentmemory
/Applications/Tailscale.app/Contents/MacOS/Tailscale serve status
```

agentmemory 沒有設定開機自啟。Mac 重開後要重新啟動 `agentmemory`，再確認 Serve 仍轉到 3111。Tailscale Serve 設定和 agentmemory 行程分開管理；重啟其中一邊，不會替另一邊完成啟動。

# 🧩 它適合補哪一塊

固定不變的專案規範適合放在 `AGENTS.md` 或 `CLAUDE.md`，每次工作都能直接讀到。agentmemory 則可保存工作中累積的內容，例如曾經查過的根因、做過的取捨、跨 session 的交接，以及從專案事件整理出的教訓。

擷取範圍取決於 Agent 提供的 hook 和 plugin；搜尋品質也受記憶內容、索引和設定影響。官方說明指出，啟用遠端模型 provider 或分享功能時，資料才會離開本機。密鑰過濾可以減少敏感資料被保存的機會，使用前仍應確認整合會擷取什麼，以及是否啟用了遠端服務。

如果你要讓 Codex、Claude Code 等工具接續同一份程式調查，可以先接一個 Agent，確認哪些事件有被保存、搜尋能不能找回，再決定是否擴到其他工具。規則不多、工作也集中在同一段對話時，一份短小的專案說明通常就夠用。

---

# 📚 參考資料

- [rohitg00/agentmemory（GitHub）](https://github.com/rohitg00/agentmemory)
- [AgentMemory 官方文件](https://www.agent-memory.dev/docs)
- [安裝與連接 Agent](https://www.agent-memory.dev/docs/quickstart)

本文依 GitHub README 與官方文件於 2026-11-26 查閱的內容整理；各 Agent 的支援事件及安裝方式可能不同，請以對應整合文件為準。
