# docker命令
## 容器管理
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
--privileged，当使用--privileged=true选项运行容器时，Docker会赋予容器几乎与主机相同的权限
```
实例

使用镜像nginx:latest以交互模式启动一个容器,在容器内执行/bin/bash命令。
```bash
runoob@runoob:~$ docker run -it nginx:latest /bin/bash
root@b8573233d675:/# 
```
### docker start :启动一个或多个已经被停止的容器
```
docker start  [OPTIONS] CONTAINER [CONTAINER...]
```
### docker stop :停止一个运行中的容器
```
docker stop [OPTIONS] CONTAINER [CONTAINER...]
```
### docker restart :重启容器
```
docker restart  [OPTIONS] CONTAINER [CONTAINER...]
```
### docker rm ：删除一个或多个容器。
```
docker rm [OPTIONS] CONTAINER [CONTAINER...]
```

OPTIONS说明：
- -f :通过 SIGKILL 信号强制删除一个运行中的容器。

- -l :移除容器间的网络连接，而非容器本身。

- -v :删除与容器关联的卷。
### docker pause :暂停容器中所有的进程。
```
docker pause CONTAINER [CONTAINER...]
```
### docker unpause :恢复容器中所有的进程。
```
docker unpause CONTAINER [CONTAINER...]
```
### docker create ：创建一个新的容器但不启动它
用法同 docker run
### docker ps : 列出容器
```
docker ps [OPTIONS]
```
```
OPTIONS说明：

-a :显示所有的容器，包括未运行的。

-f :根据条件过滤显示的内容。

--format :指定返回值的模板文件。

-l :显示最近创建的容器。

-n :列出最近创建的n个容器。

--no-trunc :不截断输出。

-q :静默模式，只显示容器编号。

-s :显示总的文件大小。
```
### docker inspect : 获取容器/镜像的元数据。
### docker attach :连接到正在运行中的容器。
退出容器后容器会stop
### docker exec ：在运行的容器中执行命令
退出容器后容器不会stop
```
docker exec [OPTIONS] CONTAINER COMMAND [ARG...]
```
```
OPTIONS说明：

-d :分离模式: 在后台运行

-i :即使没有附加也保持STDIN 打开

-t :分配一个伪终端
```
实例：
```
runoob@runoob:~$ docker exec -it  mynginx /bin/bash
root@b1a0703e41e7:/#
```
### docker logs : 获取容器的日志
```
docker logs [OPTIONS] CONTAINER
```
```
OPTIONS说明：

-f : 跟踪日志输出

--since :显示某个开始时间的所有日志

-t : 显示时间戳

--tail :仅列出最新N条容器日志
```
### docker cp :用于容器与主机之间的数据拷贝。
```
docker cp [OPTIONS] CONTAINER:SRC_PATH DEST_PATH
```
```
docker cp [OPTIONS] SRC_PATH CONTAINER:DEST_PATH
```
### docker commit :从容器创建一个新的镜像。
```
docker commit [OPTIONS] CONTAINER [REPOSITORY[:TAG]]
```
```
OPTIONS说明：

-a :提交的镜像作者；

-c :使用Dockerfile指令来创建镜像；

-m :提交时的说明文字；

-p :在commit时，将容器暂停。
```
实例：将容器a404c6c174a2 保存为新的镜像,并添加提交人信息和说明信息。
```
runoob@runoob:~$ docker commit -a "runoob.com" -m "my apache" a404c6c174a2  mymysql:v1 
sha256:37af1236adef1544e8886be23010b66577647a40bc02c0885a6600b33ee28057
runoob@runoob:~$ docker images mymysql:v1
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
mymysql             v1                  37af1236adef        15 seconds ago      329 MB
```
## 镜像操作

### docker pull :获取镜像
```
docker pull [OPTIONS] NAME[:TAG|@DIGEST]
```
### docker push : 将本地的镜像上传到镜像仓库,要先登陆到镜像仓库
```
docker push [OPTIONS] NAME[:TAG]
```
实例：上传本地镜像myapache:v1到镜像仓库中。
```
docker push myapache:v1
```
### docker search : 从Docker Hub查找镜像
```
docker search [OPTIONS] TERM
```
```
OPTIONS说明：

