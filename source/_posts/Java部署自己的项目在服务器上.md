---
title: 部署Java项目
date: 2019年6月28日 16:00:00
categories: Java
tags: [deployment]
---

## 如何部署自己的项目在服务器上

> 主要是点来点去的。   

### 一、登录服务器

点击IDEA菜单栏的`tools`->`Deplment`，输入服务器的账号和密码，测试一下，查看是否连接成功。这里连接道康服务器，之前已经输入过账号和密码，连接成功。

`tools`->`start SSH session`,终端进入道康服务器。

输入指令`ps aux | grep visualcensus`过滤查找之前运行的进程号。

杀死之前的那个进程`kill 1215`，1215为上面查找的进程id。

<!--more-->

### 二、打包

当然要配合数据库，但是因为男神操作太快，这里就没能记录下来。

右侧边状态栏，点击`Maven`，在`Lifecycle`中点击`package`，将整个项目打包。等待打包完成。

### 三、上传

直接将打包好的额target包下面的`.jar`文件用鼠标拖到`Remote Host`中的visualcensus中，在命令行输入`nohup java -jar visualcensusserver-0.0.1-SNAPSHOT.jar > /dev/null 2>&1 &`y运行java工程。



