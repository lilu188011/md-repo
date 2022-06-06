# dockercompose 文件详解

```yaml
# 指定版本信息
version: '3'
# 定义服务（容器）
services:
   # 创建具体的服务（容器）
   db:
     # 指定需要使用的镜像名称
     # 镜像名:tag
     # 如果本地没有指定镜像，那么会从docker hub中下载，否则直接使用本地的镜像
     image: mariadb
     # 在运行容器时，指定需要执行的命令或者参数
     command: --character-set-server=utf8mb4 --collation-server=utf8mb4_unicode_ci
     # 指定数据持久化映射
     volumes:
       # - 数据卷名称或者宿主机文件、路径:容器中的路径
       - mysql_db:/var/lib/mysql
     # 指定容器失败时（Existed），重启策略
     restart: always
     # 指定容器中的全局变量
     environment:
      # 变量名: 变量值
      MYSQL_ROOT_PASSWORD: "123456"
      MYSQL_DATABASE: my_django
     # 指定当前容器需要加入的网络
     networks:
      - django_app_net

   django_app:
     # 指定当前服务（容器）依赖的服务
     depends_on:
      - db
     # 指定通过Dockerfile去构建镜像（Dockerfile所在路径）
     build: ./django
     # 在build下方，指定构建的镜像名称:tag
     image: desireyang/django_app:v2
     restart: always
     volumes:
       - logs:/usr/src/app/logs/
       - django_code:/usr/src/app/web_test/
     networks:
      - django_app_net

   web:
     depends_on:
       - django_app
     build: ./nginx
     image: desireyang/web:v2
     restart: always
     # 将容器中监听的端口与宿主机端口镜像映射
     ports:
       - "8444:80"
       - "8440:8000"
     volumes:
       - logs:/var/log/nginx/
     networks:
      - django_app_net

# 指定需要使用的网络
networks:
  # 指定网络的名称，默认会创建bridge桥接网络
  django_app_net:

# 指定数据卷
volumes:
    mysql_db:
    django_code:
    logs:


```

