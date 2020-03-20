# why

什么都可以docker化，docker化的php环境开发、测试、部署等等

## 前言
  使用前，默认你已经安装了 [docker](https://www.jianshu.com/search?q=docker%E5%AE%89%E8%A3%85&page=1&type=note) 和 [docker-compose](https://www.jianshu.com/p/f323aa0416da)

## 先使用阿里云镜像加速器
 
 [阿里云镜像加速器](https://dev.aliyun.com)
 
 进入之后登录，然后进到 [ 管理中心 ]，找到镜像加速器，里面有教你怎么配置本地的加速器，不再赘述
  
 ##  克隆此包
  
     git clone  https://github.com/chichoyi/dnmp.git

### 修改docker-compose.yml

- docker搭建lnmp环境，php 7.2 + nginx latest + mysql 5.7 + redis 4 + ...

 参数说明
 
    nginx:
      image: nginx:latest
      container_name: nginx
      hostname: nginx
      ports:
        - "80:80"
      volumes:
        - /data/www:/www  # /data/www 这个是宿主机的目录，可以修改指向你的代码文件夹，比如 /mnt/hgfs/www
        - ./nginx/nginx.conf:/etc/nginx/nginx.conf:ro  # 这里是配置文件，不多，可以打开看看
        - ./nginx/conf.d/:/etc/nginx/conf.d/:ro
      links:
        - php:PHP-FPM  # php容器的别名，nginx发送给php需要用到的，默认即可
      networks:
        - lnmp
      
    php:
      build: ./phpfpm  # Dockerfile 文件地址
      container_name: php
      hostname: php
      ports:
        - "9000：9000"
      links:
        - mysql:mysql
        - redis:redis
      volumes:
        - /data/www:/www  # /data/www 这个是宿主机的目录，可以修改指向你的代码文件夹，比如 /mnt/hgfs/www
        - ./phpfpm/conf/:/usr/local/etc/php/conf.d
      networks:
        - lnmp
        
        # 注意，nginx和php挂载出来的宿主机目录一定要一致(/data/www)，不然没法访问php文件
    
    mysql:
      image: mysql:5.7
      container_name: mysql
      hostname: mysql
      ports:
        - "3306:3306"
      environment:
        - MYSQL_ROOT_PASSWORD=root  #数据库的初始化密码，这里可以指定修改成你自己的
        networks:
          - lnmp
            
    redis:
      image: redis:4
      container_name: redis
      hostname: redis
      ports:
        - "6379:6379"
      networks:
        - lnmp
        
    networks:
      lnmp:
        driver: bridge
   
   
 ## 修改完之后执行命令
  
    cd dnmp
    
    docker-compose build
    
    # 修改代码目录
    vi docker-compose.yml
    # 把42行和54行的 /Users/chenyinshan/code 修改为你本机的代码目录

    vi ./nginx/conf.d/defualt.conf   修改你自己的虚拟域名和目录,容器的目录默认是/www,
    
    docker-compose up -d
    
    //如果没有目录的话请自己创建相关目录
    
 第一次执行需要花点时间下载镜像
 
 php连接mysql需要在本机hosts文件添加域名映射，例如：
 
    127.0.0.1 mysql
    127.0.0.1 nginx
    127.0.0.1 php
    127.0.0.1 redis
 
 这样就不用去查找mysql的容器的IP了
  
    # docker-compose up -d 的时候可能一些服务没有跑起来，
    # 比如mysql，可能你给的dnmp目录权限不够，要给充足权限
    
    sudo chmod -R 777 dnmp
    
## 如何composer安装php项目依赖包

    docker run --rm --interactive --tty \
    --volume $PWD:/app \
    composer install --ignore-platform-reqs --no-scripts
    
- docker 会自动拉取composer镜像然后在当前目录执行composer命令
- --ignore-platform-reqs --no-scripts 这条命令是忽略扩展要求

## docker php 如何安装扩展参考

### php的dockerfile讲解

~~~
FROM php:7.2.2-fpm

ENV PHPREDIS_VERSION 3.1.3

#for redis and mysql
RUN curl -L -o /tmp/redis.tar.gz https://github.com/phpredis/phpredis/archive/$PHPREDIS_VERSION.tar.gz \
    && tar xfz /tmp/redis.tar.gz \
    && rm -r /tmp/redis.tar.gz \
    && mkdir -p /usr/src/php/ext \
    && mv phpredis-$PHPREDIS_VERSION /usr/src/php/ext/redis \
    && docker-php-ext-install redis \
        pdo pdo_mysql \
        bcmath

#for gd
RUN apt-get update && apt-get install -y \
        libfreetype6-dev \
        libjpeg62-turbo-dev \
        libpng-dev \
    && docker-php-ext-install -j$(nproc) iconv \
    && docker-php-ext-configure gd --with-freetype-dir=/usr/include/ --with-jpeg-dir=/usr/include/ \
    && docker-php-ext-install -j$(nproc) gd

# for grpc extention
RUN  apt-get install -y \
        libmemcached-dev zlib1g-dev \
    && pecl install grpc \
    && pecl install protobuf \
    && rm -rf /usr/src/php

# for zip
RUN apt-get update \
    && apt-get install -y --no-install-recommends libzip-dev \
    && rm -r /var/lib/apt/lists/* \
    && docker-php-ext-install -j$(nproc) zip
~~~

- 定制化的php扩展，比如grpc扩展，你的项目不需要你可以注释掉，然后重新docker-compose build
- 安装php扩展之前，优先去看[官方](https://hub.docker.com/_/php)是否有提供，比如这样的docker-php-ext-install
- 如果docker php官方没有提供，那就需要自己去写dockerfile安装了，比如在./phpfpm/Dockerfile文件里面的rabbitmq，写出这些命令行的方法是，需要进去php容器，然后按照rabbitmq的安装方式一步一步去验证我的命令和依赖，最后才把命令总结出来写到dockerfile文件

- [简书有人整理的安装扩展](https://www.jianshu.com/p/20fcca06e27e)

## 命令参考

    # 你自己修改了docker-compose文件或Dockerfile文件的话，请执行
    docker-compose build
    
    # 开启 dnmp 服务（-d 后台运行）
    docker-compose up -d
    
    # 重启 dnmp 服务
    docker-compose restart
    
    # 关闭 dnmp 服务
    docker-compose down
    
    # 如果修改或增加了 nginx 服务配置
    docker restart nginx
 
 ## tip
 
   使用的都是官方的镜像，如有疑问，欢迎提出指正,
 
 ## contact me
 
 email: chichoyi@163.com
   
