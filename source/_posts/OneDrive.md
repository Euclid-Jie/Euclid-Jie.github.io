---
title: OneDrive
tags:
  - tools
categories:
  - tools
date: 2024-01-17 23:41:27
---

作为一个windows用户，我有多台设备，同时我和对象有共享文件的需求；很多时候，我希望在不同设备上接力完成工作，这时候我发现，OneDriver真的是个好东西。

## OneDrive

使用Windows设备，那肯定会附送一个OneDrive账号，下载Onedrive软件，或者登录[网页版](https://onedrive.live.com/)即可查看；简单来说，Onedrive相当于一个网盘，只要你启动了OneDrive软件，在文件资源管理器就会有一个文件夹，可以理解为挂载在本地的网盘，如下图所示

![image-20240117235040349](https://euclid-picgo.oss-cn-shenzhen.aliyuncs.com/image/202401172350397.png)

### 教育邮箱

如果你有可以正常接受验证码的校园邮箱，就可以神奇教育版本的OneDrive，这个版本的空间会更大为1T，普通版是5G，我就是两个Onedirve，一个普通一个教育，申请[地址](https://www.microsoft.com/zh-cn/education/products/office)

## 多设备同步

很多教程说的是使用`mklink -d`命令实现同步，但是实际操作中遇到命令行窗口权限问题，文件不自动上传等问题，所以我的解决步骤是：

1. 打开网页版[Onedrive](https://onedrive.live.com/)，新建文件夹

   - 普通版界面

   ![image-20240118000105796](https://euclid-picgo.oss-cn-shenzhen.aliyuncs.com/image/202401180001864.png)

   - 教育版界面

   ![image-20240118000034298](https://euclid-picgo.oss-cn-shenzhen.aliyuncs.com/image/202401180000387.png)

2. 在Windows设备上的文件资源管理器就能看到Onedrive目录下的新建文件夹，设置为始终保留在此设备上，即可

   ![屏幕截图 2024-01-18 000324](https://euclid-picgo.oss-cn-shenzhen.aliyuncs.com/image/202401180005424.png)

3. 在网页端操作，与人共享文件夹，发送邮箱后即可完成共享，被共享的人可以在已共享中查看文件夹，并添加快捷方式

   ![image-20240118000728845](https://euclid-picgo.oss-cn-shenzhen.aliyuncs.com/image/202401180007896.png)

    ​	![](https://euclid-picgo.oss-cn-shenzhen.aliyuncs.com/image/202401180009432.png)
