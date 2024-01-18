---
title: OneDrive
tags:
  - tools
categories:
  - tools
date: 2024-01-17 23:41:27
---

作为一个windows用户，我有多台设备，同时我和对象有共享文件的需求；很多时候，我希望在不同设备上接力完成工作，这时候我发现，OneDrive真的是个好东西。

## OneDrive

使用Windows设备，那肯定会附送一个OneDrive账号，下载OneDrive软件，或者登录[网页版](https://onedrive.live.com/)即可查看；简单来说，OneDrive相当于一个网盘，只要你启动了OneDrive软件，在文件资源管理器就会有一个文件夹，可以理解为挂载在本地的网盘，如下图所示

![image-20240117235040349](https://euclid-picgo.oss-cn-shenzhen.aliyuncs.com/image/202401172350397.png)

### 教育邮箱

如果你有可以正常接受验证码的校园邮箱，就可以神奇教育版本的OneDrive，这个版本的空间会更大为1T，普通版是5G，我就是两个OneDirve，一个普通一个教育，申请[地址](https://www.microsoft.com/zh-cn/education/products/office)

## 多设备同步

很多教程说的是使用`mklink -d`命令实现同步，但是实际操作中遇到命令行窗口权限问题，文件不自动上传等问题，所以我的解决步骤是：

1. 打开网页版[Onedrive](https://onedrive.live.com/)，新建文件夹

   - 普通版界面

   ![image-20240118000105796](https://euclid-picgo.oss-cn-shenzhen.aliyuncs.com/image/202401180001864.png)

   - 教育版界面

   ![image-20240118000034298](https://euclid-picgo.oss-cn-shenzhen.aliyuncs.com/image/202401180000387.png)

2. 在Windows设备上的文件资源管理器就能看到OneDrive目录下的新建文件夹，设置为始终保留在此设备上，即可

   ![屏幕截图 2024-01-18 000324](https://euclid-picgo.oss-cn-shenzhen.aliyuncs.com/image/202401180005424.png)

3. 在网页端操作，与人共享文件夹，发送邮箱后即可完成共享，被共享的人可以在已共享中查看文件夹，并添加快捷方式

   ![image-20240118000728845](https://euclid-picgo.oss-cn-shenzhen.aliyuncs.com/image/202401180007896.png)

    ​	![](https://euclid-picgo.oss-cn-shenzhen.aliyuncs.com/image/202401180009432.png)

## 用法推荐

### 设置Typora同步

Typora的主题文件夹，路径为: C:\Users\Ouwei\AppData\Roaming\Typora\themes; 快捷键相关配置文件在: C:\Users\Ouwei\AppData\Roaming\Typora\conf。

我有多个Windows设备，故此我可以在OneDrive创建对应的文件夹，按照[设备同步](#多设备同步)的步骤创建，本在本地的OneDrive中设置始终保持在此设备，如图所示：

![image-20240118231414937](https://euclid-picgo.oss-cn-shenzhen.aliyuncs.com/image/202401182314983.png)

具体的步骤为：

- step1: 在其中一个设备中（主要设备），需要将现有的C:\Users\Ouwei\AppData\Roaming\Typora\themes, C:\Users\Ouwei\AppData\Roaming\Typora\conf下的文件复制到对应的OneDrive文件夹下（上图中所示）

- step2: 删除C:\Users\Ouwei\AppData\Roaming\Typora\themes, C:\Users\Ouwei\AppData\Roaming\Typora\conf

- step3: 使用`mklink /d`实现，代码为

  ```bash
  mklink /d C:\Users\Ouwei\AppData\Roaming\Typora\themes "C:\Users\Ouwei\OneDrive - mail.bnu.edu.cn\应用\typora"
  mklink /d C:\Users\Ouwei\AppData\Roaming\Typora\conf "C:\Users\Ouwei\OneDrive - mail.bnu.edu.cn\应用\conf"
  ```

  需要注意的是：

  1、必须先删除C:\Users\Ouwei\AppData\Roaming\Typora\themes和C:\Users\Ouwei\AppData\Roaming\Typora\conf才能操作

  2、注意顺序，需要同步的地方在前，OneDrive在后

  3、如果路径中有中文需要用""处理路径

- step4: 在另一台设备上，按照step2, step3操作，即可完成
