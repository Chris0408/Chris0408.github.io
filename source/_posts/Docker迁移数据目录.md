---
title:  Docker迁移镜像数据  
tags: 
- Docker
- Linux
categories: 
- Docker
- Linux
date: 2019-8-4
---
# Docker迁移镜像数据    
需求:  
公司机房的一台物理服务器需要新加了一块2TB的SSD作为数据盘，老大要求把Docker相关数据迁移到这块新加的SSD上。
- 环境介绍  
系统：Ubuntu 14.04.5 LTS  
Docker版本: 18.06.0-ce

## 处理过程
(1) 服务器拆机，硬盘安装过程就略过了。不过有一点需要注意的是，操作物理机硬件，最好熟悉服务器型号，买好相应配件。因为买这台机器的同事之前已经离职了，没人知道这台服务器内部构造，结果第一次拆开之后发现，没有多余的硬盘电源线，有没有硬盘支架，杯具了。后来有单独买了配件，再次进行了拆机安装。  

(2) 新硬盘分区格式化。这块硬盘是2TB的，依然可以使用fdisk工具进行分区。超过2TB的fdisk就不支持了，需要使用GPT分区，之后遇到了在专门讲一下。  

(3) 分好区，编辑/etc/fstab,设置好开机挂载，执行`mount -a`经新硬盘挂载到指定目录。我挂载在`/data`目录

(4) 迁移Docker数据。
```bash
1. service docker stop 
2. umount /var/lib/docker #取消/var/lib/docker的挂载，否则下一步迁移过程中会报device busy的错误
3. mv /var/lib/docker /data/docker-data #将docker镜像迁移到新目录
4. 修改文件/etc/docker/daemon.json,新增下面这条配置
"graph":"/data/docker-data"  #修改docker镜像存储位置
5. service docker start 
6. docker info #验证目录是否更改   
7. docker images #验证镜像是否存在
```