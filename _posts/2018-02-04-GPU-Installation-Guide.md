---
layout: post
current: post
cover:  assets/images/stock/images-stock-photo-0127845.jpg
navigation: True
title: Ubuntu 16.04 + GTX 1050ti + Cuda 8.0 + cuDNN 6.0 Installation Guide
date: 2018-02-04 10:48:00
tags: [guide]
class: post-template
subclass: 'post tag-guide'
author: kiwi
comments: true
---

## Pre Read  

据消息称 NVIDIA 的 Cuda 9.1 系列并不支持 GTX 1060 以下的显卡，于是便有了这一篇经过两三个晚上折腾后的安装指南。

## Installation Guide  

* **安装 NVIDIA 显卡驱动**

    * 若之前有安装过 NVIDIA 的其它驱动，需要先将其完全卸载 ：  
    ```sudo apt-get purge nvidia*```  

    * 添加 NVIDIA 源 ：  
    ```sudo add-apt-repository ppa:graphics-drivers/ppa```  

    * 安装指定 NVIDIA Driver （这里选择的是 nvidia-387）:  
    ```sudo apt-get update```  
    ```sudo apt-get install nvidia-387 nvidia-settings```  

    * 重启系统，在终端运行 `nvidia-smi`，若能查看显卡相关信息，说明安装成功

* **安装 Cuda 8.0**

    * 前往 [NVIDIA官网](https://developer.nvidia.com/cuda-80-ga2-download-archive) 下载 `Linux-x86_64-Ubuntu-16.04-runfile[locla]` 中的安装文件，共两个

    * 按照 [官方安装指南](https://developer.nvidia.com/compute/cuda/8.0/Prod2/docs/sidebar/CUDA_Installation_Guide_Linux-pdf) 进行安装，大致需要注意这几点：

        * 完成 PRE-INSTALLATION 中的检查，确保环境 OK 

        * 重启后运行 `lspci | grep nouveau` 确保无集成显卡运行

        * 按下 `alt + ctrl + F1` 进入 `text mode`

        * 登录后运行 `sudo init 3` 并运行 `sudo service lightdm stop` 再`cd` 到 cuda 的下载目录，

        * 运行 `sudo sh cuda_8.0.61_375.26_linux.run` 安装 base installer

            * 注意： 第一条询问是否安装显卡驱动，选择 `n`，其它的都是 `y` 或默认即可
        
        * 运行 `sudo sh cuda_8.0.61.2_linux.run` 安装 patch

        * 完成后运行 `sudo service lightdm start` 再重启系统

    * 运行 Samples 检查 Cuda 是否成功安装:  
    ```cd ~/NVIDIA..../1_Utilities/deviceQuery #由自己的安装目录决定```  
    ```sudo make```  
    ```sudo ./deviceQuery```  
    若安装成功可看到CUDA版本信息

    * 设置 CUDA 的环境变量:  
    ```export PATH=/usr/local/cuda/bin${PATH:+:${PATH}}```  
    ```export LD_LIBRARY_PATH=/usr/local/cuda/lib64${LD_LIBRARY_PATH:+:${LD_LIBRARY_PATH}}```  
    ```export CUDA_HOME=/usr/local/cuda```  
    ```source ~/.bashrc```  

* **安装 cuDNN6.0**

    * 附上参考链接中的[百度网盘下载地址](https://pan.baidu.com/s/1cps654) 密码：52g7，下载完成后，切换到下载目录，输入以下命令，解压并将相关文件拷贝到CUDA的安装目录下：  
    ```tar -zxvf cudnn-8.0-linux-x64-v6.0.tgz```  
    ```sudo cp cuda/include/cudnn.h /usr/local/cuda/include/```  
    ```sudo cp cuda/lib64/libcudnn* /usr/local/cuda/lib64/ -d```  
    ```sudo chmod a+r /usr/local/cuda/include/cudnn.h```  
    ```sudo chmod a+r /usr/local/cuda/lib64/libcudnn*```  

* 之后就可以安装 anaconda 创建环境安装TF了

## Summary

最坑的就是官方不告诉我们 cuda 9.1 不支持 1050