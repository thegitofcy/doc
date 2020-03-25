# ==**_Zookeeper_**==
## 文档内容索引
### 引入一些问题

```properties
1. 什么是 zookeeper
    是一个中间件, 提供分布式应用的协调服务.
    作用于分布式系统,发挥其优势, 可以为大数据服务
    
```

==**zookeeper 的作用**==
1. ==**首脑模式**==: master 节点选举, master 节点挂了以后, 从节点就会接手工作,并且保证这个节点是唯一的.也就是所谓的首脑模式, 从而保证集群是高可用的.
2. ==**统一配置文件管理**==: 即只需要部署一台服务器, 则可以把相同的配置文件同步更新到其他所有服务器, 此操作在云计算中用的特别多. 比如修改了 Redis 统一配置.
3. ==**发布与订阅**==: 类似于消息队列 MQ, dubbo 发布者把数据存在 znode 上, 订阅者会读取这个数据.
4. ==**提供分布式锁**==. 分布式环境中不同进程之间争夺资源,类似于多线程中的锁. (简单的说就是一个服务要操作一个共享资源之前, 要先判断作为锁的临时节点存不存在, 如果存在则说明有其他服务正在访问这个共享资源, 则需要等待, 如果这个临时节点不存在, 那么创建这个临时节点, 告诉其他服务我正在用这个共享数据.)
5. ==**集群管理**==: 保证数据的强一致.(在主节点添加一个数据, 那么基于zookeeper 的特性,会将这个数据同步到其他的从节点. 并且是一模一样的数据, 此时客户端可以从任何一个节点获取到这个数据)

==**常用的 zookeeper java 客户端**==
- `zookeeper 原生 api`
    - 缺点 1: 超时重连, 不支持自动重连, 需要手动操作
    - 缺点 2: watch事件触发过后就失效, 一次性的.
    - 缺点 3: 不支持递归创建节点.
- `zkclient`
- `apache curator`(==**推荐使用**==)
    - 注册 watcher 事件, 可以长期有效
    - 提供更多解决方案且实现简单. 比如 ==**分布式锁**==
    - 提供常用的 zookeeper 工具类.


### 名词

```properties
acl: 节点的权限acl, 可以通过权限来限制用户的访问.
watcher: 对节点变化的监听事件
```


## 第一章 分布式概念和 zookeeper 简介
==**Zookeeper**==是分布式应用程序协调服务.
## 第二章 单机安装以及启动等
==**安装**==

```properties
1. 下载稳定版
2. 解压 tar -xvf xxxx.tar
3. 配置环境变量
    export ZOOKEEPER_HOME=/usr/local/env/zk/zk-master
    export PATH=$PATH:$ZOOKEEPER_HOME/bin
4. 使环境变量生效: source /etc/profile
```

==**配置说明**==
tickTime, initLimit, synclimit, clientPort,在单机版都不需要进行修改, 只需要设置 dataDir 和 dataLogDir 就行了, 然后就可以通过命令进行启动.

```properties
1. 在 zk 的 conf 文件夹下, 有 zoo_sample.cfg 文件, 复制并改名为 zoo.cfg
2. tickTime : 用于计算时间的时间单元, 默认为 2000 毫秒. 比如 session 超时, 设置为 3* tickTIme, 那么就是 6 秒后超时.zk 中的所有时间都是以 tickTime 的倍数来计算
3. initLimit: 用于集群. 允许从节点连接并同步到 master 节点的初始化连接时间. 以 tickTime 的倍数标识.默认是 10, 也即是 20 秒
4. syncLimit: 用于计算. master 主节点与从节点之间发送消息, 进行请求和应答的时间长度.(心跳机制). 默认为 5, 也即是 10 秒
5. dataDir: zk 的要存储的一些数据. 必须要配置.
6. dataLogDir: 日志目录,如果不配置的话就会和 dataDir 公用.
7. clientPort: 连接服务器的端口, 默认 2181
```

==**启动/停止/重启服务器, 并连接服务器**==

```properties
启动: zkServer.sh start
状态: zkServer.sh status
重启: zkServer.sh restart
停止: zkServer.sh stop

连接服务: zkCli.sh
```

