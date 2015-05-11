# 微信平台接入指南
为了方便大家给自己的项目接入微信公众平台服务, 我们在RESTFul API中集成了微信公众号服务. 若您对我们的RESTFul接口还不熟悉, 请先阅读: [上空云RESTFul API文档](#restful-api.html).

## 基础概念
调用微信相关API的URL格式如下: `https://skynology.com/api/1.0/weixin/{url}`.   
其中:

* url: 具体功能url部分, 如增加部门时, 是`department`

调用微信相关API时, 请在请求的Header中带上`X-Sky-Weixin-Id` 及 `X-Sky-Weixin-Type` 两个. 其中:

* X-Sky-Weixin-Type: 为公众号类型, 如 `mp`, `corp`等.
* X-Sky-Weixin-Id: 在控制台组件中,绑定了微信公众号后, 由系统产生的id.
若您的项目中只绑定了一个微信公众平台, 并且状态为 `运行中`, 可以不带上面两个Header,　系统会自动会匹配到绑定好的公众平台上．

调用时, 绝大多数可参考微信开发平台文档. 尤其POST格式的, 基本都是按微信公众平台文档上那些参数. 及少数会为了方便调用做了修改, 修改过的会标明出来, 无标明的就表示兼容微信文档上的参数.  
 
[企业号开发文档](http://qydev.weixin.qq.com/wiki/index.php)   
[订阅/服务号开发文档](http://mp.weixin.qq.com/wiki/home/index.html)

## 权限说明
因微信平台关系者关系者第三方平台, 所以调用相关API时, 若无特殊说明, 均需Master权限的签名才可访问. 可参考下面API列表中的 `Sign`列, 若无表明 `Master`, 则可以普通签名去访问.


## 绑定微信公众号
若需要使用微信相关API时, 先在我们的[管理台](#console-tutorial.html)绑定您的公众号信息. 如下图:    
![微信绑定](http://7tebuf.com2.z0.glb.qiniucdn.com/console-tutorial/wechat-setting.jpg?imageView/2/h/300)

在绑定时需注意几个地方.如上图中:   

* [标记1]处的应用Id是在您绑定好保存后由系统自动产生. 此Id为每次新绑定时会不同, 但修改绑定数据时不会变. 此Id用于调用API时的URL中. 下面的文档中将以`550273463a859a0488000001`来URL中的代替{id}. 您调用API时, 可替换您的项目中的实际ID.
* [标记2]处的URL标识可自己随机填写, 只能是英文字母. 用于区别一个项目中绑定多个微信应用的情况.

当保存完绑定数据后, 可把上图中的回调URL复制到微信公众平台后台的回调模式下的URL输入框中即可.

## 调用权限
调用微信相关API时, 需要Master权限, 也就是您传的 `X-Sky-Request-Sign`是用`Master Key`来加密的.

## 缓存
我们会在服务上缓存相关的access_token, 以避免因频繁生成access_token而被屏蔽.

## 订阅号/服务号


### URL地址
URL|Method|Sign|Description
---|------|----|-----------
menu|GET|Master|获取菜单
menu|POST|Master|创建菜单
menu|DELETE|Master|删除菜单
group|GET|Master|获取用户分组
group| POST |Master|创建用户分组
group|PUT|Master|更新用户分组
group/users|POST|Master|移动用户分组
group/users/batch|POST|Master|批量移动用户分组
users| GET |Master|获取用户列表
users/{openId}| GET |Master|获取用户基本信息
users/updateremark|PUT|Master|设置备注名
qrcode|POST| Master |创建二维码
qrcode/fetch|POST|-|下载二维码并传到文件服务器
shorturl|POST|-|长链接转短链接
oauth2/url|POST|-|创建OAuth2调用URL, 用于获取code
oauth2/token|POST|-|通过code换取网页授权access_token
oauth2/refresh|POST|Master|刷新access_token
oauth2/userinfo|POST|Master|拉取用户信息(需scope为 snsapi_userinfo)
oauth2/check|POST|-|检验授权凭证（access_token）是否有效
oauth2/login|POST|-|用上面换取到的code来登陆系统
oauth2/signature|POST|-|获取JS SDK所需的signature
template/industry|POST|Master|设置所属行业
template|POST|Master|增加模板
template/send|POST|Master|发送模板消息
media/fetch|POST|Master|抓取媒体文件到文件服务器
meterial|GET|Master|获取素材列表
meterial/count|GET|Master|获取素材总数
meterial/{meidaId}|DELETE| Master |删除指定素材
meterial/news|POST|Master|新增永久图文素材
meterial/news/{meidaId}|GET|Master|图文永久图文素材
customservice/kfaccount|GET|Master|获取客服基本信息
customservice/kfaccount/online|GET|Master|获取在线客服接待信息
customservice/kfaccount|POST|Master|添加客服账号
customservice/kfaccount|PUT|Master|设置客服信息
customservice/kfaccount/{account}|DELETE|Master|删除客服账号
customservice/kfsession|POST|Master|创建会话
customservice/kfsession|PUT|Master|关闭会话
customservice/kfsession/user/{openid}|GET|Master|获取客户的会话状态
customservice/kfsession/list/{account}|GET|Master|获取客服的会话列表
customservice/kfsession/wait|GET|Master|获取未接入会话列表
customservice/msgrecord|POST|Master|获取客服聊天记录接口
mass/send/{to}|POST|Master|高级群发消息, {to}为群发类别, 比如 预览(preview), 分组(group), 按用户openId列表(list)等等值.
mass/{msgId}|DELETE|Master|删除群发

### 用户管理
用户管理相关API内容都兼容微信官方API. 可参照上面的调用URL及官方参数格式调用.


### 发送消息

#### 高级群发消息
高级群发消息, 可发送POST请求到 `https://skynology.com/weixin/mp/<id>/mass/send/{to}`. 
其中{to}为发送群发消息方式, 值可选 `group`, `list`, `preview`三个. 分别对应按分组(所有人)群发, 按用户`openid`列表群发, 以及预览方式等. 
如发送给组别为2的用户发送一条文本消息, 可POST如下JSON到`https://skynology.com/weixin/mass/send/group`.


```json
{
   "filter":{
      "is_to_all":false
      "group_id":"2"
   },
   "text":{
      "content":"CONTENT"
   },
    "msgtype":"text"
}
```
### OAuth2

#### 获取URL
可POST如下JSON数据到 `https://skynology.com/api/1.0/weixin/oauth2/url`上, 为了安全起见, 需传入一个 `request_key`, 等后续操作要带上 request_key,  以便服务器区分用户调用.

```json
{
	"redirect_uri":"",
	"response_type":"",
	"scope":"",
	
	// 此字段为上空云增加. 用于区别用户调用. 最长40位 
	request_key: "test"
}
```
成功后后台会返回编码过后的URL.

```json
{
	// 编码过后的URL
	url: "",
	// 后台会根据您传下的 request_key 自动生成
	"state": "xxxddddxxx"
}
```
当转到此URL后, 微信服务器会转到上面你传入的`redirect_uri`对应的地址上, 并带有code及state等参数.

#### 换取access_token
当上一步得到code后, 可用此code换取网页授权access_token. POST刚得到的code到 `https://skynology.com/api/1.0/weixin/oauth2/token`上即可换取access_token等信息.

```json
{
	// 在 获取URL 时传入的 request_key
	"request_key": "test",
	
	"code": "刚得到的code"
}
```
当code正确时, 将返回access_token等信息.

```json
{
   "access_token":"ACCESS_TOKEN",
   "expires_in":7200,
   "refresh_token":"REFRESH_TOKEN",
   "openid":"OPENID",
   "scope":"SCOPE"
}
```

#### 登陆上空云平台
code除了换取access_token以外, 也可以直接登陆上空云服务用户. 当登陆成功后, 返回上面的access_token等信息外, 还会返回`SessionToken`, 以便后续调用RESTful API时判断用户信息. 可POST数据到 `https://skynology.com/api/1.0/weixin/oauth2/login`.

```json
{

	"code":"刚得到的code",

	// 在_User表中存放用户微信openId的字段名
	// 需已经把openId存到此字段上. 否则无法找到对应用户.
	"field": "",
	
		// 在 获取URL 时传入的 request_key
	"request_key": "test"
}
```
若成功找到用户, 将返回用户基本信息 + `sessionToken`字段 + `_weixinToken` 字段.   
`_weixinToken`字段是刚才code换取的access_token等信息. 如下格式:

```json
{
	"objectId": "5506e16e3a859a3ab5000001",
	"username": "William",
	"email":"email@email.com",
	....
	"sessionToken":"127b296a487694bfd6760a0b48df8beab7fe2908",
	"_weixinToken": {
        "AccessToken": "xxxxx...xxx",
        "RefreshToken": "xxxx...xxx",
        "ExpiresAt": 1426520547,
        "OpenId": "o3ekhs_Lqq4QShTVfaC3eQw-xxx",
        "Scopes": [
            "snsapi_userinfo"
        ]
    }
}
```

### 媒体文件
一般媒体交互可直接参考微信的[JS-SDK](http://mp.weixin.qq.com/wiki/7/aaa137b55fb2e0456bf8dd9148dd613f.html). 但为了方便大家操作, 提供了以下API帮助大家开发.

#### 抓取媒体文件到文件服务器
当您有媒体文件Id(media_id)时, 可将此媒体文件直接下载到文件服务器, 以便永久保存. 可POST一个JSON数据到 `https://skynology.com/weixin/media/fetch`, 如下:

```json
{
	// 您要上传的Media文件的ID
	"media_id":"qI6_Ze_6PtV7svjolgs-rN6stStuHIjs9_DidOHaj0Q-mwvBelOXCFZiq2OsIU-p"
}
```sessionToken
成功上传后, 将会将返回文件资源表对应信息,如:

```json
{
	"objectId":"550ff32b3a859a215d000002",
	"url":"http://xx.com/xx",
	"size":1135,
	...
	"createdAt":"2015-03-23T11:07:51.712Z"
}
```
### 二维码

#### 创建二维码
创建二维码的API是极少数修改原微信格式中的一个. 为了方便调用, 把JSON格式改成一个层次, 而非微信API中的多层次JSON. 创建一个临时二维码时, 可POST如下JSON到
`https://skynology.com/weixin/qrcode`:

```json
{
	// 若创建永久二维码时, 需传入 "QR_LIMIT_STR_SCENE"
	"action_name": "QR_SCENE",
	"expire_seconds": 1800,
	"scene_id": 1,
	
	// 若创建永久二维码时, 也可用字符串形式的ID.
	"sence_str": ""
}
```
创建成功时, 会返回如下格式.

```json
{
	"ticket":"gQH47joAAAAAAAAAASxodHRwOi8vd2VpeGluLnFxLmNvbS9xL2taZ2Z3TVRtNzJXV1Brb3ZhYmJJAAIEZ23sUwMEmm3sUw==",
	"expire_seconds": 1800,
	"url":"http:\/\/weixin.qq.com\/q\/kZgfwMTm72WWPkovabbI"
}

```

#### 上传二维码图片到文件服务器
当上一步创建成功后, 可用拿到的`ticket`把二维码图片上传到文件服务器上, POST如下JSON到 `https://skynology.com/weixin/qrcode/fetch`即可.

```json
{
"ticket":"gQH47joAAAAAAAAAASxodHRwOi8vd2VpeGluLnFxLmNvbS9xL2taZ2Z3TVRtNzJXV1Brb3ZhYmJJAAIEZ23sUwMEmm3sUw=="
}
```
上传成功时, 会返回文件的`objectId`以及七牛返回的`hash`及`key`两个字段.

#### 长链接转短链接接口
可POST一个请求到`https://skynology.com/weixin/mp/550273463a859a0488000001/shorturl`.

```json
{
	"action":"long2short",
	"long_url": "https://www.skynology.com/xxx/xx/xx/x/xx//x/x"
}
```
成功将返回JSON:

```json
{
	"short_url":"http://w.url.cn/s/Acod61i"
}
```

## 企业号

### 企业号API调用URL
URL|Method|Sign|Description
---|------|----|----------
menu|GET|Master|获取菜单
menu|POST|Master|创建菜单
menu|DELETE|Master|删除菜单
department|GET|Master|获取所有部门信息
department/{departmentId}|GET|Master|获取指定部门信息
department|POST|Master|创建部门
department/{departmentId}|PUT|Master|修改指定部门信息
department/{departmentId}|DELETE|Master|删除指定部门信息
tag|GET|Master|获取标签信息
tag|POST|Master|创建标签
tag/{tagId}|PUT|Master|修改指定标签信息
tag/{tagId}|DELETE|Master|删除指定标签
tag/{tagId}/users|GET|Master|获取指定标签下的用户列表
tag/{tagId}/users|POST|Master|增加用户到指定标签下
tag/{tagId}/users|DELETE|Master|从指定标签下删除用户
users/{userId}| GET |Master|获取用户基本信息
users|POST|Master|新增用户
users/{userId}|PUT|Master|修改用户
users/{userId}|DELETE|Master|删除用户
users|DELETE|Master|批量删除用户
users/{userId}|POST|Master|邀请用户
oauth2/url|POST|Master|创建OAuth2调用URL, 用于获取code
oauth2/userinfo?agentId=&code=|GET|Master|通过code换取用户信息
oauth2/signature|POST|-|获取JS签名
message|POST|Master|发送消息
media/fetch|POST|Master|上传媒体文件到文件服务器

所以	API的参数已经兼容微信官方平台. 所以请参考上面的URL, 再看官方给出的参数调用户即可. 

[微信企业号官方文档](http://qydev.weixin.qq.com/wiki/index.php?title=%E9%A6%96%E9%A1%B5)

## 云代码中的调用.
云代码基础知识, 请先参考 [上空云云代码说明](http://developer.skynology.com/cloud-code.html).

可在云代码中调用上述这些API. 并且在 [GO SDK](https://github.com/skynology/go-sdk) 中增加了几个方法.

### GOSDK
当云代码中直接调用上述微信相关API时, 可参考如下步骤.

#### 初始化微信参数
当您在调用微信相关API前, 需先直接要调用的微信平台. (如果项目只绑定了一个公众平台, 可省略此方法)

```go
// 第一个参数为在管理台绑定微信公众号后系统给出的应用ID.
// 第二个参数是公众号的类型. 也就是在API中 `/weixin/`后面的字段.
// 若是订阅号/服务号相关API, 传入 "MP", 企业号则传入 "corp".
app.InitWeixin('550273463a859a0488000001', 'mp')

```
当初始化微信参数后, `GO SDK` 准备了四个方法来发起`GET`, `POST`, `PUT`, `DELETE` 等请求.

#### 发起GET请求

```go
 func (app *App) GetWeixin(url string) (result map[string]interface{}, err *APIError)
```
menu
可传入想要发起的GET请求地址. `url`参数非完整的 HTTP URL. 而是微信API通用部分以外的URL. 如您要获取菜单信息时, 
`url`不用传入 `https://skynology.com/550273463a859a0488000001/menu`. 而是传入 `menu`即可. 前面的`https://skynology.com/weixin/`部分为通用信息. 

```go
// 获取所有菜单信息
result, err := app.GetWeixin("menu")
	
// 获取openid为 "o3ekhs_Lqq4QShTVfaC3eQw-7juc"的用户信息
result, err := app.GetWeixin("users/o3ekhs_Lqq4QShTVfaC3eQw-7juc")


```

#### 发起POST请求
```go
 func (app *App) PostWeixin(url string, data interface{}) (result map[string]interface{}, err *APIError)
```
发起POST请求, 基本调用同Get请求差不多, 只是多了一个 `data` 参数, 该参数为您要POST到服务上的数据.

#### 发起PUT请求
```go
 func (app *App) PutWeixin(url string, data interface{}) (result map[string]interface{}, err *APIError)
```
发起PUT请求, 基本调用同Get请求差不多, 只是多了一个 `data` 参数, 该参数为您要PUT到服务上的数据.


#### 发起DELETE请求
```go
 func (app *App) DeleteWeixin(url string, data interface{}) (result map[string]interface{}, err *APIError)
```
发起DELETE请求, 基本调用同Get请求差不多. 默认删除一个对像时, 只传url即可, 若调用批量删除API时, 可传入 `data` 参数.

### Helper
在云代码中默认引用了 [云代码交互类型](https://github.com/skynology/cloud-types)中微信部分的`corp`及`mp`等包.
所以在云代码中您可以很容易的创建或获取微信相关类型. 希望下列一些代码能让您理解此概念.

```go
	
	// 将微信服务器POST过来的XML转换为指定ReqText类型.
	reqText, err := mp.GetText(req.ExtraData)
	
	// 返回一个文本信息给刚发消息过来的用户.
	resText := mp.NewResText(reqText.FromUserName, reqText.ToUserName, time.Now().Unix(), "您好, 欢迎使用上空云服务&_&")
	h.Render(resText)

```