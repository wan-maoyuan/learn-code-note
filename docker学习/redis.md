## docker 安装并启动redis

1. 下载redis镜像
```shell
docker pull redis 
```

2. 启动redis镜像，设置端口号，密码
```shell
docker run -d --name redis -p 6379:6379 --restart=always  redis  --requirepass "TCsoft123.."
```

3. docker容器设置开机自启动
```shell
--restart具体参数值详细信息
no - 容器退出时，不重启容器
on-failure - 只有在非0状态退出时才从新启动容器
always - 无论退出状态是如何，都重启容器
```

4. 如果创建时未指定 --restart=always ,可通过 update 命令更改
```shell
docker update --restart=always 容器ID
```

5. 进入redis容器中
```shell
docker exec -it redis redis-cli -a 'TCsoft123..'
```