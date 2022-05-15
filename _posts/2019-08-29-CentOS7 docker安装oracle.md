---
layout: post
title:  "CentOS7 docker安装oracle"
date:   2019-08-29 12:20:06 +0800--
categories: [数据库, 运维, ]
tags: [oracle, docker, ]  

---

# CentOS7 docker安装oracle

https://blog.csdn.net/shao_yc/article/details/104423387

### 1 安装docker-ce 

```
安装依赖的软件包
yum install -y yum-utils device-mapper-persistent-data lvm2

配置Docker的阿里云yum源
yum-config-manager --add-repo http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo

安装docker-ce
yum install -y docker-ce

启动Docker服务
systemctl start docker

获取阿里云的oracle镜像
docker pull registry.cn-hangzhou.aliyuncs.com/helowin/oracle_11g

查看获取的镜像
docker images
```



### 2 使用docker安装oracle

```
1.默认启动容器方式

docker run -d -it -p 1521:1521 --name oracle11g --restart=always registry.cn-hangzhou.aliyuncs.com/helowin/oracle_11g

持久化启动方式如下：

docker run -d -it -p 1521:1521 --name oracle --restart=always --mount source=oracle_vol,target=/home/oracle/app/oracle/oradata registry.cn-hangzhou.aliyuncs.com/helowin/oracle_11g

一般用默认启动方式就可以了，若是需要将数据保存到本地的采用持久化方式。
--mount表示要将Host上的路径挂载到容器中。
source=oracle_vol为Host的持久化卷，若未提前创建会自动创建，可通过docker volume instpect 【容器名】查看volume的具体位置，target为容器中的路径

2.查看容器，容器状态up表示在运行状态
docker ps

3.进入容器
docker exec -it 【容器id或名称】 bash

4.切换到root账户（默认进入之后是oracle账户）
su root
输入密码：helowin（密码都是一样的）

5.编辑环境变量
vi /etc/profile 添加以下内容：
    export ORACLE_HOME=/home/oracle/app/oracle/product/11.2.0/dbhome_2
    export ORACLE_SID=helowin
    export PATH=$ORACLE_HOME/bin:$PATH
source /etc/profile 使配置生效

6.创建软链接
ln -s $ORACLE_HOME/bin/sqlplus /usr/bin

7.切换到oracle用户，登录sqlplus
su - oracle
sqlplus /nolog
conn /as sysdba

修改sys、system用户密码：
alter user system identified by YOUR_PASSWORD;
alter user sys identified by YOUR_PASSWORD;
alter profile default limit PASSWORD_LIFE_TIME UNLIMITED;

8.创建用户（可选，根据需要）
用一个具有dba权限的用户登录（sysdba)，然后输入以下语句
create user 用户名 identified by 密码;
grant connect,resource,dba to test;

9.使用Navicat链接Oracle
注意下面的服务名填写helowin
```

![image-20200513140841269](/Users/silince/Develop/博客/blog_to_git/assets/imgs/image-20200513140841269.png)