# phper构建开发环境

## php 构建容器

```sudo docker run -d -p 127.0.0.1:9001:9000 --name hostname  -w $(pwd) -v $(pwd):$(pwd) php-dev```

## MySQL 构建容器

```sudo docker run -itd -p 127.0.0.1:33060:3306 --name hostname --hostname=mysql-test -e MYSQL_ROOT_PASSWORD=test mysql```

## Nginx 构建容器



