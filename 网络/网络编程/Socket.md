

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

**ServerSocket 服务端**

```java
public class Server {

    // 断开指令
    private static final String QUIT = "quit";
    // 服务端要监听的端口, 客户端可以通过这个端口连接到此服务端
    private static final int DEFAULT_PORT = 8888;

    public static void main(String[] args) {
        ServerSocket serverSocket = null;
        try {
            // 绑定端口
            serverSocket = new ServerSocket(DEFAULT_PORT);
            while (true) {
                // 阻塞式连接
                Socket socket = serverSocket.accept();
                int clientPort = socket.getPort();
                System.out.println("建立连接,  客户端 port: " + clientPort);
                BufferedReader reader = new BufferedReader(new InputStreamReader(socket.getInputStream()));
                BufferedWriter writer = new BufferedWriter(new OutputStreamWriter(socket.getOutputStream()));
                String msg = null;
                // 循环读取客户端发送的信息
                while ((msg = reader.readLine()) != null) {
                    System.out.println("接收客户端信息: " + msg);
                    // 向客户端返回信息
                    writer.write(msg + "\n");
                    writer.flush();
                    // 如果接受到的客户端信息是断开指令, 则退出此循环
                    if (QUIT.equals(msg)) {
                        System.out.println("客户端: " + clientPort + " 已断开连接");
                        break;
                    }
                }
            }
        } catch (IOException e) {
            e.printStackTrace();
        } finally {
            if (null != serverSocket) {
                try {
                    serverSocket.close();
                    System.out.println("关闭 ServerSocket");
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
        }
    }
}
```



**Socket 客户端**

```java
public class Client {

    // 断开指令
    private static final String QUIT = "quit";
    // 要连接的服务端的 IP
    private static final String DEFAULT_SERVER_HOST = "127.0.0.1";
    // 要连接的服务端的端口
    private static final int DEFAULT_SERVER_PORT = 8888;

    public static void main(String[] args) {
        Socket socket = null;
        BufferedWriter writer = null;
        try {
            // 创建客户端 Socket
            socket = new Socket(DEFAULT_SERVER_HOST, DEFAULT_SERVER_PORT);

            BufferedReader reader = new BufferedReader(new InputStreamReader(socket.getInputStream()));
            writer = new BufferedWriter(new OutputStreamWriter(socket.getOutputStream()));

            BufferedReader consalReader = new BufferedReader(new InputStreamReader(System.in));
            while (true) {
                // 读取控制台输入的息息
                String msg = consalReader.readLine();
                // 向服务端发送信息
                writer.write(msg + "\n");
                writer.flush();
                // 读取服务端返回的信息
                String s = reader.readLine();
                System.out.println("服务端返回: " + s);
                // 如果控制台输入的时退出指令, 则退出
                if (QUIT.equals(msg)) {
                    break;
                }
            }
        } catch (IOException e) {
            e.printStackTrace();
        } finally {
            if (null != writer) {
                try {
                    writer.close();
                    System.out.println("关闭 socket");
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
        }
    }
}
```



# 3. 