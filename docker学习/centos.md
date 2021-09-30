## windows 10平台

- 下载centos
```shell
docker pull centos:7.5.1804
```

- 启动centos容器
```shell
docker run  -itd  -p 81:80 -p 8888:8888 --privileged=true -v D:/data:/data --name="centos"  cf49811e3cdb  /usr/sbin/init
```

- 进入centos容器中
```shell
docker exec -it centos bash
```

- 退出容器
```shell
exit
```