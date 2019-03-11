---
title: docker中nginx配置HTTPS证书
categories:
 - DevOps
tags: docker,CentOS,nginx,https
---

 因小程序要求，所有请求必须是https协议，这里我手动配置了一次证书。
 
1. 申请证书
腾讯云和阿里云都有提供一个一年的免费证书，购买完证书，需要配置域名。
![ali_ssl_ca](https://raw.githubusercontent.com/xuguangwu/blog/master/_posts/images/ali_ssl_ca.png)

2. 域名与ip做映射
![domain_ip_mapping](https://raw.githubusercontent.com/xuguangwu/blog/master/_posts/images/domain_ip_mapping.png)

3. CentOS7.2上命令及步骤

cd /home

mkdir -p nginx/{conf.d,conf.crt,html,logs}

touch nginx/nginx.conf

vi nginx/nginx.conf
````
    user  nginx;
    worker_processes  auto;
    
    error_log  /var/log/nginx/error.log warn;
    pid        /var/run/nginx.pid;
    
    events {
        worker_connections  2048;
    }
    
    http {
        include       /etc/nginx/mime.types;
        default_type  application/octet-stream;
    
        sendfile        on;
        keepalive_timeout    65;
        client_max_body_size 10M;
    
        include /etc/nginx/conf.d/*.conf;
    }
````

touch nginx/conf.d/default.conf
````
upstream web{
    server 127.0.0.1:30081;
}

server {
        listen       443 default ssl; #容器内端口，可在宿主机映射成想要的端口
#       listen       [::]:443;
        server_name  20youk.xyz;

#        ssl on;
        ssl_certificate /mnt/ca/1_20youk.xyz_bundle.crt; #需要修改该证书路径
        ssl_certificate_key /mnt/ca/2_20youk.xyz.key; #需要修改该证书路径
        ssl_session_timeout 5m;
        ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
        ssl_ciphers ECDHE-RSA-AES128-GCM-SHA256:HIGH:!aNULL:!MD5:!RC4:!DHE;
        ssl_prefer_server_ciphers on;

        location / {
            root /usr/share/nginx/html;
        }

        location ^~/dolphin-api/ {
           proxy_pass http://10.20.24.243:30081; #需要修改服务地址
         }

}
````

启动容器，这里我将下载下来的证书放在了/mnt/ca目录下，分别是
/mnt/ca/1_20youk.xyz_bundle.crt,/mnt/ca/2_20youk.xyz.key
````
docker run -d \
    -p 30443:443 \
    -v $(pwd)/nginx/conf.d:/etc/nginx/conf.d:ro \
    -v $(pwd)/nginx/nginx.conf:/etc/nginx/nginx.conf:ro \
    -v $(pwd)/nginx/logs/nginx:/var/log/nginx \
    -v $(pwd)/nginx/html:/tmp/html \
    -v /mnt/ca:/mnt/ca \
    --name nginx \
    nginx
````

开放主机的30443端口，在云平台的安全组策略中，放开30443端口
````
firewall-cmd --zone=public --add-port=30443/tcp --permanent
firewall-cmd --reload
firewall-cmd --zone= public --query-port=30443/tcp
firewall-cmd --zone= public --remove-port=30443/tcp --permanent
````

至此，可以用https协议访问
curl https://20youk.xyz:30443/dolphin-api/api/userInfo


![nginx_file_tree](https://raw.githubusercontent.com/xuguangwu/blog/master/_posts/images/nginx_file_tree.png)


Notices:
1. 因为服务是部署在docker内，涉及到公网ip，内网ip，以及docker内的ip和dns服务解析问题，需要注意相关的ip配置，
我就遇到了https可以访问静态页面，但是访问proxy转发的服务的时候报502，因为我在配置proxy的时候转发到了公网ip，
但是在本机无法解析公网ip，配置成内网ip后就可以访问了。
2. 如果443端口被占用，可以在nginx中配置别的端口，证书是与域名绑定的，可以同时配置nginx监听多个端口，公用一个域名，
然后通过域名+端口的方式来区分访问的服务
3. 如果宿主机无法解析域名，可在/etc/hosts中配置域名与ip，ip设为127.0.0.1
4. 我使用的nginx版本为1.15.5,出现问题可以通过nginx的日志来排查问题