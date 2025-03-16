---
title: GitHub Action构建Shiroi Docker镜像
description: Mix-Space，它属于前后端分离博客系统，你可以把前端和后端分离部署在不同的地方，在之前你可以把前端部署在Vercel云函数上，达到缓解服务器压力和提升访问速度的效果。
date: 2024-10-21 19:24:26
categories: [技术探索]
tags: [Mix-Space,Docker]
---

## 前言

Mix-Space，它属于前后端分离博客系统，你可以把前端和后端分离部署在不同的地方，在之前你可以把前端部署在Vercel云函数上，达到缓解服务器压力和提升访问速度的效果。

但是随着Vercel调整Hobby免费套餐的额度，Vercel免费套餐已经越来越不够用了，这个时候我们可以通过Docker将Shiro部署到自己的服务器上来解决问题，但是我在使用Shiroi（Shiro的闭源捐赠版）的时候遇到了问题：原作者innei并没有提供它的可用Docker镜像。

这样的话你就需要在自己服务器上对Shiroi进行构建，但是对于配置较低（低于2G内存）的云服务器来说这是比较困难的，基本上会导致服务器爆内存假死。

Innei给出的解决方案是使用Github Action完成构建，将构建后的产物直接推送到你的服务器上，达到缓解服务器压力的作用。

但是这个方案在我看来存在以下局限性：

- 需要你在服务器安装相关依赖：Node.js，PM2，Sharp，但是部分用户（比如我）使用的是1Panel管理服务器，不大想要在服务器安装额外依赖。

- 输出目录被固定在了服务器的`root`目录，不好更改。

- 你需要向GitHub仓库存储你的服务器登录信息，比如SSH密钥等。

- 项目本身有回滚功能，但一般用户可能不需要此功能，也会占用大量的服务器空间。

总之我是不大想要折腾这套方案，那么有啥更好的办法嘛？

欸，你说Docker部署不就行了，虽然Innei没给你Docker镜像，但是我们可以自己造啊！

## 思路

我们当然不能直接在自己服务器上构建镜像，构建Docker镜像产生的资源占用并不会比直接构建站点静态文件要少。

那我们可以把innei的思路拿过来用一下，用Github Action进行Docker镜像构建不就可以了？

但是光构建也不行，你还需要找个地方把镜像放下来，而我也不想使用直接推送服务器的办法，这也需要在Github这里存储服务器登录信息，虽然是`secret`存储应该能保证安全性，但是谁又能说准这个事情呢？

而且部分用户服务器在国内，Github Action的主动推送速度也不见得一定多好。

## 选择

最后我的选择方案是使用Github Action进行构建，然后将镜像传到Github Packages，Github Packages默认会对私有库镜像进行私有，也保障镜像不会直接泄露。

Docker 对镜像仓库的管理共分为 3 个层级，依次是命名空间 (namespace) 、镜像仓库 (repository) 和 标签 (tag)：

- 命名空间以名称作为标识，一个命名空间可管理多个镜像仓库。

- 镜像仓库通过名称标识，一个镜像仓库中可保存一个镜像（image）的多个版本。

- 镜像版本通过标签进行区分。

基于以上层级关系，一个完整的镜像路径 `{namespace}/{repository}:{tag}` 可以唯一确定一个镜像。

新建一个私有库（注意一定是私有仓库）并在`.github/workflows`目录下新建yml工作流文件，填入如下内容：

