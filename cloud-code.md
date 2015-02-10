# 云代码使用说明
我们在后台提供了简易的云代码功能。 分`hook`及`自定义函数`两类。 云代码目前只支持Go语言版本。

## Hook
`Hook`为操作Resource时的自定义逻辑。 如保存前后，更新前后及删除前后等。 当你设置了云代码后， 系统将自动在您进行各类操作时调用。   
如下图， 点击 Hook时， 右侧功能区会列出你的项目中所有Resource的列表。 点击具体Hook的`编辑`按钮可跳到编辑区。

![Hook列表](http://7tebuf.com2.z0.glb.qiniucdn.com/cloud-code/hook.jpg?imageView/2/h/300)

其中 **编辑** 按钮有背景颜色表示:

* 红色: 按钮标识已经保存了云代码到服务器，但还未部署。
* 绿色: 标识已经部署过并正在运行的的云代码。
* 无背景色: 未云代码。


无论是Hook还是自定义函数， 都有三个传入的参数可调用。  
 
* app: 实例化好的 go sdk。以Master权限。 所以云代码对您的项目有高级管理权限。
* req: Request对象。 请参考 [Request对象](#Request对象)。
* h: Helper类。 请参考 [Helper类](#Helper)

云代码后台函数类似下面的格式:

```go
function YourCloudCode(app *skynology.App, req *CloudRequest, h *Helper) {
	// 这里是你在管理后台写的云代码。

}
```


## 自定义函数
当默认的Resource+Hook的方式无法满足您的需求， 想自己写些简单的逻辑并返回信息给前台时。 可创建**自定义函数**。

可在项目管理后台云代码功能区设置自定义函数。
![自定义函数列表](http://7tebuf.com2.z0.glb.qiniucdn.com/cloud-code/func.jpg?imageView/2/h/300)

当新增或编辑函数时， 在代码编辑页面上部是函数名及函数描述。 函数名必须由英文字母组成。 描述只用于描述你创建函数的用途， 便于区分。

![函数编辑](http://7tebuf.com2.z0.glb.qiniucdn.com/cloud-code/function-editor.jpg?imageView/2/h/300)

当写完函数内云代码后， 需先保存到服务器。 再发布到云代码服务器后才生效。
> 函数名在创建过后不允许再修改。 若有此需求， 请另增加一个函数即可。

自定义函数发布后， 可调用RESTfulAPI的基础地址+'functions/{name}'来调用。如我们调用上面的calendar函数时, 可往下面地址发送一个GET请求即可。 

```http
https://skynology.com/api/1.0/functions/calendar
```
> 自定义函数也是需要项目验证的。详情请参考 [RESTful 签名](/restful-api.html#签名)

## Request对象
在云代码中可以获取用户前台Post的信息及Session信息等。 在云代码中以 `req` 命名传入。 详细类型参考下面的结构体(CloudRequest)。

```go
type CloudRequest struct {

	// 资源Id.
	// 在对单个资源操作时才会有值
	ObjectId string 

	// Post 或 Put 等操作时前台传入的值或Get要返回的对象
	Data map[string]interface{} 

	// 用户操作时的Session对象
	Session map[string] 
}

// 云代码Session
type CloudSession struct {

	// Session 用户Id
	UserId string 

	// 是否以Master Key权限调用
	Master bool

	// 用户角色列表
	Roles []string
}
```

## Helper
为了方便您写云代码。 我们封装了一个简单的Helper类。 并以 **h** 为名传入到您的云代码中。

### Set
修改Object的字段值。   
*可用于: `保存前`，`更新前`，`查询时`*

```go
h.Set(fieldName string, value interface{})
```
如我想在某个Resource的保存前检查title字段是否大于10， 若长度大于10将截取前面10个字符时， 可在hook列表的`保存前`中写如下云代码。

```go
title := req.Data["title"].(string)
if len(title) > 10 {
	h.Set("title", title[:10])
}

```

### Protect
保护某个字段不被修改。   
*可用于: `更新前`*

```go
h.Protect(fieldName string)
```
当设置了Protect的字段无法进行更新修改。 

如只允许用户修改自己账户的密码, 可在Resource(_User)的`更新前`hook中设置:

```go
if req.Session.UserId != req.ObjectId {
	h.Protect('password')
}
```
当一个用户想更新其他用户的账户密码时， 将不会成功， 但也不会提示信息给前台。

### Cancel
取消操作并返回API错误码给前台。   
*可用于: `查询时`, `保存前`, `更新前`, `删除前`*

```go
h.Cancel(message string)
```
当在用户操作时， 返回错误码给前台时， 可用此方法。 默认Code:-1.   
**当此方法并非最后一行时，注意后面跟一行 return**

如上面的Protect的列子， 当要更新其他用户的密码时， 提示错误信息给用户时:

```go
if req.Session.UserId != req.ObjectId {
	h.Cancel('您没有权限修改其他用户的账户密码')
	return
}
// 若这里还有其他代码， 上面的Cancel后需跟return， 以防止执行全部代码。
```
此时RESTful服务器将返回错误码给用户，http状态码为:400。 内容如下:

```json
{
	"code": -1,
	"error": "您没有权限修改其他用户的账户密码"
}
```


### CancelWithCode
同Cancel， 只是多了自定义Code.

```go
h.Cancel(message string, code int)
```

### Log
记录Log到服务器。 可在管理平台的云代码功能里查看Log.    
*可用于: 所有Hook*

```go
h.Log(message string)
```

> 普通用户的项目只保存2天内的Log. 

### Render
打印Json信息到前台。

```go
h.Render(data interface{})
```

*只在自定义函数中可用。 在所有Hook中无法使用此方法*