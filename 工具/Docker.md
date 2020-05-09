#  Docker

# 1. 安装

**Mac 系统**:

直接去管网下载, 然后安装.



**Linux 系统**:

安装: `yum install docker`

启动: `systemctl start docker`



**Windows 系统**:

不清楚....



## 2. 简单使用

### 2.1 命令

```properties
docker --version : 查看 Docker 版本
docker images : 查看所有镜像
docker ps : 查看正在运行的 Docker 容器
docker ps -a : 查看所有运行过的容器. 可以理解为历史记录
docker run -itd <image id> : 运行指定的镜像. 运行后, 会生成一个容器
docker start <container id> : 启动指定的容器.
docker stop <container id> : 停止指定的容器
docker rm <container id> : 删除指定的容器
docker rmi <image id> : 删除指定的镜像
```



## 命令

```shell
docker run -itd <image id> : 启动指定 ID 的镜像.
docker run -itd --name <container name> <image id> : 启动指定 ID 的镜像, 并且给自动生成的容器起名为 <container name>
docker run -itd -p 80:8080 -v /usr/local/localjar.jar:/usr/hello.jar --name hello d23bdf5b1b1b java -jar /usr/hello.jar
启动镜像 ID 为 d23bdf5b1b1b 的镜像, 它会自动生成一个容器,并给容器起个名字叫做 hello, 并将容器的80 端口和本地的 8080 端口映射, 将本地的 /usr/local/localjar.jar 挂载到容器的/usr/下,然后起名为 hello.jar, 然后使用 java -jar 命令运行 jar 包.
```



## 自定义镜像

自定义镜像步骤:

1. 要自定义镜像, 首相要把镜像启动起来. `docker run -itd <image id>`
2. 然后将 jar 包拷贝到容器中. `docker cp /local/path/xxx.jar <container id>:/docker/container/path/xxx.jar`
3. 然后将容器打包成镜像, 其实这时候已经可以运行了. `docker commit <container id> <image name>`
4. 但是为了可移植, 需要将打包的镜像导出成 tar 包. `docker save -o xxxx.tar <image id>`
5. 然后在要运行的 Linux 中导入这个 tar 包. 导入就会有个这个镜像.`docker load -i xxx.tar`
6. 可能需要给镜像命名.`docker tag <image id> name`
7. 最后就可以启动镜像了. `docker run -itd -p 80:8080 --name 给镜像生成的容器命名 <image id> java -jar /docker/container/path/xxx.jar`



**真实操作:**

```properties
[root@cy ~]# docker images   查看所有镜像
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
docker.io/java      latest              d23bdf5b1b1b        3 years ago         643 MB
[root@cy ~]# docker run -itd d23bdf5b1b1b		启动空白镜像
7641bbfe0798134769eed8aa8c9af17358851b18591c5c0c9f671b1844a01978
[root@cy ~]# docker ps		查看镜像启动后生成的容器
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
7641bbfe0798        d23bdf5b1b1b        "/bin/bash"         3 seconds ago       Up 2 seconds                            jolly_poincare
[root@cy ~]# docker cp /usr/local/cy/app/design-0.0.1-SNAPSHOT.jar 7641bbfe0798:/usr/hello.jar   将本地 jar 包拷贝到容器中的 /usr下, 并命名为 hello.jar
[root@cy ~]# docker commit 7641bbfe0798 java/hello   将包含 jar 包的容器打包成镜像, 并命名为 java/hello
sha256:3af214ca4cdd8244604b2c5c56e0902a9cf4b5265324e59ccc71250ea2912b4f
[root@cy ~]# docker images	查看所有镜像,此时会多一个刚刚打包的镜像
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
java/hello          latest              3af214ca4cdd        18 seconds ago      663 MB
docker.io/java      latest              d23bdf5b1b1b        3 years ago         643 MB
[root@cy ~]# docker save -o hello.tar 3af214ca4cdd		将此镜像打包成 tar 包.
[root@cy ~]# ll
总用量 662764
-rw------- 1 root root 678665216 5月   8 22:56 hello.tar
-rw-r--r-- 1 root root         0 8月   4 2019 log-6379.log
[root@cy ~]# docker rmi 3af214ca4cdd		因为是本地测试,所以将最初的那个镜像删掉
Untagged: java/hello:latest
Deleted: sha256:3af214ca4cdd8244604b2c5c56e0902a9cf4b5265324e59ccc71250ea2912b4f
Deleted: sha256:c9f5bbf7ad3b5ebd37c6e28bae5782ef6c79587126eec235a581ac2cd7460883
[root@cy ~]# docker images		已经删掉了
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
docker.io/java      latest              d23bdf5b1b1b        3 years ago         643 MB
[root@cy ~]# docker load -i hello.tar		加载 tar 包
41de7ff8fb7f: Loading layer [==================================================>] 19.41 MB/19.41 MB
Loaded image ID: sha256:3af214ca4cdd8244604b2c5c56e0902a9cf4b5265324e59ccc71250ea2912b4f
[root@cy ~]# docker images			加载后, 会生成一个镜像, 但是没有名字,需要指定名字
REPOSITORY          TAG                 IMAGE ID            CREATED              SIZE
<none>              <none>              3af214ca4cdd        About a minute ago   663 MB
docker.io/java      latest              d23bdf5b1b1b        3 years ago          643 MB
[root@cy ~]# docker tag 3af214ca4cdd java/hello		为生成的镜像命名
[root@cy ~]# docker images		查看镜像
REPOSITORY          TAG                 IMAGE ID            CREATED              SIZE
java/hello          latest              3af214ca4cdd        About a minute ago   663 MB
docker.io/java      latest              d23bdf5b1b1b        3 years ago          643 MB
[root@cy ~]# docker run -itd -p 80:8080 --name hello 3af214ca4cdd java -jar /usr/hello.jar  启动镜像
ed5018fbdabdb305d3f831408c6f45307968e4a4ac6af465693a7deb5f0d4c76
[root@cy ~]# docker ps   查看容器
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                  NAMES
ed5018fbdabd        3af214ca4cdd        "java -jar /usr/he..."   3 seconds ago       Up 2 seconds        0.0.0.0:80->8080/tcp   hello
7641bbfe0798        d23bdf5b1b1b        "/bin/bash"              3 minutes ago       Up 3 minutes                               jolly_poincare
[root@cy ~]#   本地访问, 就可以了
```



