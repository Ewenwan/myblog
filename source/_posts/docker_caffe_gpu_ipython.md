---
title: 不会装cuda配caffe的小白的救赎—玄学DL
tdate: 2017/5/26 15:04:12

categories:
- 计算机视觉
tags:
- nvidia驱动
- 深度学习
- caffe
- docker
- deeplearning
- python
---

DL如今已经快成为全民玄学了，感觉离民科入侵不远了。唯一的门槛可能是环境不好配，特别是caffe这种依赖数10种其它软件打框架。不过有了docker之后，小白也能轻松撸DL了。

<!--more-->
大致流程如下，通过docker pull一个GPU版本的caffe 的image,然后安装nvidia-docker 和 


