# 基本概念
	Broker：简单来说就是消息队列服务器实体。
　　Exchange：消息交换机，它指定消息按什么规则，路由到哪个队列。
　　Queue：消息队列载体，每个消息都会被投入到一个或多个队列。
　　Binding：绑定，它的作用就是把exchange和queue按照路由规则绑定起来。
　　Routing Key：路由关键字，exchange根据这个关键字进行消息投递。
　　vhost：虚拟主机，一个broker里可以开设多个vhost，用作不同用户的权限分离。
　　producer：消息生产者，就是投递消息的程序。
　　consumer：消息消费者，就是接受消息的程序。
　　channel：消息通道，在客户端的每个连接里，可建立多个channel，每个channel代表一个会话任务。
	消息持久
	1) 将交换机置为可持久；
	2) 将通道置为可持久
	3) 消息发送时设置可持久。
	当我们“生产”了一条可持久化的消息，尝试中断MQ服务，启动消费者获取消息，消息依然能够恢复。相反，则抛出异常。
# 使用过程
	消息队列的使用过程大概如下：
	（1）客户端连接到消息队列服务器，打开一个channel。
　　（2）客户端声明一个exchange，并设置相关属性。
　　（3）客户端声明一个queue，并设置相关属性。
　　（4）客户端使用routing key，在exchange和queue之间建立好绑定关系。
　　（5）客户端投递消息到exchange。
exchange接收到消息后，就根据消息的key和已经设置的binding，进行消息路由，将消息投递到一个或多个队列里。
exchange也有几个类型，完全根据key进行投递的叫做Direct交换机，例如，绑定时设置了routing key为”abc”，那么客户端提交的消息，只有设置了key为”abc”的才会投递到队列。对key进行模式匹配后进行投递的叫做Topic交换机，符号”#”匹配一个或多个词，符号”*”匹配正好一个词。例如”abc.#”匹配”abc.def.ghi”，”abc.*”只匹配”abc.def”。还有一种不需要key的，叫做Fanout交换机，它采取广播模式，一个消息进来时，投递到与该交换机绑定的所有队列。
RabbitMQ支持消息的持久化，也就是数据写在磁盘上，为了数据安全考虑，我想大多数用户都会选择持久化。消息队列持久化包括3个部分：
　　（1）exchange持久化，在声明时指定durable => 1
　　（2）queue持久化，在声明时指定durable => 1
　　（3）消息持久化，在投递时指定delivery_mode => 2（1是非持久化）
如果exchange和queue都是持久化的，那么它们之间的binding也是持久化的。如果exchange和queue两者之间有一个持久化，一个非持久化，就不允许建立绑定。
# 基本例子
## amqp_task.php
```
<?php
$conn_args = array(
    'host'=> '192.168.6.30',
    'port'=>'5672',
    'login'=>'dpjia',
    'password'=>'dpjia',
    'vhost'=>'/'
);
//创建连接
$conn = new AMQPConnection($conn_args);
if($conn->connect()){
    echo "连接成功\n";
}else{
    die("连接失败\n");
}
//创建channel
$channel = new AMQPChannel($conn);
//创建exchange交换机
$ex_name = 'ex_dpjia';
$ex = new AMQPExchange($channel);
$ex->setName($ex_name);
$ex->setType(AMQP_EX_TYPE_DIRECT);
$ex->setFlags(AMQP_DURABLE);
echo "Exchange status:".$ex->declareExchange()."\n";
$message = json_encode(array('Hello World3!','php3','c++3:'));

$routing_key = '';
for($i=0;$i<10;$i++){
    if($routing_key == 'key'){
    	$routing_key = 'key2';
    }else{
        $routing_key = 'key';
    }
    $ex->publish($message,$routing_key);
}

$qu_name = 'qu_dpjia3';
$qu = new AMQPQueue($channel);
$qu->setName($qu_name);
$qu->setFlags(AMQP_DURABLE);
echo 'queue status:'.$qu->declare()."\n";

echo 'queue bind: '.$qu->bind($ex_name,'route.key')."\n";

$channel->startTransaction();
echo "send: ".$ex->publish($message, 'route.key'); //将你的消息通过制定routingKey发送
$channel->commitTransaction();
$conn->disconnect();
//发送消息
//echo "Send Message:".$ex->publish("TEST MESSAGE，key_1 by xust" . date('H:i:s', time()), 'key_1')."\n";
//echo "Send Message:".$ex->publish("TEST MESSAGE，key_2 by xust" . date('H:i:s', time()), 'key_2')."\n";
?>
```

## amqp_worker.php
```
<?php
$conn_args = array(
        'host'=> '192.168.6.30',
        'port'=>'5672',
        'login'=>'dpjia',
        'password'=>'dpjia',
        'vhost'=>'/'
);
//创建连接
$conn = new AMQPConnection($conn_args);
if($conn->connect()){
        echo "连接成功\n";
}else{
        die("连接失败\n");
}
$ex_name = 'ex_dpjia';
$qu_name = 'qu_dpjia';

$channel = new AMQPChannel($conn);
/*
$ex = new AMQPExchange($channel);
$ex->setName($ex_name);
$ex->setType(AMQP_EX_TYPE_DIRECT);
$ex->setFlags(AMQP_DURABLE);
echo "Exchange status:".$ex->declare()."\n";
*/
//创建队列 queue

$qu = new AMQPQueue($channel);
$qu->setName($qu_name);
$qu->setFlags(AMQP_DURABLE);
echo $qu->declare();
//绑定交换机和队列
$qu->bind($ex_name,'roukey2');

//阻塞模式接收消息
//echo "Message:\n";
//$qu->consume('processMessage',AMQP_AUTOACK); //自动ACK应答
$msg = $qu->get(AMQP_AUTOACK);
if($msg){
        var_dump($msg->getBody());
}

$conn->disconnect();

//消息回调函数 processMessage

function processMessage($envelope,$queue){
        var_dump($envelope->getRoutingKey);
        $msg = $envelope->getBody();
        echo $msg."\n";
}

?>
```