--automated :只列出 automated build类型的镜像；

--no-trunc :显示完整的镜像描述；

-f <过滤条件>:列出收藏数不小于指定值的镜像。
```
实例：从 Docker Hub 查找所有镜像名包含 java，并且收藏数大于 10 的镜像
```
runoob@runoob:~$ docker search -f stars=10 java
NAME                  DESCRIPTION                           STARS   OFFICIAL   AUTOMATED
java                  Java is a concurrent, class-based...   1037    [OK]       
anapsix/alpine-java   Oracle Java 8 (and 7) with GLIBC ...   115                [OK]
develar/java                                                 46                 [OK]
isuper/java-oracle    This repository contains all java...   38                 [OK]
lwieske/java-8        Oracle Java 8 Container - Full + ...   27                 [OK]
nimmis/java-centos    This is docker images of CentOS 7...   13                 [OK]

```
### docker images : 列出本地镜像。
```
docker images [OPTIONS] [REPOSITORY[:TAG]]
```
```
OPTIONS说明：

-a :列出本地所有的镜像（含中间映像层，默认情况下，过滤掉中间映像层）；

--digests :显示镜像的摘要信息；

-f :显示满足条件的镜像；

--format :指定返回值的模板文件；

--no-trunc :显示完整的镜像信息；

-q :只显示镜像ID。
```
### docker rmi : 删除本地一个或多个镜像。
```
docker rmi [OPTIONS] IMAGE [IMAGE...]
```
```
OPTIONS说明：

-f :强制删除；

--no-prune :不移除该镜像的过程镜像，默认移除；
```
### docker tag : 标记本地镜像，将其归入某一仓库。
```
docker tag [OPTIONS] IMAGE[:TAG] [REGISTRYHOST/][USERNAME/]NAME[:TAG]
```
实例：将镜像ubuntu:15.10标记为 runoob/ubuntu:v3 镜像。
```
root@runoob:~# docker tag ubuntu:15.10 runoob/ubuntu:v3
root@runoob:~# docker images   runoob/ubuntu:v3
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
runoob/ubuntu       v3                  4e3b13c8a266        3 months ago        136.3 MB
```
### docker build 命令用于使用 Dockerfile 创建镜像。
```
docker build [OPTIONS] PATH | URL | -
```
实例: 使用当前目录的 Dockerfile 创建镜像，标签为 runoob/ubuntu:v1。
```
docker build -t runoob/ubuntu:v1 . 
```
使用URL github.com/creack/docker-firefox 的 Dockerfile 创建镜像。
```
docker build github.com/creack/docker-firefox
```
也可以通过 -f Dockerfile 文件的位置：
```
$ docker build -f /path/to/a/Dockerfile .
```
在 Docker 守护进程执行 Dockerfile 中的指令前，首先会对 Dockerfile 进行语法检查，有语法错误时会返回：
```
$ docker build -t test/myapp .
Sending build context to Docker daemon 2.048 kB
Error response from daemon: Unknown instruction: RUNCMD
```
### FROM 和 RUN 指令的作用
FROM：定制的镜像都是基于 FROM 的镜像，这里的 nginx 就是定制需要的基础镜像。后续的操作都是基于 nginx。

RUN：用于执行后面跟着的命令行命令。有以下俩种格式：

shell 格式：
```
RUN <命令行命令> # <命令行命令> 等同于，在终端操作的 shell 命令。
```
exec 格式：
```
RUN ["可执行文件", "参数1", "参数2"]
# 例如：
# RUN ["./test.php", "dev", "offline"] 等价于 RUN ./test.php dev offline
```
注意：Dockerfile 的指令每执行一次都会在 docker 上新建一层。所以过多无意义的层，会造成镜像膨胀过大。例如：
```
FROM centos
RUN yum -y install wget
RUN wget -O redis.tar.gz "http://download.redis.io/releases/redis-5.0.3.tar.gz"
RUN tar -xvf redis.tar.gz
```
以上执行会创建 3 层镜像。可简化为以下格式：
```
FROM centos
RUN yum -y install wget \
    && wget -O redis.tar.gz "http://download.redis.io/releases/redis-5.0.3.tar.gz" \
    && tar -xvf redis.tar.gz
