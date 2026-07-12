# 🤖 AGENTS.md — New Post Workflow (Hexo)

> 對 AI Agent 寫的：給後面接手的你（跟我同個工具）一份**實戰驗證過**的 checklist。
> 上一個 agent 在這個工作流摔過：**漏 commit、忘 push、繞太多確認問題**。
> 把這份讀完，**直接幹**、不要再問 4-選項-1。

---

## 🎯 30 秒總覽

1. 寫 markdown → `source/_posts/YYYY-MM-DD-slug.md`
2. `yarn run build`（驗證渲染）→ exit 0
3. `git add` + `git commit` + `git push git@github.com-chiisen:chiisen/docs.git main`

整個流程正常 < 30 秒。**不要拆 7 輪確認問題**。使用者偏好：「先這樣」、「先不做」、「phased, do not proactively open new work」。

---

## 📝 Step 1 — 寫文章

### Frontmatter（所有既有文章都是這個格式）

```yaml
---
title: 🧠 文章標題（emoji + 中英混用）
date: YYYY-MM-DD 00:00:00
tags:
  - Tag1
  - Tag2
---
```

**已實證的慣例：**
- `title` 一定要有 emoji + 你的主題關鍵詞（看 `2026-03-06-ReAct-Agent.md` 是 `🤖 ReAct-Agent`）
- `date` 鎖 `00:00:00`
- `tags` 4 個左右，已有的 tag：`AI / Agent / Python / LLM / Go / Terminal / TUI / ASR / TTS`

### 內文風格

- 開頭：`## 標題` + **粗體定位句** `**✅ 這篇文章做什麼**`
- 一張 `![x](../images/x.png)` 封面圖（可有可無）
- `<!-- more -->` 控制摘要
- 大標：`# 🌟/🛠️/🏗️` emoji 大標
- 內文用 `checklist`：`🚀 **粗體詞**：說明`
- 結尾：原始來源連結 + 參考資料

### 換行習慣 ⚠

**Hexo 專案所有現存文章都是 CRLF**（`file *.md` 顯示 `with CRLF line terminators`）。Agent 通常寫出 LF。**寫完後必須轉換為 CRLF**，跟倉庫保持一致。
⚠️ **【安全警告】Windows/PowerShell 環境下，切勿使用 `Set-Content` 或 `Out-File` 來轉換或存檔！**
在預設的 Windows PowerShell 下，這些指令會自動將檔案格式轉為 `ANSI (Big5)`，把原本 UTF-8 中的中文字與 emoji 全部損壞成亂碼或 `?`。

* **在 Linux/Mac 上**，可用 `sed`：
  ```bash
  sed -i 's/$/\r/' source/_posts/YYYY-MM-DD-slug.md
  ```
