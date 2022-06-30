## nginx 常用配置

### 一、 负载均衡

```json
http {
   upstream  LoadBalance-server {
       server   www.site.com:8080;
       server   www.site.com:8081;
   }

   server {
       listen       80;
       server_name  localhost;
       location / {
        proxy_pass http://LoadBalance-server;    
      }

    }

}

```

### 二、动静分离

![image-20220630162340564](https://raw.githubusercontent.com/lilu188011/img-repo/master/image-20220630162340564.png)

