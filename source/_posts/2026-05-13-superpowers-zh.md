---
title: 🦸 superpowers-zh：中文增強的 skill 框架，也是觀摩 prompt 設計的好教材
date: 2026-05-13 00:00:00
tags:
  - AI
  - Agent
  - Claude
  - Tool
---

## 🦸 superpowers-zh：中文增強的 skill 框架，也是觀摩 prompt 設計的好教材

**✅ 這篇做什麼：[`obra/superpowers`](https://github.com/obra/superpowers) 的英文版早就存在，這篇介紹它的中文增強版 [`jnMetaCode/superpowers-zh`](https://github.com/jnMetaCode/superpowers-zh)。除了中文友善與一鍵適配 26 款工具，更值得看的是它把 21 個 skill 當成教材，讓人觀摩「要怎麼寫 prompt，模型才會穩定照著做」。**

<!-- more -->

不少人已經在用英文版 `superpowers`，看到中文版的第一個反應大概是「這不是重複造輪子嗎」。我一開始也這樣想。實際裝起來用了一陣子，覺得還是值得寫一篇，因為它處理的問題跟英文版不太一樣。

英文上游沒有過時，它就是方法論的源頭，版本也跟得最緊。中文版多出來的是中文語境的表達、26 款工具的一鍵安裝，以及四個針對中文團隊另外寫的 skill。

我推薦它的另一個理由更實際：21 個 `SKILL.md` 本身就是 prompt 設計的範例。翻過一輪之後，我對「怎麼寫指令模型才會照做」總算有點具體概念，不再只是憑感覺調字句。

這個部落格專案本身就在用它（`.agents/skills/` 下有 20 個 skill），所以下面有些觀察是實際跑出來的，不只是讀 README。

---

# 📦 專案速覽

| 項目 | 內容 |
| --- | --- |
| 專案 | `jnMetaCode/superpowers-zh` |
| 定位 | AI 編程工作方法論 skills 的中文增強版 |
| 上游 | [obra/superpowers](https://github.com/obra/superpowers)（MIT） |
| 語言 / 授權 | JavaScript / MIT |
| 版本 | v1.7.14（撰稿時） |
| 建立時間 | 2026-03（撰稿時約 8,300 Star、775 Fork） |

---

# 🔧 還有在維護嗎？

中文 fork 最怕的就是翻譯完就放生，所以我先查了這件事。2026-10-20 的狀況：

| 指標 | 數據 |
| --- | --- |
| 最後 commit | 2026-10-19（前一天） |
| 近 30 天 commit | 34 筆 |
| 最新 release | v1.7.14（2026-09-28） |
| 發版節奏 | 約每月一次（05-10 / 06-20 / 07-13 / 07-17 / 09-28） |
| npm latest | `1.7.14`，與 GitHub release 一致 |

有在動，但落後上游一個世代，而且幾乎是一個人維護。

- 上游 `obra/superpowers` 已到 v7.0.0（2026-10-10，brainstorming 重寫）；這邊已發佈的 v1.7.14 對應的是 v6.4.2
- 主幹已經有 `feat(brainstorming): 同步上游 v7.0.0 重寫` 這筆 commit，但 release notes 標著「未發布」。想用新功能，得等下一次發版或自己 clone 主幹
- commit 幾乎都來自同一帳號。專案本身有在動，風險在於維護者只有一位

本機 `.agents/skills/` 只有 20 個 skill，缺 `diagnosing-superpowers`（上游 v6.4.1 才加入），算一算那份安裝至少兩個月沒更新了。重跑一次 `npx superpowers-zh` 就會補齊。

---

# 🇹🇼 用過英文版的人，該看的是這幾點

## 翻譯之外還動了什麼

| 維度 | 英文上游 | 中文版 |
| --- | --- | --- |
| 支援工具 | 6 款 | 26 款 |
| 安裝方式 | 每款一條不同的 plugin 指令 | 一條 `npx` 自動偵測專案裡用了什麼 |
| 語言 | 英文 | 中文（技術術語保留英文） |
| Git 平台 | GitHub 為主 | GitHub + Gitee + Coding + 極狐 GitLab + CNB |
| CI 範例 | GitHub Actions | 加上 Gitee Go、Coding CI、極狐 CI、`.cnb.yml` |
| Code Review 風格 | 西方直白 | 中文分級話術（三級標註） |
| Commit 規範 | 無 | Conventional Commits 中文適配 |
| 文件排版規範 | 無 | 中英文空格、全半形標點、術語保留 |
| `mcp-builder` / `workflow-runner` | 已移除 | fork 保留繼續維護 |
| 卸載方式 | 手動刪檔 | 哨兵註解精確切除，不誤刪你的內容 |

## 四個原創 skill

這四個是另外寫的，不是翻譯。

| Skill | 解什麼問題 | 實際內容（節錄） |
| --- | --- | --- |
| `chinese-code-review` | 國內團隊 review 的兩個極端：太客氣放過 bug、照搬西方直白讓人下不來臺 | 「你必須改成 X」改成「建議考慮用 X，因為 Y」；反饋分三級：必須修復、建議修改、僅供參考 |
| `chinese-commit-conventions` | commit 中英混雜、沒有團隊規範 | Conventional Commits 中文 type 對照（feat、fix、docs、perf 等）、commitlint 加 husky 加 commitizen 的中文模板、conventional-changelog 中文配置 |
| `chinese-documentation` | 中英文擠在一起、全形半形混用、一股機翻味 | 中英文之間加空格、全形標點、術語保留英文，對齊《中文文案排版指北》 |
| `chinese-git-workflow` | GitHub 不穩、CI 方案照搬國外水土不服 | Gitee、Coding、極狐 GitLab、CNB、GitHub 的平台對比（CI 方案、MR 或 PR、製品庫、適合場景） |

講白一點，這四個的預設場景偏中國大陸生態。台灣團隊大概只會用到前三個（review 語氣、commit 規範、文件排版）。`chinese-git-workflow` 除非真的在用 Gitee 或極狐，否則參考價值有限。

## 術語不硬翻

技術術語保留英文，skill、commit、worktree 這些不翻，只翻說明文字。硬翻反而更難懂，這點比想像中重要。

簡中與繁中兩份 README 都有。

官方文件說，15 個翻譯 skill 只有兩處增量，會在正文標「本節是 superpowers-zh 的增量內容」，audit 也會讓未標註的增量直接報錯。如果你擔心 fork 偷偷改方法論，這條值得記著：真要對照，看那兩處就好。

## 連什麼時候該用中文 skill 都寫成路由

`using-superpowers` 裡有一張路由表，偵測到中文註釋、中文 README、`.gitee` 目錄或中文 commit 歷史，就優先叫用中文系列 skill。

它還寫明中文 skill 與翻譯 skill 是疊加使用，不互斥。做 code review 時，`requesting-code-review` 管流程、`chinese-code-review` 管語氣，比二選一的設計合理。

小提醒：四個 `chinese-*` 的 `description` 都寫「僅在使用者顯式呼叫時才用」，跟路由表的「必須優先呼叫」語氣並不一致。實務上把它們當參考資料，需要時打 `/chinese-xxx` 最保險。

---

# 🧩 21 個 skill 的組成

| 類別 | 數量 | 說明 |
| --- | --- | --- |
| 翻譯自上游 | 15 | 方法論核心，逐節翻譯，技術術語保留英文 |
| 中國原創 | 4 | 上面那四個 `chinese-*` |
| 上游歷史保留 | 2 | `mcp-builder`、`workflow-runner`（上游已移除，此 fork 續維護） |

15 個翻譯 skill 涵蓋寫程式最常出事的環節：頭腦風暴、寫計畫、執行計畫、TDD、系統化除錯、code review、完成前驗證、git worktree、撰寫 skill。

---

# 🎓 觀摩：skill 的 prompt 怎麼寫

這些招數在任何「寫給 LLM 的指令」場景都用得上。

### 1. `description` 只寫「什麼時候用」

每個 skill 的 frontmatter 長這樣：

```yaml
---
name: systematic-debugging
description: 遇到任何 bug、測試失敗或異常行為時使用，在提出修復方案之前執行
---
```

它沒有寫「先重現、再隔離、再假設、最後修復」。這是刻意的，`writing-skills` 裡明白寫著：描述裡如果摘要了工作流，模型會照著那句摘要做，不去讀正文。

原則：`description` 是觸發器（trigger），不是目錄（table of contents）。

同一份教材還舉了三個反例：寫「幫助處理測試」太抽象、用第一人稱、把關鍵詞塞進無關的 skill。這幾種寫法在專案裡很常見。

### 2. 用「鐵律」把邊界釘死

`systematic-debugging` 開頭是這樣：

```
不做根因調查，不許提修復方案
```

`verification-before-completion` 則是：

```
沒有新鮮的驗證證據，不許宣稱完成
```

一行、放在 code block 裡、語氣是禁止句。效果比一整段「請務必注意」強得多，因為它給模型一個可以自我檢查的判準，而不是模糊的期待。

### 3. 用標籤做硬門檻

`brainstorming` 裡有一段被 `<HARD-GATE>` 包住：

> 在你展示設計方案並獲得使用者批准之前，不要呼叫任何實現技能、編寫任何程式碼、搭建任何專案或採取任何實現行動。這適用於所有專案，無論看起來多簡單。

XML 標籤本身沒有魔法，但它把規則從敘述裡切出來，也給模型一個「這裡是規則區、不是建議區」的結構訊號。

### 4. 步驟寫成可勾選的檢查清單

`brainstorming` 的流程不是散文，是編號任務：

1. 探索專案脈絡（檔案、文件、最近的 commit）
2. 提供視覺伴侶（若涉及視覺問題）
3. 提出澄清問題，每次一個
4. 提出 2 到 3 種方案，附權衡分析與推薦
5. 展示設計，每節展示後取得批准
6. 寫入設計文件並 commit
7. 規格自檢（佔位符、矛盾、模糊性、範圍）
8. 請使用者審查書面規格
9. 過渡到實作，呼叫 `writing-plans`

少了「每次一個」，模型會一次丟五個問題，使用者只回一半。

### 5. 直接堵住「合理化」

`verification-before-completion` 有一張表：

| 藉口 | 現實 |
| --- | --- |
| 「應該能行了」 | 執行驗證命令 |
| 「我有信心」 | 信心 ≠ 證據 |
| 「就這一次」 | 沒有例外 |
| 「Linter 通過了」 | Linter ≠ 編譯器 |
| 「代理說成功了」 | 獨立驗證 |
| 「我累了」 | 疲勞 ≠ 藉口 |

這是把「模型會怎麼偷懶」當成需求來寫。`writing-skills` 甚至把這件事方法化：先跑 baseline 讓模型在沒有 skill 的情況下犯錯，記錄它用的每一個藉口，再針對那些藉口寫反駁，重跑驗證。

整個流程其實就是 TDD 的紅、綠、重構。

### 6. 章節順序：判準放在做法之前

`systematic-debugging` 的順序是：

```
概述 → 鐵律 → 何時使用 → 四階段流程 → 常見藉口
```

先講為什麼，再講不能違反的底線，再講適用範圍，最後才是操作細節與防呆。模型比較不會埋頭做錯方向。

### 7. 對 token 斤斤計較

`writing-skills` 要求：不要在 SKILL.md 裡列出所有參數，引用 `--help` 就好；不要重複別的 skill 的流程，引用它就好；範例要短。

skill 會進 context，寫得越肥，留給真正工作的空間越小。

> 📖 想讀原文的話：上面這些寫法都來自英文上游 [github.com/obra/superpowers](https://github.com/obra/superpowers)。對照中英兩版看同一段規則，比單讀一種語言更容易抓到哪些字是刻意留的、哪些是翻譯後的語氣。有興趣可以直接翻 `skills/` 目錄，每個 `SKILL.md` 都是獨立可讀的樣本。

---

# 🤖 安裝：一條指令掃描 26 款工具

```bash
cd /your/project
npx superpowers-zh
```

它會偵測專案用了哪些工具，把 21 個 skill 放到正確位置。偵測不到就自己指定：

```bash
npx superpowers-zh --tool cursor
```

多專案共用則用全局安裝（v1.7.0+）：

```bash
npx superpowers-zh --global --tool claude
```

專案級優先、全局兜底，兩者可以共存。Claude Code 另外支援 plugin marketplace 安裝，之後升級走 `claude plugin update`。

---

# ⚠️ 三個實務雷點

1. 別在 `~` 跑專案級安裝。會把 skills 與 bootstrap 檔寫進家目錄，污染所有專案。v1.2.1 起會拒絕，舊版不會。想全域生效請用 `--global`。
2. 手動 `cp -r skills` 是低保版。只搬了 skill 檔，`hooks/`（SessionStart 觸發）與 bootstrap 引導都不見了。結果是 AI 不會主動呼叫，每次都得手動喊。
3. 卸載靠哨兵註解。追加到 `CLAUDE.md` 的內容包在 `<!-- superpowers-zh:begin/end -->` 之間，卸載時精確切除。辨識不可靠就跳過並警告。

---

# 🔍 實際裝起來的樣子

```
.agents/
├── AGENTS.md          # 說明有哪些 skill、何時該用
└── skills/
    ├── brainstorming/
    ├── chinese-commit-conventions/
    ├── systematic-debugging/
    ├── verification-before-completion/
    └── …（共 20 個）
```

實際裝起來有兩件事值得記。

- 本機 20 個，上游現在 21 個。少的是 `diagnosing-superpowers`（用 transcript 取證、每條結論帶 `path:line`），它是上游 v6.4.1 才加入的，代表這份安裝已經兩個多月沒更新。skill 清單會隨版本變動，重跑一次 `npx superpowers-zh` 比手動補檔可靠。
- 規則真的會改變行為。這個 repo 的 commit 是 `<type>(<scope>): <繁體中文主旨>`，而且 `AGENTS-NEW-POST-WORKFLOW.md` 特別交代「不要順手清理 CRLF 噪音」。這種「知道團隊在意什麼」的判斷，通用英文 skill 給不了。

---

# 🆚 和英文上游怎麼選

| 情境 | 建議 |
| --- | --- |
| 只裝 Claude Code、英文無礙、想跟最新版 | 直接用上游 `obra/superpowers` |
| 同時用 Codex、Cursor、OpenCode 等多款 | 中文版一條指令省下重複設定 |
| 團隊在 Gitee、Coding、極狐 GitLab、CNB | 中文版有對應 Git 工作流與 CI 範例 |
| 需要中文 commit、文件排版、review 語氣 | 中文版補了這四塊上游沒有的內容 |
| 想學 prompt 怎麼寫才管得住模型 | 兩版都能讀，中文版讀起來快得多 |

---

# 📚 參考資料

- [jnMetaCode/superpowers-zh（GitHub）](https://github.com/jnMetaCode/superpowers-zh)
- [繁體中文 README](https://github.com/jnMetaCode/superpowers-zh/blob/main/README.zh-Hant.md)
- [上游英文原版 obra/superpowers（GitHub）](https://github.com/obra/superpowers)，想對照原文、看 prompt 原始寫法就從這裡
- [npm: superpowers-zh](https://www.npmjs.com/package/superpowers-zh)