## 第三章 Zookeeper 基本数据模型
### 3.1 zookeeper 数据模型介绍

1. ==**Zookeeper 的基本数据模型是一个树形结构**==. 也可以理解为 Linux 的目录结构.
2. 每一个节点都称为`znode`, 这个节点可以有==**子节点**==, 也可以有==**数据**==
3. 每个` znode`分为==**临时节点**==和==**永久节点**==. 临时节点在客户端断开后消失.(Session 失效后, 临时节点也会消失)
4. 每个` znode`都各自有自己的版本号, 可以通过命令行来显示节点信息.
5. 每当` znode`节点数据发生变化, 那么该节点的版本号会==**累加1**==.
6. 删除/修改过时节点,版本号不匹配则会报错.(两个人都对节点进行修改, 那么版本号会变成 2, 再变成 3, 这时删除节点,传入的版本号不是 3 的话, 则会报错)
7. 每个` znode`节点存储的数据不宜过大, ==**几 K 就差不多了**==
8. 节点可以设置==**权限 acl**==,可以通过权限来限制用户的访问.

### 3.2 Zookeeper 数据模型基本操作
==**客户端连接**==

```properties
启动服务器: zkServer.sh start
查看当前详情: zkServer.sh status
客户端连接: zkCli.sh
```

==**查看znode 结构**==

```properties
查看: ls path  或者是 ls2 path
```

==**关闭客户端连接**==

```properties
关闭服务器: zkServer.sh stop
关闭客户端连接: ctrl + c
```

其他基本操作查看附录.(创建, 删除等)

## 第四章 Zookeeper 特性
### 4.1 session 和心跳机制
==**zookeeper 中 session:**==: 
1. zookeeper客户端与服务端之间的连接存在会话.
2. 每个会话都可以设置超时时间.
3. 心跳结束, session 就会过期.(一旦没有心跳, session 就会过期)
4. session 过期, 那么通过这个 session 创建的临时节点就会被删除
5. ==**心跳机制**== : 客户端端服务向服务端发送心跳, 只要心跳存在, 就不会断开连接, session 就一直存在

### 4.2 watcher 机制
==**针对每个节点的操作, 都会有一个监督者-->watcher**==,.
1. 换句比较好理解的话: ==**当监控的某个节点发生了变化,就会触发 watcher 事件,并且触发后, watcher 会立即销毁**==.
2. zookeeper 中的 watcher 是==**一次性的**==, 触发后就会==**立即销毁**==. 使用一个框架进行操作的时候, 可以设置为永久性的.
3. 父节点和子节点的增删改操作都会触发其 watcher.
4. 针对不同类型的操作,触发的 watcher 事件也不同. 也就是说对节点的增删改操作触发的watcher 事件是不同的.
    1. (子)节点创建事件.
    2. (子)节点删除事件.
    3. (子)节点数据变化事件.
    4. 基于不同的事件, 那么就可以在判断事件类型的前提下, 做不同的操作.

#### 4.2.1 父节点创建 watcher 事件(zookeeper 版本不同,命令可能不同)
==**父节点和子节点的增删改的操作都可以触发 Wather**==

==**父节点 watcher 事件类型**==
1. `NodeCreated` : 节点创建事件.
2. `NodeDataChanged`: 节点改变事件.
3. `NodeDeleted`: 节点删除事件


==**注意: 在命令行中,watcher 事件为一次性的,所以在设置完, 并触发 watcher 事件后, 需要重新设置 watcher 事件.**==

```properties
创建 watcher事件 命令: 
1. "stat path [watch]"
    例如: stat /cy watch. 针对 /cy节点创建 watcher 事件, 那么接下来对 /cy 节点的增删改操作都会出发 watcher 事件. 创建 watcher 事件的时候, /cy 节点可以不存在!! 当/cy 节点不存在的时候, 虽然会报"node does not exist: /cy"的错误信息, 但是不影响创建 watcher 事件.
    此命令是 3.5 版本之前有用, 写这个文档时用的是 3.5.6 版本, 命令为 "stat -w /cy"
2. "get path [watch]". 3.5 版本之前有用, 写这个文档时用的是 3.5.6 版本, 命令为 "get -w /cy"

```

