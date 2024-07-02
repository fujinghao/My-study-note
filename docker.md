## docker命令
### docker run ：创建一个新的容器并运行一个命令
```bash
OPTIONS说明：
-d: 后台运行容器，并返回容器ID；

-i: 以交互模式运行容器，通常与 -t 同时使用；

-P: 随机端口映射，容器内部端口随机映射到主机的端口

-p: 指定端口映射，格式为：主机(宿主)端口:容器端口

-t: 为容器重新分配一个伪输入终端，通常与 -i 同时使用；

--name="nginx-lb": 为容器指定一个名称；

--dns 8.8.8.8: 指定容器使用的DNS服务器，默认和宿主一致；

-e username="ritchie": 设置环境变量；

-m :设置容器使用内存最大值；

--volume , -v: 绑定一个卷
```
实例

使用镜像nginx:latest以交互模式启动一个容器,在容器内执行/bin/bash命令。
```bash
runoob@runoob:~$ docker run -it nginx:latest /bin/bash
root@b8573233d675:/# 
```
### docker start :启动一个或多个已经被停止的容器
docker start [OPTIONS] CONTAINER [CONTAINER...]
### docker stop :停止一个运行中的容器
docker stop [OPTIONS] CONTAINER [CONTAINER...]
### docker restart :重启容器
docker restart [OPTIONS] CONTAINER [CONTAINER...]