# 上空云 RESTFUL API 文档

## API调用地址及格式
Restful API URL 格式为: [https://skynology.com/api/1.0/login](https://skynology/api/1.0/login) . 

其中 `https://skynology.com/api`为跟域名,  ` 1.0 ` 为当前API版本号, ` login ` 为具体功能地址. 详细功能参考 [API列表](#API列表)


## 注意事项
* 所有API调用数据格式均为 `JSON`. 需设置http头 `Content-Type : application/json`
* 所有API调用需设置http头 `X-SKY-Application-Id : <项目对应的applicationId>`
* 所有API调用需要传入签名, 设置http头 ` X-SKY-Request-Sign : <签名内容> `, 详情参考 [签名相关文档](#签名)
* 当用户登陆后, 需传入session字段, 以便识别. 设置http头 ` X-SKY-Session-Token : <session 内容>`


## API列表

### 资源操作相关
URL|Method|Description
---|------|-----------
resources/\<resourceName>|POST|创建对象
resources/\<resourceName>|GET|查询对象
resources/\<resourceName>/\<objectId>|GET|获取对象
resources/\<resourceName>/\<objectId>|PUT|更新对象
resources/\<resourceName>/\<objectId>|DELETE|删除对象

### 用户相关
URL|Method|Description
---|------|-----------
users|POST|注册用户
users/\<objectId>|GET|获取用户
users/\<objectId>|PUT|更新用户
users/\<objectId>/resetPassword|PUT|修改登录密码
users/\<objectId>|DELETE|删除用户
login|POST|登陆账号
requestResetPassword|POST|申请修改密码邮件
requestVerifyEmail|POST|申请验证邮箱邮件

### 角色相关
URL|Method|Description
---|:------:|-----------
roles|POST|创建角色
roles/\<objectId>|GET|查询角色
roles/\<objectId>|GET|获取角色
roles/\<objectId>|PUT|更新角色
roles/\<objectId>|DELETE|删除角色

### 文件相关
URL|Method|Description
---|------|-----------
files/fetchFromURL|POST|发送邮件

### 邮件相关
URL|Method|Description
---|------|-----------
email/\<objectId>|POST|发送邮件

### 统计数据相关
URL|Method|Description
---|------|-----------
statistics/api|GET|查询API调用统计信息


## 签名

所有的API调用都需要传入http头 签名信息, 也就是 ` X-SKY-Request-Sign `, 其值的格式为 ` timestamp,sign[,master] ` .

其中:

* timestamp（必须） - 客户端产生本次请求的 unix 时间戳，精确到毫秒.
* sign（必须）- 将 timestamp 加上 app key(或者 master key) 组成的字符串做 MD5 签名。
* master （可选）- 字符串 "master"，当使用 master key 签名请求的时候，须加上这个后缀, 以便我们分辨您是用 master key 来做签名的.

假设您有一项目信息如下: 

* ApplicationId  : `5451f38f9d40a887a1000005`
* ApplicatoinKey : `b1e47980eceec514d65912c93ae2097c016b59e8`
* MasterKey      : `b363cbfee6e08383c23ddfaf8169bf6ef0d594a2`


那么我们用ApplicationKey来做签名, `X-SKY-Request-Sign:1415856734899,2bd0e59e0e8ad68fa0491e7db7a90756`, 其中:

* 1415856734899: 为当前UNIX时间戳.
* 2bd0e59e0e8ad68fa0491e7db7a90756: `为时间戳+ApplicationKey`的MD5值, 也就是对(1415856734899b1e47980eceec514d65912c93ae2097c016b59e8)加MD5值.


如果用MasterKey来做签名, 那么签名值的Http header就为: `X-SKY-Request-Sign:1415856734899,c4e3b8c3a43ad0bd49d7c2fff5f19960,master`. 其中:

* 1415856734899: 为当前UNIX时间戳.
* c4e3b8c3a43ad0bd49d7c2fff5f19960: `为时间戳+MasterKey`的做MD5值, 也就是对(1415856734899b363cbfee6e08383c23ddfaf8169bf6ef0d594a2)加MD5值.
* master: 为我们后台分辨您是用master key来做签名的标记.

> 注意: UNIX时间戳最好为您调用API时的时间, 如果此值若是 **10分种** 以前的时间, 后台将拒绝本次请求.

### 响应格式
所用API请求, 若成功, 将会返回一个JSON对象.
成功与否将按HTTP返回的状态码来判断, 状态码为`2XX`时, 表明本次请求成功.

若返回的状态为`4XX`时, 代表您的请求失败. 此时会返回一个表示错误的JSON对象.
如下格式:

```json
{
	"code": 202,
	"error": "用户名已经被占用",
	"error_en": "Username has already been taken"
}
```

更多错误代码, 可参考: [错误代码列表](http://doc.skynology.com/api-errors.html)



## 资源操作
在上空云中的用户可创建多个资源(resource),  每个资源类中可创建多个对象(object). 每个对象可有多个字段(column). 

若把这些概念同数据库中相关概念比较, 您也许更容易理解.

上空云|数据库|备注
-----|-----|---
资源(resource)|表(table)|
对象(object)|行(row)|
字段(attribute)|列(column)|


每一个对象将会有几个默认字段: `objectId`, `ACL`, `createdAt`, `updatedAt`. 其中:

* `objectId` : 对象的Id, 主键, 全局唯一标识. 为24位字符串. 如: `5458e0da9d40a82718000001`. 当您创建对象时, 系统会自动创建此字段, 且后续不允许修改.
* `ACL`: 对象的权限列表, 为一个JSON对象. 详情查看下面的 [ACL](#ACL) 说明
* `createdAt`: 对象创建时间, 当创建对象时系统自动生成, 后续操作不允许修改.
* `updatedAt`: 最后一次修改对象的时间, 比如修改对象中的字段等时, 系统会自动更新此字段. 默认和 `createdAt` 相同.

> 系统默认创建的资源会有更多的字段. 比如 保存[用户](#用户)信息的资源 `_User`. [角色](#角色)信息的资源 `_Role` 会有更多默认字段.


### ACL
`ACL` 为JSON格式, 用来说明 `谁` 对当前对象有 `什么` 权限. ACL只允许用 [用户](#user) 的 `objectId`, [角色](#role) 的 `name` 字段来授权. 如下:

```json
{
	"objectId": "5458e0da9d40a82718000001",
	"createdAt": "2015-01-05T14:08:08:008Z",
	"updatedAt": "2015-01-05T14:08:08:008Z",
	"ACL":{
		"5458e0da9d40a82718000111": {
			"read": true,
			"write": true,
		},
		"role:employee":{
			"read": true,
			"write":false,
		}
	}
}
```

上面的 `ACL` 表示, 对当前对象(`5458e0da9d40a82718000001`), 只有 `objectId` 为 `5458e0da9d40a82718000111`的用户有读取及写入权限; 名称(`name`)为 `employee`的角色有读取权限, 但无写入权限.

从上面可以看出, 用 用户来授权时, 直接设置用户 `objectId` 即可, 用角色来授权时, 用 `'role': + '角色名(name)'` 来授权.
授权值有 `read` 标识读取权限, `write`来标识写入权限(包括`更新`, `删除`).

当一个对象对所有用开放时, 可设置


假设您在后台创建了一个 Resource, 用于存放活动相关数据 资源名为: `event`.

### 创建对象

当创建对象时, 发送一个 `POST` 请求到资源对应的URL(`https://skynology/api/1.0/event`)即可, 内容为相对应的JSON数据. 如:

```json
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

```json
{
	"objectId": "5458e2c39d40a827f9000001",
	"createdAt": "2015-01-04T09:00:00:001Z"
}
```

另在HTTP header中返回一个Location的字段, 内容为刚创建对象的HTTP地址, 如:

```html
Location: https://skynology/api/1.0/resources/event/5458e2c39d40a827f9000001
```

### 更新对象
需要更新对象数据时, 可发送一个PUT请求到对象对应的URL上, 如: `https://skynology/api/1.0/resources/event/5458e2c39d40a827f9000001`.

假设要更新上面 `event` 对象的 active为false. 直接PUT相关的数据.

```json
{
	"active": false
}
```
请求成功时, 将会返回一个JSON对象, 包含`objectId`及`updatedAt`字段.

```json
{
	"objectId": "5458e2c39d40a827f9000001",
	"updatedAt": "2015-01-04T10:08:21:001Z"
}
```


#### 计数器
可对任何数字字段做增加(Inrement) 或 减少(Decrement) 操作.
如我们想更新上面活动的报名人数, 可按如下格式PUT数据即可.

```json
{
	"joinUserCount":{"__op":"Increment", "amount":1}
}
```
其中:

* "__op": "Increment" 为固定需要.
* "amont"为递增/递减的数值. 也可是其他数值, 如: -1, 可递减1位.


#### 数组
为方便操作数组类型字段, 可用下面几种操作来更新.

* Add: 增加一个值到指定的数组字段, 可重复.
* AddUnique: 增加一个值到数组内, 只有此值不存在于原数组内, 才会添加到数组内.
* Remove: 从数组内删除指定的值.

若我们想给上面的活动的增加两个VIP门票价格, 可PUT一个如下JSON.

```json
{
	"ticketPrice": {"__op":"AddUnique", "objects":[500, 1000]}
}
```
其中:

* "__op" 为操作类型, "AddUnique" 代表操作类型, 我们也可传"Add"或"Remove".
* "objects" 为需要增加/删除的值, 需数组类型. 比如上面我们增加两个VIP票 500, 1000时, 就以数组格式传入.

#### 关系
关系(Relation)类型其实是一个 `objectId`数组, 所以相关操作参考上面数组即可.

### 删除对象
删除一个对象时, 发送一个DELETE请求到对象对应的URL上即可. 如删除上面的活动对象, 可发送DELETE请求到
 `https://skynology/api/1.0/resources/event/5458e2c39d40a827f9000001` 上即可.

删除成功后会返回一个JSON对象. 包括 "objectId" 及 "deletedAt" 字段.


## 查询
查询一个 resource 时可发送一个GET请求到URL即可, 如对上面活动做查询时, 可发送GET请求到 `https://skynology/api/1.0/resources/event `

<a id="query_where" data-title="查询条件" data-parent="query"></a>
### 查询条件
查询时可传一个编码过的JSON字符串到, 传到URL的where参数上. where参数兼容 MongoDB 查询参数.

通常查询匹配时可直接传对应的键值即可. 如我们要查询所有开放中的活动, 可传以下URL.

`https://skynology/api/1.0/resources/event?where={active:true}`

查询成功后将返回一个JSON对象, 其中 `results` 字段将包含查询结果, 具体格式如下:

```json
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

下面一些查询列子 :

查询报名人数大于等于30人的活动

```
https://skynology/api/1.0/resources/event?where={joinUserCount:{"$gte":30}}
```

查询举行时间在 `2015-01-01` 至 `2015-01-07` 其间的活动.

```
https://skynology/api/1.0/resources/event?where={startTime:{"$gte":"2015-01-01T00:00:01:001Z"},endTime:{"$gte":"2015-01-07T23:59:59:999Z"}}
```
<a id="query_order" data-title="排序" data-parent="query"></a>
### 排序
可在查询URL上增加 `order=字段名` 来做排序, 若需多个字段来排序, 用逗号`,`来分割字段名即可. 字段名前增加负号 `-` 可倒序. 如:

查询报名人数大于20人, 并用开始时间由近到远排序, 可写为如下:

```
https://skynology/api/1.0/resources/event?where={joinUserCount:{"$gte":20}}&order=startTime
``` 
若按远到近排序, 可写为:


```
https://skynology/api/1.0/resources/event?where={joinUserCount:{"$gte":20}}&order=-startTime
``` 
若同一天有多个活动, 再按人数多到少排序:


```
https://skynology/api/1.0/resources/event?where={joinUserCount:{"$gte":20}}&order=-startTime,-joinUserCount
``` 

### 获取指定字段

若您不想获取所有字段, 只想要指定的字段, 可传入 `select` 字段即可. 如下面查询将只返回活动标题及描述. 其他字段将不返回.

```
https://skynology/api/1.0/resources/event?select=name,description
``` 


### 分页
可用 `skip` 及 `take` 来做分页操作, 其中 `take` 取值范围需在 1 ~ 1000之内. 

如每获取30条活动记录, 可:

```
https://skynology/api/1.0/resources/event?take=30
``` 
获取第三页的30条记录:

```
https://skynology/api/1.0/resources/event?skip=60&take=30
``` 


### 对象总数
当我们用 `skip` 及 `take` 做分页查询时, 通常也需要所查询对象的总匹配数. 我们只需要查询时传入 `count=1` 参数即可.

```
https://skynology/api/1.0/resources/event?skip=60&take=30&count=1
``` 
查询成功后在返回的JSON对象中的 `count` 字段值为总匹配数量.

```json
{
	"results":[
		....
	], 
	"count": 403
}
```
如果我们只想查询总数, 而不需要返回数据时, 可把 `take` 设置为 0, 此时 `results`字段将不返回数据.

## 用户
用户相关操作除了 [注册用户](#注册用户) 和 [登陆](#登陆) 外, 其他操作都需要带 `X-SKY-Session-Token` 来确定用户身份.

### 注册用户
注册用户时, 只需发送POST请求到用户资源URL(`https://skynology/api/1.0/users`)上即可.

注册用户时 `username` 及 `password` 字段是必须有值的. `username`字段上有唯一索引, 所有注册已经有的用户名时会返回错误.
`password` 字段我们会在后台进一步加密, 并且前台API查询时不会返回`password`.

注册一个新用户, 可向 `https://skynology/api/1.0/users` 发送如下字段即可.

```json
{
	"username": "skynology",
	"password": "123456",
	"phone": "186000000000",
	"email": "username@skynology.com"
}
```
请求成功后, 返回的http状态和资源操作一样, 为:201. 并会有一个http header `Location:https://skynology/api/1.0/users/546576929d40a80551000002`.

返回的内容一个JSON对象. 包含 `objectId`, `createdAt`, `sessionToken` 三个字段. 

```json
{
	"objectId": "546576929d40a80551000002",
	"createdAt": "2015-01-06T11:12:32:008Z",
	"sessionToken": "5a0593c3d551f9f5b830c07a1d321cf03f3acec3"
}
```

其中 `sessionToken` 为用户在后台的session标识. 可以每次API 请求时在 http header 中设置 `X-SKY-Session-Token: 5a0593c3d551f9f5b830c07a1d321cf03f3acec3`, 以便后台判断用户并做相对应的授权.

登陆用户账号时, 发送POST请求到 `https://skynology/api/1.0/login`. 请求内容需是一个包含 `username` 及 `password` 字段的JSON数据. 如:

```json
{
	"username": "skynology",
	"password": "123456"
}
```

登陆成功后会返回包含 用户的 `objectId`, `createdAt`, `updatedAt` 及 `sessionToken` 字段的JSON对象.

### 更新用户
更新用户信息时, 直接PUT需要更新的字段到用户的URL上, 并且需要登陆用户才可修改, 所以同时需要 
http header 有传 `X-SKY-Session-Token`才可修改, 必如我们要更新上面用户的手机号, 可PUT如下JSON到 `https://skynology/api/1.0/users/546576929d40a80551000002`:

```json
{
	"phone": "18688888888"
}
```
请求成功后, 将返回 `objectId` 和 `updatedAt` 字段了.

```json
{
	"objectId": "",
	"updatedAt": "2015-01-08T10:14:13:009Z"
}
```

### 修改密码
用户密码是不允许直接更新来做修改的. 需要更新时, 需要POST旧密码及新密码到 `https://skynology/api/1.0/users/<objectId>/resetPassword` 即可. 如:

```json
{
	"old_password": "123456",
	"new_password": "123456789"
}
```

### 申请验证邮箱
当需要验证用户邮箱时, 发送POST请求到 `https://skynology/api/1.0/users/<objectId>/requestVerifyEmail`.

```json
{
	"name": "William"
}
```

当调用验证用户邮箱时, 系统会从后台邮件模板列表(`_EmailTemplate`)中查找 `name` 为 `verifyEmail` 的模板,  并将上面JSON中 `email` 占位符相关知识请看 [发送邮件](#发送邮件). 
如果您有修改过模板, 并增加了自定义占位符, 请在上面JSON对象中增加相对应的字段.

若您不希望用上空云提供的验证页面, 可在上面的JSON中加自定义字段 `link` 来覆盖系统提供的默认验证地址, 系统会在link所对应的URL后面增加三个参数 `applicationId`, `email`, `token`. 如
`http://mycustomurl.com?applicationId=<当前项目ID>&email=<上面JSON对应的邮箱>&token=<验证码>`.

当您需要做验证时,可向 `https://skynology/api/1.0/verifyEmail` 发送一个PUT请求, 请求JSON内容为上面URL里接收到的参数, 其中email及token为必须字段.


>  模板中的 {{link}}, {{project}} 两个占位符为保留字段, 会在系统后台自动填充。 可随意调整在模板中的位置.


### 删除用户
删除用户时, 发送一个DELETE请求到用户URL即可. 如删除上面用户时, 可发送DELETE到 `https://skynology/api/1.0/users/546576929d40a80551000002` .

## 角色
做过各种系统的用户都会碰到 `角色`, 不管您是做ERP还CMS等, 一般都会碰到用角色来授权用户. 角色可以是一个特定的划分, 包含一个或多个用户, 比如一般系统都会有一个角色叫 `admin`, 用来做一些高安全性的操作. 

上空云也提供了角色服务, 一个角色主要包含 `name`, `users`, `ACL` 及 `parents` 字段. 其中 `name`为角色名, 全局应唯一, 用来分辨/识别角色. `users` 为指向 用户的 relation(`objectId数组`), `parents` 是为了角色继承而设置的 relation, 可存放父角色的 `objectId`. 

> 角色涉及项目安全及用户权限相关, 所以建议直接在我们管理后台去操作或由Rest API来操作. 若有需求放到前台或其他客户端去操作时需谨慎.

### 创建角色
创建角色, 可POST数据到 `https://skynology/api/1.0/roles`.

```json
{
	"name": "Admin",
	"ACL": {
		"*": {
			"read": true
		}
	},
	"users": ["546576929d40a80551000002", "546576929d40a80551000003", "546576929d40a80551000004"]
}
```
当创建成功时, 返回的http状态码为 201. http header 中带roles的URL `Location: https://skynology/api/1.0/roles/5458e0da9d40a82718000001`.

### 获取角色
获取角色信息可发送一个GET请求到角色的URL上, 如上面创建时返回的Location地址 `Location: https://skynology/api/1.0/roles/5458e0da9d40a82718000001` 将会返回角色的JSON信息.

```json
{
	"objectId": "5458e0da9d40a82718000001",
	"name": "Admin", 
	"ACL": {
		"*": {
			"read": true
		},
		"role:Admin": {
			"read": true,
			"write": true
		}
	},
	"users": ["546576929d40a80551000002", "546576929d40a80551000003", "546576929d40a80551000004"],
	"parents": [],
	"createdAt": "2015-01-08T12:01:01:999Z",
	"updatedAt": "2015-01-08T12:01:01:999Z"
}
```

### 更新角色
更新角色时可发送PUT请求到角色URL. 角色字段中的 `users` 及 `parents` 为 Relation 类型, 所以更新时, 可参考 [更新数组对象](#数组) .

若我们把 `546576929d40a80551000009`这个用户加入到`Admin`数组中. 可发送如下JSON到 `https://skynology/api/1.0/roles/5458e0da9d40a82718000001`.

```json
{
	"users":{ "__op":"AddUnique", "objects":["546576929d40a80551000009"] }
}
```

<a id="role_delete" data-title="删除角色" data-parent="role"></a>
### 删除角色
删除角色时, 发送DELETE请求到角色的URL. 如删除上面 `Admin` 角色时, 发送 DELETE 到 `https://skynology/api/1.0/roles/5458e0da9d40a82718000001`.

## 文件
文件服务我们选择第三方平台: [七牛](http://www.qiniu.com), 为国内存储行业一线平台. 

### 获取上传Token
上传文件到七牛平台时, 需要设置上传凭证, 详细概念可参考 [上传凭证（UploadToken）](http://developer.qiniu.com/docs/v6/api/overview/security.html) .

在上空云, 您可以发送GET请求到 `https://skynology/api/1.0/files/token` 获取上传相关信息.
请求成功后将返回如下JSON对象:

```json
{
	"vendor": "qiniu",
	"bucket": <bucket名>,
	"token": <上传Token信息>
}
```

当拿到此信息后可参考七牛对应文档去上传文件. 上传成功后, 上空云系统自动将在 `_File`资源中创建对应信息, 并返回文件相关JSON信息.

> 上传 Token 过期时间为： 1个小时。

### 删除文件
删除文件时, 发送DELETE请求到文件对应的资源URL(不是文件存储于七牛的URL)即可. 如:
`https://skynology/api/1.0/files/646576929d40a80551000001`.

## 邮件
多数项目都会涉及到发送邮件的功能, 不管是账号激活, 密码验证 或者是发送邮件通知等. 所以上空云给大空提供了发送邮件功能.

### 创建邮件模板
发送邮件前, 需要预先创建好邮件模板文件. 模板格式使用 [mustache](http://mustache.github.io/).
我们在后台预先创建好模板资源表 `_EmailTemplate`. 主要字段有 `name`,  `subject` 及 `content` 字段. 其中:

* `name`   : 系统用0特殊识别符, 唯一约束。 如发送验证帐号邮件时， 系统会到找 `name`: `verifyEmail`的模板。
* `subject`: 为邮件标题模板
* `content`: 邮件内容模板.

比如我们要创建一个模板, 可POST请求到 `https://skynology/api/1.0/resources/_EmailTemplate`. 如:

```json
{
	"subject": "激活{{project}}账号",
	"content": "您好{{user}}, 
				欢迎您注册{{project}}, 请您激活你的账号, 激活地址为:{{url}}. 
				谢谢!"
}
```
创建成功后, 将同其他资源对象创建一样, 返回包含 `objectId` 及 `createdAt`, `updatedAt` 的JSON对象. 并会在http header 中邮件模板的URL. `Location: https://skynology/api/1.0/resources/_EmailTemplate/946576929d40a80551000001`

### 发送电邮
当在已经创建了邮件模板后, 可POST一个JSON数据到 `https://skynology/api/1.0/email/send/<objectId>`. JSON对象中的字段, 将会替换模板中的占位符. 比如上面创建的这个用户账号激活的邮件, 可发送POST请求到模板的URL上. `https://skynology/api/1.0/email/946576929d40a80551000001`.

```json
{
	"to": ["user1@skynology.com", "user2@skynology.com"],
	"data": {
		"project": "上空云",
		"user": "William",
		"url": "http:/www.skynology.com/xxxx/xxxxxxxx-xxx-xxx"
	}
}
```

其中 `to` 为收件人的邮件地址数组, 可有多个收件人. `data` 为需要替换模板中占位符的数据. 当邮件发送时, 邮件标题及内容将被替换成如下:

```
标题: 激活上空云账号   
内容: 您好William,   
     欢迎注册上空云, 请激活您的账号, 激活地址为: http:/www.skynology.com/xxxx/xxxxxxxx-xxx-xxx   
	 谢谢!
```


> 另邮件内容以 `HTML` 格式发送, 所以可以模板中带有HTML标记.

## 数据统计
我们提供各类的统计/分析信息给大家.

### API调用信息统计
API调用数据直接关系到大家的 `$`. 所以相信大家也会非常关心此信息.
我们会在每日凌晨 1点到4间某一时间统计前一天的各种API调用情况. 并生成报表, 存于系统中. 

查询API统计信息, 可发送GET请求到 `https://skynology/api/1.0/statistics/api`. 此功能接受两个参数, `startDate` 和 `endDate`. 参数类型为日期格式, 如 `2015-01-01`., 比如我们要查询1号到7号的API调用数据. 可GET请求: `https://skynology/api/1.0/statistics/api?startDate=2015-01-01&endDate=2015-01-07`.

查询成功后, 将返回:

```json
[
	{
		objectId: "5465f5639d40a807b8000030",
		date: "2015-01-01", // 统计日期
		apiGetCount: 1031,  // GET请求数
		apiPostCount: 56,   // POST请求数
		apiPutCount: 7629,  // PUT请求数
		apiDeleteCount: 185, // DELETE请求数
		fileGetCount: 41338, // 文件GET请求数
		filePutCount: 342,  // 文件PUT请求数
		fileSpace: 868977,  // 文件磁盘使用量. 单位为Byte.
		fileTransfer: 81157, // 文件流量使用量. 单位为Byte.
		sendEmailCount: 364,  // 邮件发送量. 
	}, 
	...
]
```