#### 4.2.2 子节点 watcher
==**子节点的 watcher 都是以父节点为基础, 可以理解为父节点的子节点发生了什么.**==

==**子节点 watcher 事件类型**==
1. `NodeChildrenChanged` : 创建和删除父节点的子节点都会出发此事件.

==**子节点创建和删除事件**==:
1. 使用 `ls /父节点 watch(ls -w /父节点)`创建 watcher 事件, ==**创建**==父节点的子节点和==**删除**==父节点的子节点, 都会出发NodeChildrenChanged事件
2. 使用 `ls /父节点 watch(ls -w /父节点)`创建 watcher 事件, ==**修改父节点的子节点, 不会触发 watcher 事件**==


```properties
创建watcher 事件:
    ls /父节点 watch : 为父节点创建子节点事件.
    此时, 执行 create /cy/ch 会触发NodeChildrenChanged事件. 执行 delete /cy/ch 也会触发NodeChildrenChanged事件.
```

==**子节点的修改事件**==
==想要触发子节点的修改事件, 则需要把子节点当做父节点创建 watch 事件, 不能以子节点的身份去创建, 所以触发的是 NodeDataChanged事件.==

```properties
创建子节点的修改事件:
    1. stat /父节点/子节点 watch.(stat -w /父节点/子节点).
    2. get /父节点/子节点 watch.(get -w /父节点/子节点).
也就是直接在子节点上创建时间, 而不是通过父节点来触发子节点的时间. 会触发 NodeDataChanged 事件
```

#### 4.2.3 watcher 事件使用场景.
==**统一资源配置**==
假如有集群. 在其中一台机器上进行了配置文件的更新, 那么就可以设置这个节点watcher 事件, 然后进行更新操作, 那么就可以收到一个 watcher 事件的通知, 就可以去修改客户端的配置.


### 4.3 ACL(access control lists) 权限控制
#### 4.3.1 ACL 介绍
- ==**针对节点可以设置相关的读写等权限, 目的是为了保障数据安全性.**==
- ==**权限 permissions 可以指定不同的权限范围以及角色**==

==**ACL的构成**==
zookeeper通过==**[scheme:id:permissions]**==来构成权限列表
- `scheme`: 代表采用的某种权限机制(5 种)
    - `world`: 所有用户. world下只有一个 id,即只有一个用户, 也就是 anyone, 组合的写法是: ==**world:anyone:[permissions]**==
    - `auth`: 代表认证登录,需要先注册, 注册用户有权限就可以, 写法为: ==**auth:user:password:[permissions]**==, 密码为明文.
    - `digest`: 需要对密码加密才能访问, 和auth差不多, 只不过输入的密码必须是经过加密的密码, 写法为: ==**digest:username:base64(sha1(password)):[permissions]**==
    - `ip`: 当设置为 ip 指定的 ip 地址, 此时限制 ip 进行访问. 形式为: ==**ip:192.168.1.1:[permissions]**==
    - `super`: 代表超级管理员, 拥有所有权限.
- `id`: 代表允许访问的用户
- `permissions`: 权限组合字符串
    - 权限字符串缩写 : ==**crdwa**==
        - `c`: create, 代表创建子节点,指拥有创建当前==**节点的子节点**==的权限.
        - `r`: read. 获取当前节点/子节点列表. 拥有获取当前节点/子节点列表的权限
        - `w`: write. 设置节点数据
        - `d`: delete. 删除子节点. 拥有删除当前==**节点的子节点**==的权限.(但是可以删除当前节点, 只不过只有当前节点没有子节点的时候才可以进行删除)
        - `a`: admin. 设置权限. 当前用户可以去设置当前节点的权限.

#### 4.3.2 ACL 命令行
- ==**getAcl**==: 获取某个节点的 ACL 权限信息
- ==**setAcl**==: 设置某个节点的 ACL 权限信息
- ==**addauth 权限机制 用户名:明文密码**== : 输入认证授权信息, 也就是==**注册登录**==. 注册时输入明文密码(digest 时也是明文密码), 但是在 zookeeper 系统内, 密码是以加密的形式存在的.

