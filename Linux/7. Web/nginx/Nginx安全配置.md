## Nginx DDOS防御

### 限制每秒请求数

- ngx_http_limit_req_module模块通过漏桶原理来限制单位时间内的请求数，一旦单位时间内请求数超过限制，就会返回503错误。配置需要在两个地方设置:
  - nginx.conf的http段内定义触发条件，可以有多个条件
  - 在location内定义达到触发条件时nginx所要执行的动作

```C

http {
  	limit_req_zone $binary_remote_addr zone=one:10m rate=10r/s;
	//触发条件，所有访问ip 限制每秒10个请求  ...
	server {
  		...
		location ~ \.php$ {
			//执行的动作,通过zone名字对应
    		limit_req zone=one burst=5 nodelay; 
 		}
 	}
}
```

- 参数说明：
  - $binary_remote_addr: 二进制远程地址;
  - zone=one: 10m 定义zone名字叫one，并为这个zone分配10m内存，用来存储会话（二进制远程地址），1m内存可以保存16000会话;
  - rate=10r/s; 限制频率为每秒10个请求
  - burst=5 允许超过频率限制的请求数不多于5个，假设1、2、3、4秒请求为每秒9个，那么第5秒内请求15个是允许的，反之，如果第一秒内请求15个，会将5个请求放到第二秒，第二秒内超过10的请求直接503，类似多秒内平均速率限制;
  - nodelay 超过的请求不被延迟处理，设置后15个请求在1秒内处理;

### 限制IP连接数

- ngx_http_limit_conn_module的配置方法和参数与http_limit_req模块很像，参数少，要简单很多

```C
http {
  	limit_conn_zone $remote_addr zone=addr:10m; //触发条件  ...  
	server { 
  		...   
		location /download/ { 
   			limit_conn addr 1; // 限制同一时间内1个连接，超出的连接返回503 
    	} 
   	}
}

```

### 举例:

- 此配置将限制每个IP的连接速率为每秒5个请求，同时在短时间内允许10个突发请求。如果超过了这些限制，nginx将拒绝连接或请求。

```C
http {
    limit_conn_zone $binary_remote_addr zone=conn_limit_per_ip:10m;
    limit_req_zone $binary_remote_addr zone=req_limit_per_ip:10m rate=5r/s;

    limit_conn conn_limit_per_ip 10;
    limit_req zone=req_limit_per_ip burst=10 nodelay;
}
```

---

### 白名单设置

- http_limit_conn和http_limit_req模块限制了单ip单位时间内的并发和请求数，但是如果nginx前面有lvs或者haproxy之类的负载均衡或者反向代 理，nginx获取的都是来自负载均衡的连接或请求，这时不应该限制负载均衡的连接和请求，就需要geo和map模块设置白名单：

```C
geo $whiteiplist {
	default 1; 10.11.15.161 0; 
} map $whiteiplist $limit {
	1 $binary_remote_addr; 0 ""; 
} 
limit_req_zone $limit zone=one:10m rate=10r/s;
limit_conn_zone $limit zone=addr:10m;
```

- 参数说明：
- geo模块定义了一个默认值是1的变量whiteiplist，当在ip在白名单中，变量whiteiplist的值为0，反之为1
- 如果在白名单中–> whiteiplist=0 --> $limit=“” --> 不会存储到10m的会话状态（one或者addr）中 --> 不受限制,反之，不在白名单中 --> whiteiplist=1 --> $limit=二进制远程地址 -->存储进10m的会话状态中 --> 受到限制

### 使用 Nginx 的缓存

- Nginx 的缓存可以将请求的响应缓存下来，从而减少对后端服务器的请求。可以通过以下配置启用
- 使用 Nginx 的缓存

