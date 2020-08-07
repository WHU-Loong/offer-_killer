
# docker学习手记
from:
《第一本docker书》&&docker官方文档
dev：tlinux
[TOC]
## 简介
容器不需要模拟层和管理层，仅仅使用操作系统的系统调用接口，降低单个容器所需要的开销，使得宿主机可以运行更多的容器
### docker特性
* 建模简单
* 开发人员与运维人员职责逻辑可以分离
* 使得应用程序具备可移植性
* 鼓励面向服务的架构和微服务架构 单个容器运行单个进程 应用程序或服务可以表示为一系列内部互联的容器

### docker组件
1. docker客户端和服务器
2. docker镜像
3. registry
4. docker容器

plus：写时复制：在fork之后exec之前两个进程用的是相同的物理空间（内存区），子进程的代码段、数据段、堆栈都是指向父进程的物理空间，也就是说，两者的虚拟空间不同，但其对应的物理空间是同一个。当父子进程中有更改相应段的行为发生时，再为子进程相应的段分配物理空间。

### 安装docker
tlinux中已经集成docker 无需下载和设置代理
将当前用户加docker群组 避免每次操作都要获取管理员权限
1. sudo groupadd docker
2. sudo gpasswd -a ${USER} docker
3. sudo systemctl restart docker
4. newgrp docker

### 配置docker守护进程
运行docker守护进程时 可以使用-H调整守护进程绑定监听接口的方式

sudo  /usr/bin/docker -d -H tcp://0.0.0.0:2375
或者
export DOCKER_HOST="tcp://0.0.0.0.:2375"

## docker入门
1. docker info
2. docker run --name bob -i -t ubuntu /bin/bash
-i :保证容器中STDIN开启
-t ：为创建的容器分配一个伪tty终端
/bin/bash:在创建容器结束后就会执行容器中的该命令
-name:为容器指定名字
3. sudo docker start bob
4. docker attach bob
重新附着到容器的会话上，注意attach之前容器必须为启动状态
5. sudo docker run --name NAME -d ubuntu /bin/sh -c "while true;do echo hello world ;sleep 1;done"
-d docker 放到后台运行
6. docker logs --tail 10 -t daemon_dave
--tail:查看最新日志
-t：显示时间戳
7. docker top daemon_deve
查看容器中的进程
8. docker exec -d daemon_dave(容器名字) touch /etc/new_config_file
-d ：后台启动 不需要交互
docker exec -t -i daemon_dave /bin/bsah
9. docker stop daemon_dave 
停止守护式容器
10. docker run --restart=always --name daemon_dave -d ubuntu /bin/sh -c "while true;do echo hello world;sleep1; done"
restart==onfailure:6 最多重启五次
11. docker inspect daemon_dave
docker inspect --format='{{.State.Running}}'daemon_dave
--format选定查看结果(go语言模板)
12. 删除容器
docker rm
删除全部 docker rm `docker ps -a -q` 此处为反引号

## docker镜像和仓库
### 镜像
本地镜像保存在Docker宿主机的var/lib/docker目录下
/var/lib/docker/containers 下看到所有容器
grubfs->rootfs->其他只读文件->可写空间进行cow
1. 拉取镜像
docker pull
2. 查找镜像
docker search 从docker Hub查找镜像
3. 构建镜像
a. docker commit(不推荐)
4. docker rmi 删除镜像

b. docker build和dockerfile文件
(新建目录 目录都处于上下文中)
FROM：基础镜像
MAINTAINER：作者信息
ENV:设置环境变量
RUN：
EXPOSE：开放端口
执行成功一条指令就进行一次提交
docker build -t ='仓库/名称：标签' .
1.RUN命令出错 
2.避免缓存 docker build --no-cache -t 
3.书写Dockerfile顶部格式尽量相同 可以命中缓存

### 仓库
用户仓库/顶层仓库

### dockerfile指令
CMD 容器启动时要做的命令
ENTRYPOINT 不容易被覆盖
WORKDIR 指定工作目录
ENV 设置环境变量 可以留存到该镜像创建的容器的环境变量中
USER 指定以何种用户运行
VOLUME 向基于镜像创建的容器添加卷(持久化目录)
ADD 将构建环境下的文件和目录复制到镜像中去
copy
ONBUILD:为镜像提供触发器，只可继承一次

## Docker构建并连接
对于容器与容器的连接，我们有多种办法
但是为了容器具有更良好的密闭性，尽量选择link的方式来连接父容器和子容器
EX：
对于一个webapp和一个redis，他们在不同容器中，将他们使用link连接起来
1. docker run -d --name redis XX/redis
2. docker run -p 4567 --name webapp --link redis:db -t -i -v $PWD/webapp:/opt/webapp XX/sinatra /bin/bash

或者可以使用Fig以yaml的格式来定义一组服务。