上边是手动操作的,命令比较多, 可以使用脚本来创建 Dockerfile .

### **Dockerfile**

步骤:

1. 创建一个文件夹, 名字随便起, 例如 test, 然后进入 test
2. 在 test 下创建文件 Dockerfile, 没有后缀
3. 将 jar 包拷贝到 test 下.
4. 编辑 Dockerfile 文件, 写脚本
5. 使用 `docker build -t imageName .` 构建. 因为当前操作是在 test 目录下进行的, 所以最后是点



真实操作:

```shell
[root@cy ~]# mkdir test		创建目录
[root@cy ~]# cd test/		进入目录
[root@cy test]# cp /usr/local/cy/app/design-0.0.1-SNAPSHOT.jar ./hello.jar   将 jar 包拷贝到当前目录下
[root@cy test]# ll
总用量 18956
-rw-r--r-- 1 root root 19410774 5月   8 23:15 hello.jar
[root@cy test]# touch Dockerfile		创建 Dockerfile, 必须是这个名字
[root@cy test]# vim Dockerfile		编写脚本
[root@cy test]# ll
总用量 18960
-rw-r--r-- 1 root root       54 5月   8 23:16 Dockerfile
-rw-r--r-- 1 root root 19410774 5月   8 23:15 hello.jar
[root@cy test]# docker images		运行前, 确定当前是只有一个镜像.
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
docker.io/java      latest              d23bdf5b1b1b        3 years ago         643 MB
[root@cy test]# docker build -t java/test .		在 test 下执行脚本
Sending build context to Docker daemon 19.41 MB
Step 1/3 : FROM java
 ---> d23bdf5b1b1b
Step 2/3 : MAINTAINER cy
 ---> Running in 82ebbf96dd95
 ---> c21ff468e56c
Removing intermediate container 82ebbf96dd95
Step 3/3 : COPY hello.jar /usr/hello.jar
 ---> 7f4f3175fd0b
Removing intermediate container a67a3b3e1604
Successfully built 7f4f3175fd0b
[root@cy test]# docker images		查看镜像, 发现成功生成镜像
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
java/test           latest              7f4f3175fd0b        4 seconds ago       663 MB
docker.io/java      latest              d23bdf5b1b1b        3 years ago         643 MB
[root@cy test]#
```



Dockerfile 脚本

```shell
FROM java
MAINTAINER cy
COPY hello.jar /usr/hello.jar
```

 

OK . 洗洗睡觉吧.



## 附录

### 1. 出现过的坑

#### 1.1 docker pull 时报错

`error pulling image configuration: Get https://production.cloudflare.docker.com/registry-v2/docker/registry/v2/blobs/sha256/d2/d23bdf5b1b1b1afce5f1d0fd33e7ed8afbc084b594b9ccf742a5b27080d8a4a8/data?verify=1588903806-jZWaVVz21dWbmSq5zjuJM7i7rVw%3D: dial tcp: lookup production.cloudflare.docker.com: no such host`

此错误一般是因为系统时间不一致造成的, 使用 `ntpdate time.windows.com`命令来同步时间后, 应该就解决了.



