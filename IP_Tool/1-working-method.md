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

```
{
	"code": 0,
	"data": [
		{
			"ip": "122.241.31.86",
			"port": 4231
		},
		{
			"ip": "183.165.227.0",
			"port": 4258
		},
		{
			"ip": "1.49.226.71",
			"port": 4267
		},
		{
			"ip": "114.99.11.136",
			"port": 4225
		},
		{
			"ip": "221.10.104.76",
			"port": 4231
		}
	],
	"msg": "0",
	"success": true
}
```

具体Python代码如下：

```
def check_zm():
	IP_API = 'http://webapi.http.zhimacangku.com/getip?num=5&type=2&pro=&city=0&yys=0&port=1&pack=234772&ts=0&ys=0&cs=0&lb=4&sb=0&pb=4&mr=1&regions='
	res = json.loads(requests.get(IP_API).text) # 将
	code = res['code'] 
	if code == 0:
		return 1, res['data']
	return 0, res['msg']
```
