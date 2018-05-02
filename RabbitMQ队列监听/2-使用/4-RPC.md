# 概述
本demo主要在模拟amqplib包中的rpc模式,又将原例子中的client端拆分成了web端和接口层
![img](../assets/images/six.png)
# web端
web端如果需要获取返回值,需要定义好需要监听的返回队列.(如果不需要,这部分省略)web端调用发送信息的接口,把需要传递的参数和返回队列的name发送到接口层,需要注意的是,返回队列的创建在接口调用之前.

## 参数
序号 | 参数类型 | 名称 | 是否必须 | 说明
--- | --- | --- | --- | ---
1 | string | back_queue | 是 | 返回队列名
2 | string | url | 是 | 接口地址
3 | string | prama | 是 | 接口业务参数

## 代码示例
```
var q = 'ListenUploadToJson'
var open = require('amqplib').connect('amqp://dpjia:dpjia@192.168.6.30:5672')
const request = require('request')

open.then(conn => {
  return conn.createChannel()
})
.then(ch => {
  console.log('===jian ting zhong')
  return ch.assertQueue(q, {
	  durable: false
  }).then(ok => {
    return ch.consume(q, msg => {
      if (msg !== null) {
        console.log(msg.content.toString())
        ch.ack(msg)
      }
    })
  })
}).catch(console.warn)

request({
	url: 'http://192.168.6.20/DPOpenAPI/api/1.0/functions/log/rpc_client?n=12&qu=' + q,
	headers: {
		'x-dp-key':'0c31e550cfdab86f2c2ea59327907798',
		'x-dp-id':'cfdab86f2c2ea593',
		'x-dp-token':'153715265c41f0634016be0916dc3ce6'
	}
}, (error, res, body) => {
	console.log(body)
})
//本例子在获取返回值这里具体应用时候需要做相应改变,通过判断correlation_id,来确定返回值是否具体为本次的返回if($res->get(curr_id) == correlation_id)
```

## 返回
![img](../assets/images/fanhui.png)

# 接口层
接口层获取到参数之后,把这些消息通过消息队列发送到server端执行,成功后返回具体id(channel标识)到web端,失败则返回具体错误

## 代码示例
```
<?php
defined('BASEPATH') OR exit('No direct script access allowed');
require_once APPPATH.'/models/1.0/common/Class_m.php';

use PhpAmqpLib\Connection\AMQPStreamConnection;
use PhpAmqpLib\Message\AMQPMessage;

class Rpc_client extends Class_m{

	private $connection;
    private $channel;
    private $callback_queue;
    private $response;
    private $corr_id;

	public function __construct() {
		parent::__construct();
		$this->connection = new AMQPStreamConnection('192.168.6.30','5672','dpjia','dpjia');
		$this->channel = $this->connection->channel();
		//queue_delcare参数 1:参数名,2,3:持久化,4:是否独占,5:
	}

	public function get_rpc_client($args){
		//DEMO SIX
		$n = $args['n'];
		$res = array();
		$this->callback_queue = $args['qu'];
		$this->response = null;
		$this->corr_id = uniqid();

		$msg = new AMQPMessage(
			strval($n),
			array(
				'correlation_id' => $this->corr_id,
				'reply_to' => $this->callback_queue
				)
			);
		$this->channel->basic_publish($msg,'','rpc_queue');
		
		$res['correlation_id'] = $this->corr_id;
		return array($res,'200');
	}
}
```

## 返回
```
{"correlation_id": "5966ea91d7d53"}
```

# 服务层
服务层需要consume监听发送的队列,在回调函数中执行耗时久的异步任务,执行完成之后,把返回结果,和通道id,publish到返回队列里,由于web端在接收着返回信息,返回队列里的消息会发送到web端.
服务层使用ci 的cli方式实现

## 代码示例
```
<?php
defined('BASEPATH') OR exit('No direct script access allowed');

use PhpAmqpLib\Connection\AMQPStreamConnection;
use PhpAmqpLib\Message\AMQPMessage;

class Rpcserver extends CI_Controller{
// php index.php rqserver/Rpcserver fbi_server
	public function __construct(){
		parent::__construct();
		if(! is_cli()) {
    		echo 'cannot exec'.PHP_EOL;
    		exit;
		}
		//server端可以有对db的操作
		$this->load->database();
	}

	public function fbi_server(){
		$connection = new AMQPStreamConnection('192.168.6.30','5672','dpjia','dpjia');
		$channel = $connection->channel();
		$channel->queue_declare('rpc_queue',false,false,false,false);
		echo "waitting request \n";
		$callback = function($req){
			$n = intval($req->body);
			$num = $this->fib($n);
			echo $num."\n";
			$msg = new AMQPMessage(
				strval($num),
				array('correlation_id' => $req->get('correlation_id'))
				);
			echo $req->get('reply_to');
			$req->delivery_info['channel']->basic_publish($msg,'',$req->get('reply_to'));
			$req->delivery_info['channel']->basic_ack($req->delivery_info['delivery_tag']);
		};
		$channel->basic_qos(null,1,null);
		$channel->basic_consume('rpc_queue','',false,false,false,false,$callback);

		while (count($channel->callbacks)) {
			$channel->wait();
		}
		$channel->close();
		$connection->close();
	}

	private function fib($n){
		//api
		// 接收到上传模型
		// 模型转换成json文件
		// 模型和json文件再上传到cdn
		// 返回模型的cdn地址和json的cdn地址
		if($n == 0){
			return 0;
		}elseif ($n == 1) {
			return 1;
		}else{
			return $this->fib($n-1) + $this->fib($n-2);
		}
	}
}
```

## 执行命令 
php index.php rqserver/Rpcserver fbi_server