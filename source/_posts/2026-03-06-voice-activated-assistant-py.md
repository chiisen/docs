---
title: 🎙️ 語音互動助理 (Voice-Activated Assistant)
date: 2026-03-06 00:00:00
tags:
  - Python
  - AI
  - ASR
  - TTS
---

## 語音互動助理 (Voice-Activated Assistant) 🎙️

**✅ 基於 Python 開發的高效能、隱私優先的本地端語音代理系統。**

這是一個基於 Python 開發的高效能、隱私優先的本地端語音代理系統。透過整合最新的 Qwen3 ASR 與 TTS 技術，實現流暢的語音指令識別與自動化回應。

![VoiceActivatedAssistant](../images/VoiceActivatedAssistant.png)

<!-- more -->

# 🌟 核心特性
- 🚀 **極速本地推論**：使用 Qwen3-ASR 與 Qwen3-TTS，支援串流輸出，具備極低首包延遲。
- 🗣️ **純淨自然發音**：內建 OpenCC 簡繁轉換機制，確保發音皆為標準的國語/普通話，避免 TTS 模型朗讀繁體中文時產生混淆。
- 🤫 **隱私與安全**：語音轉文字 (ASR) 結果僅暫存於記憶體 (RAM)，程式結束後自動釋放，不留任何磁碟紀錄。
- 🧠 **智慧停頓偵測 (VAD)**：內建 0.8 秒連續靜音判斷，精準識別一段話的結束點。
- 🚦 **狀態機協調**：當 TTS 播放時自動暫停 ASR 監聽，完美解決「自己聽到自己講話」的自我回饋問題。
- 🛠️ **JSON 驅動規則**：透過簡單的 JSON 設定檔定義關鍵字、優先序與多樣化的回覆模式。

# 🛠️ 技術棧
- **Language**: Python 3.10+
- **ASR**: Qwen3-ASR (1.7B / 0.6B)
- **TTS**: Qwen3-TTS
- **VAD**: Silero VAD
- **Concurrency**: Threading + Python Queue

# 🏗️ 專案宗旨
`voice-activated-assistant.py` 的開發初衷是為了解決依賴雲端 API 的延遲與隱私問題，打造一個完全在本地端執行的 AI 語音代理系統。它專注於快速響應與離線可用性，讓使用者能夠安全、流暢地與本地智慧互動。

### 我的 Github 專案

[🔗 我的 Github 專案: voice-activated-assistant.py](https://github.com/chiisen/voice-activated-assistant.py)  
✅ 一個隱私優先的本地端語音代理系統。歡迎 Star 🌟 收藏與提供建議！

---
