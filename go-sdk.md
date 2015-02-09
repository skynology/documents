# 上空云 GO SDK 使用说明
我们上空云后台服务是用GO语言写的。 所以GO SDK我们是不能少的。

## 安装

### go get安装
直接在命令行执行下面命令即可安装到本地

```go
go get -u github.com/skynology/go-sdk
```

您也可以直接从github上下载SDK源码安装。

[GO SDK](https://github.com/skynology/go-sdk)


### 初始化SDK
当安装完成后， 需要在您的项目中引用库文件。

```go
import "github.com/skynology/go-sdk"

```

并且初始化app对象。 需传入 `applicationId` 和 `applicationKey`。

```go
app := skynology.NewApp('application id', 'application key')

```

## 对象
`Object` 类是用来操作 `对象`。 `Object` 所操作的资源类， 需在 [管理后台](https://console.skynology.com) 中已经创建过的。 若未在后台创建的类， 在保存，更新， 删除等操作时， 将直接返回错误码。

如同 `基础概念` 中讲的一样， 对象有几个默认字段。 在 `Object` 中有一些默认属性。如下:

* `ObjectId`: 对象Id字段。
* `ACL`: 对象权限字段。
* `CreatedAt` 及 `UpdatedAt`: 对象创建时间及最后更新时间。
* `ResourceName`: 当前对象对应的资源类名。(一般是您初始化`Object`时传入的。) 

### 创建对象操作类

```go

// 创建Object对象, 如操作 'MyUser' 这个 Resource内的对象
object := app.NewObject('MyUser')

// 用资源名及 objectId 来初始化， 一般可用于更新已经存在的对象
object := app.NewObjectWithId('MyUser', '520c7e1ae4b0a3ac9ebe326a')


```

### 设置字段

当初始化完成后， 我们可以用 `Set` 这个方法来设置对象的字段。 `Set`包含两个参数， 第一个为字段名，第二个参数为字段值。 字段值需和后台设置的数据类型相匹配。


```go

// 创建Object对象
object := app.NewObject('MyUser')
object.Set("name", "Jacky Chan")
object.Set("age", 50)
object.Set("likes", []string{'python', 'golang', 'javascript'})
object.Set("married", true)


```

### 更新字段
更新字段同 [设置字段](#设置字段) 基本上是相同的， 普通更新直接使用上面的 `Set` 这种方式即可，我们会SDK中计算出您设置了哪些字段， 并且只更新您设置过的字段。

除了上面说到的更新字段方式外，我还有一些特殊的更新字段方式。

#### 计数器
我们除了在创建资源类时可以设置字段为 **自动更新** 以外， 也可以在SDK中**递增**或**递减** `数值` 类型字段的值。 您也可以直接更新字段值， 但是若有多人同时做了相同的更新值操作， 将会发生覆盖先更新的值的情况。您使用 `计数器` 方式去更新时， 将不会发覆盖的情况。

您可用下面的方式去**递增**或**递减**值:

```go

// 关注人数自动加一
object.Increment("followCount")

// 关注人数增加5
obj.Increment("followCount", 5);

// 关注人数减一
obj.Increment("followCount", -1);

// 关注人数减5
obj.Increment("followCount", -1);
```

#### 数组字段
当您需要更新数组字段时， 除了上面的直接设置 `Set`方式去覆盖外， 也可以用下面提供的几个方法去更新数组内的值。


```go

object := app.NewObjectWithId("MyUser", "520c7e1ae4b0a3ac9ebe326a")

// 对 likes 数组内增加3个值。 当前方法不检查值的唯一性。 如下面的 'golang' 在数组内出现两个。
object.AddValueToArrayFromList("likes", []string{"golang", "c#", "php"})


// 对 likes 数组内增加2个值。 将检查值的唯一性。 如下面的 'golang' 将不会被加到数组内。
object.AddUniqueValueToArrayFromList("likes", []string{'golang', 'java'});

// 从数组内删除2个值
object.RemoveFromArrayFromList("likes", []string{"python", "javascript"});


// 也可单独增加或删除一个值
object.AddValueToArray("likes", "php")
object.AddUniqueValueToArray("likes", "php")
object.RemoveFromArrayFromList("likes", "php")

```
#### ACL字段
因 **ACL** 字段为一个特殊的JSON Object类型。 所以我们在GO SDK中提供了几个常用的方法来设置此字段。

##### 用户Id来设置

```go
// 设置读权限
object.SetReadAccessByUserId("54a923429760e37fd1000002", true)
// 设置写权限
object.SetWriteAccessByUserId("54a923429760e37fd1000002", false)

```

上面设置了用户 `54a923429760e37fd1000002` 对当前对象的权限。用户有查看权限， 而无更新或删除权限。
除了普通用户Id外， 也支持特殊通配符。有以下通配符:

* 星号(*): 型号代表所有人。 包括未匿名用户。 当未设置对象ACL时， 默认就是所有人(\*)都有读写权限。
* 井号(#): 井号代表已经登录用户。 需先使用 [用户登录功能](#用户) 登录过才能查看或编辑此对象。

##### 用角色名来设置 

```go
// 设置读权限
object.SetReadAccessByRoleName("admin", true)
// 设置写权限
object.SetWriteAccessByRoleName("admin", false)
```
上面设置了名称为 `admin` 的角色对当前对象有读写权限。 角色相关概念， 

### 保存对象
当设置完类的字段后， 可以使用 `Save` 方法来保存当前的对象。 保存会有两种情况， 一种是完全**新增**时； 另一种是**更新**已有对象。 不管是哪一种， 我们都是用同一个方法去保存。 SDK后台会自动分辨出是属于哪一种。

`Save` 方法返回两个值， 第一个为`bool`类型。判断保存是否成功。 第二个为`APIError`对象。 当保存失败时可从这里得到失败信息。


```go
object := app.NewObject('MyUser')
object.Set("name", "Jacky Chan")
object.Set("age", 50)

successed, apierror := object.Save()

```

> 当保存是新增操作时, 会更新当前对象的`objectId`字段。

### 查询对象
当想获取指定 Object 信息时， 可用`Query`类的`GetObject`方法来获取对象。

```go

query := app.NewQuery("MyUser")

object, apierror := query.GetObject("520c7e1ae4b0a3ac9ebe326a")

```


### 删除对象
删除对象时， 可调用 `delete` 方法。

```go
object := app.NewObjectWithId("MyUser", "520c7e1ae4b0a3ac9ebe326a")
successed, apierror := object.Delete()
if successed {
	// do something
}

```


## 查询
在你引用的SDK下 `Query` 类用于查询资源对象。

### 普通查询
当查询多条记录时， 可用下面方法去初始化 `Query` 对象。

```go
// 创建查询对象
query := app.NewQuery("MyUser")

// 设置查询条件, 如找名字为 'Jet Lee'的人员
query.Equal("name", 'Jet Lee")

```

不等于

```go

// 设置查询条件, 如找名字不等于为 'Jacky Chan'的人员
query.NotEqual("name", "Jacky Chan")

```

针对数值类型， 可以使用比较方法。

```go
// 小于: age < 30 的人员
query.LessThan("age", 30)

// 小于等于: age <= 30 的人员
query.LessOrEqual("age", 30)

// 大于: age >= 30 的人员
query.GreaterThan("age", 30)

// 大于等于: age >= 30 的人员
query.GreaterOrEqual("age", 30)
```

针对字段串类型的字段。 提供了下面的方法。

```go
// 代表查询所有名字为 'Jacky' 开头的人员。
query.StartWith("name", "Jacky");

// 代表查询所有名字为 'Lee' 结尾的人员， 如 'Jet Lee' 将被搜索到。
query.EndWith("name", "Lee")

// 代表查询所有名字中包含 'Mike' 的人员。如 'Mike Jordan', 'Mike Jackson'。
query.Contains("name", "Mike");
```

可以用 `In` 或 `NotIn` 方法来查询匹配给出数组中的值。

```go
// 只要年龄为 30, 35, 50 中的任何一个， 就会被查询到。
query.In('age', []float64{30, 35, 50});

// 查询名称不是 'Jet Lee' 和 'Jacky Chan'的人员
query.NotIn('name', []string{"Jet Lee",  "Jacky Chan"]);
```

针对数组类型的字段， 也可用上面的 `in` 及 `notIn` 方法。 如:

```go
// 只要某人员的 `likes`列表里有 'golang', 'python' 就会被查询到。
query.In("likes", []string{"golang", "python"});

```

除了 `In` 及 `NotIn`, 我们还提供了一个方法 `MatchAll`。 用来查询匹配所有给出值的对象。 如:

```go
// 查询 `likes` 字段包含 'golang' 和 'python' 两个值的人员
query.MatchAll("likes", []string{"golang", "python"});
```

除了基本的查询条件外， 我们也提供了查询数量及排序等功能。

```go
// 先用名字排序， 再用年龄从大到小排。
query.OrderBy('name').OrderByDescending('age');

// 忽略60条记录， 再从第61个开始返回20条数据
query.Skip(60).Take(20);

```

上面的 `Skip` 及 `Take` 可以做分页。 但一般我做分页时都需知道查询到的数据总数。 此时我们可以设置 `Count` 来返回总数。

```go
query.Count(true);
```

当设置完所有条件后，我们用 `Find` 方法来获取对象。此方法也可返回三个值, 第一个为查询到的`Object`数组， 第二个查到到的总数据数， 第三个是	`APIError` 对象。

```go

objectList, totalCount, err := query.Find()
if err != nil {
	// return?
}


```

## 用户
在SDK中专门提供了一个 `User`类来操作用户。 用户类继承自 `Object`类， 所以多数操作相同。 只是增加了一些特殊的操作， 如 用户注册， 登录， 登出等。

### 字段
在`User` 中除了 `Object` 默认的字段外， 另外增加了 `UserName`, `Email`, `Phone` 以及 `SessionToken` 等字段。


### 注册用户
注册用户时， 可调用 `Register` 方法。 其中`UserName`及`Password`为必须填写的字段。

```go

user := app.NewUser()
user.UserName = "Jet Lee";
user.Password = "qr4hFHEG4k1z5suHLOuc";
user.Email = "jetlee@email.com";
user.Phone = "18600000000";
user.Set("age", 40)
user.Set("likes", []string{"javascript", "bash", "c#"})

successed, err := user.Register()
if successed {
	// do something...
}

```

### 用户登录
我们提供了如几个方法来做用户登录。

1. 用户名 + 密码 来登录

```go
// err 为 APIError对象
user, err := app.LoginWithUserName("jetlee", "qr4hFHEG4k1z5suHLOuc") 

```

2. 邮箱地址 + 密码 来登录

```go
// err 为 APIError对象
user, err := app.LoginWithUserName("jetlee@email.com", "qr4hFHEG4k1z5suHLOuc") 
```
3. 手机号码 + 密码 来登录

```go
// err 为 APIError对象
user, err := app.LoginWithUserName("18600000000", "qr4hFHEG4k1z5suHLOuc")
```

### 当前已登录用户
当用户登录过后， 会将用户信息保存到本地。 以后可直接用下面的方法来获取已登录的用户信息。

```go

// 若已经有登录过， 将会返回登录的用户。
user := app.CurrentUser()
if user == nil {
	// do something
}

```

### 登出用户
当用户已经登录后， 默认是没有失效时间的。 可调用下面的方法来登出用户。

```go
successed, err := user.Logout()

```


### 删除用户
因 `User` 是继承自 `Object`, 所以删除时， 可直接用 `Object` 的 `Delete` 方法。

```go
successed, err := user.Delete()
```

## 角色
我们不太建议在SDK中去操作角色， 因为关系到整个系统的权限问题。 所以您可以在管理后台操作或用 [Restful API](http://developer.skynology.com/restful-api.html#角色) 来操作角色。