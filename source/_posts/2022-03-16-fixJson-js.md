---
title: 🛠️ fixJson.js
date: 2022-03-16 00:00:00
tags:
  - Javascript
  - JSON
  - Tool
  - CLI
---

## 🛠️ fixJson.js

**✅ 修正不正常的 JSON 資料格式，並提供強大的偵錯功能！ 🧹**

當你遇到格式錯誤的 JSON 導致解析失敗時，不需要再辛苦地開啟 IDE 或其他工具進行偵錯。fixJson.js 提供了偵錯提示功能，能夠在修復失敗或遇到語法錯誤時，精準指出錯誤發生的行號與位置，幫助你快速定位問題！

<!-- more -->

# 📦 安裝

```bash
npm install fix-json-format
```

# ✨ 功能特性

- 修正缺少引號的 key/value
- 修正缺少逗號
- 修正多餘逗號
- 修正不平衡括號
- 修正空值欄位
- 支援 null/true/false 布林值
- 支援數字欄位
- 支援時間格式 (ISO/一般格式)
- 支援 IP 格式 (::ffff:x.x.x.x)
- 支援多行 JSON
- 自動修復漏逗號
- 錯誤提示 - 修復失敗時顯示錯誤行號與位置
- 性能優化 - 減少重複 replace 次數
- TypeScript 支援 - 完整類型定義
- ESM/CommonJS 支援 - 支援兩種模組系統
- CLI 工具 - 命令列介面

# 📝 程式碼範例

## CLI 命令列

```bash
# 從檔案讀取
fix-json-format input.json

# 從標準輸入讀取
cat input.json | fix-json-format

# 輸出到檔案
fix-json-format input.json -o output.json

# 顯示說明
fix-json-format --help
```

## ESM / TypeScript

```javascript
import { fix_json } from "fix-json-format";
```

## CommonJS

```javascript
const { fix_json } = require("fix-json-format");
```

## 使用方式

```javascript
// 回傳物件格式 (預設) - 包含錯誤資訊
const { result, error } = fix_json(str);
if (error) {
  console.error(error); // 輸出: JSON Syntax Error at line 1, column 10: ...
}

// 向後相容 - 回傳字串
const strResult = fix_json(str, { returnObject: false });
```

# 🧪 測試

```bash
npm test # Jest 單元測試
npm run test:legacy # 原有測試

# 本地開發測試
npm link # 連結本地指令
fix-json-format --version # 顯示版本
fix-json-format --help # 顯示說明
echo '{name:"test"}' | fix-json-format # 修復 JSON

# 或直接使用 npx (不需安裝)
npx fix-json-format --version
echo '{name:"test"}' | npx fix-json-format
```

# 📋 更新日誌

## v1.0.3

- 新增 CLI 命令列工具

## v1.0.2

- 新增 TypeScript 類型定義 (.d.ts)
- 新增 ESM 模組支援
- 同步版本號

## v1.0.1

- 新增多餘逗號自動修復
- 新增不平衡括號自動修復
- 新增字串值缺少引號修復
- 效能優化 (Stage 分組 + 替換表)

## v1.0.0

- 初始版本
- 修正缺少引號的 key/value
- 修正缺少逗號
- 修正空值欄位
- 支援 null/true/false 布林值
- 支援數字欄位
- 支援時間格式
- 支援 IP 格式
- 支援多行 JSON
- 錯誤提示功能

---

### 我的 Github 專案

[🔗 我的 Github 專案: fixJson.js](https://github.com/chiisen/fixJson.js)  
✅ 支援命令列與程式碼引入。歡迎 Star 🌟 或是 Fork 出去打造你的自定義版本！
