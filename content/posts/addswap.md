---
title: 设置Swap增加小内存Linux服务内存并且设置持久化
subtitle: 设置Swap增加小内存Linux服务内存并且设置持久化
date: 2023-08-23T22:37:01+08:00
draft: false
author:
  name: waka
  link:
  email:
  avatar: /images/avatar.jpg
description: sawp, linux
keywords: swap, linux
license:
comment: true
weight: 0
tags:
  - sawp
  - linux
categories:
  - linux
  - swap
hiddenFromHomePage: false
hiddenFromSearch: false
summary:
resources:
  - name: featured-image
    src: featured-image.jpg
  - name: featured-image-preview
    src: featured-image-preview.jpg
toc: true
math: false
lightgallery: false
password:
message:
repost:
  enable: false
  url:

# See details front matter: https://fixit.lruihao.cn/documentation/content/#front-matter
---
### 第一步: 创建一个空白文件
切换到root权限

使用命令dd if=/dev/zero of=/opt/swapfile bs=1M count=1024来创建一个1GB的空白文件到/opt/swapfile。同样的可以按照这个方法创建其他大小的空白文件。我一般会创建和内存一样大小的交换空间

设置一下权限chmod 600 /opt/swapfile

### 第二步: 创建交换空间
使用命令mkswap /opt/swapfile来将这个空白文件变成一个swap文件。只有该文件才能将对应的磁盘空间作为一个临时内存。

### 第三步: 应用交换文件
使用命令swapon /opt/swapfile将该交换空间应用到系统中。此时执行free -h可以看到swap一行多出了1GB空间

### 第四步：设置重启自动挂载
您所描述的步骤是在 Linux 系统中创建并启用一个交换空间文件。在这些步骤之后，您需要确保交换空间在系统重启后仍然存在和启用。以下是让这个交换空间在重启后仍然存在和启用的步骤：

1. 编辑 `/etc/fstab` 文件：这个文件包含了系统启动时要挂载的文件系统和交换空间的信息。您可以使用文本编辑器（如 `sudo nano` 或 `sudo vim`）来编辑这个文件。

   ```
   sudo nano /etc/fstab
   ```

2. 在 `/etc/fstab` 文件的末尾添加以下行，以将交换空间文件自动挂载为交换空间：

   ```
   /opt/swapfile none swap sw 0 0
   ```

   其中，第一列是交换空间文件的路径，第二列是文件系统类型（none 表示没有实际的文件系统），第三列是选项（`swap` 表示该文件是一个交换空间），最后两列是与文件系统检查和备份相关的选项。

3. 保存并退出编辑器。

4. 重新启动系统：重启您的计算机，交换空间应该在系统启动时自动挂载。

您可以使用以下命令来验证交换空间是否在系统重启后仍然存在：

```
swapon --show
```

这将显示当前启用的交换空间。如果您看到 `/opt/swapfile` 列出在其中，说明您成功地让交换空间在重启后保留下来。

请注意，在进行任何与系统配置相关的更改时，请谨慎操作并确保备份重要数据。不正确的配置可能会导致系统问题。
<!--more-->
