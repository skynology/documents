# 微信平台接入指南
为了方便大家给自己的项目接入微信公众平台服务, 我们在RESTFul API中集成了微信公众号服务. 若您对我们的RESTFul接口还不熟悉, 请先阅读: [上空云RESTFul API文档](#restful-api.html).

## 基础概念
调用微信相关API的URL格式如下: `https://skynology.com/api/1.0/weixin/{type}/{id}/{url}`.   
其中:

* type: 为公众号类型, 如 `mp`, `corp`等.
* id: 在控制台组件中,绑定了微信公众号后, 由系统产生的id.
* url: 具体功能url部分, 如增加部门时, 是`department`

调用时, 绝大多数可参考微信开发平台文档. 尤其POST格式的, 基本都是按微信公众平台文档上那些参数. 及少数会为了方便调用做了修改, 修改过的会标明出来, 无标明的就表示兼容微信文档上的参数.  
 
[企业号开发文档](http://qydev.weixin.qq.com/wiki/index.php)   
[订阅/服务号开发文档](http://mp.weixin.qq.com/wiki/home/index.html)

## 绑定微信公众号
若需要使用微信相关API时, 先在我们的[管理台](#console-tutorial.html)绑定您的公众号信息. 如下图:    
![微信绑定](http://7tebuf.com2.z0.glb.qiniucdn.com/console-tutorial/wechat-setting.jpg?imageView/2/h/200)

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
URL|Method|Description
---|------|-----------
weixin/mp/\<id>/menu|GET|获取菜单
weixin/mp/\<id>/menu|POST|创建菜单
weixin/mp/\<id>/menu|DELETE|删除菜单
weixin/mp/\<id>/group|GET|获取用户分组
weixin/mp/\<id>/group| POST |创建用户分组
weixin/mp/\<id>/group|PUT|更新用户分组
weixin/mp/\<id>/group/users|POST|移动用户分组
weixin/mp/\<id>/group/users/batch|POST|批量移动用户分组
weixin/mp/\<id>/users| GET |获取用户列表
weixin/mp/\<id>/users/{openId}| GET |获取用户基本信息
weixin/mp/\<id>/users/updateremark|PUT|设置备注名
weixin/mp/\<id>/qrcode|POST|创建二维码
weixin/mp/\<id>/qrcode/upload|POST|下载二维码并传到文件服务器
weixin/mp/\<id>/shorturl|POST|长链接转短链接
weixin/mp/\<id>/oauth2/url|POST|创建OAuth2调用URL, 用于获取code
weixin/mp/\<id>/oauth2/token|POST|通过code换取网页授权access_token
weixin/mp/\<id>/oauth2/refresh|POST|刷新access_token
weixin/mp/\<id>/oauth2/userinfo|POST|拉取用户信息(需scope为 snsapi_userinfo)
weixin/mp/\<id>/oauth2/check|POST|检验授权凭证（access_token）是否有效
weixin/mp/\<id>/template/industry|POST|设置所属行业
weixin/mp/\<id>/template|POST|增加模板
weixin/mp/\<id>/template/send|POST|发送模板消息
weixin/mp/\<id>/media/upload|POST|上传媒体文件到文件服务器
weixin/mp/\<id>/news|POST|上传图文消息素材
weixin/mp/\<id>/mass/send/{to}|POST|高级群发消息, {to}为群发类别, 比如 预览(preview), 分组(group), 按用户openId列表(list)等等值.
weixin/mp/\<id>/mass/{msgId}|DELETE|删除群发

### 用户管理
用户管理相关API内容都兼容微信官方API. 可参照上面的调用URL及官方参数格式调用.


### 发送消息

#### 高级群发消息
高级群发消息, 可发送POST请求到 `https://skynology.com/weixin/mp/<id>/mass/send/{to}`. 
其中{to}为发送群发消息方式, 值可选 `group`, `list`, `preview`三个. 分别对应按分组(所有人)群发, 按用户`openid`列表群发, 以及预览方式等. 
如发送给组别为2的用户发送一条文本消息, 可POST如下JSON到`https://skynology.com/weixin/mp/550273463a859a0488000001/mass/send/group`.


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
可POST如下JSON数据到 `https://skynology.com/api/1.0/weixin/mp/550273463a859a0488000001/oauth2/url`上, 

```json
{
	"redirect_uri":"",
	"state":"",
	"response_type":"",
	"scope":""
}
```
成功后后台会返回编码过后的URL.

```json
{
	// 编码过后的URL
	url: ""
}
```
当转到此URL后, 微信服务器会转到上面你传入的`redirect_uri`对应的地址上, 并带有code及state等参数.

#### 换取access_token
当上一步得到code后, 可用此code换取网页授权access_token. POST刚得到的code到 `https://skynology.com/api/1.0/weixin/mp/550273463a859a0488000001/oauth2/token`上即可换取access_token等信息.

```json
{
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
code除了换取access_token以外, 也可以直接登陆上空云服务用户. 当登陆成功后, 返回上面的access_token等信息外, 还会返回`SessionToken`, 以便后续调用RESTful API时判断用户信息. 可POST数据到 `https://skynology.com/api/1.0/weixin/mp/550273463a859a0488000001/oauth2/login`.

```json
{
	"code":"刚得到的code",

	// 在_User表中存放用户微信openId的字段名
	// 需已经把openId存到此字段上. 否则无法找到对应用户.
	"field": ""
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

#### 上传媒体文件到文件服务器
当您有媒体文件Id(media_id)时, 可将此媒体文件上传到文件服务器, 以便永久保存. 可POST一个JSON数据到 `https://skynology.com/weixin/mp/550273463a859a0488000001/media/upload`, 如下:

```json
{
	// 您要上传的Media文件的ID
	"media_id":"qI6_Ze_6PtV7svjolgs-rN6stStuHIjs9_DidOHaj0Q-mwvBelOXCFZiq2OsIU-p"
}
```
成功上传后, 将会返回上传文件的`objectId`以及七牛的服务器返回的`hash`及`key`,如:

```json
{
    "hash": "Fq4E7h1myo2XXtXX2sHV5BkRRT-",
    "key": "Fq4E7h1myo2XXtX2sHV5BkRRT-.jpeg",
    "objectId": "55090a7e3a859a04ab000001"
}
```
### 二维码

#### 创建二维码
创建二维码的API是极少数修改原微信格式中的一个. 为了方便调用, 把JSON格式改成一个层次, 而非微信API中的多层次JSON. 创建一个临时二维码时, 可POST如下JSON到
`https://skynology.com/weixin/mp/550273463a859a0488000001/qrcode`:

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
当上一步创建成功后, 可用拿到的`ticket`把二维码图片上传到文件服务器上, POST如下JSON到 `https://skynology.com/weixin/mp/550273463a859a0488000001/qrcode/upload`即可.

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
URL|Method|Description
---|------|-----------
weixin/corp/\<id>/menu|GET|获取菜单
weixin/corp/\<id>/menu|POST|创建菜单
weixin/corp/\<id>/menu|DELETE|删除菜单
weixin/corp/\<id>/department|GET|获取所有部门信息
weixin/corp/\<id>/department/{departmentId}|GET|获取指定部门信息
weixin/corp/\<id>/department|POST|创建部门
weixin/corp/\<id>/department/{departmentId}|PUT|修改指定部门信息
weixin/corp/\<id>/department/{departmentId}|DELETE|删除指定部门信息
weixin/corp/\<id>/tag|GET|获取标签信息
weixin/corp/\<id>/tag|POST|创建标签
weixin/corp/\<id>/tag/{tagId}|PUT|修改指定标签信息
weixin/corp/\<id>/tag/{tagId}|DELETE|删除指定标签
weixin/corp/\<id>/tag/{tagId}/users|GET|获取指定标签下的用户列表
weixin/corp/\<id>/tag/{tagId}/users|POST|增加用户到指定标签下
weixin/corp/\<id>/tag/{tagId}/users|DELETE|从指定标签下删除用户
weixin/corp/\<id>/users/{userId}| GET |获取用户基本信息
weixin/corp/\<id>/users|POST|新增用户
weixin/corp/\<id>/users/{userId}|PUT|修改用户
weixin/corp/\<id>/users/{userId}|DELETE|删除用户
weixin/corp/\<id>/users|DELETE|批量删除用户
weixin/corp/\<id>/users/{userId}|POST|邀请用户
weixin/corp/\<id>/oauth2/url|POST|创建OAuth2调用URL, 用于获取code
weixin/corp/\<id>/oauth2/userinfo?agentId=&code=|GET|通过code换取用户信息
weixin/corp/\<id>/oauth2/signature|POST|获取JS签名
weixin/corp/\<id>/message|POST|发送消息
weixin/corp/\<id>/media/upload|POST|上传媒体文件到文件服务器

所以	API的参数已经兼容微信官方平台. 所以请参考上面的URL, 再看官方给出的参数调用即可. 

[微信企业号官方文档](http://qydev.weixin.qq.com/wiki/index.php?title=%E9%A6%96%E9%A1%B5)

## 云代码中的调用.
云代码基础知识, 请先参考 [上空云云代码说明](http://developer.skynology.com/cloud-code.html).

可在云代码中调用上述这些API. 并且在 [GO SDK](https://github.com/skynology/go-sdk) 中增加了几个方法.

### GOSDK
当云代码中直接调用上述微信相关API时, 可参考如下步骤.

#### 初始化微信参数
当您在调用微信相关API前, 需先直接要调用的微信平台. 

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
`url`不用传入 `https://skynology.com/weixin/mp/550273463a859a0488000001/menu`. 而是传入 `menu`即可. 前面的`https://skynology.com/weixin/mp/550273463a859a0488000001/`部分为通用信息. 

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