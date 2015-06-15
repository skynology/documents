# OAuth2 接口使用指南
为了方便大家用第三方登陆您的项目, 我们提供了简单的OAuth接口. 若您对我们的RESTFul接口还不熟悉, 请先阅读: [上空云RESTFul API文档](#restful-api.html).   
若您有其他OAuth相关需求, 请直接在联系我们, 我们会在第一时间评估, 并集成.


## 绑定OAuth账号
若需要使用OAuth相关API时, 先在我们的[管理台](#console-tutorial.html)绑定您的公众号信息. 在`组件`下的`OAuth`子页面进行绑定.


## API列表

### 新浪微博
URL|Method|Description
---|------|-----------
oauth/weibo/url |POST| 获取登陆URL
oauth/weibo/token |POST| 获取access token, 需传code
oauth/weibo/login |POST| 用code登陆系统
 
### 腾讯QQ
URL|Method|Description
---|------|-----------
oauth/weibo/url |POST| 获取登陆URL
oauth/weibo/token |POST| 获取access token, 需传code
oauth/weibo/login |POST| 用code登陆系统


## 新浪微博

#### 获取URL
可POST如下JSON数据到 `https://skynology.com/api/1.0/oauth/weibo/url`上, 为了安全起见, 需传入一个 `request_key`, 等后续操作要带上 request_key,  以便服务器区分用户调用.

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
	
	"redirect_uri":"",
	"code": "刚得到的code"
}
```
当code正确时, 将返回access_token等信息.

```json
{
	"access_token": "ACCESS_TOKEN",
	"expires_in": 1234,
	"uid":"12341234"
}
```

#### 登陆上空云平台
code除了换取access_token以外, 也可以直接登陆上空云服务用户. 当登陆成功后,  返回用户信息, 并会附带字段`SessionToken`, 以便后续调用RESTful API时判断用户信息. 可POST数据到 `https://skynology.com/api/1.0/oauth/weibo/login`.

```json
{
	// 在 获取URL 时传入的 request_key
	"request_key": "test",
	
	"redirect_uri":"",
	"code": "刚得到的code",

	// 在 '_User' 表中绑定微博uid的字段名, 下列名为'uid'
	"field": "uid"
}
```
若您通过 `换取access_token` 步骤拿到了 `access_token` 及 `uid`. 可直接传入上面的json中, 此时系统将不会再读取 `code` 字段, 直接使用您传的`uid`字段去查询用户. 当成功登陆时, 将返回用户信息及 `sessionToken`字段.


```json
{
	"objectId": "5506e16e3a859a3ab5000001",
	"username": "William",
	"email":"email@email.com",
	....

	//　session信息, 默认有效为30天, 
	// 可自行传入 'expired_in`字段来控制, 单位为秒
	"sessionToken":"127b296a487694bfd6760a0b48df8beab7fe2908",
	
	"_weiboToken": {
        "access_token": "xxxxx...xxx",
        "uid": "xxxx...xxx"
    }
}
```


## 腾讯QQ

#### 获取URL
可POST如下JSON数据到 `https://skynology.com/api/1.0/oauth/qq/url`上, 为了安全起见, 需传入一个 `request_key`, 等后续操作要带上 request_key,  以便服务器区分用户调用.

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
当上一步得到code后, 可用此code换取网页授权access_token. POST刚得到的code到 `https://skynology.com/api/1.0/weixin/qq/token`上即可换取access_token等信息.

```json
{
	// 在 获取URL 时传入的 request_key
	"request_key": "test",
	
	"redirect_uri":"",
	"code": "刚得到的code"
}
```
当code正确时, 将返回access_token等信息.

```json
{
	"access_token": "j5wewKnZLKAmWN7IjjKz",
	"expires_in": 1234,
	"refresh_token": "FeOrXfTaKY9PjYfpX8uX",
	"openid":"mGwFuslX1Wyc9VEyQAsauuhayQv2ak"
}
```

#### 登陆上空云平台
code除了换取access_token以外, 也可以直接登陆上空云服务用户. 当登陆成功后,  返回用户信息, 并会附带字段`SessionToken`, 以便后续调用RESTful API时判断用户信息. 可POST数据到 `https://skynology.com/api/1.0/oauth/qq/login`.

```json
{
	// 在 获取URL 时传入的 request_key
	"request_key": "test",
	
	"redirect_uri":"",
	"code": "刚得到的code",

	// 在 '_User' 表中绑定微博openid的字段名, 下列名为'openid'
	"field": "qqopenid"
}
```
若您通过 `换取access_token` 步骤拿到了 `access_token` 及 `openid`. 可直接传入上面的json中, 此时系统将不会再读取 `code` 字段, 直接使用您传的`openid`字段去查询用户. 当成功登陆时, 将返回用户信息及 `sessionToken`字段.


```json
{
	"objectId": "5506e16e3a859a3ab5000001",
	"username": "William",
	"email":"email@email.com",
	....

	//　session信息, 默认有效为30天, 
	// 可自行传入 'expired_in`字段来控制, 单位为秒
	"sessionToken":"127b296a487694bfd6760a0b48df8beab7fe2908",
	
	"_qqToken": {
        "access_token": "xxxxx...xxx",
        "refresh_token": "rT1LmXudZLq3hT94Yar4"
        "openid": "xxxx...xxx"
    }
}
```