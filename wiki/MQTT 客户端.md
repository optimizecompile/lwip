---
title: MQTT 客户端
date: 2026-06-16
tags:
  - lwIP
  - MQTT
  - IoT
---

# MQTT 客户端

## 概述

MQTT (Message Queuing Telemetry Transport) 是轻量级的发布/订阅消息协议，广泛用于 IoT 场景。

## 源文件

- `src/apps/mqtt/mqtt.c` - MQTT 实现
- `include/lwip/apps/mqtt.h` - 头文件
- `include/lwip/apps/mqtt_opts.h` - 配置选项

## 启用 MQTT

```c
// lwipopts.h
#define LWIP_MQTT           1
#define MQTT_MAX_TOPIC_LEN    64
#define MQTT_MAX_MSG_LEN      128
```

## MQTT 连接

```c
#include "lwip/apps/mqtt.h"

// 连接参数
mqtt_client_t *client;
struct mqtt_connect_client_info_t ci = {
    .client_id = "lwip_client",
    .client_user = "user",
    .client_pass = "password",
    .keep_alive = 60,
    .will_topic = "status",
    .will_msg = "offline"
};

// 连接回调
static void mqtt_incoming_data_cb(void *arg, const char *topic, u32_t tot_len) {
    // 处理接收到的数据
}

static void mqtt_incoming_publish_cb(void *arg, const char *topic, u32_t tot_len) {
    // 发布回调
}

static void mqtt_connection_cb(mqtt_client_t *client, void *arg, mqtt_connection_status_t status) {
    if (status == MQTT_CONNECT_DISCONNECTED) {
        // 连接断开
    } else if (status == MQTT_CONNECT_ACCEPTED) {
        // 连接成功，订阅主题
        mqtt_sub_unsub(client, "sensor/temp", 1, mqtt_sub_cb, 0);
    }
}

// 建立连接
err_t err = mqtt_client_connect(&client, 
                                 ip_addr_ip4(&broker_ip), MQTT_PORT,
                                 mqtt_connection_cb, &ci);
```

## 发布消息

```c
// 发布 QoS 0
mqtt_publish(client, "sensor/temp", "25.5", 4, MQTT_QOS_0, MQTT_RETAIN_OFF, NULL, NULL);

// 发布回调
static void pub_fin_cb(void *arg, err_t result) {
    if (result == ERR_OK) {
        // 发布成功
    }
}

mqtt_publish(client, "sensor/temp", "25.5", 4, MQTT_QOS_1, MQTT_RETAIN_OFF, pub_fin_cb, NULL);
```

## 订阅主题

```c
static void mqtt_sub_cb(void *arg, err_t result) {
    if (result == ERR_OK) {
        // 订阅成功
    }
}

// 订阅 QoS 1
mqtt_sub_unsub(client, "actuator/relay", 1, mqtt_sub_cb, 1);
```

## 接收消息

```c
// 设置数据回调
mqtt_set_inpub_callback(client, mqtt_incoming_publish_cb, mqtt_incoming_data_cb, NULL);

// 在回调中处理
static void mqtt_incoming_data_cb(void *arg, const char *topic, u32_t tot_len) {
    char *payload = malloc(tot_len + 1);
    // 读取完整数据
    // ...
    payload[tot_len] = '\0';
    
    printf("Received: %s\n", payload);
    free(payload);
}
```

## 完整示例

```c
void mqtt_example(void) {
    mqtt_client_t *client = mqtt_client_new();
    
    struct mqtt_connect_client_info_t ci = {
        .client_id = "lwip_subscriber",
        .keep_alive = 60
    };
    
    // 连接
    mqtt_client_connect(client, &broker_ip, MQTT_PORT, 
                       mqtt_connection_cb, &ci);
    
    // 设置回调
    mqtt_set_inpub_callback(client, pub_cb, data_cb, NULL);
}

static void mqtt_connection_cb(mqtt_client_t *client, void *arg, 
                               mqtt_connection_status_t status) {
    if (status == MQTT_CONNECT_ACCEPTED) {
        mqtt_sub_unsub(client, "home/+/temperature", 1, sub_cb, 1);
    }
}
```

## MQTT 配置

```c
// lwipopts.h
#define MQTT_REQ_TIMEOUT_MS      30000   // 请求超时
#define MQTT_KEEPALIVE           60      // 保活间隔
#define MQTT_MAX_TOPIC_LEN       64      // 最大主题长度
#define MQTT_MAX_MSG_LEN         128     // 最大消息长度
#define MQTT_OUTPUT_RINGBUF_SIZE 256     // 输出缓冲区
```

## QoS 级别

| 级别 | 描述 | 可靠性 |
|------|------|--------|
| MQTT_QOS_0 | 至多一次 | 最低 |
| MQTT_QOS_1 | 至少一次 | 中等 |
| MQTT_QOS_2 | 恰好一次 | 最高 |

## 断开连接

```c
mqtt_disconnect(client);
```

## 相关文件

- [[TCP 协议]] - MQTT 底层传输
- [[学习路径]] - 推荐学习顺序
