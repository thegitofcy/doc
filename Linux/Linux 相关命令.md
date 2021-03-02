# 1. 常规的简单命令

## 1. 顶层命令

man : man 命令可以通过一些参数快色查询 Linux 帮助手册, 并格式化显示

`man socket` : 显示 socket 帮助手册







# 2. 系统操作相关

### 1. 抓取程序所有的线程对内核有没有发生系统调用

`strace -ff -o ./xxx/xxx java xxx.java`

-ff:

-o: 表示输出.  -o ./xxx/xxx 输出到的目录



### 2. 查一个进程有多少线程?

基于 Linux 一切皆文件的理念, 所有的进程和线程都可以是一个文件夹.

/proc/ : proc 一个程序的线程都会在这个文件下

```shell
jps  --查看所有进程, 拿到进程号, 假如此时是 2878
cd /prpc/2878   -- 进入当前进程号的目录下, 这个目录下是当前进程的所有相关的东西. 主要关注当前目录下的 fd 和 task 目录.
cd task   -- 当前目录下有多少文件就是有多少个线程
cd fd   	--  文件描述符. 是从 0 开始的数字. 任何一个程序都有 IO. 最少有三个. 0: 标准输入, 1 标准输出, 2:错误输出. 
```





# 调用服务



## 1. curl

==**curl 参数**== : -d:短形式. --data: 长形式

```properties
-o : 将结果保存在到指定文件
-d : 请求参数
-v : 进入Verbose模式, 以获取更详细的请求信息
-i : 获取Response Header 信息和 Body 信息
-I : 获取 Response Header 信息
```



### 1.常用命令

```shell
curl www.baidu.com   // 访问www.baidu.com
curl -o /path/file www.baidu.com  // 访问当前 URL, 并将结果保存在 file 文件内
curl -v -o /path/file www.baidu.com // 如果觉得 curl 的结果不太详细, 可以通过 -v 进入Verbose模式, 获取更详细的结果.
curl --trace dump www.baidu.com // 如果觉得-v 还不够详细, 可以使用 --trace file 将更详细的结果保存在 file 文件中. 
curl --tace--ascii dump www.baidu.com // --trace 结果是 16 进制的, 使用--tace--ascii 获取看的明白的结果.
```



### 2. 关于 header

```shell
curl -i www.baidu.com // 获取 Response 的 Header信息以及 Body 信息.
curl -I www.baidu.com // 只获取 Response 的 Head 信息
```



### 3. POST

```shell
curl -d key1=value1&key2=value2 http://example.com // 发送POST 请求
curl -d key1=value1 -d key2=value2 http://example.com // 发送多组数据, curl 会自动将数据连接起来
curl -d @filename http://example.com // 还可以把 post 参数放在文件内, 然后发送请求
```