* **在 Windows (PowerShell) 上**，必須使用 .NET 類別安全地以 **無 BOM UTF-8** 寫入：
  ```powershell
  $path = "source/_posts/YYYY-MM-DD-slug.md"; $content = [System.IO.File]::ReadAllText($path, [System.Text.Encoding]::UTF8); [System.IO.File]::WriteAllText($path, $content.Replace("`n", "`r`n"), (New-Object System.Text.UTF8Encoding($false)))
  ```

---

## 🔨 Step 2 — Build 驗證

```bash
yarn run clean
yarn run build
```

- 應該看到 `177 files generated` 之類（會隨文章數變）
- exit code = 0
- HTML 在 `public/YYYY/MM/DD/YYYY-MM-DD-slug/index.html`
- ⚠ **`yarn run build` 不是 deploy、不會 git add 任何東西** — 它只驗證渲染對不對

---

## 🔄 Step 3 — Commit + Push

### git add

```bash
git add source/_posts/YYYY-MM-DD-slug.md
```

**只 add 這個檔案**。不要 add `themes/next/` (CRLF 噪音)、不要 add `public/` (在 .gitignore)、不要 add `.yarn/install-state.gz`。

### Commit message

跟既有風格對齊（最近 3 篇用 emoji）：

```bash
git commit -m "📝 docs: 新增 <一句話總結>"
```

### Push URL ⚠⚠⚠

**不要用 `git push origin main`**，會觸發 4 個 pushurl = 3 個 fork 沒權限 = Permission denied。

用**直接 URL 繞開**：

```bash
git push git@github.com-chiisen:chiisen/docs.git main
```

### 驗證 push 成功

```bash
git ls-remote git@github.com-chiisen:chiisen/docs.git refs/heads/main
```

本機 HEAD SHA 應跟遠端 SHA 字串一致。

---

## 🚨 不要再犯的錯

| 錯的事 | 為什麼壞 | 正確做法 |
|---|---|---|
| 寫完 `yarn run build` 就以為「完成」 | build 不會 commit、不會 push | 跑 build 驗渲染後，**才**開始 git |
| `git push origin main` | 觸發 4 pushurl，3 個 fork fail | 直接 URL `git@github.com-chiisen:chiisen/docs.git` |
| 把 `.yarn/install-state.gz` 一起 add | 已被 untrack（見 a5420f0） | 用 `git status --short` 檢查 staging |
| 順手清理 CRLF 339 modified | 你說了 postpone 339 | **別動** |
| 開「4 選項 1」確認 push 細節 | user 偏好 phased + KISS | 直接 commit + push，中斷不超過 5 句話 |
| 漏 commit 新文章 | user 工作流階段沒注意到 | 完成清單每 1-2 步再掃一次 `git status --short` |
| 推完說「已 push」卻沒附 SHA | 沒附證據 = 沒做 | `git ls-remote` 印遠端 SHA |
| 使用 PowerShell 的 `Set-Content` 或 `Out-File` 轉 CRLF | 在 Windows PowerShell 下預設會將編碼轉成 ANSI (Big5)，導致 UTF-8 中文毀損變亂碼 | 用 .NET 的 `WriteAllText` 指定無 BOM 的 UTF-8 編碼安全轉換 |

---

## ✅ 完整單次流程（給 agent 抄的）

```bash
# 寫完 source/_posts/YYYY-MM-DD-slug.md

sed -i 's/$/\r/' source/_posts/YYYY-MM-DD-slug.md  # CRLF 統一
yarn run build                                         # 渲染驗證
git add source/_posts/YYYY-MM-DD-slug.md
git commit -m "📝 docs: 新增 <主題>"
git push git@github.com-chiisen:chiisen/docs.git main

# 驗證
git ls-remote git@github.com-chiisen:chiisen/docs.git refs/heads/main
# SHA 應跟本機 HEAD 一致
```

整個跑完 < 60 秒。看起來很長、其實 5 條命令。

---

## 📁 倉庫 quick reference

| 項目 | 位置 |
|---|---|
| 文章 | `source/_posts/YYYY-MM-DD-slug.md` |
| 圖片 | `source/images/` |
| 主題 | `themes/next/` |
| Hexo 設定 | `_config.yml` |
| 前一顆範例 commit | `5fa4714`（大型推理機器） |
| 推 push URL | `git@github.com-chiisen:chiisen/docs.git` |
| origin remote (不要用 push) | `git@github.com-chiisen:chiisen/docs.git` (但 origin 有 4 個 pushurl) |

---

## ⚠ 已知沒解決的事

1. **`themes/next/` 339 modified CRLF**：user 決定 postpone，agent 不要動
2. **`yarn.lock` 是 2026-03-04 舊版**：依賴 bump 靠 dependabot，agent 不要手動 update
3. **branch protection 提醒「Changes must be made through a pull request」**：admin bypass 允許 push 直進 main，但未來可能被堵。這條是 GitHub 警告、不是錯誤。

---

_本文件由 2026-07-10 後設會話寫入，作者 chiisen。更新我前請先讀《myself》\(你自己\)。_
