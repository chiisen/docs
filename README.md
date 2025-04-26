# docs
使用 Hexo 架設簡單的分享技術文件並發佈到"cloudflare"，包含介紹文章: "全面瞭解 n8n v1.88.0 的重磅更新 —— MCP Server"、"教會你用 Google AI Studio 提早結束工作回家"、"C# 設計模式學習筆記與程式碼範例"、"JavaScript 設計模式學習筆記與程式碼範例"、"Functional Design Pattern 是一個結合函數式程式設計和設計模式的概念"

## Hexo
Hexo 其實就是一個基於 Node.js 開發的網誌框架

## 安裝需求
你需要安裝 Node.js 與 Git

## 安裝 Hexo
```
npm install hexo-cli -g
```
查詢現在安裝的 hexo 版本號，測試一下是否安裝成功
```
hexo version
```
或
```
hexo -v
```

## 建立 Hexo 專案
```
hexo init <資料夾名稱>
```
我通常把新建的資料夾檔案複製到專案的更目錄  
這樣就不需要 cd <資料夾名稱>  
直接初始化 Hexo 專案  
```
npm install
```

## 啟動 Hexo 專案
```
hexo server
```
通常是 http://localhost:4000/  
直接點擊可以開啟瀏覽器瀏覽網頁  
如果報錯
```
FATAL Permission denied. You can't use port 4000.
```
可以已用指令更換 port
```
hexo server -p 5000
```

# 清除靜態檔案與快取
```
hexo clean
```

# generate 產生靜態檔案
```
hexo generate
```

# 新增文章
```
hexo new <title>
```

# 發佈到遠端
```
hexo deploy
```

# 發佈到 cloudflare
去註冊一個 cloudflare 帳號  
選取左邊選單 "計算(Workers)" 中  
建立 "Worker 和 Pages"  
![Worker 和 Pages](./images/cloudflare_workers_and_pages.png)

## 組建組態
![組建組態](./images/cloudflare_build.png)
### 組件命令
```
npm run build
```
### 組建輸出目錄
```
public
```
### 變數與秘密
![變數與秘密](./images/NODE_VERSION.png)
```
NODE_VERSION 20.11.0
```
發佈網址: https://docs-axs.pages.dev

