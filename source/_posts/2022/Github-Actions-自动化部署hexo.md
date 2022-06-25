---
title: Github Actions 自动化部署hexo
date: 2022-06-25 21:37:51
index_img:
banner_img:
categories:
tags:
---

使用Github Actions 自动化部署hexo。

<!-- more -->

## 背景

`hexo`博客需要备份的有两个`branch`，我们可以通过脚本在`hexo deploy`命令时自动备份`hexo`分支，也可以在备份`hexo`时，使用`Github Actions`来自动化部署，这里记录一下自动化部署的过程。



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



### 添加secrets

复制`hexo-deploy-key.pub`文件内容，在仓库中`Settings > Deploy keys`添加Deploy key。

[![jkHP54.jpg](https://s1.ax1x.com/2022/06/25/jkHP54.jpg)](https://imgtu.com/i/jkHP54)

复制`hexo-deploy-key`文件内容，在仓库中`Settings > Secrets`添加秘钥，名称为`DEPLOY_KEY`。

[![jkHer6.jpg](https://s1.ax1x.com/2022/06/25/jkHer6.jpg)](https://imgtu.com/i/jkHer6)

## Workflow配置

在blog目录文件中新建`.github/workflows/deploy.yml`:

```yaml
# This is a basic workflow to help you get started with Actions

name: Blog deploy

# Controls when the workflow will run
on:
  # Triggers the workflow on push or pull request events but only for the main branch
  push:
    branches: [ hexo ]

  # Allows you to run this workflow manually from the Actions tab
  workflow_dispatch:

# A workflow run is made up of one or more jobs that can run sequentially or in parallel
jobs:
  # This workflow contains a single job called "build"
  build-and-deploy:
    # The type of runner that the job will run on
    runs-on: ubuntu-latest

    # Steps represent a sequence of tasks that will be executed as part of the job
    steps:
      # Checks-out your repository under $GITHUB_WORKSPACE, so your job can access it
      - uses: actions/checkout@v2

      # Runs a set of commands using the runners shell
      - name: Set Node
        uses: actions/setup-node@v2
        with:
          node-version: '14'

      - name: Npm Install
        run: |
          npm install hexo-cli -g
          npm install

      - name: Set Key
        env:
          DEPLOY_KEY: ${{ secrets.DEPLOY_KEY }}
        run: |
          mkdir -p ~/.ssh
          echo "$DEPLOY_KEY" > ~/.ssh/id_rsa
          chmod 600 ~/.ssh/id_rsa
          ssh-keyscan github.com >> ~/.ssh/known_hosts
          git config --global user.email "chzaaow@gmail.com"
          git config --global user.name "m1ria"

      - name: Hexo Deploy
        run: |
          hexo clean
          hexo generate
          gulp
          hexo deploy
```

