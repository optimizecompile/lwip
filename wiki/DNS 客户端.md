---
title: DNS 客户端
date: 2026-06-16
tags:
  - lwIP
  - DNS
---

# DNS 客户端

## 概述

DNS (Domain Name System) 客户端用于将域名解析为 IP 地址。

## 源文件

- `src/core/dns.c` - DNS 实现
- `include/lwip/dns.h` - 头文件

## 启用 DNS

```c
// lwipopts.h
#define LWIP_DNS               1
#define DNS_MAX_SERVERS        2        // 最多 DNS 服务器
#define DNS_MAX_NAME_LENGTH    256      // 域名最大长度
```

## API

### 解析域名

```c
#include "lwip/dns.h"

// 同步解析（会阻塞）
err_t dns_gethostbyname(const char *hostname, ip_addr_t *addr,
                        dns_found_callback_t found, void *callback_arg);

// 异步示例
static void dns_callback(const char *name, const ip_addr_t *ip, void *arg) {
    if (ip) {
        printf("Resolved %s to %s\n", name, ipaddr_ntoa(ip));
    } else {
        printf("DNS query failed for %s\n", name);
    }
}

dns_gethostbyname("example.com", &resolved_ip, dns_callback, NULL);
```

### 获取服务器

```c
// 获取 DNS 服务器 IP
const ip_addr_t *dns_server = dns_getserver(0);

// 设置 DNS 服务器
dns_setserver(0, &new_dns_ip);
```

## 示例

### 简单查询

```c
ip_addr_t ip;
err_t result = dns_gethostbyname("api.example.com", &ip, NULL, NULL);

if (result == ERR_OK) {
    printf("IP: %s\n", ipaddr_ntoa(&ip));
}
```

### 异步查询

```c
void fetch_url(const char *domain) {
    dns_gethostbyname(domain, NULL, on_dns_resolved, NULL);
}

static void on_dns_resolved(const char *name, const ip_addr_t *ip, void *arg) {
    if (ip) {
        // 使用解析到的 IP
        http_get(ip);
    }
}
```

## 配置选项

```c
// lwipopts.h
#define DNS_LOCAL_HOSTLIST     1
#define DNS_LOCAL_HOSTLIST_LEN 4

// 本地主机列表（可选）
static const char *local_hostlist[] = {
    "mydevice.local",
    "localhost"
};
```

## DNS 缓存

lwIP DNS 客户端会缓存查询结果直到 TTL 超时。

## 相关文件

- [[TCP 协议]] - 使用 DNS 解析后的 IP 建立连接
- [[UDP 协议]] - DNS 使用 UDP 协议
