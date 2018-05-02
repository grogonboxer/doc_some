# php-amqplib官方文档
url:http://www.rabbitmq.com/tutorials/tutorial-one-php.html
#测试demo:
url:
http://**.**.com/openapi/1.0/function/log/*

# 附上一个基本的例子:
## TaskWithEx.php
```
public function get_taskWithEx(){
	//DEMO THREE
	$connection = new AMQPStreamConnection('192.168.6.30','5672','dpjia','dpjia');
	$channel = $connection->channel();

	$channel->exchange_declare('logs_dpj','fanout',false,false,false);
//没有queue绑定的exchange 会丢失,但如果有receive正在监听,消息还是可以被获取到<不建议>
	//$channel->queue_declare('task_qu',false,true,false,false);

	$data = 'abc bcd cde def efg';

	$msg = new AMQPMessage($data,
				array('delivery_mode' => AMQPMessage::DELIVERY_MODE_PERSISTENT)
			);
	$channel->basic_publish($msg,  'logs_dpj');

	echo 'send'.$data."\n";
	
	$channel->close();
	$connection->close();
	}
```
## WorkerWithEx.php
```
public function get_workerWithEx(){
	//DEMO TWO
	$connection = new AMQPStreamConnection('192.168.6.30','5672','dpjia','dpjia');
	$channel = $connection->channel();

	$channel->exchange_declare('logs_dpj', 'fanout', false, false, false);

	list($queue_name, ,) = $channel->queue_declare("", false, false, true, false);

	$channel->queue_bind($queue_name, 'logs_dpj');

	echo ' [*] Waiting for logs. To exit press CTRL+C'."\n";

	$callback = function($msg){
	  echo $msg->body."\n";
	};

	$channel->basic_consume($queue_name, '', false, true, false, false, $callback);

	// while(count($channel->callbacks)) {
	//     $channel->wait();
	// }

	$channel->close();
	$connection->close();
	}
```