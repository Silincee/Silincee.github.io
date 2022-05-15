---
layout: post
title:  "CentOS7的docker下载下来启动失败问题（Job for docker.service failed because the control process exited with）的解决方案"
date:   2019-12-19 12:30:06 +0800--
categories: [运维, ]
tags: [docker, ]  
---
# CentOS7的docker下载下来启动失败问题（Job for docker.service failed because the control process exited with）的解决方案

会出现的错误：

```
Job for docker.service failed because the control process exited with error code. See "systemctl status docker.service" and "journalctl -xe" for details.
```

如果忽略错误进行操作，就会出现：

`Cannot connect to the Docker daemon at unix:///var/run/docker.sock. Is the docker daemon running?`
因为docker就没有启动成功

解决方案：

```
vim /etc/sysconfig/docker-storage
将配置文件修改为：
DOCKER_STORAGE_OPTIONS="--selinux-enabled --log-driver=journald --signature-verification=false"
vim /etc/docker/daemon.json
写入指定参数：
{ "storage-driver": "devicemapper" }然后重启docker服务
systemctl restart docker
```

没有出现错误信息，说明问题已经解决

