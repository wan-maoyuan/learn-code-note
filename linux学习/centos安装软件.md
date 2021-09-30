## 安装java8

1. 查看java版本
```shell
yum search java | grep jdk
```

2. 安装java8
```shell
yum install java-1.8.0-openjdk java-1.8.0-openjdk-devel
```

3. 查看java8安装位置
```shell
ll /user/lib/jvm
```

4. 设置环境变量
```shell
vi /etc/profile


#set java environment
JAVA_HOME=/user/lib/jvm/java-1.8.0-openjdk-1.8.0.191.b12-1.el7_6.x86_64
JRE_HOME=$JAVA_HOME/jre
CLASS_PATH=.:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar:$JRE_HOME/lib
PATH=$PATH:$JAVA_HOME/bin:$JRE_HOME/bin
export JAVA_HOME JRE_HOME CLASS_PATH PATH
```

5. 刷新配置，使其生效
```shell
source /etc/profile
```

6. 查看java版本
```shell
java -version
```

## 安装nginx

> centos7系统库中默认是没有nginx的rpm包的，所以我们自己需要先更新下rpm依赖库

1. 使用yum安装nginx需要包括Nginx的库，安装Nginx的库
```shell
rpm -Uvh http://nginx.org/packages/centos/7/noarch/RPMS/nginx-release-centos-7-0.el7.ngx.noarch.rpm
```

2. 使用下面命令安装nginx
```shell
yum install nginx
```

> 配置文件在 /etc/nginx/conf.d/ 

3. 启用nginx，开机自启动
```shell
systemctl enable nginx
```

4. 启动nginx
```shell
systemctl start nginx
```

5. 查看nginx状态
```shell
systemctl status nginx
```

6. 停止nginx
```shell
systemctl stop nginx
```

7. 重新加载nginx
```shell
systemctl reload nginx
```

## 安装redis
