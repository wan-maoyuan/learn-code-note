## docker安装并启动nacos

1. 下载nacos镜像
```shell
docker pull nacos/nacos-server
```

2. 启动nacos镜像
```shell
docker run -d -p 8848:8848 -e MODE=standalone --restart always --name nacos nacos/nacos-server
```