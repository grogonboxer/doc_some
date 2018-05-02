## 缓存使用说明
	目前缓存的相关文件都在 models/1.0/cache 目录下。
	具体使用文件在：cache文件下的  ensure 方法 。
	
### 缓存调取
```{
	$this->load->model('1.0/cache/Cache','cache');
	$data = $this->cache->ensure($cache_key, 
			function($params){
				//具体获取数据的方法	
			},
		$expire ,$params);
}

```
### 请求参数
| 参数名     | 说明    | 是否必须    | 默认值 |
| ----------|-----|:-------------:| -----:|
| params    | 获取数据时需要的参数 | 是  | 无|
| expire    | 过期时间 | 是  | 3天|
| cache_key    | 缓存的唯一表示key | 是  | 无 |


### cache_key的命名方式
	 用户类型、用户ID、具体功能业务的key值，用下划线拼接





	
	