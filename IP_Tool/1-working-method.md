# 1. 工作原理

阅读本文需要一定的Python爬虫基础知识。

### 0x00 基本原理

首先新建一个易班轻应用，观察其数据变化。

（假装有图）

通过切换网络和热点可以发现易班轻应用对于独立访客这一项指标的统计标准就是使用不同的IP进行访问，便可累计独立访客量。那么我们只需要使用不同的IP对轻应用页面进行请求即可。

通过小学二年级的计算机知识可以知道，我们可以import `requests` 库使用参数 `proxies` 引入代理IP，对轻应用Url进行get请求便可实现提升独立访客的效果。

### 0x01 IP代理获取

打开摆渡进行搜索，随便找一个代理池，本项目使用的是芝麻代理（以下简称芝麻），读者可根据实际需求对代码中的代理池API进行替换。（具体配置见[#i-zhu-ce-dai-li-chi](../2-using.md#i-zhu-ce-dai-li-chi "mention")）

本项目内置的API的返回信息格式为Json，结构如下：

```json
{
	"code": 0,
	"data": [
		{
			"ip": "122.241.31.86",
			"port": 4231
		},
		...
	],
	"msg": "0",
	"success": true
}
```

具体Python代码如下：

```python
def check_zm():
	IP_API = 'http://webapi.http.zhimacangku.com/getip?num=5&type=2&pro=&city=0&yys=0&port=1&pack=234772&ts=0&ys=0&cs=0&lb=4&sb=0&pb=4&mr=1&regions='
	res = json.loads(requests.get(IP_API).text) # 处理返回信息为Json
	code = res['code'] 
	if code == 0: #状态码为0则输出IP，反之则输出错误信息
		return 1, res['data']
	return 0, res['msg']
```

通过上方的函数我们可以实现对API连通性的测试，由于学校的宽带每天都会轮换IP，所以增加了错误信息返回便于调试。

然后我们对返回的IP代理进行简单的处理，将其处理为可直接供给requests模块使用的`{'method':'ip:port'}`格式，编写一个简单的函数。

```python
def Deal_IP():
	flag, data = check_zm() # 检查返回值是否包含IP
	IPS = []
	if flag:
		for ips in data: # 遍历返回数据进行处理
			proxymeta = str(ips["ip"]) + ':' + str(ips["port"])
			IP = {"http": proxymeta} # 拼接IP并添加到字典中 
			IPS.append(IP) 
		return IPS # 返回字典​
```

通过以上两个函数可以实现对代理池连接性检验，并且获取IP代理。

### 0x02 访问轻应用Url

易班轻应用Url形如：`https://q.yiban.cn/app/index/appid/1234`，那么可以通过修改其后缀的Appid即可拼接得到轻应用Url。

```python
def Run(App_id, IP):
	url = 'https://q.yiban.cn/app/index/appid/' + App_id
	try:
		res1 = requests.get(url, proxies=IP, headers={'UA': xxx)
		return 1
	except requests.exceptions.SSLError:  # 易班接口出错
		return 0
```

通过一个简单的get方法请求轻应用Url即可提高轻应用独立访客量。

但需注意的是，易班对于短时间内大量访问会有防火墙保护，过于频繁的访问会触发其**DDos防火墙**，目前观测到的效果是从触发起到当日24时的全部访问均**不计入独立访客**。且正常情况下切换代理访问易班也有几率不被记录访客，实测一秒请求一次效果最佳，成功率为80%（假设获取的IP均可用的情况下，实际操作成功率为75%\~60%）。

### 0x03 主函数实现

本文档仅简述脚本工作原理，关于可视化版本的多线程实现不在本文档讨论范围中。

