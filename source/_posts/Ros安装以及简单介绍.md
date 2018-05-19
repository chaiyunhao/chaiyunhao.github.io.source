---
title: Ros安装以及简单介绍
date: 2018-05-16 20:07:38
comments: true
tags:
    - ROS
---

### 什么是ROS
ROS(Robot Operating System)按照字面意思就是一个开源的机器人软件平台。
目前ROS开发支持c++和Python语言。

### 如何安装ROS

#### 操作系统和ROS版本选择
这里推荐使用操作系统版本为 Ubuntu 16.04 ROS版本为 kinetic

#### apt安装
1. 添加地址到sources.list
sudo sh -c 'echo "deb http://packages.ros.org/ros/ubuntu $(lsb_release -sc) main" > /etc/apt/sources.list.d/ros-latest.list'

2. 设置key
sudo apt-key adv --keyserver hkp://ha.pool.sks-keyservers.net:80 --recv-key 421C365BD9FF1F717815A3895523BAEEB01FA116

3. 安装
首先执行
```
sudo apt-get update
```
然后选择安装哪些ROS相关的包，这里推荐全量安装，包含ROS, rqt, rviz, robot-generic libraries, 2D/3D simulators, navigation and 2D/3D perception
```
sudo apt-get install ros-kinetic-desktop-full
```

#### rosdep初始化
```
sudo rosdep init
rosdep update
```

#### 环境变量设置

如果你想要每次启动系统或者打开新的终端都不需要重新设置ROS的环境变量，那么需要将ROS的环境变量添加到bashrs里
```
echo "source /opt/ros/kinetic/setup.bash" >> ~/.bashrc
source ~/.bashrc
```
或者每次打开端口执行 `source /opt/ros/kinetic/setup.bash`

#### 安装其他依赖

```
sudo apt-get install python-rosinstall python-rosinstall-generator python-wstool build-essential
```

#### 验证安装

终端执行命令 `roscore`

如果可以正常运行，则输出相关内容，最终将 roscore 的服务运行起来，可以在终端看到一下内容
```
started core service [/rosout]
```
