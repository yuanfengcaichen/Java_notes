## Deepin中安装

1. 切换到root用户
2. 执行apt install rabbitmq-server
3. 安装gedit
4. 修改~/.bashrc文件
5. 启动RabbitMQ

### 切换到root用户

```cmd
su root
//如果没有root用户，先设置root用户密码
sudo passwd root

//之后再执行su root
```

### 执行安装命令

```cmd
apt install rabbitmq-server
```

### 安装gedit

```cmd
apt install gedit
```

### 修改~/.bashrc文件

```cmd
gedit ~/.bashrc
```

打开文件后，在文件的最下方添加：

```txt
export RABBITMQ_HOME="$PATH:/usr/lib/rabbitmq/lib/rabbitmq_server-3.7.8"
```

输入命令让配置生效：source ~/.bashrc

### 启动RabbitMQ

1. 输入命令 rabbitmq-server

2. 执行下列命令停止或启动服务

   rabbitmqctl stop
   rabbitmqctl start

### 开启web管理插件

```cmd
rabbitmq-plugins enable rabbitmq_management
```

默认账户和密码都是guest

使用浏览器登录：[http://localhost:15672](http://localhost:15672/)