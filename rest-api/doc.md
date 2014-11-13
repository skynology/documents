# 上空云 RESTFUL API 文档

<br>
## API调用地址及格式
Restful API url 格式为: [https://api.skynology.com/1.0/login](https://api.skynology.com/1.0/login) . 

其中 `https://api.skynology.com`为跟域名,  ` 1.0 ` 为当前API版本号, ` login ` 为具体功能地址. 详细功能参考 [API列表](#api_list)

<br>
## 注意事项
* 所有API调用数据格式均为 `JSON`. 需设置http头 `Content-Type : application/json`
* 所有API调用需设置http头 `X-SKCloud-Application-Id : <项目对应的applicationId>`
* 所有API调用需要传入签名, 设置http头 ` X-SKCloud-Request-Sign : <签名内容> `, 详情参考 [签名相关文档](#signature)
* 当用户登陆后, 需传入session字段, 以便识别. 设置http头 ` X-SKCloud-Session-Token : <session 内容>`

<br>
<a name="api_list"></a>
## API列表

<a name="api_list_resource"></a>
### [资源操作相关](#resource)
URL|Method|Description
---|------|-----------
resources/\<resourceName>|POST|创建对象
resources/\<resourceName>|GET|查询对象
resources/\<resourceName>/\<objectId>|GET|获取对象
resources/\<resourceName>/\<objectId>|PUT|更新对象
resources/\<resourceName>/\<objectId>|DELETE|删除对象

<a name="api_list_user"></a>
### [用户相关](#user)
URL|Method|Description
---|------|-----------
users|POST|注册用户
users/\<objectId>|GET|获取用户
users/\<objectId>|PUT|更新用户
users/\<objectId>/resetPassword|PUT|修改登录密码
users/\<objectId>|DELETE|删除用户
login|POST|登陆账号

<a name="api_list_role"></a>
### [角色相关](#role)
URL|Method|Description
---|:------:|-----------
roles|POST|创建角色
roles/\<objectId>|GET|查询角色
roles/\<objectId>|GET|获取角色
roles/\<objectId>|PUT|更新角色
roles/\<objectId>|DELETE|删除角色

<a name="api_list_file"></a>
### [文件相关](#file)
URL|Method|Description
---|------|-----------
files/fetchFromURL|POST|发送邮件

<a name="api_list_email"></a>
### [邮件相关](#email)
URL|Method|Description
---|------|-----------
email/\<objectId>|POST|发送邮件

<a name="api_list_statistics"></a>
### [统计数据相关](#statistics)
URL|Method|Description
---|------|-----------
statistics/api|GET|查询API调用统计信息

<br>
<a name="signature"></a>
## 签名

所有的API调用都需要传入http头 签名信息, 也就是 ` X-SKCloud-Request-Sign `, 其值的格式为 ` timestamp,sign[,master] ` .

其中:

* timestamp（必须） - 客户端产生本次请求的 unix 时间戳，精确到毫秒.
* sign（必须）- 将 timestamp 加上 app key(或者 master key) 组成的字符串做 MD5 签名。
* master （可选）- 字符串 "master"，当使用 master key 签名请求的时候，须加上这个后缀, 以便我们分辨您是用 master key 来做签名的.

<br>
** 假设您有一项目信息如下: **

* ApplicationId  : `5451f38f9d40a887a1000005`
* ApplicatoinKey : `b1e47980eceec514d65912c93ae2097c016b59e8`
* MasterKey      : `b363cbfee6e08383c23ddfaf8169bf6ef0d594a2`


那么我们用ApplicationKey来做签名, `X-SKCloud-Request-Sign:1415856734899,2bd0e59e0e8ad68fa0491e7db7a90756`, 其中:

* 1415856734899: 为当前UNIX时间戳.
* 2bd0e59e0e8ad68fa0491e7db7a90756: `为时间戳+ApplicationKey`的MD5值, 也就是对(1415856734899b1e47980eceec514d65912c93ae2097c016b59e8)加MD5值.


如果用MasterKey来做签名, 那么签名值的Http header就为: `X-SKCloud-Request-Sign:1415856734899,c4e3b8c3a43ad0bd49d7c2fff5f19960,master`. 其中:

* 1415856734899: 为当前UNIX时间戳.
* c4e3b8c3a43ad0bd49d7c2fff5f19960: `为时间戳+MasterKey`的做MD5值, 也就是对(1415856734899b363cbfee6e08383c23ddfaf8169bf6ef0d594a2)加MD5值.
* master: 为我们后台分辨您是用master key来做签名的标记.

> 注意: UNIX时间戳最好为您调用API时的时间, 如果此值若是 **10分种** 以前的时间, 后台将拒绝本次请求.

### 相应格式
所用API请求, 若成功, 将会返回一个JSON对象.
成功与否将按HTTP返回的状态码来判断, 状态码为`2XX`时, 表明本次请求成功.

若返回的状态为`4XX`时, 代表您的请求失败. 此时会返回一个表示错误的JSON对象.
如下格式:

```
{
	"code": 202,
	"error": "用户名已经被占用",
	"error_en": "Username has already been taken"
}
```

更多错误代码, 可参考: [错误代码列表](https://github.com/skcloud/api-errors)


<br>
<a name="resource"></a>
## 资源操作

假设您在后台创建了一个 Resource, 用于存放活动相关数据 资源名为: `event`.



<a name="resource_post"></a>
### 创建对象

当创建对象时, 发送一个 `POST` 请求到资源对应的URL(`https://api.skynology.com/1.0/event`)即可, 内容为相对应的JSON数据. 如:

```
{
	"subject": "周六线下活动",
	"description": "上空云组织线下活动~~~",
	"startTime": "2015-01-04T14:00:00:901Z",
	"endTime": "2015-01-04T18:00:00:805Z",
	"joinUserCount": 0,
	"active":true,
	"ticketPrice": [30, 100, 150]
}
```
如果创建成功, HTTP返回状态为 `201`, 内容为含 `objectId` 和 `createdAt`的JSON数据.如:

```
{
	"objectId": "5458e2c39d40a827f9000001",
	"createdAt": "2015-01-04T09:00:00:001Z"
}
```

另在HTTP header中返回一个Location的字段, 内容为刚创建对象的HTTP地址, 如:

```
Location: https://api.skynology.com/1.0/resources/event/5458e2c39d40a827f9000001
```

<a name="resource_put"></a>
### 更新对象
需要更新对象数据时, 可发送一个PUT请求到对象对应的URL上, 如: `https://api.skynology.com/1.0/resources/event/5458e2c39d40a827f9000001`.

假设要更新上面 `event` 对象的 active为false. 直接PUT相关的数据.

```
{
	"active": false
}
```
请求成功时, 将会返回一个JSON对象, 包含`objectId`及`updatedAt`字段.

```
{
	"objectId": "5458e2c39d40a827f9000001",
	"updatedAt": "2015-01-04T10:08:21:001Z"
}
```
<a name="resource_put_increment"></a>
#### 计数器
可对任何数字字段做增加(Inrement) 或 减少(Decrement) 操作.
如我们想更新上面活动的报名人数, 可按如下格式PUT数据即可.

```
{
	"joinUserCount":{"__op":"Increment", "amount":1}
}
```
其中:

* "__op": "Increment" 为固定需要.
* "amont"为递增/递减的数值. 也可是其他数值, 如: -1, 可递减1位.

<a name="resource_put_array"></a>
#### 数组
为方便操作数组类型字段, 可用下面几种操作来更新.

* Add: 增加一个值到指定的数组字段, 可重复.
* AddUnique: 增加一个值到数组内, 只有此值不存在于原数组内, 才会添加到数组内.
* Remove: 从数组内删除指定的值.

若我们想给上面的活动的增加两个VIP门票价格, 可PUT一个如下JSON.

```
{
	"ticketPrice": {"__op":"AddUnique", "objects":[500, 1000]}
}
```
其中:

* "__op" 为操作类型, "AddUnique" 代表操作类型, 我们也可传"Add"或"Remove".
* "objects" 为需要增加/删除的值, 需数组类型. 比如上面我们增加两个VIP票 500, 1000时, 就以数组格式传入.

<a name="resource_put_relation"></a>
#### 关系
关系(Relation)类型其实是一个 `objectId`数组, 所以相关操作参考上面数组即可.

<a name="resource_delete"></a>
### 删除对象
删除一个对象时, 发送一个DELETE请求到对象对应的URL上即可. 如删除上面的活动对象, 可发送DELETE请求到
 `https://api.skynology.com/1.0/resources/event/5458e2c39d40a827f9000001` 上即可.

删除成功后会返回一个JSON对象. 包括 "objectId" 及 "deletedAt" 字段.



<br>
<a name="query"></a>
## 查询
查询一个 resource 时可发送一个GET请求到URL即可, 如对上面活动做查询时, 可发送GET请求到 `https://api.skynology.com/1.0/resources/event `

<a name="query_where"></a>
### 查询条件
查询时可传一个编码过的JSON字符串到, 传到URL的where参数上. where参数兼容 MongoDB 查询参数.

通常查询匹配时可直接传对应的键值即可. 如我们要查询所有开放中的活动, 可传以下URL.

`https://api.skynology.com/1.0/resources/event?where={active:true}`

查询成功后将返回一个JSON对象, 其中 `results` 字段将包含查询结果, 具体格式如下:

```
{
	"results": [
		{
			"subject": "周六线下活动",
			"description": "上空云组织线下活动~~~",
			"startTime": "2015-01-06T14:00:00:901Z",
			"endTime": "2015-01-06T18:00:00:805Z",
			"joinUserCount": 0,
			"active":true,
			isFree: false,
			"ticketPrice": [30, 100, 150]
		},
		{
			"subject": "锻炼身体, 从我做起 -> 1/12日爬梧桐山",
			"description": "爬大梧桐山",
			"startTime": "2015-01-12T09:00:00:001Z",
			"endTime": "2015-01-12T18:00:00:805Z",
			"joinUserCount": 0,
			"active":true,
			""isFree:true",
			"ticketPrice": []
		},
		....
	],
	"count": 405
}
```
<br>

除了直接匹配, 也可按下面列表中的关键字来做比较等查询.

关键字|描述
-----|---
$lt| 小于
$lte|小于等于
$gt|大于
$gte|大于等于
$ne|不等于
$in|包含
$nin|不包含
$exists|这个字段有值
$regex|正则匹配, 用于字符串字段.
$all|包含所有给定的值

** 下面一些查询列子 **:

查询报名人数大于等于30人的活动

```
https://api.skynology.com/1.0/resources/event?where={joinUserCount:{"$gte":30}}
```

查询举行时间在 `2015-01-01` 至 `2015-01-07` 其间的活动.

```
https://api.skynology.com/1.0/resources/event?where={startTime:{"$gte":"2015-01-01T00:00:01:001Z"},endTime:{"$gte":"2015-01-07T23:59:59:999Z"}}
```
<a name="query_order"></a>
### 排序
可在查询URL上增加 `order=字段名` 来做排序, 若需多个字段来排序, 用逗号`,`来分割字段名即可. 字段名前增加负号 `-` 可倒序. 如:

查询报名人数大于20人, 并用开始时间由近到远排序, 可写为如下:

```
https://api.skynology.com/1.0/resources/event?where={joinUserCount:{"$gte":20}}&order=startTime
``` 
若按远到近排序, 可写为:


```
https://api.skynology.com/1.0/resources/event?where={joinUserCount:{"$gte":20}}&order=-startTime
``` 
若同一天有多个活动, 再按人数多到少排序:


```
https://api.skynology.com/1.0/resources/event?where={joinUserCount:{"$gte":20}}&order=-startTime,-joinUserCount
``` 

<a name="query_select"></a>
### 获取指定字段

若您不想获取所有字段, 只想要指定的字段, 可传入 `select` 字段即可. 如下面查询将只返回活动标题及描述. 其他字段将不返回.

```
https://api.skynology.com/1.0/resources/event?select=name,description
``` 
<a name="query_skip"></a>
### 分页
可用 `skip` 及 `take` 来做分页操作, 其中 `take` 取值范围需在 1 ~ 1000之内. 

如每获取30条活动记录, 可:

```
https://api.skynology.com/1.0/resources/event?take=30
``` 
获取第三页的30条记录:

```
https://api.skynology.com/1.0/resources/event?skip=60&take=30
``` 

<a name="query_count"></a>
### 对象总数
当我们用 `skip` 及 `take` 做分页查询时, 通常也需要所查询对象的总匹配数. 我们只需要查询时传入 `count=1` 参数即可.

```
https://api.skynology.com/1.0/resources/event?skip=60&take=30&count=1
``` 
查询成功后在返回的JSON对象中的 `count` 字段值为总匹配数量.

```
{
	"results":[
		....
	], 
	"count": 403
}
```
如果我们只想查询总数, 而不需要返回数据时, 可把 `take` 设置为 0, 此时 `results`字段将不返回数据.