```
如上，以 && 符号连接命令，这样执行后，只会创建 1 层镜像。
### 开始构建镜像
在 Dockerfile 文件的存放目录下，执行构建动作。

以下示例，通过目录下的 Dockerfile 构建一个 nginx:v3（镜像名称:镜像标签）。

注：最后的 . 代表本次执行的上下文路径，下一节会介绍。
```
$ docker build -t nginx:v3 .
```
### 上下文路径
上一节中，有提到指令最后一个 . 是上下文路径，那么什么是上下文路径呢？
```
$ docker build -t nginx:v3 .
```
上下文路径，是指 docker 在构建镜像，有时候想要使用到本机的文件（比如复制），docker build 命令得知这个路径后，会将路径下的所有内容打包。

解析：由于 docker 的运行模式是 C/S。我们本机是 C，docker 引擎是 S。实际的构建过程是在 docker 引擎下完成的，所以这个时候无法用到我们本机的文件。这就需要把我们本机的指定目录下的文件一起打包提供给 docker 引擎使用。

如果未说明最后一个参数，那么默认上下文路径就是 Dockerfile 所在的位置。

注意：上下文路径下不要放无用的文件，因为会一起打包发送给 docker 引擎，如果文件过多会造成过程缓慢。

### Dockerfile实例
```
FROM centos:7.6.1810

COPY openGauss-5.0.0-CentOS-64bit.tar.bz2 .
COPY gosu-amd64 /usr/local/bin/gosu

ENV LANG en_US.utf8

#RUN yum install -y epel-release

RUN set -eux; \
    yum install -y bzip2 bzip2-devel curl libaio&& \
    groupadd -g 70 omm;  \
    useradd -u 70 -g omm -d /home/omm omm;  \
    mkdir -p /var/lib/opengauss && \
    mkdir -p /usr/local/opengauss && \
    mkdir -p /var/run/opengauss  && \
    mkdir /docker-entrypoint-initdb.d && \
    tar -jxf openGauss-5.0.0-CentOS-64bit.tar.bz2 -C /usr/local/opengauss && \
    chown -R omm:omm /var/run/opengauss && chown -R omm:omm /usr/local/opengauss && chown -R omm:omm /var/lib/opengauss &&  chown -R omm:omm /docker-entrypoint-initdb.d && \
    chmod 2777 /var/run/opengauss && \
    rm -rf openGauss-5.0.0-CentOS-64bit.tar.bz2 && yum clean all

RUN set -eux; \
    echo "export GAUSSHOME=/usr/local/opengauss"  >> /home/omm/.bashrc && \
    echo "export PATH=\$GAUSSHOME/bin:\$PATH " >> /home/omm/.bashrc && \
    echo "export LD_LIBRARY_PATH=\$GAUSSHOME/lib:\$LD_LIBRARY_PATH" >> /home/omm/.bashrc

ENV GOSU_VERSION 1.12
RUN set -eux; \
     chmod +x /usr/local/bin/gosu


ENV PGDATA /var/lib/opengauss/data

COPY entrypoint.sh /usr/local/bin/
RUN chmod 755 /usr/local/bin/entrypoint.sh;ln -s /usr/local/bin/entrypoint.sh / # backwards compat

ENTRYPOINT ["entrypoint.sh"]

EXPOSE 5432
CMD ["gaussdb"]
```