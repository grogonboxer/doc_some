# 概述
	MQ全称为Message Queue, 消息队列（MQ）是一种应用程序对应用程序的通信方法。应用程序通过读写出入队列的消息（针对应用程序的数据）来通信，而无需专用连接来链接它们。消息传递指的是程序之间通过在消息中发送数据进行通信，而不是通过直接调用彼此来通信，直接调用通常是用于诸如远程过程调用的技术。排队指的是应用程序通过 队列来通信。队列的使用除去了接收和发送应用程序同时执行的要求。其中较为成熟的MQ产品有IBM WEBSPHERE MQ等等。
# 安装Erlang

## 1.安装基础包:gcc ncurses-devel
```yum install gcc```
```yum install ncurses-devel```
## 2.去官网下载对应版本的Erlang
(1)编译安装erlang
(2)rpm包安装
由于本次安装erlang是为了支持rabbitmq的安装,并非为了Erlang的开发使用
所以选择rpm包安装
下载地址:http://www.rabbitmq.com/releases/erlang/ 根据环境不同选择不同版本
本机选用版本: erlang-19.0.4-1.el6.x86_64.rpm
## 3.安装Erlang
```rpm -ivh erlang-19.0.4-1.el6.x86_64.rpm```
## 4.安装完后输入“erl”以下提示即为安装成功：
```
[root@localhost ~]# erl
Erlang/OTP 19 [erts-8.0.3] [source] [64-bit] [async-threads:10] [hipe] [kernel-poll:false]
Eshell V8.0.3  (abort with ^G)
1> 
```
# 安装RabbitMQ
安装 RabbitMQ必须在安装Erlang之后进行,如果Erlang没有安装,请查看Linux安装Erlang
## 1.下载RabbitMQ
下载地址:http://www.rabbitmq.com/releases/
## 2.导入key文件 
```rpm --import http://www.rabbitmq.com/rabbitmq-signing-key-public.asc```
## 3.安装RabbitMQ
```yum install -y rabbitmq-server-3.6.10-1.el6.noarch.rpm```
## 4.添加远程访问用户，并分配权限
1）添加用户
```rabbitmqctl add_user dpjia ```
2）配置用户组
```rabbitmqctl set_user_tags administrator```
3）分配权限
```rabbitmqctl set_permissions -p / dpjia '.*' '.*' '.*'```
## 5.开启可视化插件
```rabbitmq-plugins enable rabbitmq_management```
## 6.启动RabbitMQ
启动RabbitMQ服务命令为: rabbitmq-server start (用户关闭连接后,自动结束进程)
后台运行RabbitMQ服务命令为: rabbitmq-server
/etc/init.d/rabbitmq-server start
关闭服务命令为: /etc/init.d/rabbitmq-server stop

## 测试:远程访问地址:
http://192.168.6.30:15672
用户名:dpjia 密码:dpjia 测试是否能够登录
登录标识 RabbitMQ安装成功

