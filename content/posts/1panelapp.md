---
title: 1pane Linux管理面板安装第三方软件库和设置自动更新
subtitle: 1pane Linux管理面板安装第三方软件库和设置自动更新
date: 2023-08-07T03:00:37+08:00
draft: false
author:
  name: waka
  link:
  email:
  avatar: /images/avatar.jpg
description: 1pane Linux管理面板安装第三方软件库和设置自动更新
keywords: 1pane, Linux, 管理面板, 安装第三方软件库, 设置自动更新
license: 
comment: true
weight: 0
tags:
  - 1pane
  - Linux
categories:
  - 1pane
  - Linux
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
## 1panel Liunx管理面板安装第三方软件库并设置自动更新教程

### 1、准备：安装好的1panel面板

1panel文档：https://1panel.cn/docs/

默认 `1Panel` 安装在 `/opt/` 路径下，如果不是可按需修改。

关于大陆电磁环境复杂，`github` 网络连接可能有问题，可以自行搜寻解决方式，如 `ghproxy` 等。

项目地址：https://github.com/okxlin/appstore

### 2、使用方法

默认`1Panel`安装在`/opt/`路径下，如果不是按需修改以下。

#### 2.1

- 方式一：使用`git` 方式获取应用到`/opt/1panel/resource/apps/local`文件夹下

```
# 克隆名为 localApps 的分支的仓库到 /opt/1panel/resource/apps/local/appstore-localApps 目录下
git clone -b localApps https://github.com/okxlin/appstore /opt/1panel/resource/apps/local/appstore-localApps

# 将 /opt/1panel/resource/apps/local/appstore-localApps/apps 目录下的所有文件复制到 /opt/1panel/resource/apps/local/ 目录下
cp -rf /opt/1panel/resource/apps/local/appstore-localApps/apps/* /opt/1panel/resource/apps/local/

# 删除 /opt/1panel/resource/apps/local/appstore-localApps 目录及其内容
rm -r /opt/1panel/resource/apps/local/appstore-localApps
```



然后应用商店刷新本地应用即可。

##### 设置自动更新：

1、在1panel面板中添加一个计划任务，类型是执行shell脚本

2、在脚本内容里添加如下脚本：

```shell
#!/bin/bash

# 设置远程仓库URL和本地目标路径
remote_repo="https://github.com/okxlin/appstore"
branch="localApps"
local_path="/opt/1panel/resource/apps/local/appstore-localApps"

# 执行git clone命令
git clone -b $branch $remote_repo $local_path

# 检查git clone命令是否成功执行
if [ $? -eq 0 ]; then
  echo "Git clone 完成，开始复制文件..."
  # 复制文件到指定目录
  cp -rf $local_path/apps/* /opt/1panel/resource/apps/local/
  if [ $? -eq 0 ]; then
    echo "文件复制完成，开始清理..."
    # 删除临时文件夹
    rm -r $local_path
    if [ $? -eq 0 ]; then
      echo "清理完成"
    else
      echo "清理失败，请检查错误信息"
    fi
  else
    echo "文件复制失败，请检查错误信息"
  fi
else
  echo "Git clone 失败，请检查错误信息"
fi
```

3、设置为每天执行一次或者每2小时执行一次

其余方式可参考项目readme有详细介绍

![app-list.png](https://s1.imagehub.cc/images/2023/08/07/app-list.png)


<!--more-->
