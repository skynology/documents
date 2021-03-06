# 上空云 基础概念

上空云是一个提供云端后台数据存储为主， 继承周边功能集成的平台。 为减轻开发人员的后台开发成本， 并提供完善及全面的第三方服务集成。 让开发人更专注自己的 "前端"开发。

## 基础模块
我们主要以应用数据存储为主。在管理后台创建好您想要的资源表(resource)， 即可调用对应的 RESTful API。RESTful 详细说明， 请看[RESTful API文档](http://developer.skynology.com/restful-api.html)

### 资源表
后台数据是以资源表=Resource来做为基础存储。 并每个Resource下可以存方无数的Object。每个Object可以设置多个Attribute. 我们把这些概念对比下传统的数据库将更容易理解。

上空云|数据库|备注
-----|-----|---
资源(resource)|表(table)|-
对象(object)|行(row)|-
字段(attribute)|列(column)|-

我们后台是用NoSQL来做为存储介质。 所以整个数据是 模式自由(Scheme-Free)的。所以您可以任何时候对 Object 增加或删除字段(Attribute).

#### 数据类型
资源表提供比传统数据库更多的数据类型， 方便大家操作时更加简便合理。

类型|说明|备注
---|----|---
String|字符串型|
Number|数值型|包括整形及浮点型
Boolean|布尔型| true 或 false
Date|日期时间型|存放格式采用ISO 8601格式。不带时区。
Object|JSON 对象类型|JSON 对象，如 {"name":"skynology","location":"SZ"}
Array|数组型|JSON 数组， 可包含我们所有支持的所有类型
ACL|权限描述类型|Access Control List. 对对象的访问权限说明。 参考 [ACL说明](#ACL)
Pointer|对象关联类型|存放另外一个Object 的 objectId 字段。方便做类似传统数据库的外键关联。
SessionUser|当前登陆用户Id|特殊的Pointer类型, 自动保存当前已经登陆用户的objectId.
Relation|对象关联列表|特殊数组类型， 存放Pointer字段。也就是一个以 objectId 组成的列表。
GeoJSON|地理位置|GeoJSON为表示地理位置信息的JSON对象。参考 [GeoJSON Spec](http://geojson.org/geojson-spec.html)


#### 默认字段
每个Object创建后， 系统会增加如下几个默认字段。 这些字段一般为不可写。由系统自动生成。

* objectId: 对象的Id标识, 主键。 全局唯一。格式一般为24位字符串。如: "54abd2a29760e369ca000001"。
* ACL: 权限字段。 默认为公开权限， 所为人都有读写权限。如: {"*":{"read":true,"write":true}}。
* createdAt: 创建日期。 当Object被创建时， 系统自动生成的时间。
* updatedAt: 对象修改时间。 每次对Object做修改时，系统自动生成的时间。 默认同 createdAt 相同。

### ACL
每个 Object 都有一个字段为`ACL`， 用来描述此 Object 的读写权限说明。 值为JSON格式, ACL只允许用 [用户](#user) 的 `objectId`, [角色](#role) 的 `name` 字段以及几个通配符来授权. 如下一个Object:

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

上面 Object 的 `ACL` 表示, 对当前对象(`5458e0da9d40a82718000001`), 只有 `objectId` 为 `5458e0da9d40a82718111111`的用户有读取及写入权限; 名称(`name`)为 `employee`的角色有读取权限, 但无写入权限.

从上面可以看出, 按用户来授权时, ACL的Key直接设置用户 `objectId` 即可, 用角色来授权时, 用 `'role': + '角色名(name)'` 来授权.
授权值有 `read` 标识读取权限, `write`来标识写入权限(包括`更新`, `删除`).

当一个对象是完全开放时， 可用通配符 星号(*) 来做为key. 如下面的设置代表所有人都有阅读权限， 但无编辑权限(包括更新,删除):

```json
{
	"ACL":{
		"*": {"read":true, "write":false}
	}
}
```

有些场景， 我们只允许登录的用户才可查看或编辑。 此时用 通配符 **井号(#)** 来做为Key。 如下面设置代表， 只有已经登录的用户才可查看及编辑。

```json
{
	"ACL":{
		"#": {"read":true, "write":false}
	}
}
```
>
> 除了针对每一条数据设置ACL来做访问控制外, 我们也可以给每个Resource的`GET`,`FIND`,`POST`,`PUT`以及`DELETE`等
> RESTful方法设置访问控制权限. 请参考 [设置资源访问权限](/console-tutorial.html#设置资源访问权限)


## 默认资源表
当你在管理控制台创建一个项目时， 系统自动为您创建了几个常用的资源表， 以方便您操作。 系统默认创建的表一般用 下划线(_)来开头。 所有的系统表都是继承自默认的资源表， 所以会有默认字段 `objectId`,`ACL`,`createdAt`, `updatedAt`。 除了这些每个表都会有自己的所需的一些额外默认字段。 默认字段一般不可以修改。

### 用户表
用户表 "_User" 为系统创建的默认资源表， 用于存放系统用户信息。 用户表除了所有表外。 还默认加了一些字段。 主要几个如下：

字段名|必填|说明|备注
----|:----:|-------|----
username|\*|登录用户名| 
password|\*|登录密码| 
email|\_|邮箱地址|
phone|\_|手机号码|
emailVerified|\_|邮箱验证标记| 默认为空， 当用户验证了邮箱地址后， 会变为 true.
phoneVerified|\_|手机验证标记| 默认为空， 当用户验证了手机号后， 会变为 true.
sessionToken|\_|session标记|用户登录后， 保存的session标记。

### 角色表
角色表 "_Role" 为存放项目用户角色的地方。

字段名|必填|说明|备注
----|:----:|-------|----
name|\*|角色名|每个角色的名字， 全局唯一。 名字需以英文字符串及数字组成， 不可包含特殊字符及中文单词。
users|\_|用户列表|角色包含的用户Id列表。 Relation类型。
parents|\_|父级角色列表|角色名组成的列表。Relation类型。


### 文件表
角色表 "_File" 是用于记录上传的文件信息。除了默认字段外， 还有增加了以下一些常用的字段。 多数字段内容无法修改， 都由后台自动生成。

字段名|必填|说明|备注
----|:----:|-------|----
key|\*|文件标识|为文件唯一标识， 一般为系统自动生成。
name|\_|文件名|为上传时的文件名。
size|\_|文件大小| 上传的文件大小。 单位为: 字节(Byte)。
bucket|\_|空间名|上传文件在云端存放的空间。
mimeType|\_|文件信息|
ext|\_|文件扩展名|
hash|\_|文件的hash值|用于区别相同文件。
exif|\_|照片参数信息|若上传的文件是图片类型，并且的exif信息， 将保存在此。Object类型。
imageInfo|\_|图片信息
createUser|\_|上传人员|保存上传用户Id。

### 邮件模板表
邮件模板表 "_EmailTemplate" 是存放邮件模板的地方。 当您调用发送邮件功能时， 将从此表找相对应的模板文件。

字段名|必填|说明|备注
----|:----:|-------|----
name|\*|模板名称|模板名称， 唯一性要求。 有些特殊邮件将由名称来查找， 如验证用户邮箱等。
subject|\*|邮件标题|存放发送邮件时的标题信息。 内容可以有占位符。
content|\*|邮件正文|邮件正文模板。可含占位符。

### 短信模板表
短信模板表 "_SMSTemplate" 为存放短信模板文件的地方。 当您调用发送短信的功能时， 将从此表找模板文件。

字段名|必填|说明|备注
----|:----:|-------|----
name|\*|模板名称|模板名称，全局唯一性要求。
templateId|\*|模板Id|一般指第三方提供商的模板Id。
content|\*|模板正文|短信内容模板。可含占位符。


## 第三方集成
我们的文件存储，短信， 邮件服务， 移动推送等功能都支持集成第三方服务。让您自己来选择使用哪一家的服务。

设置这些功能， 请您参考 [管理控制台使用说明](http://developer.skynology.com/console-tutorial.html) 和 [RESTful API使用说明](http://developer.skynology.com/restful-api.html)

> 若您有想集成的服务， 或有第三方服务商推荐， 请直接联系我们哦^_^.
> 可发送邮件到 sdk[AT]skynology.com。