#### 4.3.3 设置权限机制
==**world**==
```properties
world 权限机制: 
    getAcl /cy/autest
        'world,'anyone
        : cdrwa
        world:anyone:cdrwa : 代表任何用户都可以进行增删改的操作, 并且可以进行 a 操作, 也就是设置权限.
    setAcl /cy/autest world:anyone:crwa : 设置crwa 权限,可以创建子节点, 读取节点/子节点数据, 设置节点数据, 修改权限, 但是没有删除当前节点子节点的权限. 但是可以删除当前节点, 只不过删除当前节点前需要保证当前节点没有子节点.
```

==**auth**==

```properties
auth 权限机制: auth:user:pwd:permission
    要使用 auth,就需要先注册, 也就是 addauth进行注册.
    注册 : addauth 权限机制 username:passwd. 例如: addauth digest cy:cy, 使用 digest 权限机制, 注册用户名为 cy, 密码为 cy 的用户
    设置 ACL : setAcl /cy/autest auth:cy:cy:crda
```

==**digest**==

```properties
digest:
    如果已经登录了, 则需要先退出, ctrl + c.
    setAcl /cy/autest digest:cy:加密的密码:权限字符串. 
```

==**ip**==

```properties
ip :
    只有指定 ip 才有指定操作.
    setAcl /cy/autest ip:x.x.x.x:crdwa
```

==**super**==

```properties
super 超级用户:
这种权限机制有最高的权限.
创建 super 步骤 :
    1. 修改 zkServer.sh , 增加 super 管理员
    2. 重启 zkServer

修改 zkServer.sh 文件,找到"nohup"字样,在 -Dzookeeper.root.logger 的双引号后边加入以下值:
"-Dzookeeper.DigestAuthentiationProvider.superDigest=super用户名:密文密码base64(SHA1(passwd))"

可以查看源码: DigestAuthentiationProvider类下的 superDigest()方法.
```

#### 4.4.4 ACL权限控制 使用场景
==**开发/测试环境分批. 开发者无权操作测试库的节点,只能看. 涉及到命名空间**==

==**生产环境上控制指定 ip 的服务可以访问指定的节点,防止混乱.(坏处是如果服务是动态 ip, 则需要频繁的去修改配置)**==


## 第五章 zookeeper 四字命令(four letter words)
- ==**zookeeper可以通过它自身提供的简写命令来和服务器进行交互.**==
- ==**需要使用到 nc 命令, 安装: yum install nc**==
- ==**nc命令使用方式:**== echo [command] | nc [ip] [port]


```properties
stat : 查看 zookeeper 的状态信息,以及是否 mode.(mode 是指当前是集群模式或者单例模式)
```

## 第六章 zookeeper 集群
涉及知识:
1. ==**主从节点**==
2. ==**心跳机制(选举模式)**==
3. ==**这时在其中一台上边创建一个节点,就会同步到其他两台上边去.**==

选举模式: 假如有 master, slave1, slave2, 如果 master 挂掉了, 那么剩下的两个 slave 节点就会进行选举, 假如胜出的是 slave1, slave1 就会升级为 master, 那么剩下的就是一个 master, 还有一个 salve2. 如果此时之前的 master 又启动起来了,它就会加入到集群中,会成为 salve1, 而不会还变成 master.

### 6.1 集群搭建注意点
- 配置数据文件 myid 1/2/3 对应 server.1/2/3
- 通过 ==**zkCli.sh -server [ip]:[port]**==检测集群是否配置成功.

### 6.2 单机伪分布式集群
就是在一台服务器上搭建伪分布式集群.
主要就是 ==**port**== 不一致.

