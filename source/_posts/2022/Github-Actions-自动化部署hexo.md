---
title: Github Actions 自动化部署hexo
index_img: 'https://s1.ax1x.com/2022/06/26/jkxrr9.jpg'
banner_img: 'https://s1.ax1x.com/2022/06/26/jkxsbR.jpg'
categories: 博客搭建
tags:
  - Hexo
  - Github Actions
  - 持续集成
abbrlink: b4e9f04c
date: 2022-06-25 21:37:51
---

使用Github Actions 自动化部署hexo。

<!-- more -->

## 背景

`hexo`博客需要备份的有两个`branch`，我们可以通过脚本在`hexo deploy`命令时自动备份`hexo`分支，也可以在备份`hexo`时，使用{% label primary @Github Actions %}来自动化部署，这里记录一下自动化部署的过程。



## 流程

### 生成ssh秘钥

运行命令在`.ssh`文件夹下生成秘钥`hexo-deploy-key`和公钥`hexo-deploy-key.pub`文件。

```bash
➜  ~/.ssh ssh-keygen -f hexo-deploy-key -t rsa -C "chzaaow@gmail.com"
Generating public/private rsa key pair.
Enter passphrase (empty for no passphrase):
Enter same passphrase again:
Your identification has been saved in hexo-deploy-key
Your public key has been saved in hexo-deploy-key.pub
The key fingerprint is:
SHA256:oXFz9fhr52svjuDewqCwyUBNPaKRaMu1gaGk8bWrL/s chzaaow@gmail.com
The key's randomart image is:
+---[RSA 3072]----+
|.+o...      .    |
|==o+o.o    . o   |
|= +=+...+ . . .  |
| oo... + +   .   |
| .  . . S     .  |
|  ...   .      . |
|  .o + . o.   o .|
|  ..+ .  .oo o.+ |
|  .+E    .o.o.oo=|
+----[SHA256]-----+

```



### 上传公私钥		

复制`hexo-deploy-key.pub`文件内容，在仓库中`Settings > Deploy keys`添加Deploy key。

[![jkHP54.jpg](https://s1.ax1x.com/2022/06/25/jkHP54.jpg)](https://imgtu.com/i/jkHP54)

复制`hexo-deploy-key`文件内容，在仓库中`Settings > Secrets`添加秘钥，名称为`DEPLOY_KEY`。

[![jkHer6.jpg](https://s1.ax1x.com/2022/06/25/jkHer6.jpg)](https://imgtu.com/i/jkHer6)

## Workflow配置

在blog目录文件中新建`.github/workflows/deploy.yml`:

```yaml
name: HEXO CI

on:
  push:
    branches:
    - hexo

jobs:
  build:
    runs-on: ubuntu-latest
    strategy:
      matrix:
        node-version: [14.x]

    steps:
      - uses: actions/checkout@v1

      - name: Use Node.js ${{ matrix.node-version }}
        uses: actions/setup-node@v1
        with:
          node-version: ${{ matrix.node-version }}

      - name: Configuration environment
        env:
          HEXO_DEPLOY_PRI: ${{secrets.DEPLOY_KEY}}
        run: |
          mkdir -p ~/.ssh/
          echo "$HEXO_DEPLOY_PRI" > ~/.ssh/id_rsa
          chmod 600 ~/.ssh/id_rsa
          ssh-keyscan github.com >> ~/.ssh/known_hosts
          git config --global user.name "m1ria"
          git config --global user.email "chzaaow@gmail.com"
      - name: Install dependencies
        run: |
          npm i -g hexo-cli
          npm i
      - name: Deploy hexo
        run: |
          hexo clean && hexo generate && hexo deploy
```

{% note warning %}

on.push指定branch推送时执行工作流。

secrets.DEPLOY_KEY指定秘钥。

{% endnote %}



## 配置hexo

在仓库的配置文件{% label primary @_config.yml %}修改：

```yaml
# Deployment
## Docs: https://hexo.io/docs/one-command-deployment
deploy:
  type: git
  repo: git@github.com:m1ria/m1ria.github.io.git
  branch: master
```



## 效果

检查一下运行成功。

[![jkxlvQ.jpg](https://s1.ax1x.com/2022/06/26/jkxlvQ.jpg)](https://imgtu.com/i/jkxlvQ)