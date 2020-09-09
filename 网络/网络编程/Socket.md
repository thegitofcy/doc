# 1. Socket

## Socket 简介

`Socket`作为网络通讯中的端口来使用, 即 `ip + port`



**在 java 中的实现**

`Socket`: 在应哟端的 Socket 端点.

` ServerSocket`: 代码服务器端的 Socket 端点



原理:

<img src="/Users/cy/develop/doc/local/picture/Socket/image-20200909164107602.png" alt="image-20200909164107602" style="zoom:33%;" />



- ServerSocket bind一个端口, 这样, ServerSocket 就会监听这个端端口, 就可以接受到客户端发送到这个端口的信息
- 调用 `accept()` 方法, 这个方法是阻塞的, 一旦调用后, ServerSocket 就会一直等待连接.
- 客户端 Socket connect, 连接上后, 就可以通过 IO 在连个端点之间进行数据传输.



# 2. 简单演示