==**搭建方式:**==
```properties
1. 将 zookeeper 解压包复制三份.
2. 修改第一个.
    1. 根据 zookeeper 文件夹名称修改路径. 端口就是用默认的 2181 就行
    2. 在 dataDir 数据文件目录下创建一个文件: "myid"
        方法: vim myid
        写入一个数字  1. 然后保存
            
    2. 修改 master 配置文件 zoo.cfg
        加入以下内容:
            server.1=master 节点 ip:数据同步端口:选举模式使用端口
            server.2=master 节点 ip:数据同步端口:选举模式使用端口
            server.3=master 节点 ip:数据同步端口:选举模式使用端口
            
            例如:
            server.1=10.211.55.8:2888:3888
            server.1=10.211.55.8:2889:3889
            server.1=10.211.55.8:2890:3890
            
3. 修改第二个.
    1. 根据zookeeper 文件名修改 dataDir 路径
    2. 修改端口为 2182
    3. 在 dataDir 数据文件目录下创建一个文件: "myid"
        方法: vim myid
        写入一个数字  2. 然后保存
            
    2. 修改 master 配置文件 zoo.cfg
        加入以下内容:
            server.1=master 节点 ip:数据同步端口:选举模式使用端口
            server.2=master 节点 ip:数据同步端口:选举模式使用端口
            server.3=master 节点 ip:数据同步端口:选举模式使用端口
            
            例如:
            server.1=10.211.55.8:2888:3888
            server.1=10.211.55.8:2889:3889
            server.1=10.211.55.8:2890:3890

            
4. 修改第三个.
    1. 根据zookeeper 文件名修改 dataDir 路径
    2. 修改端口为 2183
    3. 在 dataDir 数据文件目录下创建一个文件: "myid"
        方法: vim myid
        写入一个数字  3. 然后保存
            
    2. 修改 master 配置文件 zoo.cfg
        加入以下内容:
            server.1=master 节点 ip:数据同步端口:选举模式使用端口
            server.2=master 节点 ip:数据同步端口:选举模式使用端口
            server.3=master 节点 ip:数据同步端口:选举模式使用端口
            
            例如:
            server.1=10.211.55.8:2888:3888
            server.2=10.211.55.8:2889:3889
            server.3=10.211.55.8:2890:3890

5. 分别进入三个zookeeper文件夹的 bin 目录, 启动三台机器, 并使用客户端连接

这时在其中一台上边创建一个节点,就会同步到其他两台上边去.
```

### 6.3 在多台服务器上搭建 zookeeper 集群
#### 6.3.1 集群搭建
==**需要注意**==: 环境变量的配置不同, ip 配置不同, 端口号可以相同.

==**搭建步骤**==

```properties
1. 分别在三台服务器上安装 zookeeper, 并配置环境变量.
2. 修改配置文件 zoo.cfg
    1. 端口号都可以使用默认的 2181
    2. server.1=10.211.55.8:2888:3888
       server.2=10.211.55.9:2888:3888
       server.3=10.211.55.10:2888:3888
```

分别启动三台服务器上的 zookeeper, 使用` zkServer.sh status`命令查看当前 zookeeper 的信息.

==**验证: 注意需要关闭防火墙**==
在一台机器上边创建一个节点, 在其他服务器上可以 get 到就说明搭建成功.


#### 6.3.2 选举模式测试
- 使用` zkServer.sh status` 查看当前 zookeeper 状态.
- 在 leader 上使用` zkServer.sh stop` 命令停止 zookeeper, 或者直接关闭服务器.
- 在另外两台上边执行` zkServer.sh status`, 会发现其中一台升级为了 leader.
- 这时启动之前的 leader, 会发现这台重新启动的 zookeeper 会变为 follower.


## 第七章 使用 zookeeper 原生 API 进行客户端开发(不常用)
### 7.1 建立客户端和 zk 服务端的连接

