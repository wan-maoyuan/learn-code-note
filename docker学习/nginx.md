## docker安装并启动nginx容器

1. 下载nginx镜像
```shell
docker pull nginx
```

2. 启动nginx镜像
```shell
docker run  -itd  -p 81:80  -v D:/data:/data --name="nginx"  nginx
```

3. 进入nginx容器中
```shell
docker exec -it nginx bash
```

> nginx.conf配置文件在 /etc/nginx/config.d下面