## 使用docker运行sqlserver

- 先去docker仓库拉去sqlserver镜像
```shell
sudo docker pull mcr.microsoft.com/mssql/server:2019-CTP3.2-ubuntu
```

- 使用docker run 命令启动容器，其中要注意的是使用-v参数指定了sqlserver2019_data目录挂载到容器的/var/opt/mssql目录，这个目录是用来存储数据库文件的，所以最好挂载到外容器外部，避免因为不小心删除容器而丢失数据
```shell
sudo mkdir /hd2/sqlserver2019_data
```

```shell
sudo docker run -e "ACCEPT_EULA=Y" -e "SA_PASSWORD=dev@123," -p 14330:1433 --name sqlserver2019 -v /hd2/sqlserver2019_data:/var/opt/mssql  -d mcr.microsoft.com/mssql/server:2019-CTP3.2-ubuntu
```

- 使用docker ps 命令查看容器运行情况，可以看到sqlserver2019正在运行
```shell
sudo docker ps -a
```

- 进入sqlserver容器内部
```shell
sudo docker exec -it sqlserver2019 "bash"
```

- 命令行登录sqlserver
```shell
/opt/mssql-tools/bin/sqlcmd -S localhost -U SA -P "dev@123,"
```

