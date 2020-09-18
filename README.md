# phper构建开发环境

## php 构建容器

```sudo docker run -d -p 9001:9000 --name hostname  -w $(pwd) -v $(pwd):$(pwd) php-dev```

## MySQL 构建容器

```sudo docker run -itd -p 33061:3306 -v $(pwd)/database/mysql/conf:/etc/mysql/conf.d -v $(pwd)/logs/mysql:/var/log/mysql -v $(pwd)/data:/data --name mysqltest --hostname=mysql-test2 -e MYSQL_ROOT_PASSWORD=test mysql-test```

## Nginx 构建容器
```sudo docker run -it -d nginx-test --name nginx-test nginx```

## 关于容器的环境变量

+ php配置文件目录：　```/usr/local/etc/php```
+ nginx 配置文件目录： ```/etc/nginx```


## docker-compose

``` cd php-dev && docker-compose up -d --build ```

## 应用目录

```
├── apps           # 应用代码库
├── cache          # 缓存数据库服务
│   ├── memcached
│   └── redis
├── database       # 数据库服务
│   ├── mongodb
│   ├── mysql
│   ├── postgresql
│   └── redis
├── logs            # 日志服务
│   ├── memcached
│   ├── mysql
│   ├── nginx
│   ├── php-fpm
│   ├── postgresql
│   └── redis
├── nginx           # nginx服务
│   ├── conf.d
│   └── Dockerfile
├── php-fpm         # php-fpm服务
│   ├── conf
│   └── Dockerfile
└── README.md
```



## 一些问题

#### php与nginx需要一个环境里吗，主要是PHP脚本是不是要在同一个容器里

不需要，已经测试，只要保证fast_cgi能够找到对应的脚本就行，

需要保证php容器脚本路径跟nginx里路径一样

测试：

1. 新建一个index.php,
2. 让nginx,php的工作目录都包含他，nginx, php分别在两个容器内

#### 创建容器使用ipv6，而host未开启抓发ipv6,可能导致host无法转发

+ net.ipv4.ip_forward=1
+ net.ipv6.conf.all.forwarding=1

#### 通过-v挂载到容器里的目录权限似乎跟宿主机器的目录权限有区别？待测试

-v的方式默认是read-write方式

#### 默认创建容器使用的是brige

使用默认brige的容器可以互联

#### php-fpm 机器连通mysql, 需要一个授权信息，保证能够链接到内部网络

#### 是否可以将三个容器整合到一起，只暴露80端口，其他端口都不暴露？这样子操作是否会更好些？