### 0. 学习资料参考：

- [嵌入式Linux学习笔记（7）-Socket网络编程_嵌入式linux socket网络编程-CSDN博客](https://blog.csdn.net/m0_74800695/article/details/142451293)

### 1.  `Socket` 基本概念：

- `Socket` 是一个支持不同进程间通信的抽象接口，它使得应用进程能够通过网络协议 (如 `TCP` 或 `UDP`) 进行通信。`Socket` 本身并不包含具体的网络通信细节，它依赖操作系统提供的底层网络接口（如 `Berkeley Socket API`）来实现数据传输
- `Socket` 是网络通信的端点，它既可以作为服务端接受来自客户端的连接请求，也可以作为客户端发起连接请求

### 2.  `Socket` 类型：

- 流式 `Socket` (`SOCK_STREAM`)：
	- 基于 `TCP` 协议，提供可靠的、面向连接的、双向的字节流通信。`TCP` 协议保证数据按顺序可靠地到达，适用于要求高可靠性的应用场景
- 数据包 `Socket` (`SOCK_DGRAM`)：
	- 基于 `UDP` 协议，是一种无连接的通信方式，数据包是独立的，不保证数据的顺序和可靠性。适用于对实时性要求较高、能容忍丢包的应用
- 原始 `Socket` (`SOCK_RAW`)：
	- 允许直接访问底层协议，如 `IP` 或 `ICMP`，它功能强大但使用较为不便，主要用于协议开发

> [!NOTE] `TCP` 与 `UDP`
> - `TCP` 是一种面向连接的协议，提供可靠的数据传输。`TCP` 通过建立一个连接，将数据分为多个小数据包，并为每个数据包分配一个序列号，使得它们能按顺序传输到目标节点。如果数据包丢失或损坏，`TCP` 会进行重传，确保所有数据包都被传输，并按顺序重新组装
> 	- `TCP` 还提供了流量控制和拥塞控制的机制，以确保网络稳定和高效
> 	- `TCP` 适用于需要可靠传输、顺序传输和流量控制的应用程序，如网页浏览、文件传输和电子邮件等
> 	  
>  ----------------------------------------------------------------------
> 	  
> - `UDP` 是一种无连接的协议，提供不可靠的数据传输。`UDP` 将数据分为小的数据包，并将它们发送到目标节点，但不保证它们能按顺序送达（尽最大努力传输 `best-effort`）。同时 `UDP` 不会进行检验重传，也不提供流量控制和拥塞控制
> 	- 对比 `TCP` 较为严谨的传输方式，`UDP` 显得就没有那么“繁杂“。因此 `UDP` 的传输速度相对较快，但在不可靠的网络环境下可能会有数据丢失或乱序
> 	- `UDP` 适用于对实时性要求较高，但对数据可靠性要求不高的应用程序，如视频流传输，语音通话和在线游戏等

### 3.  `Socket` 模型：

- `Socket` 通常采用客户端-服务器模型,，依据不同的通信协议有不同的流程：
- `TCP/IP` 通信流程（三次握手建立连接，四次挥手终止连接）：
	- 建立：
	  1. 客户端向服务器发送 `SYN`（同步）请求
	  2. 服务器接收到 `SYN` 包后，回复确认信息（`SYN-ACK` 包）给客户端（完成第一次握手）
	  3. 客户端接收到确认信息后，再次向服务器发送确认消息（`ACK` 包）（完成第二次握手）
	  4. 服务器接收到确认消息后，连接建立成功，开始进行数据传输

	- 数据传输（上传为例）：
	  1. 客户端将需要传输的数分成较小的数据包，并为每个数据包分配一个序列号，并将数据包传输到服务器
	  2. 服务器接收到数据包后，发送 `ACK` 包给客户端，表示已经接收到数据
	  3. 客户端接收到确认信息后，再次向服务器发送下一个数据包
	  4. 服务器不断接收和发送数据包，直到所有数据传输完成

	- 终止：
	  1. 数据传输完成后，客户端发送一个连接释放请求（`FIN` 包）给服务器
	  2. 服务器接收到连接释放请求后，发送确认信息（`ACK` 包）给客户端，表示同意释放连接（完成第一次挥手）
	  3. 客户端接收到确认信息后，不可以再向服务端发送消息，但仍可以接收服务端发送的消息（完成第二次挥手）
	  4. 服务器如果没有要发送的消息，此时也发送一个连接释放请求给客户端
	  5. 客户端接收到连接释放请求后，发送确认消息给服务器，表示同意释放连接（完成第三次挥手），此时客户端关闭
	  6. 服务器接收到确认消息后关闭（完成第四次挥手）

---

- `UDP` 通信流程（无需建立与终止，直接进行通信）：
	1. 客户端将需要传输的数据打包成数据报，并指定目标主机的 `IP` 地址和端口号。
	2. 客户端将数据报发送给目标主机
	3. 服务器接收到数据包后，根据目标端口号对数据进行处理
	4. 服务器根据需要向客户端发送响应数据报
	5. 客户端接收到服务器的响应数据报，进行相应的处理
	6. 数据传输完成后，连接自动关闭，不需要进行连接释放

---

- 实例代码（`TCP` 为例）：
1. 服务器端 `server.c`：

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <arpa/inet.h>

#define PORT 8080       // 服务器端口号
#define MAX_CLIENTS 5   // 最大客户端连接数

int main(){
    int server_fd;                  // 服务器文件描述符
    int new_socket;                 // 客户端文件描述符
    struct sockaddr_in addr;        // 实例化服务器地址结构体
    char buffer[1024] = {0};        // 缓冲区

    int addr_len = sizeof(addr);

    // 创建 socket
    if((server_fd = socket(AF_INET, SOCK_STREAM, 0)) == 0){
        perror("socket failed");
        exit(EXIT_FAILURE);
    }

    // 设置服务器地址结构体
    addr.sin_family = AF_INET;          // 配置 IPv4
    addr.sin_addr.s_addr = INADDR_ANY;  // 接受任意 IP 地址连接
    addr.sin_port = htons(PORT);        // 指定端口号

    // 绑定 socket 到地址和端口
    if(bind(server_fd, (struct sockaddr *)&addr, addr_len) < 0){
        perror("bind failed");
        exit(EXIT_FAILURE);
    }

    // 启动监听
    if (listen(server_fd, MAX_CLIENTS) < 0){
        perror("listen failed");
        exit(EXIT_FAILURE);
    }

    printf("Server is listening on port %d ... \n", PORT);

    // 接受客户端连接
    if ((new_socket = accept(server_fd, (struct sockaddr *)&addr, (socklen_t*)&addr_len < 0))){
        perror("accept failed");
        exit(EXIT_FAILURE);
    }

    // 输出客户端连接信息
    printf("New connection: %s:%d\n", inet_ntoa(addr.sin_addr), ntohs(addr.sin_port));

    // 读取客户端发送的消息
    while(1){
        int valread = read(new_socket, buffer, 1024);
        if (valread == 0){
            printf("Client disconnected\n");
            break;
        }

        printf("Client: %s\n", buffer);

        // 发送回复消息给客户端
        send(new_socket, "Hello from server", 18, 0);
    }

    // 关闭连接；
    close(new_socket);
    close(server_fd);

    return 0;
}
```

2. 客户端 `client.c`：

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <unistd.h>
#include <arpa/inet.h>

#define SERVER_IP "192.168.7.141"       // 服务器 IP 地址
#define SERVER_PORT 8080                // 服务器端口号

int main(){
    int sock;
    struct sockaddr_in server_addr;
    char buffer[1024] = {0};            // 创建缓冲区
    char message[] = "Hello from client";

    // 创建 socket
    if ((sock = socket(AF_INET, SOCK_STREAM, 0)) < 0){
        perror("Socket creation falied");
        exit(EXIT_FAILURE);
    }

    // 设置服务器地址
    server_addr.sin_family = AF_INET;
    server_addr.sin_port = htons(SERVER_PORT);

    // 将服务器 IP 地址从文本转换为网络字节序
    if (inet_pton(AF_INET, SERVER_IP, &server_addr.sin_addr) <= 0){
        perror("Invalid address / Address not supported");
        exit(EXIT_FAILURE);
    }

    // 发送数据到服务器
    send(sock, message, strlen(message), 0);
    printf("Message sent to server: %s\n", message);

    // 接收服务器的响应
    int valread = read(sock, buffer, 1024);
    printf("Server response: %s \n", buffer);

    // 关闭连接
    close(sock);
    return 0;

}
```