```C
http {
    proxy_cache_path /var/cache/nginx levels=1:2 keys_zone=my_cache:10m inactive=60m;
    server {
        location / {
             proxy_cache my_cache;
             proxy_cache_valid 200 60m;
             proxy_cache_valid 404 1m;
           }
    }
}

- 反向代理缓存配置
```C
http {
    proxy_cache_path /var/cache/nginx levels=1:2 keys_zone=my_cache:10m inactive=60m;
    
    server {
        listen 80;
        server_name example.com;
        location / {
            proxy_pass http://backend;
            proxy_cache my_cache;
            proxy_cache_valid 200 60m;
            proxy_cache_valid 404 1m;
            proxy_cache_key "$scheme$request_method$host$request_uri";
        }
    }
    
    upstream backend {
        server backend.example.com;
    }
}
#这个配置文件定义了一个名为my_cache的缓存区，缓存路径为/var/cache/nginx，缓存数据的存储级别为1:2，最长缓存时间为60分钟。这个缓存区被用于代理到backend服务器的请求。
#其中，proxy_cache_valid指令定义了对于不同的HTTP响应码，缓存可以保留多长时间。在这个示例中，对于HTTP响应码为200的响应，缓存可以保留60分钟，对于HTTP响应码为404的响应，缓存只能保留1分钟。
#proxy_cache_key指令定义了用于生成缓存键的字符串。这个字符串是由$scheme、$request_method、$host和$request_uri这些变量组成的。这个缓存键将作为缓存数据的唯一标识符。

#pragma endregion请注意，在上面的配置文件中，我们仅仅缓存了HTTP响应码为200和404的响应。如果您希望缓存所有的响应，可以使用以下的proxy_cache_valid指令：
proxy_cache_valid any 60m;
#这个指令将允许缓存任何响应，且最长缓存时间为60分钟。

```

//这将缓存状态码为 200 的响应 60 分钟，并缓存状态码为 404 的响应 1 分钟。
```

---

### 防止HTTP慢速攻击

- 此配置将限制客户端在10秒内必须发送请求或数据，同时将缓冲区大小限制为1k。如果连接不稳定或缓慢，nginx将关闭连接并拒绝请求。

```C
http {
    client_body_timeout 10s;
    client_header_timeout 10s;
    send_timeout 10s;

    client_body_buffer_size 1k;
    client_header_buffer_size 1k;
    client_max_body_size 1k;
}
```

## Nginx防止SQL注入

- 禁用server_tokens指令，不暴露版本号

```C
#建议配置在http 全局

Server_tokens off;
```

- 禁用不需要的HTTP方法

```C
#  一般的网站和应用程序，你应该只允许GET，POST，和HEAD并禁用其他
#  http 444 代表无响应 
 
location / {
    # 允许 GET、POST、HEAD 请求
    limit_except GET POST HEAD {
        deny all;
    }
}
```

- 设置缓冲区大小限制

```C
# 为了防止对您的Nginx Web服务器的缓冲区溢出攻击，坐落在一个单独的文件以下指令（创建的文件名为/etc/nginx/conf.d/buffer.conf为例）
 
client_body_buffer_size  1k;
client_header_buffer_size 1k;
client_max_body_size 1k;
large_client_header_buffers 2 1k;
 
# 包含此配置
include /etc/nginx/conf.d/*.conf;
 
 
```

- 设置access error日志

```C
# nginx.conf配置文件中，error_log、access_log前的#去掉
 
# 示例如下：
 
log_format  nsfocus  '$remote_addr - $remote_user [$time_local] '
              ' "$request" $status $body_bytes_sent "$http_referer" '
              ' "$http_user_agent" "$http_x_forwarded_for"'; 
 
access_log  logs/access.log  nsfocus;
error_log   logs/error.log   nsfocus;
 
# nsfocus是设置配置文件格式的名称
```

- 反向代理隐藏主机信息

```C
# 建议配置在http
 
proxy_hide_header X-Application-Context;
 
或者
 
proxy_hide_header X-Powered-By; proxy_hide_header Server;
```

- SSL安全加固

```C
ssl_protocols 协议版本指定为TLSv1.2  TLSv1.3；
 
# 指定加密模式 禁用弱加密模式
ssl_ciphers ECDHE-RSA-AES128-GCM-SHA256:ECDHE:ECDH:AES:HIGH:!NULL:!aNULL:!MD5:!3DES:!ADH:!RC4:!DH:!DHE;
```
