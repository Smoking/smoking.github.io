---
title: docker搭建mysql+phpadmin+wordpress+ngnix
date: 2019-05-17 17:27:31
author: smoking
img: https://www.dreamhost.com/blog/wp-content/uploads/2017/07/wordpress-logo-stacked-rgb.png
top: false
cover: true
coverImg: /medias/featureimages/3.jpg
toc: true
mathjax: false
summary: 利用docker搭建 lnmp 环境运行wordpress
categories: shell
tags:
  - shell
  - docker
---


## 环境

*   系统版本
![1557827558608.png](https://upload-images.jianshu.io/upload_images/1728667-e9d083668ecceec9.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

*   docker 版本
       ![1557827461114.png](https://upload-images.jianshu.io/upload_images/1728667-ff58720c7ff7f977.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

*   docker-compose
    具体安装过程参考官方文档即可
![1557827644821.png](https://upload-images.jianshu.io/upload_images/1728667-1c102bd2981dda1c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

## docker-compose.yml

#### 建一个 docker-compose.yml文件

##### mysql

```yaml
services:
    mysql:
    	# 选中mariadb的最新版本
        image: mariadb:latest
        # 端口
        expose:
          - "3306"
        #映射本地 当前目录下mysql文件夹持久化
        volumes:
          - ./mysql:/var/lib/mysql
        #环境变量设置用户名密码
        environment:
          - MYSQL_ROOT_PASSWORD=123456
          - MYSQL_USER=wordpress
          - MYSQL_PASSWORD=123456
          - MYSQL_DATABASE=wordpress
          - MYSQL_RANDOM_ROOT_PASSWORD=1
        #挂掉自动重启
        restart: always
```

##### wordpress


```yaml
wordpress:
	# 选中带有php-fpm 的版本，wordpress docker上有很多版本，根据自己情况选择需要的版本
    image: wordpress:5.2.0-php7.3-fpm
    # 把wordpress的主体文件夹映射到本地 wordpress目录
    volumes:
      - ./wordpress:/var/www/html
    # 环境变量 根据mysql 设置的填入
    environment:
      - WORDPRESS_DB_HOST=mysql
      - WORDPRESS_DB_NAME=wordpress
      - WORDPRESS_DB_USER=wordpress
      - WORDPRESS_DB_PASSWORD=123456
    # 设置依赖
    depends_on:
      - mysql
    restart: always
```

##### phpmyadmin

```yaml
phpmyadmin:
    image: phpmyadmin/phpmyadmin:latest
    ports:
      - "8080:80"
    # 根据mysql设置相关环境变量
    environment:
        - PMA_HOST=mysql
        - PMA_USER=wordpress
        - PMA_PASSWORD=123456
    depends_on:
          - mysql
          - nginx
    restart: always
```


##### nginx

```yaml
nginx:
    image: nginx:latest
    ports:
      - '80:80'
      - '443:443'
    # 映射本地，加载本地的配置
    volumes:
      - ./nginx:/etc/nginx/conf.d
      - ./logs/nginx:/var/log/nginx
      - ./wordpress:/var/www/html  #这里选择本地wordpress即 wordpress。docker中的目录
    depends_on:
      - wordpress
    restart: always
```


## nginx配置

根据上面配置在当前目录下创建一个nginx文件夹，存放配置文件,配置如下

```nginx
server {
listen 80;
server_name localhost;

    root /var/www/html;
    index index.php;

    access_log /var/log/nginx/access.log;
    error_log /var/log/nginx/error.log;

    location / {
        try_files $uri $uri/ /index.php?$args;
    }

    location ~ \.php$ {
        try_files $uri =404;
        fastcgi_split_path_info ^(.+\.php)(/.+)$;
        fastcgi_pass wordpress:9000;
        fastcgi_index index.php;
        include fastcgi_params;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        fastcgi_param PATH_INFO $fastcgi_path_info;
    }
}
```

## 启动容器

​	启动：docker-compose up

## QA

如果wordpress 或者phpadmin 链接数据库失败，单独重启一下 wordpress 或者phpadmin 即可

## 待优化

*  相互依赖的容器 启动顺序控制
    [docker compose 服务启动顺序控制](https://www.cnblogs.com/wang_yb/p/9400291.html)


[github 地址](https://github.com/Smoking/docker-lnmp-wordpres.git)

- - -


[深圳利程电子有限公司](https://www.lcptcheater.com)