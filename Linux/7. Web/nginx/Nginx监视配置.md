## 取当前 Nginx 服务器的连接数、请求处理数、读取/写入/等待的连接数等信息
- 确认 Nginx 是否已编译并安装了 stub_status 模块。您可以使用以下命令检查 Nginx 是否已启用此模块：
```nginx -V 2>&1 | grep -o with-http_stub_status_module```

- 打开 Nginx 配置文件。默认情况下，Nginx 的配置文件位于 /etc/nginx/nginx.conf并添加以下代码
```C
server {
    listen 127.0.0.1:8080; # 监听的端口号和 IP 地址
    server_name localhost; # 域名或主机名

    location /nginx_status {
        stub_status on; # 启用 stub_status 模块
        access_log off; # 禁用访问日志
        allow 127.0.0.1; # 允许访问的 IP 地址
        deny all; # 禁止其他 IP 地址访问
    }
}

//在上述代码中，我们定义了一个名为 nginx_status 的虚拟主机，并启用了 stub_status 模块。我们还禁用了访问日志以提高性能，并定义了仅允许来自本地主机的 IP 地址访问此页面。
```

- 使用 curl 命令测试 stub_status 页面是否正常工作：
```curl http://127.0.0.1:8080/nginx_status```

- 如果一切正常，您应该会看到一个类似于以下内容的页面：
```
Active connections: 1
server accepts handled requests
 10 10 10 
Reading: 0 Writing: 1 Waiting: 0
```