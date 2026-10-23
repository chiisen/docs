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
`subtitle` 會顯示在站名下方，樣式是一顆標籤。這是站台自己的發版流水號，不是 Hexo 套件版本。Hexo 實際版本看 `package.json` 的 `hexo.version`（目前 8.1.2）。

有內容更新就加 1，用數字加，不要用字串接在後面。`1.0.109` 的下一版是 `1.0.110`。

```yaml
title: "⭐️ 文章分享(持續累積中🏃‍♂️)"
subtitle: "v1.0.109"
```

`package.json` 的 `version` 維持 `0.0.0`。這個專案是 `private: true`，那個欄位不會出現在首頁。

