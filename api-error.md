# RESTful API 错误码详解

## 格式
服务器端api返回的错误代码列表.


服务器api返回的http status 为 4XX 或 5XX 时, 将会显示这些错误. 

返回的json格式为

```json
{
	"code": 202,
	"error": "用户名已经被占用",
	"error_en": "Username has already been taken."
}
```

## 错误列表

### 1 

* 信息: ``` Internal server error. No information available ```
* 含义: 服务器内部错误或者参数错误，一般是因为传入了错误的参数，或者没有在本文档里明确定义的运行时错误，都会以代码 1 指代。

###  10
* 信息: ``` Permission denied ```
* 含义: 无权限操作

### 30
* 信息: ``` Project name is missing or empty ```
* 含义: 没有提供项目名称, 或项目名称为空

### 31
* 信息: ``` Invalid Project name, A project name contains only a-zA-Z0-9 characters and starts with letter ```
* 含义: 非法的 Project 名称，Project 名称是大小写敏感的，并且必须以英文字母开头，有效的字符仅限在英文字母、数字以及下划线

### 32
* 信息: ``` Project are already exists ```
* 含义: 项目已经存在, 名称重复

### 80
* 信息: ``` Invalid paramters ```
* 含义: 无效的参数或参数不全

### 81
* 信息: ``` The token is expired ```
* 含义: Token 已失效

### 82
* 信息: ``` Invalid invite code ```
* 含义: 无效的邀请码

### 83
* 信息: ``` Please verify phone & email before create project ```
* 含义: 先验证您的邮箱和手机号

### 100
* 信息: ``` The connection to the servers failed ```
* 含义: 无法建立 TCP 连接到服务器，通常是因为网络故障，或者我们服务器故障引起的

### 101
* 信息: ``` resource or attribute doesn't exist ```
* 含义: 查询的 resource 不存在，或者要关联的 Pointer 对象不存在

### 102
* 信息: ``` The object/attribute already exist
* 含义: 查询的 resource 或者 字段 已经存在.

### 103
* 信息: ``` Missing or invalid esourcename. objectnames are case-sensitive ```
* 含义:  非法的 resource 名称，resource 名称是大小写敏感的，并且必须以英文字母开头，有效的字符仅限在英文字母、数字以及下划线

### 104
* 信息: ``` Missing object id. ```
* 含义: 缺少 objectId，通常是在查询的时候没有传入 objectId，或者 objectId 非法, objectId 为标准的mongodb objectId,  可[参考此文章](http://api.mongodb.org/java/current/org/bson/types/ObjectId.html)

### 105
* 信息: ``` Invalid key name. Keys are case-sensitive. They must start with a letter, and a-zA-Z0-9_ are the only valid characters ```
* 含义: 无效的 key 名称，也就是 Resource 的列名无效，列名必须以英文字母开头，有效的字符仅限在英文字母、数字以及下划线

### 106
* 信息: ``` Malformed pointer. Pointers must be arrays of a resourcename and an object id ```
* 含义:  无效的 Pointer 格式，Pointer 必须为形如 ```{resourceName: 'GameScore', objectId:'xxxxxx'}``` 的 JSON 对象

### 107
* 信息: ``` Malformed json object. A json dictionary is expected ```
* 含义: 无效的 JSON 对象，解析 JSON 数据失败。

### 108
* 信息: ``` Tried to access a feature only available internally ```
* 含义: 此 API 仅供内部使用。

### 109
* 信息: ``` Cannot insert duplicate object in resource ```
* 含义: 无法插入重复对象到资源类。

### 111 
* 信息: ``` Field set to incorrect type ```
* 含义: 想要存储的值不匹配列的类型，请检查你的数据管理平台中列定义的类型，查看存储的数据是否匹配这些类型

### 112 
* 信息: ``` Invalid channel name. A channel name is either an empty string (the broadcast channel) or contains only a-zA-Z0-9_ characters and starts with a letter ```
* 含义: 推送订阅的频道无效，频道名称必须不是空字符串，只能包含英文字母、数字以及下划线，并且只能以英文字母开头

### 114
* 信息: ``` Invalid device token ```
* 含义:  iOS 推送存储的 deviceToken 无效，如何存储 installation 请阅读

### 116 
* 信息: ``` The object is too large ```
* 含义: 要存储的对象超过了大小限制，我们限制单个对象的最大大小在 16M

### 119
* 信息: ``` That operation isn't allowed for clients ```
* 含义:  该操作无法从客户端发起

### 120
* 信息: ``` The results were not found in the cache ```
* 含义: 查询结果无法从缓存中找到，SDK 在使用从查询缓存的时候，如果发生缓存没有命中，返回此错误

### 121
* 信息: ``` Keys in NSDictionary values may not include '#39; or'.' ```
* 含义: JSON 对象中 key 的名称不能包含 ``` $ ``` 和 ``` . ```符号

