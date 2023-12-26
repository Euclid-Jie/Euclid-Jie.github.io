---
title: Useful GitHub Action
tags:
  - github
  - python
categories:
  - null
date: 2023-12-25 22:51:26
---
最近在琢磨定时执行任务，实现对微博热搜榜单的抓取，并实现存档。

最开始以为需要一台服务器，但是我发现了`gtihub action`这个好东西，对于简单的`python`任务，最快可以每`5min`执行一次。

借助`github action`，我的想法很快落地，项目地址为：[Euclid-Jie/weibo_hotlist](https://github.com/Euclid-Jie/weibo_hotlist)

## GitHub Action

### 简介

> [GitHub Actions 文档 - GitHub 文档](https://docs.github.com/zh/actions)

在`repo`里的`.github/workflows`下配置一个`.yml`文件，`github`就会提供一个执行环境，按照此`.yml`中的步骤进行执行，可以设置为定时任务，或者是被某个动作触发执行，例如`push`，`pull request`等

## How to use

其实这个很简单，直接上例子，并配上说明

### 定期执行`python`程序

主要流程为在`github`提供的运行机器上，`clone repo` -> `checkout branch` -> `run start.sh`运行程序抓取数据比写入 -> 使用`git`提交代码并`push`

```yaml
name: 5min定时抓取

on:
  workflow_dispatch:
  schedule:
  - cron: '*/5 * * * *'

jobs:
  build:
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@main
        with:
          persist-credentials: false
          fetch-depth: 0

      - name: run start.sh
        run: |
          pip install pandas
          bash ./start.sh
          
      - name: Set timestamp
        id: timestamp
        run: echo "::set-output name=timestamp::$(date +'%Y-%m-%d %H:%M')"

      - name: Git add and commit
        run: |
          git config --global user.email "ouweijie123@outlook.com"
          git config --global user.name "crawer actioner"
          git add .
          git commit -m "craw_5minly_${{ steps.timestamp.outputs.timestamp }}"

      - name: GitHub Push
        uses: ad-m/github-push-action@master
        with:
          github_token: ${{ secrets.REPO_TOKEN }}
          branch: master
```

以上`.yml`中需要注意的点有：

1. `cron: '*/5 * * * *'`实现了每`5min`执行一次，但事实上可能比这个频率低，`1h`运行较为稳定

2. 其中执行`python`的是`.sh`，具体内容为`python3 ./weibo_hot_list_serach.py`

3. 使用`git`需要配置`name`和`mail`，其中`name`是必须的

4. 使用`git push`时，需要开启`action`的权限，不然会报错，如图

   ![image-20231225231602980](https://euclid-picgo.oss-cn-shenzhen.aliyuncs.com/image/202312252316187.png)

5. 如果这个`repo`是公开的，`secrets.REPO_TOKEN`是非必须的

### 每个月进行存档

主要流程为在`github`提供的运行机器上，`clone repo` -> `checkout branch` -> 将`hotlist`目录下的`.csv`文件压缩到`archive`目录下，并删除原始`.csv`文件-> 使用`git`提交代码并`push`

目的是为了减少`repo`的空间占用，压缩比率在$15\%$左右

```yaml
name: 每月存档

on:
  workflow_dispatch:
  schedule:
  - cron: '0 0 1 * *'

jobs:
  build:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout code
        uses: actions/checkout@main

      - name: Create archive directory
        run: |
          if [ ! -d "archive" ]; then
            mkdir archive
          fi

      - name: Set timestamp
        id: timestamp
        run: echo "::set-output name=timestamp::$(date +'%Y-%m')"

      - name: Create archive
        run: tar -czvf "archive/monthly_archive_${{ steps.timestamp.outputs.timestamp }}.tar.gz" hotlist/*.csv

      - name: Delete original data
        run: rm -rf hotlist/*.csv

      - name: Git add and commit
        run: |
          git config --global user.email "ouweijie123@outlook.com"
          git config --global user.name "archiver actioner"
          git add .
          git commit -m "monthly_archive_${{ steps.timestamp.outputs.timestamp }}"
          git push
```

以上`.yml`中需要注意的点有：

先看上个例子中的注意事项

1. `cron: '0 0 1 * *'`实现了每个月的1号执行一次

2. 直接进行`git push`也行，不一定要使用别人的工具