```java
/**
 * @program: zookeeper
 * @description: 需要实现 watcher, 并重写 process 方法,  以接收 watcher 事件通知.
 * @author: cy
 **/
public class Example implements Watcher {

//    private static final String connectString = "10.211.55.8:2181";
    private static final String connectString = "10.211.55.8:2181,10.211.55.9:2181,10.211.55.10:2181";
    private static final Integer sessionTimeout = 5000;
    public static void main(String[] args) throws IOException, InterruptedException {
        /**
         * String connectString : ip:port
         *      可以使单个 ip:port
         *      可以使多个 ip:port, ip1:port1, ip2:port2, ip3:port3...  多个 IP 的话就是连接集群
         * int sessionTimeout : session会话超时时间, 单位是毫秒
         * Watcher watcher : 通知事件. 如果有对应的事件触发, 则会收到一个通知; 如果不需要通知, 那就传个 null
         * long sessionId : 会话 id
         * byte[] sessionPasswd : 会话密码, 当会话丢失后, 可根据会话id 和会话密码重新获取会话
         * boolean canBeReadOnly : 可读. 当这个物理机断开连接后, 还是可以收到数据, 只不过是旧数据, 建议 false, 不推荐使用.
         */
        ZooKeeper zooKeeper = new ZooKeeper(connectString, sessionTimeout, new Example());
        System.out.println("客户端开始连接 zookeeper, ip : " + connectString);
        System.out.println("连接状态: " + zooKeeper.getState());
        Thread.sleep(2000);
        System.out.println("连接状态: " + zooKeeper.getState());
    }

    @Override
    public void process(WatchedEvent event) {
        System.out.println("接收到 watcher 通知 : " + event);
    }
}
```

### 7.2 会话重连机制

```java
public class Example2 implements Watcher {

//    private static final String connectString = "10.211.55.8:2181";
    private static final String connectString = "10.211.55.8:2181,10.211.55.9:2181,10.211.55.10:2181";
    private static final Integer sessionTimeout = 5000;
    public static void main(String[] args) throws IOException, InterruptedException {
        /**
         * String connectString : ip:port
         *      可以使单个 ip:port
         *      可以使多个 ip:port, ip1:port1, ip2:port2, ip3:port3...  多个 IP 的话就是连接集群
         * int sessionTimeout : session会话超时时间, 单位是毫秒
         * Watcher watcher : 通知事件. 如果有对应的事件触发, 则会收到一个通知; 如果不需要通知, 那就传个 null
         * long sessionId : 会话 id
         * byte[] sessionPasswd : 会话密码, 当会话丢失后, 可根据会话id 和会话密码重新获取会话
         * boolean canBeReadOnly : 可读. 当这个物理机断开连接后, 还是可以收到数据, 只不过是旧数据, 建议 false, 不推荐使用.
         */
        ZooKeeper zooKeeper = new ZooKeeper(connectString, sessionTimeout, new Example2());
        long sessionId = zooKeeper.getSessionId();
        byte[] sessionPasswd = zooKeeper.getSessionPasswd();
        System.out.println("客户端开始连接 zookeeper, ip : " + connectString);
        System.out.println("连接状态: " + zooKeeper.getState());
        Thread.sleep(2000);
        System.out.println("连接状态: " + zooKeeper.getState());

        Thread.sleep(1000);
        ZooKeeper zooKeeper1 = new ZooKeeper(connectString, sessionTimeout, new Example2(), sessionId, sessionPasswd);
        System.out.println("开始会话重连 : " + zooKeeper1.getState());
        Thread.sleep(1000);
        System.out.println("重连状态: " + zooKeeper1.getState());

    }

    @Override
    public void process(WatchedEvent event) {
        System.out.println("接收到 watcher 通知 : " + event);
    }
}
```

### 7.3 节点的增删改查




### 7.4 watcher 和 ACL 相关操作

## 第八章 Apache Curator (比较常用)
### 8.1 重试策略
- `RetryPolicy retryPolicy = new ExponentialBackoffRetry(最大重试时间, 最大重试次数)` : ==**最大重试 5 次,每次最大重试时间指定的时间, 如果还没重连, 就放弃掉 **==
- `RetryPolicy retryPolicy = new RetryNTimes(重试次数, 重试间隔时间)` : ==**重试指定的次数**==
- `RetryPolicy retryPolicy = new RetryOneTime(重试间隔的时间)` : ==**如果第一次连接失败, 只会再重试一次**==
- `RetryPolicy retryPolicy = new RetryForever()` : ==**永远都会重试, 不推荐**==
- `RetryPolicy retryPolicy = new RetryUntilElapsed(最大重试时间, 每次重试间隔)` : ==**重试时间超过最大重试时间后, 就不再重试**==

### 8.2 创建连接和关闭连接

