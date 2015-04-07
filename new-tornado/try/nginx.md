# 使用nginx部署tornado

一个代理服务器是一台中转客户端资源请求到适当的服务器的机器。一些网络安装使用代理服务器过滤或缓存本地网络机器到Internet的HTTP请求。因为我们将运行一些在不同TCP端口上的Tornado实例，因此我们将使用反向代理服务器：客户端通过Internet连接一个反向代理服务器，然后反向代理服务器发送请求到代理后端的Tornado服务器池中的任何一个主机。代理服务器被设置为对客户端透明的，但它会向上游的Tornado节点传递一些有用信息，比如原始客户端IP地址和TCP格式。

我们在Nginx线上环境使用了 `nginx-upload-module` 将文件上传委托给nginx插件完成。对于Tornado来说，上传文件会造成整个系统阻塞。

并且绝不要在线上环境上将静态文件转发到Tonrado，应但使用nginx来放置静态文件，并设置超时时间。

这里给出我们在线环境当中使用的supervisord配置文件（仅列出Tornado相关部分）。

```
user nginx;
worker_processes 5;

error_log /var/log/nginx/error.log;

pid /var/run/nginx.pid;

events {
    worker_connections 1024;
    use epoll;
}

http {
    upstream tornadoes {
        server 127.0.0.1:8000;
        server 127.0.0.1:8001;
        server 127.0.0.1:8002;
        server 127.0.0.1:8003;
    }
    include /etc/nginx/mime.types;
    default_type application/octet-stream;

    access_log /var/log/nginx/access.log;

    keepalive_timeout 65;
    proxy_read_timeout 200;
    sendfile on;
    tcp_nopush on;
    tcp_nodelay on;
    gzip on;
    gzip_min_length 1000;
    gzip_proxied any;
    gzip_types text/plain text/html text/css text/xml
               application/x-javascript application/xml
               application/atom+xml text/javascript;

    # Only retry if there was a communication error, not a timeout
    # on the Tornado server (to avoid propagating "queries of death"
    # to all frontends)
    proxy_next_upstream error;
    server {
        listen 80;
        server_name www.example.org *.example.org;
        client_max_body_size 100M;

        location /upload {
            upload_pass @after_upload;

            # Store files to this directory
            upload_store /tmp;

            # Allow uploaded files to be world readable
            upload_store_access user:rw group:rw all:rw;

            # Set specified fields in request body
            upload_set_form_field $upload_field_name.name "$upload_file_name";
            upload_set_form_field $upload_field_name.content_type "$upload_content_type";
            upload_set_form_field $upload_field_name.path "$upload_tmp_path";

            # Inform backend about hash and size of a file
            upload_aggregate_form_field "$upload_field_name.md5" "$upload_file_md5";
            upload_aggregate_form_field "$upload_field_name.size" "$upload_file_size";

            upload_pass_form_field "somearg";

            upload_cleanup 200 400 404 499;
        }

        location /static/ {
            root /var/www/static;
            if ($query_string) {
                expires max;
            }
        }

        location @after_upload {
            proxy_pass   http://tornadoes;
        }

        location / {
            proxy_pass_header Server;
            proxy_set_header Host $http_host;
            proxy_redirect off;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Scheme $scheme;
            proxy_pass http://tornadoes;
        }
    }
}
```

有关于Nginx的配置，您可以查看Nginx文档的详细解释。该配置可以直接在线上环境使用，如果要使用https，只要添加一个server块即可。