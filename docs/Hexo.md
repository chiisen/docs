# Hexo
Hexo 其實就是一個基於 Node.js 開發的網誌框架

# 安裝需求
你需要安裝 Node.js 與 Git

# 安裝 Hexo
```shell
npm install hexo-cli -g
```
查詢現在安裝的 hexo 版本號，測試一下是否安裝成功
```shell
hexo version
```
或
```shell
hexo -v
```

# 建立 Hexo 專案
```shell
hexo init <資料夾名稱>
```
我通常把新建的資料夾檔案複製到專案的更目錄  
這樣就不需要 cd <資料夾名稱>  
直接初始化 Hexo 專案  
```shell
npm install
```

# 啟動 Hexo 專案
```shell
hexo server
```
通常是 http://localhost:4000/  
直接點擊可以開啟瀏覽器瀏覽網頁  
如果報錯
```shell
FATAL Permission denied. You can't use port 4000.
```
可以已用指令更換 port
```shell
hexo server -p 5000
```

# 升級專案內的 Hexo 版本
```
npm install hexo@latest --save
```

# 清除靜態檔案與快取
```shell
hexo clean
```

# generate 產生靜態檔案
```shell
hexo generate
```

# 新增文章
```shell
hexo new <title>
```

# 發佈到遠端
```shell
hexo deploy
```

## 編譯 hexo 專案
```shell
npm run build
```
編譯完會產生最重要的 public 內容  

## 組建輸出目錄
```shell
public
```

## 更新首頁的版本號
修改 `_config.yml` 的 `Hexo 1.0.5` 上的版本號  
```yaml
title: "⭐️ Sam 的文章分享(持續累積中🏃‍♂️)(cloudflare + Hexo 1.0.5 版)"
```
一般 npm 套件是修改 `package.json` 的 0.0.0 版本號  
```shell
"version": "0.0.0",
```
但是目前沒辦法顯示在畫面上，所以才修改 Hexo 上的標題設定檔案 `_config.yml` 的版本號  