```java
CuratorFramework client = CuratorFrameworkFactory.builder()
                        .connectString("ip:port")
                        .sessionTimeoutMs(1000).retryPolicy(上面其中的一种重试策略)
                        .build();
client.start()

client.close();// 关闭连接
```

### 8.3 zookeeper 命名空间和创建节点
#### 8.3.1 namespace 的创建
==**namespace的作用**==: 添加 namespace 成功后, 连接成功后, 这个客户端以后所有的操作都会基于这个 namespace. 类似于 excelipse 的工作空间. 成功后, 在命令行,在==根节点会出现一个指定的名称的节点==(名称和 namespace 一样)

==创建 namespace==
```java
CuratorFramework client = CuratorFrameworkFactory.builder()
                        .connectString("ip:port")
                        .sessionTimeoutMs(1000).retryPolicy(上面其中的一种重试策略)
                        .namespace("workspace").build();
```

#### 8.3.2 创建节点

```java
// 如果指定了namespace, 那么创建的节点就是基于 namespace 的. 也就是 namespace 的子节点.
client.create.creatingParentsIfNeeded() // creatingParentsIfNeeded(): 如果有父节点, 会创建它的父节点,就是可以递归创建多级节点.
    .withMode(CreateMode.PERSISTENT)    // 临时节点/持久节点/临时顺序节点/持久顺序节点
    .withACL(Ids.OPEN_ACL_UNSAFE)       // 指定 ACL 权限
    .forPath("/cy/testNode", "节点值".getBytes());
```

### 8.4 修改节点和删除节点
==**修改节点值**==
```java
client.setDate()
    .withVersion(版本号) // 如果版本号不一致, 则会报 BadVersionException 异常.
    .forPath("节点路径", "节点值".getBytes())
```

==**删除节点**==

```java
client.delete()
    .guaranteed()   // 如果删除失败, 那么在后端还是会继续删除, 直到成功. 比如发生了网络抖动,也会保证这个节点可以被删除
    .deletingChildrenIfNeeded() // 如果有子节点, 就删除它的子节点.
    .withVersion(版本号)   // 如果版本号不一致, 则会报 BadVersionException 异常.
    .forPath("/cy/testNode");
```

### 8.5 查询节点相关信息
#### 8.5.1 查询节点数据和信息

```java
Stat stat = new Stat(); // 节点状态对象
byte[] data = clent.getData()
    .storingStatIn(stat) // 此方法可加可不加, 如果加上,则会在获取数据的同时,将节点状态赋值到 stat 对象中.
    .forPath("/cy/testNode");
sout("节点数据为 : " + new String(data));
sout("节点版本号为 :" + stat.getVersion()) ;
```

#### 8.5.2 查询子节点

```java
List<String> childNodes = client.getChildren()
    .forPath("/cy/testNode");
```

#### 8.5.3 判断节点是否存在

```java
Stat statExist = client.checkExists()
    .forPath("/cy/testNode");
sout(statExist);    // 如果节点不存在, 那么返回的结果 statExist 就为 null.
```

### 8.6 curator 之 watcher 
#### 8.6.1 usingWatcher() 触发后就销毁

```java
public class myWatcher implements CuratorWatcher {
    @Override
    public void process(WatchedEvent event) {
        ...
    }
}

client.getDate
    .usingWatcher(new myWatcher())  // 使用usingWatcher, 那么 watcher 事件只会触发一次, 触发后就销毁
    .forPath("/cy/testNode");
```

#### 8.6.2 nodeCache 一次注册, N 次监听.

```java
final NodeCache nodeCache = new NodeCache(client, "/cy/testNode");  // 将节点作为缓存
nodeCache.start(true); // 如果参数为 true, 那么在建立连接的时候就会把节点数据缓存到本地. 默认为 false, 不会缓存到本地
if (nodeCache.getCurrentDate() != null) {
    sout("节点初始化数据不为空 : " + new String(nodeCache.getCurrentDate().getData()));
} else {
    sout("节点初始化数据不为空");
}
nodeCache.getListenable().addListtener(new NodeCacheListener(){
    // 只要节点发生变化, 就会触发这个方法
    public void nodeChanged() {
        if (nodeCache.getCurrentData() != null) {
            String data = new String(.getData());
            sout("节点路径: " + nodeCache.getCurrentData().getPath + ", 节点数据: " + data);
        } else {
            sout("节点被删除");
        }
    }
});
```

