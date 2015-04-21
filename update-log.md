# 系统更新日志

## 2015-04-21 16:00

### 控制台
* 增加资源访问权限功能. 明细请查看 [设置资源访问权限](/console-tutorial.html#设置资源访问权限)

## 2015-04-17 23:00

### RESTful API
* 查询时增加Include功能. 可把`Pointer`及`Relation`字段对应的资源给查出来.
* 对一些api增加Master签名要求, 如微信平台相关的api.
* 增加cors. 默认对 `skynology.com`相关的URL.

### GO SDK
* 增加RESTful API的Include对应的方法

### 微信SDK
* 取消了URL中带的`type`及`id`, 放Header中. 若项目只绑定了一个微信平台, 可不设置这两个Header.
* 修改一些已知bug.

## 2015-03-30 00:20

### RESTful API
* [增加地图相关API](/restful-api.html#地图相关)
* [增加数组内修改数据内对象方法](/restful-api.html#数据内对象)
* [增加微信SDK](/weixin-api.html)
* [增加直接从URL下载文件到服务器API](/restful-api#抓取指定URL文件)
* 数据类型增加`SessionUser`.
* 云代码增加执行前有效性检查
* 云代码调用等待时间从3秒改为5秒
* 修复其他已知bug

### GO SDK
* 增加调用微信相关方法
* Object增加SetMulti方法, 可直接传入一个 `map[string]interface{}`

### 控制台
* 改变数据操作方式. 从以往双击弹出modal, 变为双击变为输入框
* 云代码增加微信相关hook
* 组件增加绑定微信公众号及地图功能

### 其他
