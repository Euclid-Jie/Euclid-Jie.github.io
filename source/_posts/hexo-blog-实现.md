---
title: hexo blog 实现
date: 2023-10-15 22:08:30
tags: 
  - node
  - hexo
---

> 主要参照于[此文](http://blog.haoji.me/build-blog-website-by-hexo-github.html?from=xa)，但是针对自己的需求做了一些改变

## 准备工作

### 安装`node.js`

``node.js`是`js`运行的必要条件，关于`node.js`的安装，各种教程很多，直接从[官网](https://nodejs.org/en)下载安装即可

-  检测是否安装成功

  > 在`cmd`中运行以下代码，如果返回版本号，即成功

  ```shell
  node -v
  ```

- `npm`工具

  安装`node.js`会附赠此工具，`npm`用于管理`node.js`的包，对我而言，我将`node.js`类比于`Python`解释器，将`npm`类比于`pip`

  > 同样在`cmd`中运行以下代码，如果返回版本号，即成功

  ```shell
  npm -v
  ```

### 确认`node.js`的版本

据我了解，如果想要运行`hexo`，需要的`node.js`版本需要再14以上。由于我之前还部署过`gitbook`，本机的`node.js`版本为`v10.24.1`，所以需要更新`node.js`版本，但是如果我要兼容多个版本的`node.js`当然也是可以的，用到下文中的`nvmv`即可实现

### `nvmw`工具

> 用于管理多个`node.js`版本，由于我是windows11，故选择此工具

- 直接使用`npm`进行安装即可

  > 默认会安装在`~\user\nvmw`路径下

  ```shell
  npm install -g nvmw
  ```

- 检测是否安装成功，注意此处是大写的`V`，如果返回版本号，即安装成功

  ```shell
  nvmw -V
  ```

- 使用`nvmw`安装指定版本的`node.js`

  > 可以将此步骤理解为，使用`conde`配置多个`python`虚拟环境，以下代码演示了安装`v16.17.1`

  ```shell
  nvmw install v16.17.1
  ```

- 查看当前的`node.js`可用版本

  > 将返回可用版本，并标识当前所处的版本，如果没有想要的版本，可以使用`nvmw`安装

  ```shell
  nvmw ls
  ```

- 切换`node.js`版本

  > 如果当前的版本，不是自己想要的版本，可以进行切换，以下代码演示了切换为`v10.24.1`版本

  ```shell
  nvmw switch v10.24.1
  ```

## 使用`hexo`构建博客

> `hexo`是一个成熟的博客生成，管理系统，有丰富的插件，主题生态，官网[链接](http://hexo.io)

### 安装`hexo`

- 使用`npm`进行全局安装`hexo`

  ```shell
  npm install -g hexo
  ```

### 初始化本地repo

> 从本步骤开始，会与网络上的主流教程不同，如果你有一定的`git`的使用经验，并希望对你的创作内容进行版本控制，我更推荐使用我的方案

- 创建一个文件夹目录，并命名为`{github_username}.github.io`

  > 关于这个文件夹的命名，主要是为了与`GitHub`的`repo`保持一致

- 使用`hexo`初始化

  > 以下的步骤，如果不加说明，都是在项目的`root`目录下进行

  `init`结束后，文件下会出现，诸如`node_mudules`，`publish`，`source`，`themes`等文件夹，说明已经初始化成功

  ```shell
  hexo init
  ```

- 使用`hexo`生产静态网页页面

  > 这一步的操作，可以理解为使用`hexo`工具对`source`目录下的的`md`格式文件，进行静态网页转换，相关的网页文件会保存在`public`目录下

  虽然还什么都没写，但是会有一个默认的介绍`hexo`的`hello_world.md`文件存放在`source`目录下

  ```shell
  hexo g
  ```

  > 如果想要清空生成的内容，使用
  ```shell
  hexo clean
  ```

- 预览网页

  会在本地4000端口生成一个网页(也可以指定端口），可以进行预览，后续只要把这个网页托管到githug.io或自己的服务器并绑定域名，既可以实现个人博客构建

  ```shell
  hexo s
  hexo s -p 90000
  ```

### 使用`hexo`组织博客内容

- 创建新的博客

  将成功创建名为"postName"的`md`文件在`source`下，编辑此文件，既可以实现博文撰写

  ```shell
  hexo new "postName"
  ```

###  切换博客`theme`

官方[网站](https://hexo.io/themes/)列出了很多不同风格的模板，网上的各种[推荐](https://www.zhihu.com/question/24422335/answer/3127401635)也很多，可根据自己的喜好选择

- 下载theme，以下为`cactus`示例

  > 将该主题下载在`thems`目录下，具体配置可以看主题[说明](https://github.com/probberechts/hexo-theme-cactus)

  ```shell
  git clone https://github.com/probberechts/hexo-theme-cactus.git themes/cactus
  ```
  由于我对主题的部分细节进行了修改，所以需要`clone`fork的版本
  ```shell
  git clone git@github.com:Euclid-Jie/hexo-theme-cactus.git themes/cactus
  ```
  

- 修改配置文件

  修改`_config.yml`中的`theme: landscape`改为`theme: yilia`，然后重新执行`hexo g`来重新生成

## 设置远程Repo

此时已经有了一个本地的`repo`，可以实现`md`转网页，并预览，只要将这个网页托管到网站上，既可以实现博客部署。后续如果博客的内容进行了修改，只要使用`hexo`重新生成后，将生成的网页内容`push`到远程，即可实现博客更新

### 在GitHub和创建repo

- 创建一个名为`{github_username}.github.io`的`repo`

- 绑定本地分支

  先在本地`repo`中使用`git`初始化，然后设置远程为刚刚创建的`repo`

- `pages`设置

  在远程`repo`的设置中，`pages`卡下，设置`branch`为`master/docs`

  > 意思为，需要托管的网页文件，存放在`~/docs`目录下，针对此目录下的文件，构建静态网页即可

![指定github pages 目录](https://euclid-picgo.oss-cn-shenzhen.aliyuncs.com/image/202310152308113.png)

### 推送设置

上文中设置了静态网页文件目录为`~/docs`，故需要在本地文件中进行配置

- 修改`_config.yml`

  ```shell
  # Directory
  source_dir: source
  public_dir: docs  # 改publish为docs
  ```

- 设置`.gitignore`文件

  可以根据自己的需求配置，需要注意的是，一定不要忽略`docs`目录，以下为我的配置

  ```shell
  .DS_Store
  Thumbs.db
  db.json
  *.log
  node_modules/
  public/
  .deploy*/
  _multiconfig.yml
  .history/
  themes/
  ```

- 推送流程

  在完成所有设置后，依次使用，即可实现博客推送

  - `hexo g` : 基于`source`目录下的`md`文件生成网页文件至`docs`目录
  - `git add .`：将所有更改暂存
  - `git commit`：提交本地更改
  -  `git push`：将本地更改推送至远程

  远程将根据当前的`~/docs`内容生成`pages`，网址为`https://{github_username}.github.io`

  

