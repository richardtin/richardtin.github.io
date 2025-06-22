---
slug: create-docusaurus-website-with-docker
title: 用 Docker 打造自己的 Docusaurus 學習筆記網站 - 一個簡單卻意外的開始
authors: [richardtin]
tags: [docker, docusaurus]
---

>「有時候，筆記不是為了記錄昨天，而是為了看見明天。」

當我第一次聽到 Docusaurus 這個名字時，我以為它只是另一個冷門的靜態網站生成器，和市面上數十種工具沒什麼兩樣。但當我真正用它整理學習筆記後，才發現它有多讓人意外：

它的設計理念很單純，卻幫我把分散的 Markdown 筆記變成了一座條理分明、可搜尋、可持續擴充的知識花園。

為了確保環境一致、易於部署，我選擇用 Docker 來管理整個 Docusaurus 專案。以下，我將以一個真實可操作的例子，分享如何從零開始，用 Docker 快速架設一個屬於自己的 Docusaurus 學習筆記網站。

<!-- truncate -->

## 一、前置條件

在開始之前，請確認您已具備：

- 安裝好 Docker（Docker Desktop / Docker Engine 均可）
- 具備基本的命令列操作能力
- 一顆願意嘗試的心

## 二、建立專案目錄

首先，在您的主機上建立一個放置筆記網站的目錄：

```bash
$ mkdir docusaurus-notes
$ cd docusaurus-notes
```

## 三、撰寫 `docker-compose.yml`

```yml
services:
  app:
    image: node:22-alpine
    user: node
    environment:
      - NODE_ENV=production
    volumes:
      - ./my-website:/app
    working_dir: /app
    command: sh -c "npm install && npm run start --host 0.0.0.0"
    labels:
      - "traefik.enable=true"
      - "traefik.http.routers.app.rule=Host(`my-website.traefik.me`)"
      - "traefik.http.routers.app.entrypoints=web"
      - "traefik.http.services.app.loadbalancer.server.port=3000"
  
  traefik:
    image: traefik:v3.4
    container_name: traefik
    command:
      - "--api.insecure=true"
      - "--providers.docker=true"
      - "--entrypoints.web.address=:80"
    ports:
      - "80:80"
      - "8080:8080"
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
```

## 四、建置並啟動服務

執行以下指令：

```bash
### 初使化 docusaurus
$ docker run --rm -ti -v .:/app -w /app node:22-alpine npx create-docusaurus@latest my-website classic --typescript

### 使用 docker compose 啟動
$ docker compose up -d
```

請在瀏覽器開啟 http://my-website.traefik.me

## 五、開始編輯您的筆記

此時，您的 Docusaurus 專案位於 my-website 目錄中，主要重點：

- `docs/`：放置 Markdown 筆記檔案
- `sidebars.js`：定義側邊欄結構
- `docusaurus.config.js`：站點名稱、Logo 與自訂設定

每當您修改 Markdown 或配置，Docker 容器中的開發伺服器會自動熱更新，無需重啟。

## 六、下一步

若想將網站發佈到正式環境，建議使用以下方式：

- 產生靜態檔案：
    ```bash
    $ npm run build
    ```

- 將 `build/` 目錄部署到任意靜態主機（如 NGINX、S3、Vercel 等）

## 七、結語

Docusaurus 與 Docker 的結合，讓筆記不僅僅是筆記，而是活的知識系統。

若您也正在尋找一個可持續、可分享、可版本管理的學習筆記解決方案，不妨從這份教學開始，一步步種下屬於自己的知識花園。
