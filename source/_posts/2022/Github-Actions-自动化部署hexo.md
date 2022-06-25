---
title: Github Actions 自动化部署hexo
date: 2022-06-25 21:37:51
index_img: https://s1.ax1x.com/2022/06/26/jkxsbR.jpg
banner_img: https://s1.ax1x.com/2022/06/26/jkxrr9.jpg
categories: 博客搭建
tags:
 - Hexo
 - Github Actions
 - 持续集成
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



### 上传公私钥		

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

jobs:
  build:
    runs-on: ubuntu-latest
    name: A job to deploy blog.
    steps:
    - name: Checkout
      uses: actions/checkout@v1
      with:
        submodules: true # Checkout private submodules(themes or something else).
    
    # Caching dependencies to speed up workflows. (GitHub will remove any cache entries that have not been accessed in over 7 days.)
    - name: Cache node modules
      uses: actions/cache@v1
      id: cache
      with:
        path: node_modules
        key: ${{ runner.os }}-node-${{ hashFiles('**/package-lock.json') }}
        restore-keys: |
          ${{ runner.os }}-node-
    - name: Install Dependencies
      if: steps.cache.outputs.cache-hit != 'true'
      run: npm ci
    
    # Deploy hexo blog website.
    - name: Deploy
      id: deploy
      uses: sma11black/hexo-action@v1.0.3
      with:
        deploy_key: ${{ secrets.DEPLOY_KEY }}
        user_name: m1ria  # (or delete this input setting to use bot account)
        user_email: chzaaow@gmail.com  # (or delete this input setting to use bot account)
        commit_msg: ${{ github.event.head_commit.message }}  # (or delete this input setting to use hexo default settings)
    # Use the output from the `deploy` step(use for test action)
    - name: Get the output
      run: |
        echo "${{ steps.deploy.outputs.notify }}"


```

## 配置hexo

在仓库的配置文件`_config.yml`修改：

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