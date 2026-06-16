---
title: HTTP 服务器
date: 2026-06-16
tags:
  - lwIP
  - HTTP
  - 应用
---

# HTTP 服务器

## 概述

lwIP 内置了 HTTP 服务器实现，支持静态文件、CGI、SSI 等功能。

## 源文件

- `src/apps/http/httpd.c` - HTTP 服务器主文件
- `src/apps/http/fs.c` - 文件系统实现
- `src/apps/http/http_client.c` - HTTP 客户端
- `include/lwip/apps/httpd.h` - 头文件

## 启用 HTTP 服务器

```c
// lwipopts.h
#define LWIP_HTTPD           1
#define HTTPD_USE_MEM_PBUF   1

// CGI 支持
#define LWIP_HTTPD_CGI       1

// SSI 支持
#define LWIP_HTTPD_SSI       1
```

## 初始化

```c
#include "lwip/apps/httpd.h"

void http_server_init(void) {
    httpd_init();
}
```

## 文件系统

HTTP 服务器使用内存文件系统，需要先运行 `makefsdata` 工具生成。

### 文件结构

```c
// src/apps/http/fs/
//  ├── index.html
//  ├── 404.html
//  └── img/
//      └── logo.gif
```

### 访问文件

```c
// 通过 URL 访问
// http://ip/index.html
```

## CGI (Common Gateway Interface)

CGI 允许动态生成内容。

### CGI 数组

```c
// 定义 CGI 处理函数
const char *cgi_params[] = {
    "/status",
    "/config",
    "/reset"
};

tCGI cgi_handlers[] = {
    { "/status", cgi_status },
    { "/config", cgi_config },
    { "/reset",  cgi_reset }
};
```

### CGI 处理函数

```c
u16_t cgi_handler(int iIndex, int iNumParams, char *pcParam[], 
                  char *pcValue[]) {
    
    if (iIndex == 0) {  // /status
        // 生成状态页面
        char status_html[256];
        snprintf(status_html, sizeof(status_html),
                 "<p>Uptime: %lu</p>", get_uptime());
        
        httpd_response(pcValue[0], status_html);
    }
    
    return httpd_index_html();  // 返回默认页面
}
```

### 注册 CGI

```c
httpd_cgi_start();
httpd_set_cgi_handlers(cgi_handlers, sizeof(cgi_handlers) / sizeof(cgi_handlers[0]));
```

## SSI (Server Side Includes)

SSI 允许在 HTML 中嵌入动态内容。

### SSI 标签格式

```html
<!-- 使用 #! 标记 -->
<html>
<body>
<p>CPU: <!--#cpu--></p>
<p>Mem: <!--#mem--></p>
</body>
</html>
```

### SSI 处理函数

```c
// SSI tags
const char *ssi_tags[] = {
    "cpu",
    "mem"
};

u16_t ssi_handler(char *pcInsert, int iInsertLen, char *pcTag) {
    if (strcmp(pcTag, "cpu") == 0) {
        return snprintf(pcInsert, iInsertLen, "%d%%", get_cpu_usage());
    }
    if (strcmp(pcTag, "mem") == 0) {
        return snprintf(pcInsert, iInsertLen, "%d KB", get_free_mem());
    }
    return 0;
}
```

### 注册 SSI

```c
httpd_ssi_start();
httpd_set_ssi_handler(ssi_handler, ssi_tags, sizeof(ssi_tags) / sizeof(ssi_tags[0]));
```

## 生成 fsdata.c

```bash
# 进入 httpd 目录
cd src/apps/http/makefsdata

# 编译
gcc -o makefsdata makefsdata.c

# 运行生成 fsdata.c
./makefsdata ../fs/
```

## HTTP 客户端

```c
#include "lwip/apps/http_client.h"

// 回调函数
static void http_client_result(void *arg, httpc_state_t *state, 
                               struct pbuf *response, err_t err) {
    if (err == ERR_OK && response) {
        // 处理响应
        pbuf_copy_partial(response, buffer, response->tot_len, 0);
    }
}

// 发起请求
httpc_connection_t settings = {
    .result_fn = http_client_result,
    .alt_connect_fn = NULL
};

httpc_get(&settings, "http://example.com/data.txt", &state);
```

## Session 支持

```c
// 启用 session
#define LWIP_HTTPD_SESSION  1

// Session 回调
void session_handler(void *arg, char *session_id) {
    // 验证 session
}
```

## 相关文件

- [[BSD Socket API]] - 底层通信
- [[Netconn API]] - 内部实现
