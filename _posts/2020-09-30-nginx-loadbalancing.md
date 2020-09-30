---
layout: post
title: "Nginx负载均衡配置"
subtitle: ""
date: 2020-09-30 01:19:22 +0800
author: "YangYongKang"
header-style: text
catalog: true
archive:
    - Nginx 
tags:
    - Nginx
---

### 一、配置
* 打开Nginx文件夹下nginx.conf文件开始配置负载均衡在http段添加以下代码

```javascript
upstream yangyk {
      server 127.0.0.1:8081;
      server 127.0.0.1:8082;
      server 127.0.0.1:8083;
      # 当前负载均衡方式为轮询,为nginx默认的方式
}

server{ 
    listen 8090; 
    server_name  blog.yangyk.com; 
    location / { 
        proxy_pass         http://yangyk; 
    } 
}
```
* yangyk: 配置反向代理服务器组名
* listen 8090: 需要监听的端口
* server_name  blog.yangyk.com: 访问地址 
* proxy_pass http://yangyk;: 表示将所有请求转发到yangyk服务器组中配置的某一台服务器上

### 二、模式
-  默认轮询,上面的upstream就是默认轮询方式，请求会随机派发到你配置的服务器上。
-  权重分配
```javascript
        upstream loadbalancing { # loadbalancing名字可以自定义
            server 127.0.0.1:8081 weight=1; 
            server 127.0.0.1:8082 weight=2;
        }
        #weight的值越高被派发请求的概率也就越高，可以根据服务器配置的不同来设置
```      
- ip哈希分配
```javascript
        upstream loadbalancing { 
               ip_hash;
               server 127.0.0.1:8081; 
               server 127.0.0.1:8082;
        }
        #同一客户端连续的Web请求都会被分发到同一服务器进行处理 
```
- 最少连接分配
```javascript
       upstream loadbalancing { 
            least_conn;
             server 127.0.0.1:8081; 
             server 127.0.0.1:8082;
        }
       #Web请求会被转发到连接数最少的服务器上
```
### 三、参数
- weight 权重    
默认为1，将请求平均分配给每台server,weight = 数值 (值越高被选中的概率也就越高)
- max_fails   
失败多少次踢出队列,默认为1。某台Server允许请求失败的次数，超过最大次数后，在fail_timeout时间内，新的请求将不会分配给这台机器。如果设置为0，Nginx会将这台Server置为永久无效状态   max_fails = 数值
- fail_timeout  
踢出队列后重新探测时间,默认为10秒,某台Server达到max_fails次失败请求后，在fail_timeout期间内，nginx会认为这台Server暂时不可用，不会将请求分配给它,fail_timeout = 60s (s = 秒)
- max_conns   
最大连接数,限制分配给某台Server处理的最大连接数量，超过这个数量，将不会分配新的连接给它。默认为0，表示不限制。max_conns = 800 为防止单机性能过载可以根据实际情况设置
### 四、Nginx启动命令
1. 查看nginx是否启动
   ```html
    ps -ef | grep nginx
   ```
2. 关闭nginx服务
   ```html
    systemctl stop nginx.service  
   ```
3. 启动nginx服务
   ```html
    systemctl start nginx.service  
   ```
4. 设置开机自启动
   ```html
    systemctl enable nginx.service
   ```
5. 停止开机自启动
   ```html
   systemctl disable nginx.service
   ```
6. 查看服务当前状态
   ```html
   systemctl status nginx.service
   ```
7. 重新启动服务
   ```html
    systemctl restart nginx.service
   ```
