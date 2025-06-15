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

下载完成后会看到一个`python-3.12.X-amd64.exe`的文件，双击即可进行安装，不关心的话，一路点继续就行。

### VsCode安装及配置

Python只是一个解释器，日常使用还需要一个IDE（集成开发环境），这里我推荐VsCode（主流的还有PyCharm）。VsCode官网[下载地址](https://vscode.download.prss.microsoft.com/dbazure/download/stable/f1e16e1e6214d7c44d078b1f0607b2388f29d729/VSCodeUserSetup-x64-1.91.1.exe)，点击即可下载。

下载完成后会看到一个`VSCodeUserSetup-x64-X.X.X.exe`，同样是双击进行安装就好，不关心的话，一路点击继续就行。

> 补充阅读：
>
> 1、项目/文件夹/文件 是什么概念？
>
> 单个文件可以完成需求，则新建一个py文件或者ipynb文件即可；
>
> 在VsCode中`新建文件`/`打开文件`即为这种模式；
>
> 如果需求比较复杂，涉及到多个文件间的交互融通，则以项目/文件夹的形式进行开发；
>
> 在VsCode中`打开文件夹`基本等于开一个项目，一般更推荐采用此种形式。
>
> 2、.py文件 or .ipynb文件的关系？
>
> 两则都是使用python作为解释器的文件，是非常常见且主流的python程序文件格式；
>
> .py文件可以很方便打开，.ipynb在社区版Pycharm是打不开的。但我们已经使用VsCode了，就不存在这种问题；
>
> .py执行时会从第一行执行到最后一行，而.ipynb文件则是以模块运行（这种模块成为cell）；
>
> 推荐使用场景为，.ipynb更适合简单的程序编写和调试，在跑通后，再转为.py执行。防杠声明：熟手当然可以直接写.py。

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

## 虚拟环境

### Why

> 补充阅读：
>
> 3、Python中`类库`的概念是什么？
>
> Python代码开头的 `from XX.YY import ZZ as FF`，就是涉及到`类库`的使用；
>
> `类库`的名字有点奇怪，一般也被称为库或则包，但是官方确实叫`类库（package）`；
>
> 通过使用其他人写好的库，可以避免重复工作（又称之为偷懒）；
>
> 安装Python时，会附赠一些基础的库，但是更多的库需要自己安装；

在上文中，我们率先安装了Python3.12，这可以理解为此台电脑中最基础的环境，代称为Base环境。在实际开发中，会遇到以下问题：

1、这个项目用不了Python3.12，得用Python3.8，那我需要重装Python吗？

不用，电脑可以装多个版本的Python，并不冲突，在项目中选好需要使用的版本就行。

2、我这个项目需要使用新版的某个库，但是我以前一直用的老版的这个库，我更新后，以前的代码跑不了，怎么办？

你可以选择，旧项目再切老版本类库，也可以再装一个Python环境，但是我更推荐虚拟环境（VenV)

### What

> 虚拟环境可以给让你的项目开发更迅速和准确

虚拟环境相当于在项目/文件夹中，复制了Base环境中的Python，作为分身。注意，只是复制了Python解释器+基础的库，项目需要的特定库，需要再安装。

如果所有的项目都用Base环境，Base中就会有大量的库，即庞大（索引耗时）也容易发生冲突。特定的项目使用特定的库，显然是最优解。补充阅读[内容](https://euclid-jie.github.io/Euclidbooktry/python/%E8%99%9A%E6%8B%9F%E7%8E%AF%E5%A2%83.html)。

### How to Creat

> 补充阅读：
>
> 4、命令行是什么？
>
> 在使用VsCode进行Python开发过程中，这些词：终端（terminal）/控制台（console）/cmd/命令行/powershell，都大差不大，以下我统称为命令行；
>
> 命令行就是输入一行代码，回车，电脑干一件事。在VsCode中，可以通过快捷键`Ctrl + ~` 使其出现。

创建虚拟环境，VsCode会有提醒，让你创建。如果没有，则在命令行执行相关指令进行创建。

使用VsCode进入某个文件夹，唤出终端，执行以下命令

```base
python -m venv .venv
```

执行结束后，目录中出现`.venv`，则创建成功，如下图：

![image-20240801224425907](https://euclid-picgo.oss-cn-shenzhen.aliyuncs.com/image/202408012244737.png)

图中有个pip.ini需要自己创建并填入以下内容，不然后续安装库会很慢

```ini
[global]
timeout = 6000
index-url=http://mirrors.aliyun.com/pypi/simple/
[install]
trusted-host=mirrors.aliyun.com
```

> 补充阅读：
>
> 4、pip是什么？
>
> 前文提到，实际中使用Python开发，需要使用很多库，那么如果管理这些包（安装/卸载/更新），推荐使用pip，这是Python安装时附送的；
>
> 最基础的命令是在命令行中执行 `pip install package_name`，以达到安装指定库的目的。详细的pip相关内容可以在[这里](https://euclid-jie.github.io/Euclidbooktry/python/pip.html)看，也是鄙人写的。

### How to Use

为了区分Base环境和虚拟环境，显然需要配置，涉及到两种情况：

1、VsCode中，选择解释器，选为虚拟环境中的`python.exe`，路径为：".venv\Scripts\python.exe"；

2、命令行中，默认是Base环境，需要激活虚拟环境后，再进行pip及其他操作。在命令行中执行以下代码：

```bash
.\.venv\Scripts\activate
```

看到命令行变成下图情况（头部多了绿色部分），则表示激活成功

![image-20240801225447743](https://euclid-picgo.oss-cn-shenzhen.aliyuncs.com/image/202408012254447.png)

## 打包Python

将python代码打包为exe，方便分享和使用。非常推荐使用虚拟环境的基础上，进行打包，因为需要对环境中的库进行编译，虚拟环境中可以只安装项目中需要的库，这样以来，打包速度更快，打包出来的exe程序，占用空间更小。

### How

安装库，使用`pyinstaller`进行打包，所以需要安装

```base
pip install pyinstaller 
```

命令行中执行打包命令，-i 指定了exe的图标，`Element_A.ico`为图标文件，`main.py`为需要打包的程序

```base
Pyinstaller -F -i .\Element_A.ico .\main.py
```

打包完成的程序路径为".\dist\mian.exe"，双击即可使用。此exe不再依赖于pyton环境，发送给其他人也可以执行。一般此exe大小在100M以内。
