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

記錄 OpenCode v2.3.2 連接既有遠端 AgentMemory 的實作流程，並安裝 17 個全域 Skills。這次配置只處理 MCP 工具與 Skills，沒裝自動捕捉 hooks，也沒在本地啟動 AgentMemory 伺服器。

<!-- more -->

AgentMemory 已經在另一台機器上跑。這台主機上的 OpenCode 只要把 MCP client 指向遠端，再裝好對應的 Skills 就行。兩者分工很明確：MCP 提供工具連線能力，Skills 則指引 Agent 什麼時候該存、查或交接記憶。

# ⚙️ 找到 OpenCode 生效的設定檔

環境裡的 `OPENCODE_CONFIG_DIR` 指向 `~/.opencode`，實際生效的設定檔是 `~/.opencode/opencode.json`。一開始習慣先看 `~/.config/opencode/opencode.json`，結果 `opencode mcp list` 一直顯示 `No MCP servers configured`，才發現看錯位置。

OpenCode v2 的 MCP server 需寫在 `mcp.servers` 底下。加入的設定如下：

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

設定裡的 `type: local` 代表 OpenCode 會在本地拉起一個 MCP stdio 行程，不是指 AgentMemory 本身跑在本機。命令直接呼叫 CLI 的 `mcp` 子命令，讓它讀取 `~/.agentmemory/.env` 中的 `AGENTMEMORY_URL` 與 `AGENTMEMORY_SECRET` 連往遠端。之所以不用獨立的 `@agentmemory/mcp` shim，是因為在當前環境下它不一定會吃滿 `.env` 設定，走 CLI 入口比較穩。

設定檔只保留啟動指令，網址與金鑰都不明文寫在裡面。

# 🧰 安裝記憶操作 Skills

使用 `skills` CLI 安裝 AgentMemory 的全域 Skills，並關閉匿名遙測：

```bash
DISABLE_TELEMETRY=1 npx skills add rohitg00/agentmemory -g -a opencode -y --copy
```

總共 17 個 Skills 會寫入 `~/.agents/skills`，涵蓋存取、查詢、回顧、交接等操作，以及 MCP 與架構的參考說明。原本專案裡的 `orca-cli` 沒被動到。

注意這裡沒裝自動捕捉 plugin 或 hooks。AgentMemory 不會自動把每段對話或工具呼叫塞進資料庫；必須由 OpenCode 透過安裝好的 Skills，主動呼叫 MCP 工具來查詢或寫入。如果需要自動擷取，得另外掛 capture plugin 並單獨驗證。

# ✅ 驗證連線

改完設定後跑 `opencode reload`，接著檢查狀態：

```bash
opencode mcp list
```

終端回報 `agentmemory connected`。因為 MCP 資源清單本來就是空的，改調唯讀的 `memory_sessions` 測試；順利撈回遠端現有的測試 session，確認通訊正常。這次沒寫入測試記憶，也沒在本機開 server。

# 🔐 憑證管理與後續

連線用的 `AGENTMEMORY_URL` 與 `AGENTMEMORY_SECRET` 放在 `~/.agentmemory/.env`，權限鎖 `600`。敏感資訊不要寫進 OpenCode 設定檔或診斷紀錄。

先前除錯時曾把憑證寫入設定，後來已全面改成由 MCP 行程載入 `.env`，並清理掉設定檔內的明文。不過因為金鑰曾暴露在工具輸出過，穩妥起見應該在伺服端輪替 secret，再回頭更新本機 `.env`。

---

# 📚 參考資料

- [rohitg00/agentmemory（GitHub）](https://github.com/rohitg00/agentmemory)
- [OpenCode MCP servers 官方文件](https://opencode.ai/docs/mcp-servers/)
- [OpenCode config 官方文件](https://opencode.ai/docs/config/)

本文記錄 OpenCode v2.3.2 與 AgentMemory 0.9.33 的實際設定結果。後續版本若調整設定格式或啟動方式，請以官方文件為準。
