## docker安装并启动mysql容器

1. 安装mysql镜像
```shell
docker pull mysql:5.7
```

2. 启动mysql镜像
```shell
docker run --name pwc-mysql -e MYSQL_ROOT_PASSWORD=123456 -p 3306:3306 -d mysql
```
- –name：给新创建的容器命名，此处命名为pwc-mysql
- -e：配置信息，此处配置mysql的root用户的登陆密码
- -p：端口映射，此处映射主机3306端口到容器pwc-mysql的3306端口
- -d：成功启动容器后输出容器的完整ID，例如上图 73f8811f669ee...
- 最后一个mysql指的是mysql镜像名字


3. mysql镜像中文乱码
- 现进入容器中
```shell
 docker exec -it mysql bash

 echo "character-set-server=utf8" >> /etc/mysql/mysql.conf.d/mysqld.cnf

 service mysql restart
```
