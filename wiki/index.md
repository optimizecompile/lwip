---
title: lwIP 学习笔记
date: 2026-06-16
tags:
  - lwIP
  - TCP/IP
  - 嵌入式
aliases:
  - lwIP Wiki
  - lwIP笔记
---

# lwIP 学习笔记

> 轻量级 TCP/IP 协议栈 - 嵌入式系统网络通信学习仓库

## 快速导航

### 核心模块
- [[lwIP 概述]] - 项目简介与架构
- [[内存管理]] - mem.c, memp.c
- [[数据包管理]] - pbuf.c
- [[网络接口]] - netif.c
- [[TCP 协议]] - tcp.c
- [[UDP 协议]] - udp.c
- [[DNS 客户端]] - dns.c

### 应用程序接口
- [[BSD Socket API]] - sockets.c
- [[Netconn API]] - api_msg.c
- [[Packet API]] - if_api.c

### 高级应用
- [[HTTP 服务器]] - httpd.c
- [[MQTT 客户端]] - mqtt.c
- [[SNMP 代理]] - snmp.c
- [[mDNS/DNS-SD]] - mdns.c
- [[DHCP 客户端]] - dhcp.c

### 学习路径
- [[学习路径]] - 从入门到精通

---

## 项目结构

```
/workspace/
├── src/
│   ├── api/          # 高级API (socket, netconn)
│   ├── apps/         # 应用层协议 (HTTP, MQTT, SNMP)
│   ├── core/         # 核心协议 (TCP, UDP, IP, ICMP)
│   ├── core/ipv4/    # IPv4 相关
│   ├── core/ipv6/    # IPv6 相关
│   ├── include/      # 头文件
│   └── netif/        # 网络接口 (Ethernet, PPP)
├── contrib/          # 第三方集成与示例
├── doc/             # 文档
└── test/           # 单元测试
```

## 资源链接

- [官方文档](https://www.nongnu.org/lwip/)
- [GitHub 仓库](https://github.com/lwip-tcpip/lwIP)
