---
title: UDP 协议
date: 2026-06-16
tags:
  - lwIP
  - UDP
  - 协议
---

# UDP 协议

## 概述

UDP (User Datagram Protocol) 是无连接的不可靠传输协议，适用于对实时性要求高但能容忍丢包的场景。

## 源文件

- `src/core/udp.c` - UDP 实现
- `include/lwip/udp.h` - 头文件

## UDP PCB

```c
struct udp_pcb {
    struct udp_pcb *next;        // 下一个 PCB
    
    ip_addr_t local_ip;         // 本地 IP
    u16_t local_port;           // 本地端口
    
    ip_addr_t remote_ip;         // 远程 IP
    u16_t remote_port;          // 远程端口
    
    // 回调函数
    void (*recv)(void *, struct udp_pcb *, struct pbuf *, 
                 const ip_addr_t *, u16_t);
    void *recv_arg;             // 回调参数
};
```

## API

### 创建与绑定

```c
#include "lwip/udp.h"

// 创建 UDP PCB
struct udp_pcb *udp_new(void);

// 绑定端口
err_t udp_bind(struct udp_pcb *pcb, const ip_addr_t *ip, u16_t port);

// 连接到远程（可选，设置默认目标）
err_t udp_connect(struct udp_pcb *pcb, const ip_addr_t *ip, u16_t port);

// 删除 PCB
void udp_remove(struct udp_pcb *pcb);
```

### 发送数据

```c
// 发送到已连接的目标或指定目标
err_t udp_send(struct udp_pcb *pcb, struct pbuf *p);
err_t udp_sendto(struct udp_pcb *pcb, struct pbuf *p, 
                 const ip_addr_t *dst_ip, u16_t dst_port);
```

### 接收数据

```c
// 注册接收回调
err_t udp_recv(struct udp_pcb *pcb, 
              void (*recv)(void *, struct udp_pcb *, struct pbuf *,
                           const ip_addr_t *, u16_t),
              void *recv_arg);
```

## 示例：UDP Echo 服务器

```c
static void recv_callback(void *arg, struct udp_pcb *pcb, struct pbuf *p,
                         const ip_addr_t *addr, u16_t port) {
    if (p != NULL) {
        // 回显数据
        udp_sendto(pcb, p, addr, port);
        pbuf_free(p);
    }
}

void udp_echo_server_init(void) {
    struct udp_pcb *pcb = udp_new();
    udp_bind(pcb, IP_ADDR_ANY, 7);
    udp_recv(pcb, recv_callback, NULL);
}
```

## UDP 与 TCP 对比

| 特性 | UDP | TCP |
|------|-----|-----|
| 连接性 | 无连接 | 面向连接 |
| 可靠性 | 不可靠 | 可靠 |
| 顺序性 | 无序 | 有序 |
| 速度 | 快 | 较慢 |
| 资源占用 | 低 | 高 |
| 数据流 | 数据报 | 字节流 |

## 配置选项

```c
// lwipopts.h
#define LWIP_UDP              1           // 启用 UDP
#define UDP_TTL               64          // 生存时间
#define MEMP_NUM_UDP_PCB      4           // UDP PCB 数量
```

## 广播与多播

```c
// 启用广播
#define UDP_BROADCAST         1

// 发送广播
udp_sendto(pcb, p, IP_ADDR_BROADCAST, port);

// 多播支持
#include "lwip/igmp.h"
igmp_joingroup(IP_ADDR_ANY, &multicast_addr);
```

## 相关文件

- [[TCP 协议]] - 可靠的面向连接协议
- [[BSD Socket API]] - Socket 接口
- [[数据包管理]] - UDP 如何使用 pbuf
