---
title: BSD Socket API
date: 2026-06-16
tags:
  - lwIP
  - API
  - Socket
---

# BSD Socket API

## 概述

BSD Socket API 是标准的 POSIX 风格网络编程接口，lwIP 提供了完整的兼容实现。

## 源文件

- `src/api/sockets.c` - Socket 实现
- `include/lwip/sockets.h` - 头文件

## 特性

> [!info] lwIP Socket 特性
> - 兼容 POSIX 标准
> - 基于 Netconn API 内部实现
> - 支持阻塞/非阻塞模式
> - 线程安全（需要 OS 支持）

## 创建与关闭

```c
#include "lwip/sockets.h"

// 创建 socket
int socket(int domain, int type, int protocol);

// 关闭 socket
int close(int s);

// 销毁 socket
int shutdown(int s, int how);
```

## 绑定与监听

```c
// 绑定地址
int bind(int s, const struct sockaddr *name, socklen_t namelen);

// 监听连接
int listen(int s, int backlog);

// 接受连接
int accept(int s, struct sockaddr *addr, socklen_t *addrlen);
```

## 连接

```c
// 连接服务器
int connect(int s, const struct sockaddr *name, socklen_t namelen);
```

## 数据传输

```c
// 发送数据
int send(int s, const void *data, size_t size, int flags);
int sendto(int s, const void *data, size_t size, int flags,
           const struct sockaddr *to, socklen_t tolen);

// 接收数据
int recv(int s, void *mem, size_t len, int flags);
int recvfrom(int s, void *mem, size_t len, int flags,
             struct sockaddr *from, socklen_t *fromlen);
```

## 示例：TCP 客户端

```c
#include "lwip/sockets.h"
#include "lwip/netdb.h"

void tcp_client_example(void) {
    int sock;
    struct sockaddr_in server;
    const char *message = "Hello";
    char buffer[1024];
    
    sock = socket(AF_INET, SOCK_STREAM, 0);
    
    server.sin_family = AF_INET;
    server.sin_port = htons(80);
    inet_aton("192.168.1.1", &server.sin_addr);
    
    connect(sock, (struct sockaddr *)&server, sizeof(server));
    send(sock, message, strlen(message), 0);
    
    int len = recv(sock, buffer, sizeof(buffer), 0);
    close(sock);
}
```

## 示例：TCP 服务器

```c
void tcp_server_example(void) {
    int sock, client_sock;
    struct sockaddr_in server, client;
    
    sock = socket(AF_INET, SOCK_STREAM, 0);
    
    server.sin_family = AF_INET;
    server.sin_port = htons(8080);
    server.sin_addr.s_addr = htonl(INADDR_ANY);
    
    bind(sock, (struct sockaddr *)&server, sizeof(server));
    listen(sock, 5);
    
    while (1) {
        socklen_t len = sizeof(client);
        client_sock = accept(sock, (struct sockaddr *)&client, &len);
        
        // 处理连接...
        close(client_sock);
    }
}
```

## 配置

```c
// lwipopts.h
#define LWIP_SOCKET           1           // 启用 Socket API
#define DEFAULT_THREAD_STACKSIZE 1024     // 栈大小
#define SELECT_MAX_FDS        16           // 最大 fd 数量
```

## select()

```c
#include "lwip/select.h"

int select(int maxfdp1, fd_set *readset, fd_set *writeset, 
           fd_set *exceptset, struct timeval *timeout);

// 使用示例
fd_set readset;
FD_ZERO(&readset);
FD_SET(sock, &readset);

struct timeval tv = { .tv_sec = 5, .tv_usec = 0 };
int ret = select(sock + 1, &readset, NULL, NULL, &tv);

if (FD_ISSET(sock, &readset)) {
    // 可读
}
```

## lwIP 扩展

```c
// 设置 socket 选项
int setsockopt(int s, int level, int optname, const void *optval, socklen_t optlen);

// 获取 socket 选项
int getsockopt(int s, int level, int optname, void *optval, socklen_t *optlen);

// 获取本地地址
int getsockname(int s, struct sockaddr *name, socklen_t *namelen);

// 获取远程地址
int getpeername(int s, struct sockaddr *name, socklen_t *namelen);
```

## 相关文件

- [[Netconn API]] - lwIP 原生 API
- [[TCP 协议]] - TCP 底层实现
- [[UDP 协议]] - UDP 底层实现