#### 8.6.3 子节点监听 PathChildrenCache

```java
// 对父节点进行监听, 监听父节点下的子节点的所有操作.
final PathChildrenCache childrenCache = new PathChildrenCache(client, "/cy/testNode", true); // 最后一个参数如果为 true, 就会将节点缓存的内容放到一个stat 中.

// POST_INITIALIZED_EVENT : 异步初始化, 初始化之后会触发事件(一般使用这种方式)
// NORMAL : 异步初始化.
// BUILD_INITIAL_CACHE: 同步初始化.
childrenCache.start(StartMode.POST_INITIALIZED_EVENT); // StartMode : 初始化方式. 此处是异步初始化.初始化后会触发事件. 但是异步不会获取到下边的列表数据
List<childDate> childDateList = childrenCache.getCurrentData(); // 获取当前数据节点的子节点数据列表
for(childDate cd : childDateList) {
    sout(new String(cd.getData()));
}
childrenCache.getListenable().addListener(new PathChildrenCacheListener(){
    public void childEvent(CuratorFrameword client, PathChildrenCacheEvent event) {
        // INITIALIZED 子节点初始化
        // CHILD_ADDED 添加子节点
        // CHILD_REMOVE 删除子节点
        // CHILD_UPDATED 修改子节点(路径和数据)
        if (event.getType().equals(PathChildrenCacheEvent.Type.INITIALIZED)) {
            sout("子节点初始化 OK");
        }
    }
});
```

### 8.7 watcher 统一配置更新集群中 N 台节点的配置文件
==**watcher 机制**==: 当修改某一个节点的配置后, 监听到这个节点发生了变化后, 会统一的去修改本地节点的配置.

```java
添加监听事件
```

### 8.8 ACL 相关操作

```java
CuratorAcl acl = new CuratorAcl();

```

## 附录
### zookeeper 操作命令

```properties
help : 查看帮助
ls / : ls 查看根目录下目录结构(zk 的数据模型就类似于目录结构.)  ls [path]
stat /: 查看当前节点的状态, 如 dataversion, calversion 等
    cZxid: 节点创建后, zk 为这个节点设置的 id
    ctime: 创建时间
    mZxid: 修改后的 zk 分配的 id
    mtime: 修改时间
    pZxid: 子节点的 id
    cversion: 子节点的版本
    dataVersion: 当前节点的版本号, 如果修改了, 会累加 1
    aclVersion: acl 权限 id
    dataLength: 数据长度
    numChildren: 子节点的个数
    ephemeralOwner: 用这个属性的值来区分节点是临时节点还是持久节点. 如果是 0x0,就是持久节点, 否则就是临时节点.
ls2 / : ls + stat
get / : 取出当前节点内的数据.

创建命令 create: 
    1. create 节点名 节点值 : 创建一个节点, 并设置值. craete /cy cy-data. 创建根目录下的 cy 节点, 并设置值为 cy-data. 这种创建方式是默认创建, 是非顺序的, 并且是持久化的.
    2. create -e 节点名 节点值 : 创建一个临时节点.
    3. create -s 节点名 节点值 : 创建一个顺序节点.
        执行命令: create -s /cy/sec seq, 会返回这样的字样: Created /cy/sec0000000003
        再次执行: create -s /cy/sec seq, 则会返回这样的字样: Created /cy/sec0000000004
        可以看出, 创建的节点名是按照顺序累加 1 的.
        
修改命令 set:
    1. set 节点名 节点值 : 将节点的值设置为指定的值. 设置后, dataVersion 会累加 1.这种方式在设置的时候不会去验证当前版本号, 只会在设置回将 dataVersion 累加 1
    2. set 节点名 节点值 当前版本号: 只有当命令中的版本号和当前版本号匹配的情况下, 才会去设置节点的值, 否则会报错.


删除命令 delete:
    1. delete 节点 : 删除指定节点.
    2. delete 节点 版本号: 如果输入的版本号和当前版本号不一致, 则不会进行删除.
    
```