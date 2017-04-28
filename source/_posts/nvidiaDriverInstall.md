---
title: ubuntu安装Nvidia驱动-cudnn-anaconda-tensorflow-tensorlayer

date: 2017/4/27 15:04:12

categories:
- 计算机视觉
tags:
- nvidia驱动
- cuda
- anaconda
- tensorflow
- tensorlayer
- 深度学习
- deeplearning
---
[TOC]

<!--more-->
# 安装Nvidia驱动

## 安装步骤
### 查询NVIDIA驱动
首先去官网(http://www.nvidia.com/Download/index.aspx?lang=en-us) 查看适合自己显卡的驱动

### 安装NVIDIA驱动
- 安装之前先卸载已经存在的驱动版本：
```
sudo apt-get remove --purge nvidia*
```

-  若电脑是集成显卡（NVIDIA独立显卡忽略此步骤），需要在安装之前禁止一项：
```
sudo service lightdm stop
```

- 执行以下指令安装驱动：
```
sudo add-apt-repository ppa:xorg-edgers/ppa
sudo apt-get update
sudo apt-get install nvidia-375 #注意在这里指定自己的驱动版本！
```

- 安装完成之后输入以下指令进行验证：
```
sudo nvidia-smi
```
若列出了GPU的信息列表则表示驱动安装成功。如果没看到，重启再试一下,linux装驱动需要重启才加载吧

## 可能出的问题

- add-apt-repository 命令不存在
```
sudo apt-get update
sudo apt-get install python-software-properties
sudo apt-get install software-properties-common
```
然后关掉terminator

- 输入nvidia-smi 说驱动没装上

重装系统换成英文版ubuntu

# anaconda

安装andaconda会自动安装很多python库和ipython notebook，并且可以提供虚拟机机制，支持多版本python共存。anaconda自动集成了最新版的MKL（math kernel libray）库，这是Intel推出的底层数值计算库。
 
## 安装anaconda

- 在anaconda官网continuum下载64位python3版本
- 在annaconda下载目录执行命令
```
bash Anaconda*.sh
```

- anaconda的license文档按q跳过，输入yes确认，按回车使用默认路径
- 输入yes把anaconda的binary路径加入~/.bashrc


## anacond的使用
用户安装的不同python环境都会被放在目录~/anaconda/envs下
- 查看已安装环境
```
conda info -e
```
- anaconda版本
```
which conda # 或者 conda -V
```
### conda的环境管理
- 创建一个python2.7的环境
```
conda create --name py27 python=2.7
```

- 使用activate激活某个环境
```
source activate py27  # linux使用此句
activate python34      # windows使用此句
```
> 激活后，会发现terminal输入的地改成py27，是因为把.bashrc里的path改成python27的路径

- 若想返回默认的python版本
```
source deactivate py27   # 返回原始版本python
```

- 删除一个已有的环境
```
conda remove --name py27 --all
```

### conda的包管理
- 安装包
```
conda install scipy
```
- 查看已经安装packages
```
conda list
```
- 查看某个指定环境的已安装包
```
conda list -n py27
```
- 查看package信息
```
conda search numpy
```
- 更新package
```
conda update -n py27 numpy
```
- 删除package
```
conda remove -n py27 numpy
```

# 安装cuda 
先下载[cuda](https://developer.nvidia.com/cuda-downloads)
然后输入命令进行安装
```
sudo sh cuda*linux.run --override
```
>启动安装程序，一直按q，输入accept接受条款 
输入n不安装nvidia图像驱动，之前已经安装过了 
输入y安装cuda 8.0工具 
回车确认cuda默认安装路径：/usr/local/cuda-8.0 
输入y用sudo权限运行安装，输入密码 
输入y或者n安装或者不安装指向/usr/local/cuda的符号链接 
输入y安装CUDA 8.0 Samples，以便后面测试

# 安装cudnn
- 将下载下来的cudnn-8.0-linux-x64-v5.1.tgz 解压之后，解压后的cuda文件夹先打开里面的include文件夹，在终端输入：
```
sudo cp cudnn.h /usr/local/cuda/include/ 
cd ~/cuda/lib64 
sudo cp lib* /usr/local/cuda/lib64/
```
- 继续更新文件链接
```
cd /usr/local/cuda/lib64/ 
sudo rm -rf libcudnn.so libcudnn.so.5 
sudo ln -s libcudnn.so.5.1.10 libcudnn.so.5 
sudo ln -s libcudnn.so.5 libcudnn.so
```
- 然后设置环境变量
```
sudo gedit /etc/profile
```
- 在末尾加入
```
PATH=/usr/local/cuda/bin:$PATH 
export PATH
```
- 保存之后创建链接文件
```
sudo gedit /etc/ld.so.conf.d/cuda.conf
```
- 加入
```
/usr/local/cuda/lib64
```
- 终端下接着输入
```
sudo ldconfig
```
使链接生效

# advance profile 工具接口
```
sudo apt-get install libcupti-dev
```

## 参考资料
《TensorFlow实战》

[Anaconda使用总结](http://www.jianshu.com/p/2f3be7781451)

[Anaconda使用教程（使用Anaconda配置多python开发环境）](http://www.afox.cc/archives/390)
