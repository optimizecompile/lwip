---
title: Netconn API
date: 2026-06-16
tags:
  - lwIP
  - API
  - Netconn
---

# Netconn API

## 概述

Netconn API 是 lwIP 原生的高级 API，比 BSD Socket 更轻量且支持异步操作。

## 源文件

- `src/api/api_msg.c` - Netconn 消息处理
- `src/api/netbuf.c` - netbuf 结构
- `include/lwip/api.h` - 头文件

## netconn 结构

```c
struct netconn {
    enum netconn_type type;        // 连接类型
    enum netconn_state state;     // 连接状态
    
    union {
        struct tcp_pcb *tcp;
        struct udp_pcb *udp;
        struct raw_pcb *raw;
    } pcb;
    
    // 邮箱用于同步
    sys_mbox_t mbox;              // 消息邮箱
    sys_sem_t sem;                // 信号量
    
    // 回调
    void (*callback)(struct netconn *, enum netconn_evt, u16_t len);
};
```

## API

### 创建与删除

```c
#include "lwip/api.h"

// 创建 netconn
struct netconn *netconn_new(enum netconn_type type);
struct netconn *netconn_new_with_callback(enum netconn_type type, 
                                          void (*callback)(struct netconn *, u8_t, u16_t));

// 删除 netconn
err_t netconn_delete(struct netconn *conn);
```

### TCP 服务器

```c
// 绑定
err_t netconn_bind(struct netconn *conn, const ip_addr_t *addr, u16_t port);

// 监听
err_t netconn_listen(struct netconn *conn);

// 接受连接
err_t netconn_accept(struct netconn *conn, struct netconn **new_conn);

// 接收数据
err_t netconn_recv(struct netconn *conn, struct netbuf **buf);

// 发送数据
err_t netconn_send(struct netconn *conn, struct netbuf *buf);

// 关闭
err_t netconn_close(struct netconn *conn);
```

### TCP 客户端

```c
// 连接到服务器
err_t netconn_connect(struct netconn *conn, const ip_addr_t *addr, u16_t port);

// 发送数据
err_t netconn_write(struct netconn *conn, const void *dataptr, size_t size, 
                    u8_t apiflags);
```

### UDP

```c
// UDP 发送
err_t netconn_send(struct netconn *conn, struct netbuf *buf);

// UDP 接收
err_t netconn_recv(struct netconn *conn, struct netbuf **buf);
```

## netbuf

netbuf 是 Netconn API 使用的数据包结构：

```c
struct netbuf {
    struct pbuf *p, *ptr;         // pbuf 和当前指针
    ip_addr_t addr;               // 源/目标地址
    u16_t port;                   // 源/目标端口
};
```

### netbuf API

```c
// 获取数据
void *netbuf_data(struct netbuf *buf, size_t *len);

// 移动到下一个数据块
err_t netbuf_next(struct netbuf *buf);

// 释放 netbuf
void netbuf_free(struct netbuf *buf);

// 复制 netbuf
struct netbuf *netbuf_copy(struct netbuf *buf);
```

## 示例：TCP 服务器

```c
void tcp_server(void) {
    struct netconn *conn, *new_conn;
    struct netbuf *buf;
    err_t err;
    
    conn = netconn_new(NETCONN_TCP);
    netconn_bind(conn, NULL, 8080);
    netconn_listen(conn);
    
    while (1) {
        err = netconn_accept(conn, &new_conn);
        if (err == ERR_OK) {
            while ((err = netconn_recv(new_conn, &buf)) == ERR_OK) {
                do {
                    netbuf_data(buf, (void**)&data, &len);
                    netconn_write(new_conn, data, len, NETCONN_COPY);
                } while (netbuf_next(buf) >= 0);
                netbuf_delete(buf);
            }
            netconn_close(new_conn);
        }
    }
}
```

## Socket vs Netconn

| 特性 | Socket | Netconn |
|------|--------|---------|
| 兼容性 | POSIX | lwIP 原生 |
| API 风格 | 文件描述符 | 回调/邮箱 |
| 异步支持 | select/poll | 邮箱机制 |
| 内存开销 | 较高 | 较低 |
| 复杂度 | 简单 | 较复杂 |

## 配置

```c
// lwipopts.h
#define LWIP_NETCONN           1           // 启用 Netconn API
#define DEFAULT_ACCEPTMBOX_SIZE   8       // 接受邮箱大小
#define DEFAULT_RECVMBOX_SIZE     16      // 接收邮箱大小
#define DEFAULT_SENDMBOX_SIZE     16      // 发送邮箱大小
```

## 相关文件

- [[BSD Socket API]] - 基于 Netconn 的兼容接口
- [[TCP 协议]] - TCP PCB
- [[UDP 协议]] - UDP PCB
