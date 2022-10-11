# ROS_notes


## 换源

```
sudo cp /etc/apt/sources.list /etc/apt/sources.list.back
sudo gedit /etc/apt/sources.list
```

将内容替换为
清华源

```

deb http://mirrors.tuna.tsinghua.edu.cn/ubuntu/ focal main restricted
deb http://mirrors.tuna.tsinghua.edu.cn/ubuntu/ focal-updates main restricted
deb http://mirrors.tuna.tsinghua.edu.cn/ubuntu/ focal universe
deb http://mirrors.tuna.tsinghua.edu.cn/ubuntu/ focal-updates universe
deb http://mirrors.tuna.tsinghua.edu.cn/ubuntu/ focal multiverse
deb http://mirrors.tuna.tsinghua.edu.cn/ubuntu/ focal-updates multiverse
deb http://mirrors.tuna.tsinghua.edu.cn/ubuntu/ focal-backports main restricted universe multiverse
deb http://mirrors.tuna.tsinghua.edu.cn/ubuntu/ focal-security main restricted
deb http://mirrors.tuna.tsinghua.edu.cn/ubuntu/ focal-security universe
deb http://mirrors.tuna.tsinghua.edu.cn/ubuntu/ focal-security multiverse

```

更新
```
sudo apt-get update
sudo apt-get upgrade
```


## ROS Installation

设置电脑以安装来自packages.ros.org的软件
```
sudo sh -c 'echo "deb http://packages.ros.org/ros/ubuntu $(lsb_release -sc) main" > /etc/apt/sources.list.d/ros-latest.list'
```

设置密钥
```
sudo apt-key adv --keyserver 'hkp://keyserver.ubuntu.com:80' --recv-key C1CF6E31E6BADE8868B172B4F42ED6FBAB17C654
```

开始安装

```
sudo apt update
sudo apt install ros-noetic-desktop-full
```

一般来说，每次打开新终端都要source一下
```
source /opt/ros/noetic/setup.bash
```


为了方便，设置自动source此脚本
```
echo "source /opt/ros/noetic/setup.bash" >> ~/.bashrc
source ~/.bashrc
```

**注意**如果安装了其他解释器，如zsh

上面*sh均更改为对应的解释器  

bash改为zsh    bashrc改为zshrc

**zsh**
```
echo "source /opt/ros/noetic/setup.zsh" >> ~/.bashrc
source ~/.zshrc
```

## Create ROS Workspace

```
mkdir -p ~/catkin_ws/src
cd ~/catkin_ws/
catkin_make
```

catkin_make之后,生成了build 和 devel
then, source这些文件中的任何一个都可以将当前工作空间设置在环境的最顶层。

```
source devel/setup.zsh
```

要保证工作区被安装脚本正确覆盖，需确定ROS_PACKAGE_PATH环境变量包含你当前的工作空间目录：
```
echo $ROS_PACKAGE_PATH

the result will be like:
/home/<username>/catkin_ws/src:/opt/ros/<distro>/share
```

## Filesystem Tools

Navigating with command-line tools such as ls and cd can be very tedious which is why ROS provides tools to help you. 

### rospack

usage and example:
```
rospack find [pkg_name]

example:
rospack find roscpp
would return
YOUR_INSTALL_PATH/share/roscpp
```

### roscd