```
name: Docker

on:
  push:
    branches:
      - main
  schedule:
     - cron: '0 3 * * *'

  repository_dispatch:
    types: [trigger-workflow]

permissions: write-all
concurrency:
  group: ${{ github.workflow }}-${{ github.ref }}
  cancel-in-progress: true

env:
  PNPM_VERSION: 9.x.x
  HASH_FILE: build_hash

jobs:
  prepare:
    name: Prepare
    runs-on: ubuntu-latest
    if: ${{ github.event.head_commit.message != 'Update hash file' }}

    outputs:
      hash_content: ${{ steps.read_hash.outputs.hash_content }}

    steps:
      - name: Checkout
        uses: actions/checkout@v4
      - name: Read HASH_FILE content
        id: read_hash
        run: |
          content=$(cat ${{ env.HASH_FILE }}) || true
          echo "hash_content=$content" >> "$GITHUB_OUTPUT"
  check:
    name: Check Should Rebuild
    runs-on: ubuntu-latest
    needs: prepare
    outputs:
      canceled: ${{ steps.use_content.outputs.canceled }}

    steps:
      - uses: actions/checkout@v4
        with:
          repository: innei-dev/shiroi
          token: ${{ secrets.GH_PAT }}
          fetch-depth: 0
          lfs: true

      - name: Use content from prev job and compare
        id: use_content
        env:
          FILE_HASH: ${{ needs.prepare.outputs.hash_content }}
        run: |
          file_hash=$FILE_HASH
          current_hash=$(git rev-parse --short HEAD)
          echo "File Hash: $file_hash"
          echo "Current Git Hash: $current_hash"
          if [ "$file_hash" == "$current_hash" ]; then
            echo "Hashes match. Stopping workflow."
            echo "canceled=true" >> $GITHUB_OUTPUT
          else
            echo "Hashes do not match. Continuing workflow."
          fi

  build:
    name: Build artifact
    runs-on: ubuntu-latest
    needs: check
    if: ${{needs.check.outputs.canceled != 'true'}}

    outputs:
      sha_short: ${{ steps.store.outputs.sha_short }}
      branch: ${{ steps.store.outputs.branch }}

    steps:
    - uses: actions/checkout@v4
      with:
        repository: innei-dev/shiroi
        token: ${{ secrets.GH_PAT }}
        fetch-depth: 0
        lfs: true

    - name: Login to Registry
      uses: docker/login-action@v3
      with:
        registry: ghcr.io
        username: ${{ github.actor }}
        password: ${{ secrets.GITHUB_TOKEN }}

    - name: Build Docker Image
      run: |
        docker build -t ghcr.io/${{ secrets.DOCKER_NAMESPACE }}/shiroi:latest .

    - name: Push Docker Image to Github
      run: |
        docker push ghcr.io/${{ secrets.DOCKER_NAMESPACE }}/shiroi:latest

    - name: Store artifact commit version
      shell: bash
      id: store
      run: |
        sha_short=$(git rev-parse --short HEAD)
        branch_name=$(git rev-parse --abbrev-ref HEAD)
        echo "sha_short=$sha_short" >> "$GITHUB_OUTPUT"
        echo "branch=$branch_name" >> "$GITHUB_OUTPUT"
  store:
    name: Store artifact commit version
    runs-on: ubuntu-latest
    needs: [build]
    steps:

      - name: Checkout
        uses: actions/checkout@v4
        with:
          persist-credentials: false
          fetch-depth: 0

      - name: Use outputs from build
        env:
          SHA_SHORT: ${{ needs.build.outputs.sha_short }}
          BRANCH: ${{ needs.build.outputs.branch }}
        run: |
          echo "SHA Short from build: $SHA_SHORT"
          echo "Branch from build: $BRANCH"
      - name: Write hash to file
        env:
          SHA_SHORT: ${{ needs.build.outputs.sha_short }}

        run: |
          echo "SHA_SHORT: $SHA_SHORT"
          echo $SHA_SHORT > ${{ env.HASH_FILE }}
      - name: Commit files
        run: |
          git config --local user.email "41898282+github-actions[bot]@users.noreply.github.com"
          git config --local user.name "github-actions[bot]"
          git add ${{ env.HASH_FILE }}
          git status
          git commit -a -m "Update hash file"
      - name: Push changes
        uses: ad-m/github-push-action@master
        with:
          github_token: ${{ secrets.GITHUB_TOKEN }}
          branch: ${{ github.ref }}
```

这样就可以实现简单的构建并上传Github Registry镜像，需要你在仓库的`secret`设置中配置一个机密变量：

- `GH_PAT`：有权限访问Shiroi仓库的Github Access Token。

- `DOCKER_NAMESPACE`：镜像的命名空间，其中不要有任何大写字符，为了好记和防冲突可能性尽量选择使用个人Github用户名的小写字符。

> 由于 GitHub action 的限制，当一个仓库在 3 个月内没有活动时，工作流会被禁用。
> @innei

所以我们采用innei的办法，每次构建结束后上传一个存储哈希值的文件，来保持仓库活动，而且在构建前对仓库哈希值进行对比，也不会出现重复构建的情况。

参考上面修改环境`secret`后，运行工作流（注意先开启你仓库设置中Github Action写入文件的权限），这样就会生成哈希值文件并且构建镜像了。

## 使用

保存工作流文件，等待运行完毕，你应该可以在仓库侧边`Packages`或者个人Github账号主页的`Package`里找到镜像文件。

在你个人服务器上拉取镜像前，你需要先在你的服务器上配置Docker私有仓库，值得注意的是你的Gitea实例必须是HTTPS地址，不然Docker会拒绝拉取不安全的私有仓库。

在你的服务器上输入以下指令登录Github Resgistry私有仓库

```
docker login ghcr.io
```

然后会提示你输入账号和有访问权限的Github Access Token，确认登录后你就可以拉取你的私有仓库镜像了，如果你是使用的1Panel的话，可以在容器的仓库设置直接设置私有仓库。

然后你也可以使用下面的compose文件进行配置安装Shiroi：

```
services:
  shiro:
    container_name: Shiroi
    image:
    restart: always
    environment:
      - NEXT_SHARP_PATH=/usr/local/lib/node_modules/sharp
      - NEXT_PUBLIC_API_URL=https://api.example.com/api/v2
      - NEXT_PUBLIC_GATEWAY_URL=https://api.example.com
    ports:
      - 127.0.0.1:2323:2323
    networks:
      - 1panel-network
networks:
  1panel-network:
    external: true
```

`image`这里填写你在软件包仓库这里看到的容器镜像信息，当然这是适用于1Panel的，常规部署也可以用下面这个

```
services:
  shiro:
    container_name: Shiroi
    image:
    restart: always
    environment:
      - NEXT_SHARP_PATH=/usr/local/lib/node_modules/sharp
      - NEXT_PUBLIC_API_URL=https://api.example.com/api/v2
      - NEXT_PUBLIC_GATEWAY_URL=https://api.example.com
    ports:
      - 127.0.0.1:2323:2323
    networks:
      - mx-network
```

## 后话

这样你就算是简单完成了，本文本质上偏专业系，而非喂饭文，如果有疑问可以评论区提问或者搭配搜索引擎食用本文。
