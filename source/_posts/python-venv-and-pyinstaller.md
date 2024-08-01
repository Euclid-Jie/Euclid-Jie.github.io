---
title: python venv and pyinstaller
tags:
  - python
categories:
  - tutorials
date: 2024-08-01 19:34:36
---
新手教程导向，方案是python+vscode，使用虚拟环境（Venv），并使用pyinstaller打包程序

> 注：以下软件的下载链接试用于大部分Windows电脑，如果是其他系统，请另行处理

## Python安装

目前主流使用的python有两个大版本，分别是python2.X和python3.X，一般使用的都是python3.X，截至发文最新为python3.12。

### 下载及安装

这是[官网下载地址](https://www.python.org/ftp/python/3.12.4/python-3.12.4-amd64.exe)，点击即可下载python3.12，因为神秘原因会比较慢，等不及的也可以找国内的镜像。

下载完成后会看到一个`python-3.12.X-amd64.exe`的文件，双击既进行安装，不关心的话，一路点继续就行。

### VsCode安装及配置

Python只是一个解释器，日常使用还需要一个IDE（集成开发环境），这里我推荐VsCode（主流的还有PyCharm）。VsCode官网[下载地址](https://vscode.download.prss.microsoft.com/dbazure/download/stable/f1e16e1e6214d7c44d078b1f0607b2388f29d729/VSCodeUserSetup-x64-1.91.1.exe)，点击即可下载。

下载完成后会看到一个`VSCodeUserSetup-x64-X.X.X.exe`，同样是双击进行安装就好，不关心的话，一路点击继续就行。

安装完成后，打开VsCode，需要安装一些插件，我将挨个说明：

- 在哪安装插件

  如图所示，最左边有个四个方格的图标，是管理插件的地方，随即可以检索需要的插件名称，选中想要的插件，然后安装即可，一般安装后，会要求重启VsCode

  > 下图会与你看到的有出入，但是凭借你的聪明才智，一定可以找到每一步需要点击的地方！

  ![image-20240801195608201](https://euclid-picgo.oss-cn-shenzhen.aliyuncs.com/image/202408011956706.png)

- 安装中文插件，这样VsCode的界面就是中文（类似上图）

  插件名：Chinese (Simplified) 

  简介：此中文（简体）语言包为 VS Code 提供本地化界面。

- 安装python插件，这样VsCode就能调用Python解释开展工作

  插件名：Python，点第一个，下载量最多的那个，发行商为Microsoft

  简介：专门为Python提供的插件。顺带说一句，VsCode还能作为C，C++，R等语言的IDE使用，很强大。

- 安装jupyter插件，这样就能使用notebook模式运行Python，获得”所见即所得“的体验

  插件名：Jupyter，点第一个，下载量最多的那个，发行商为Microsoft。安装此插件，会顺带安装一些其他的插件，不用关心。

  简介：适配使用notebook模式，进行Pyhton语言的编写和运行

最后你需要告诉VsCode你的Python解释器在哪。文件>新建文件>命名为`demo.py`，随即打开此文件，不出意外你在右下角会看到，提醒你选中Python解释器的位置。点击>选择解释器>输入解释器路径，然后选择你安装的python的位置，一般为"C:\Program Files\Python\Python312\python.exe"或者"C:\Users\Ouwei\AppData\Local\Programs\Python\Python312\python.exe"。至此配置结束。

![image-20240801200451308](https://euclid-picgo.oss-cn-shenzhen.aliyuncs.com/image/202408012017826.png)

> 以下内容下次再补充

## 虚拟环境

为什么要创建虚拟环境

### 打包Python

将python代码打包为exe，方便分享和使用

