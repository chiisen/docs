---
title: 🔌 OpenCode × AgentMemory：MCP 接遠端服務，再用 Skills 管記憶
date: 2026-11-26 00:00:00
tags:
  - AI
  - Agent
  - OpenCode
  - MCP
  - Memory
---

## 🔌 OpenCode × AgentMemory：MCP 接遠端服務，再用 Skills 管記憶

**✅ 這篇做什麼：記錄 OpenCode v2.3.2 如何連到既有的遠端 AgentMemory，並安裝 17 個全域 Skills。這次只有 MCP + Skills，沒有安裝自動捕捉 hooks，也沒有在 OpenCode 這台機器啟動 AgentMemory server。**

<!-- more -->

AgentMemory 的服務已經在另一台機器上運作。這台 OpenCode 主機要做的是把 MCP client 指向遠端服務，再安裝一組可操作記憶的 Skills。兩邊用途不同：MCP 提供工具連線，Skills 告訴 Agent 何時以及如何儲存、查詢或交接記憶。

# ⚙️ 先找到 OpenCode 真正讀取的設定檔

這台環境的 `OPENCODE_CONFIG_DIR` 指向 `~/.opencode`，所以有效設定檔是 `~/.opencode/opencode.json`。一開始查看的是 `~/.config/opencode/opencode.json`；那份不是目前 CLI 使用的檔案，因此 `opencode mcp list` 才回報 `No MCP servers configured`。

OpenCode v2 的 MCP server 要放在 `mcp.servers` 底下。這次加入的設定如下：

```json
{
  "mcp": {
    "servers": {
      "agentmemory": {
        "type": "local",
        "command": [
          "npx",
          "-y",
          "@agentmemory/agentmemory@latest",
          "mcp"
        ]
      }
    }
  }
}
```

這裡的 `type: local` 指 OpenCode 會在本機啟動一個 MCP stdio 程序，不表示 AgentMemory 的 REST server 也跑在這台機器。命令用 AgentMemory CLI 的 `mcp` 子命令；該程序會讀取 `~/.agentmemory/.env` 裡的 `AGENTMEMORY_URL` 和 `AGENTMEMORY_SECRET`，再連到既有遠端服務。獨立的 `@agentmemory/mcp` shim 在這次環境下不一定會載入 `.env` 裡的全部設定，所以採用 CLI 入口。

OpenCode 官方文件也把手動設定放在 `mcp.servers`，並將 `type: local` 定義為由 OpenCode 啟動的本機命令。設定檔只保留啟動命令，沒有內嵌網址或憑證。

# 🧰 安裝記憶操作 Skills

這次用 `skills` CLI 安裝 AgentMemory 提供的全域 Skills，並關閉匿名遙測：

```bash
DISABLE_TELEMETRY=1 npx skills add rohitg00/agentmemory -g -a opencode -y --copy
```

共安裝 17 個 Skills 到 `~/.agents/skills`。有用來儲存、查詢、回顧、交接、忘記或追蹤記憶的操作 Skills，也有 MCP、REST API、架構等參考 Skills。原本的 `orca-cli` Skill 保留，沒有被覆蓋。

這次沒有裝自動捕捉 plugin 或 hooks。AgentMemory 不會因此自動記下每一則對話或每次工具呼叫；需要由 OpenCode 選用已安裝的 Skills，透過 MCP 工具明確查詢或寫入。若要自動捕捉，得另行安裝支援 OpenCode 的 capture plugin，並另外驗證它會記錄哪些事件。

# ✅ 怎麼確認連線有成功

修改有效設定後執行 `opencode reload`，再查 MCP 狀態：

```bash
opencode mcp list
```

結果顯示 `agentmemory connected`。MCP 資源清單是空的，因此改用唯讀的 `memory_sessions` 查詢；查詢成功，讀到的是遠端原本已有的測試 session。這次沒有新增測試記憶，也沒有啟動本機 AgentMemory server。

# 🔐 憑證放哪裡，以及後續要做什麼

`~/.agentmemory/.env` 存放 `AGENTMEMORY_URL` 和 `AGENTMEMORY_SECRET`，檔案權限設為 `600`，只有擁有者可讀寫。實際值不應放進 OpenCode 設定、文章或診斷輸出。

原設定檔曾經內嵌憑證，後來已改成由 MCP 程序讀取 `.env`；`~/.config/opencode/opencode.json` 也整理為不含憑證的設定。不過，憑證曾出現在工具讀取結果中。建議在遠端服務端輪替 secret，再更新這台機器的 `.env`。本次沒有自行輪替，避免未先更新連線端就讓服務中斷。

---

# 📚 參考資料

- [rohitg00/agentmemory（GitHub）](https://github.com/rohitg00/agentmemory)
- [OpenCode MCP servers 官方文件](https://opencode.ai/docs/mcp-servers/)
- [OpenCode config 官方文件](https://opencode.ai/docs/config/)

本文記錄 OpenCode v2.3.2 與 AgentMemory 0.9.33 的實際設定結果。OpenCode 和 AgentMemory 後續版本可能調整設定格式或啟動方式；新版本請再對照官方文件。
