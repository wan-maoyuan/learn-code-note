https://www.cnblogs.com/rslai/p/8249812.html


## centos安装redis
1. 检查是否有redis yum 源
```shell
yum install redis
```

2. 下载fedora的epel仓库
```shell
yum install epel-release
```

3. 安装redis数据库
```shell
yum install redis
```

4. 安装完毕后，使用下面的命令启动redis服务
```shell
# 启动redis
service redis start
systemctl start redis.service
# 停止redis
service redis stop
systemctl stop redis.service
# 查看redis运行状态
service redis status
# 查看redis进程
ps -ef | grep redis
```

5. 设置redis为开机自动启动
```shell
chkconfig redis on
```

6. 进入redis服务
```shell
# 进入本机redis
redis-cli
# 列出所有key
keys *
```

7. 防火墙开放相应端口
```shell
# 开启6379
/sbin/iptables -I INPUT -p tcp --dport 6379 -j ACCEPT
# 开启6380
/sbin/iptables -I INPUT -p tcp --dport 6380 -j ACCEPT
# 保存
/etc/rc.d/init.d/iptables save
# centos 7下执行
service iptables save
```

## 修改redis默认端口密码
1. 打开配置文件
```shell
vim /etc/redis.conf
```

2. 修改默认端口，查找 port 6379 修改为相应端口即可

3. 修改默认密码，查找 requirepass foobared 将 foobared 修改为你的密码

4. 使用配置文件启动 redis
```shell
redis-server /etc/redis.conf &
```

5. 使用端口登录
```shell
redis-cli -h 127.0.0.1 -p 6179
```