### 122
* 信息: ``` Invalid file name. A file name contains only a-zA-Z0-9_. characters and is between 1 and 36 characters ```
* 含义: 无效的文件名称，文件名称只能是英文字母、数字和下划线组成，并且名字长度限制在 1 到 36 之间

### 124
* 信息: ``` The request timed out on the server. Typically this indicates the request is too expensive ```
* 含义: 请求超时，超过一定时间（默认 10 秒）没有返回，通常是因为网络故障或者该操作太耗时引起的

### 125
* 信息: ``` The email address was invalid. ```
* 含义: 电子邮箱地址无效

### 126
* 信息: ``` Send email failed ```
* 含义: 发送电脑失败

### 137
* 信息: ``` A unique field was given a value that is already taken ```
* 含义: 违反 resource 中的唯一性索引约束 (unique)，尝试存储重复的值

### 139
* 信息: ``` Role's name is invalid ```
* 含义: 角色名称非法，角色名称只能以英文字母、数字或下划线组成

### 140
* 信息: ``` Exceeded an application quota. Upgrade to resolve ```
* 含义: 超过应用的容量限额，请升级帐户等级

### 141
* 信息: ``` Cloud Code script had an error ```
* 含义: 云代码脚本编译或者运行报错

### 142
* 信息: ``` Cloud Code validation failed ```
* 含义: 云代码校验错误，通常是因为 beforeSave、beforeDelete 等函数返回 error

### 145 
* 信息: ``` Payment is disabled on this device ```
* 含义: 本设备没有启用支付功能

### 150
* 信息: ``` Fail to convert data to image ```
* 含义: 转换数据到图片失败

### 180
* 信息: ``` Get upload token failed ```
* 含义: 获取上传文件token失败

### 181
* 信息: ``` Invalid callback token ```
* 含义: 无效的回掉

### 182
* 信息: ``` save file to db failed ```
* 含义: 保存文件失败

### 200
* 信息: ``` Username is missing or empty ```
* 含义: 没有提供用户名，或者用户名为空

### 201
* 信息: ``` Password is missing or empty ```
* 含义: 没有提供密码，或者密码为空

### 202
* 信息: ``` Username has already been taken ```
* 含义: 用户名已经被占用

### 203
* 信息: ``` Email has already been taken ```
* 含义: 电子邮箱地址已经被占用

### 204
* 信息: ``` The email is missing, and must be specified ```
* 含义: 没有提供电子邮箱地址

### 205
* 信息: ``` A user with the specified email was not found ```
* 含义: 找不到电子邮箱地址对应的用户

### 207
* 信息: ``` Users can only be created through sign up ```
* 含义: 只能通过注册创建用户，不允许第三方登录

### 208
* 信息: ``` An existing account already linked to another user ```
* 含义: 第三方帐号已经绑定到一个用户，不可绑定到其他用户

### 210
* 信息: ``` The username and password mismatch ```
* 含义: 用户名和密码不匹配

### 211
* 信息: ``` Cloud not find user ```
* 含义: 找不到用户

### 212
* 信息: ``` The new password and old password mismatch ```
* 含义: 新旧密码不匹配, 用于修改密码

### 216

* 信息: ``` Disabled account ```
* 含义: 当前账户被禁用, 可在后台设置 'active' 字段

### 250

* 信息: ``` Linked id missing from request ```
* 含义: 连接的第三方账户没有返回用户唯一标示 id

### 251

* 信息: ``` Invalid linked session ``` 或者 ``` Invalid Weibo session ```
* 含义:  无效的账户连接，一般是因为 access token 非法引起的

### 401

* 信息: ``` Unauthorized ```
* 含义: 未经授权的访问，没有提供 App id，或者 App id 和 App key 校验失败，请检查配置

### 401
* 信息: ``` Invalid session token```
* 含义: 无效的Session Token, 未能正确验证 X-SKCloud-Session-Token.

### 403
* 信息: ``` Forbidden to xxx by resource permissions ```
* 含义: 操作被禁止，因为 resource 权限限制

### 503
* 信息: ``` Rate limit exceeded ```
* 含义: 超过流量访问限制，默认 API 并发 1000 访问每秒，通过数据管理平台每秒限制上传一个文件，并且每分钟最多上传 30 个文件，如需提升，请联系我们


## 短信服务相关

### 10801
* 信息: ``` Phone Number is missing or empty```
* 含义: 没有提供收信人手机号或手机号格式错误 

### 10802
* 信息: ``` The Params is missing or empty ```
* 含义: 请提供 [params] 参数或参数不正确

### 10803
* 信息: ``` Invalid Template Id. ```
* 含义: 未找到对应模板文件或您无权限查看此模板文件

### 10804
* 信息: ``` Need Set SMS Vendor before call SMS functions ```
* 含义: 请先设置短信服务商帐号信息

### 10805
* 信息: ``` Verify Phone Number failed ```
* 含义: 验证手机号码失败


