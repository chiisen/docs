---
title: 🛠️ mattpocock/skills：小塊、可改、給真的在寫程式的人
date: 2026-10-21 00:00:00
tags:
  - AI
  - Agent
  - Claude
  - Tool
---

## 🛠️ mattpocock/skills：小塊、可改、給真的在寫程式的人

**✅ 這篇做什麼：介紹 [mattpocock/skills](https://github.com/mattpocock/skills)。Matt Pocock 把自己 `.agents` 目錄裡每天在用的 skills 公開出來。他的對照對象是 GSD、BMAD、Spec-Kit 這類會把流程整個包走的做法：那些套件幫你排程序，但也把控制權拿走，流程本身出錯時不好修。這包故意做成小的、好改的、可以拼起來的，而且不綁某個模型。**

<!-- more -->

倉庫說明寫 "Skills for Real Engineers"，副標是 straight from my `.agents` directory。他要的是日常工程，不是 vibe coding。電子報報名人數他寫約 6 萬，連結在 [aihero.dev](https://www.aihero.dev/s/skills-newsletter)。技能索引也可以從 [skills.sh/mattpocock/skills](https://skills.sh/mattpocock/skills) 看。

---

# 🛠️ 怎麼裝

外掛會自己更新。`npx skills` 是把可編輯的檔案複製進專案，之後要自己更新。兩種都裝的話，每份 skill 會出現兩次，所以一個 agent 只選一種。

Claude Code：

```text
claude plugin install mattpocock-skills@claude-plugins-official
```

Codex：

```text
codex plugin marketplace add mattpocock/skills
codex plugin add mattpocock-skills@mattpocock
```

Codex 會在啟動時更新。GitHub Copilot CLI：

```text
copilot plugin marketplace add mattpocock/skills
copilot plugin install mattpocock-skills@mattpocock
```

Copilot 還要在 `~/.copilot/settings.json` 加一次：

```json
{
  "extraKnownMarketplaces": {
    "mattpocock": { "source": { "source": "github", "repo": "mattpocock/skills" }, "autoUpdate": true }
  }
}
```

VS Code 用 **Chat: Install Plugin From Source**，填 `https://github.com/mattpocock/skills`，每天更新。

Gemini CLI 要自己重跑才算更新：

```text
gemini skills install https://github.com/mattpocock/skills.git --path skills/engineering
gemini skills install https://github.com/mattpocock/skills.git --path skills/productivity
```

其他 agent，或想拿到可改的檔案：

```text
npx skills@latest add mattpocock/skills -a <agent>
```

`-a` 可以是 cursor、opencode、devin、windsurf、amp、pi。省略的話安裝程式會問。選 skill 時把 `setup-matt-pocock-skills` 勾進去。更新用 `npx skills@latest update`；有新 skill 要再跑一次 `add`。

裝好後，每個 repo 跑一次 `/setup-matt-pocock-skills`。它問三件事：issue tracker（GitHub、GitLab、本地檔案，或你自己描述的別的）、triage 用的標籤、文件要存哪。

---

# 🧩 他在補的四種失敗

README 把常見翻車收成四條，每條都掛一個 skill，而不是再發明一套總流程。

Agent 做的不是你要的。他引 *The Pragmatic Programmer*：沒有人一開始就完全知道自己要什麼。對人是這樣，對 agent 也是。解法是 grilling，讓 agent 把設計樹上的分支問完。非程式用 `/grill-me`，寫程式用 `/grill-with-docs`。後者多做一件事：邊問邊收共用術語。他說這兩個是最常被用的，每次要改東西之前都先跑。

Agent 太囉嗦。Eric Evans 講的 ubiquitous language 被拿來當理由：人和領域專家一開始不說同一套話，agent 被丟進專案後也是邊猜術語邊寫，一句話拆成二十個字。`/grill-with-docs` 會把術語寫進 `GLOSSARY.md`，難解釋的決定寫進 ADR。他舉自己的 `course-video-manager`：改前是「課程裡某一節被做成檔案系統上的真實位置出了問題」，改後是 "materialization cascade"。那份舊檔在釘選的 commit 裡還叫 `CONTEXT.md`，後來才改名。副作用他列了三個：命名跟著詞彙走、agent 比較找得到路、思考時少燒 token。

程式不能跑。回饋速度是上限，一次接太大的任務就不該接，這句也來自 *The Pragmatic Programmer*。對齊了需求還是寫出壞程式，通常是回饋迴圈不夠：型別、瀏覽器、自動測試。測試這邊他要 red-green-refactor，先寫會失敗的測試。`/tdd` 就是這圈。難抓的 bug 和效能退步走 `/diagnosing-bugs`：先做出會在這個 bug 上變紅的迴圈，再縮小、提假設、加觀測、修、補回歸測試，一階段過一關。

程式變成一坨。Kent Beck 要每天投資設計，Ousterhout 要深模組：很多行為，小介面。Agent 把寫碼加快，熵也加快。`/to-spec` 在寫規格前會先問你要動哪些模組。`/improve-codebase-architecture` 掃一遍加深的機會，交候選清單。他建議隔幾天跑一次。這是調查，不是救援；老程式庫找得到候選，不會替你把泥團拆開。

---

# 🗂️ 誰可以呼叫

分法只有一軸。User-invoked 只有你打出指令才會跑，工作是編排。Model-invoked 你也可以打，任務對上時 agent 自己也會拿。前者可以呼叫後者，不能再呼叫另一個 user-invoked。

工程、每天會用到的，user-invoked：

| 指令 | 做什麼 |
| --- | --- |
| `/ask-matt` | 問現在該用哪個 skill 或流程 |
| `/grill-with-docs` | 盤問，同時改 `GLOSSARY.md` 和 ADR |
| `/triage` | 依 triage 角色的狀態機移動 issue |
| `/improve-codebase-architecture` | 掃加深機會，出一份 HTML 報告，再針對你選的那項繼續問 |
| `/to-spec` | 把已經談過的內容收成規格，發到 issue tracker。不再面試一輪 |
| `/to-tickets` | 拆成 tracer-bullet tickets，寫明阻擋關係。本地檔或 tracker 上的 blocking link 都可以 |
| `/implement` | 照規格或 tickets 做，約定好的接縫走 `/tdd`，提交前走 `/code-review` |
| `/implement-spec` | 整份規格放一條整合分支，依任務圖把就緒的票交給實作 subagent，收尾仍是 `/code-review` |
| `/wayfinder` | 大到一次 session 放不下的工作，拆成 tracker 上的決策票，一張一張解到路清楚 |
| `/retro` | session 結束後，對導航、自動檢查、規範、steering 檔、工具提改進，嚴重的先講 |

Model-invoked 裡比較常被上面那些叫到的：`/tdd`、`/diagnosing-bugs`、`/domain-modeling`、`/codebase-design`（深模組：小介面後面放很多行為，接縫乾淨，透過介面測）、`/code-review`（標準和規格兩條軸，各開 subagent，避免互相污染）、`/prototype`（用完即丟，狀態問題用單份 HTML，UI 用同一路由切幾種差很多的版本）、`/pr`（PR 內文先給最小的對照，再給前後證據，並標這是單向門還是雙向門，以及爆炸半徑）、`/research`（對一手來源查完，寫成倉庫裡有引用的 Markdown，當背景 agent 跑）、`/wizard`（產生互動式 bash，帶人做只有人能做的步驟：開帳號、塞 CI secret、走第三方後台、一次性切換）。

生產力、不限寫程式：`/grill-me` 把計劃問到設計樹的分支都有答案。`/handoff` 把這輪對話壓成給下一個 agent 的交接文件。`/teach` 用目前目錄當有狀態的教學工作區，分多輪教。`/to-questionnaire` 把你一個人答不了的決定做成 Markdown 問卷，問的是寄給誰、要收回什麼，不是題目本身。`/wait-what` 在一句話沒聽懂時用，agent 用 `GLOSSARY.md` 的詞再講一次。底下的 `/grilling` 是面試原語，`/grill-me`、`/grill-with-docs`、`/triage`、`/wayfinder`、`/improve-codebase-architecture` 都站在它上面。`/writing-for-agents` 則是寫給 agent 讀的文件：skill、`AGENTS.md`、`CLAUDE.md`，以及任何靠指標才會被打開的文件。

---

# 💬 在 Codex 裡怎麼接

README 在 Codex 安裝步驟後面用的是斜線指令，例如 `/grill-me`。這段依 2026-10-21 讀到的 `main`。若你這台 Codex 只認 `$grill-me`，把下面的 `/` 換成 `$`。我沒有在本機已安裝的外掛裡核對觸發字串。

先跑過 `/setup-matt-pocock-skills`。`/to-spec`、`/to-tickets`、`/wayfinder` 都要知道 issue tracker 在哪；沒設定時，skill 會叫你先跑 setup。

文件寫明的先後不是一條固定管線，各 skill 只做自己那件事：

- `/grill-me` 只呼叫 `grilling`，把計劃問完，不寫文件。
- `/grill-with-docs` 同時呼叫 `grilling` 與 `domain-modeling`，會改 `GLOSSARY.md` 和 ADR。
- `/to-spec` 不再提問。它把已經談過的內容收成規格，發到 tracker，並貼 `ready-for-agent`。seam（對外可觀察的測試邊界）會先跟你對過。
- `/to-tickets` 拆成可獨立驗證的垂直切片，先把清單給你看，同意才發表。不關父 issue。
- `/wayfinder` 給一次 session 放不下、路還不清楚的工作。預設只做決策、不實作。問完若沒有霧，它應停下來問你，不要硬畫地圖。
- `/tdd` 是先紅後綠。寫測試前先列出 seam 並等你確認。一次一個 seam。重構不在這個迴圈，留給 `code-review`。
- `/implement` 照規格或票做，接縫走 `/tdd`，中間跑型別檢查與單一測試，結束跑整個測試套件和 `/code-review`，然後 commit 到目前分支。

下面的取捨是用法建議，不是 README 規定的順序。需求還說不清，先 grill，不要先 `/to-spec`。會改程式、名詞會重複，用 `/grill-with-docs`；還不碰 repo，用 `/grill-me`。驗收已經講得出口、範圍在一個模組，直接 `/tdd`。要留給之後的 agent，grill 完再 `/to-spec`，看過規格才 `/implement`。要拆多張票，規格點頭後再 `/to-tickets`，一張一張做。`/wayfinder` 只放在目的地仍是霧、而且明顯超過一輪的時候。

改錯字、改一句文案、改一個你已經指出的常數，直接下指令。不必開這些 skill。

`/implement` 會 commit。不想讓它提交時，在 prompt 寫「不要 commit、不要 push」。這是覆寫作者預設，skill 裡沒有開關。

每個階段分開貼。`<...>` 換成你的句子。

需求不清楚、會動到程式：

```text
/grill-with-docs
我想做：<一句話>。
已知限制：<例如不能改公開 API>。
先不要寫程式、不要開 issue、不要 commit。
問到可驗收的行為、明確不做的範圍，以及會寫進 GLOSSARY.md 的名詞為止。
```

還不碰程式：

```text
/grill-me
計畫：<一句話>。
先不要寫檔。問到每個分支都有決定，或標出仍須別人回答的問題為止。
```

bug 或小功能，驗收已經清楚：

```text
/tdd
問題：<誰在什麼操作後看到什麼>。
預期：<可觀察結果>。
必須保留：<既有行為>。
先列出要測的 seam，等我確認再寫會失敗的測試。
一次只做一個 seam。不要 commit、不要 push。
完成時貼上紅燈與綠燈的命令輸出。
```

對話已經夠，要收成規格：

```text
/to-spec
依目前對話寫規格並發到 issue tracker。
不要再面試。seam 先給我看，我同意再發表。
寫明問題、做法、驗收、不做的範圍。不要寫死檔案路徑。
```

規格你已經看過：

```text
/to-tickets <規格連結或 issue 編號>
拆成可各自驗證的垂直切片，標出阻擋關係。
先給清單：標題、被誰擋住、完成後能看到什麼。
等我同意再發表。不要關閉父 issue，不要開始寫程式。
```

目的地仍不清楚，而且超過一輪：

```text
/wayfinder
這一輪只畫地圖，不實作、不開 PR。
若你判斷沒有霧、一輪就做得完，停下來告訴我，不要建立 wayfinder:map。
```

之後一輪只解一張決策票（research 票是文件寫的例外）：

```text
/wayfinder <地圖連結>
這一輪只處理一張未認領、沒被擋住的票。
先認領再做。做完把決議寫回該票並更新地圖。不要實作目的地本身。
```

一張票開始做：

```text
/implement <票的編號或 URL>
先複述標題。只做這張票的驗收條件。
約定過的 seam 走 /tdd。結束前跑專案的型別檢查與測試，並貼輸出。
不要 commit、不要 push。
```

你要看的產出：grill 問的問題有沒有改到驗收範圍；`/grill-with-docs` 還要看 `GLOSSARY.md` 和 ADR 的名詞是不是你會用的。`/to-spec` 看 tracker 上的問題、做法、驗收、Out of Scope，以及 seam 有沒有先點頭。`/to-tickets` 看每張票是否單獨可驗證，阻擋對不對。本地 tracker 的檔在 `.scratch/<feature-slug>/issues/`，一票一檔。`/wayfinder` 看地圖的 Destination、Decisions so far、Not yet specified、Out of scope。`/tdd` 和 `/implement` 先看紅燈是不是失敗在該行為上，再看綠燈輸出和 diff 有沒有超出這張票。

---

# 📎 來源

- 倉庫：<https://github.com/mattpocock/skills>
- skills.sh：<https://skills.sh/mattpocock/skills>
- 電子報：<https://www.aihero.dev/s/skills-newsletter>
- skill 原文：<https://github.com/mattpocock/skills/tree/main/skills>
