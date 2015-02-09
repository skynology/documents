# 上空云 Node.JS SDK 使用说明
现如今， Node.js 是越来越流行了， 我们也忍不住要为你提供相关 SDK, 以方便您的开发。该SDK是基于我们的RESTful API构建。并整个SDK风格是链式语句。 如果您用过 jQuery 库， 当能体会到此种风格的方便。

## 安装

### 使用NPM安装
使用如下命令可用 npm 来把SDK安装到您的项目中。

```go
npm install skynology --save
```

### 手动安装
可直接下载源码文件， 并加入到您的项目中即可使用。 下载地址: [上空云NodeJS SDK](https://github.com/skynology/nodejs-sdk)。

### 初始化SDK
当安装完成后， 需要在您的项目中引用库文件, 并且设置 `applicationId` 和 `applicationKey`。

```javascript
var sky = require("skynology");

sky.config.initialize('application id', 'application key');

```

## 对象

### 创建对象操作类
在你引用的SDK下 `Object` 这个方法用于操作资源对象。

```javascript
var sky = require('skynology');

// 可传资源表名
var obj = sky.Object('MyUser');

// 用资源名及 objectId 来初始化
var obj = sky.Object('MyUser', '520c7e1ae4b0a3ac9ebe326a');


```

### 设置字段

当初始化完成后， 我们可以用 `set` 这个方法来设置对象的字段。 `set`包含两个参数， 第一个为字段名，第二个参数为字段值。 字段值需和后台设置的数据类型相匹配。

```javascript

// 创建对象
var obj = sky.Object('MyUser');

obj.set('name', 'Jacky Chan').set('age', 50).set('likes', ['python', 'golang', 'javascript']);

obj.set('married', true)；


```

### 更新字段
更新字段同 [设置字段](#设置字段) 基本上是相同的， 普通更新直接使用上面的 `set` 这种方式即可，我们会SDK中计算出您设置了哪些字段， 并且只更新您设置过的字段。

除了上面说到的更新字段方式外，我还有一些特殊的更新字段方式。

#### 计数器
我们除了在创建资源类时可以设置字段为 **自动更新** 以外， 也可以在SDK中**递增**或**递减** `数值` 类型字段的值。 您也可以直接更新字段值， 但是若有多人同时做了相同的更新值操作， 将会发生覆盖先更新的值的情况。您使用 `计数器` 方式去更新时， 将不会发覆盖的情况。

您可用下面的方式去**递增**或**递减**值:

```javascript
var obj = sky.Object('MyUser', '520c7e1ae4b0a3ac9ebe326a');

// 关注人数自动加一
obj.increment('followCount');

// 关注人数增加5
obj.increment('followCount', 5);

// 关注人数减一
obj.increment('followCount', -1);


// 关注人数减5
obj.increment('followCount', -5);
```

#### 数组字段
当您需要更新数组字段时， 除了上面的直接设置 `set`方式去覆盖外， 也可以用下面提供的几个方法去更新数组内的值。

```javascript

var obj = sky.Object('MyUser', '520c7e1ae4b0a3ac9ebe326a');

// 对 likes 数组内增加3个值。 当前方法不检查值的唯一性。 如下面的 'golang' 在数组内出现两个。
obj.addToArray('likes', ['golang', 'c#', 'php']);


// 对 likes 数组内增加2个值。 将检查值的唯一性。 如下面的 'golang' 将不会被加到数组内。
obj.addUniqueToArray('likes', ['golang', 'java']);

// 从数组内删除2个值
obj.removeFromArray('likes', ['python', 'javascript']);


```


### 保存对象
当设置完类的字段后， 可以使用 `save` 方法来保存当前的对象。 保存会有两种情况， 一种是完全**新增**时； 另一种是**更新**已有对象。 不管是哪一种， 我们都是用同一个方法去保存。 SDK后台会自动分辨出是属于哪一种。

可给 `save` 方法传两个function。 第一个代表保存成功后需做的动作。 第二个为保存失败后需做的动作。

```javascript

var obj = sky.Object('MyUser');

obj.set('name', 'Jet Lee').set('age', 50).set('likes', ['golang', 'python'])
	.save(function(body) {
		// do something
	}, function(error){
		// 
	});

```

### 查询对象
当想获取指定 Object 信息时， 可用资源名+objectId方式来创建 `Query` 操作类。 再调用 `get`方法即可。

```javascript
var query = sky.Query('MyUser', '54a923429760e37fd1000002');

query.get(function(object){
	// 若找不到对象时， object 为 null;
}, function(error){
	// 获取出错时
});
```

### 删除对象
删除对象时， 可调用 `delete` 方法。

```javascript
obj.delete(function(data){
	// 删除成功
}, function(error){
	// do something
});
```


## 查询
在你引用的SDK下 `Query` 这个方法用于操作资源对象。

### 普通查询
当查询多条记录时， 可用下面方法去初始化 `Query` 对象。

```javascript
// 创建查询对象
var query = sky.Query('MyUser');

// 设置查询条件, 如找名字为 'Jet Lee'的人员
query.equal('name', 'Jet Lee');
```

不等于

```javascript
// 设置查询条件, 如找名字不等于为 'Jacky Chan'的人员
query.notEqual('name', 'Jacky Chan');
```

针对数值类型， 可以使用比较方法。

```javascript
// 小于: age < 30 的人员
query.lessThan('age', 30);

// 小于等于: age <= 30 的人员
query.lessOrEqual('age', 30);

// 大于: age >= 30 的人员
query.greaterThan('age', 30);

// 大于等于: age >= 30 的人员
query.greaterOrEqual('age', 30);
```

针对字段串类型的字段。 提供了下面的方法。

```javascript
// 代表查询所有名字为 'Jacky' 开头的人员。
query.startWith('name', 'Jacky');

// 代表查询所有名字为 'Lee' 结尾的人员， 如 'Jet Lee' 将被搜索到。
query.endWith('name', 'Lee');

// 代表查询所有名字中包含 'Mike' 的人员。如 'Mike Jordan', 'Mike Jackson'。
query.contains('name', 'Mike');
```

可以用 `in` 或 `notIn` 方法来查询匹配给出数组中的值。

```javascript
// 只要年龄为 30, 35, 50 中的任何一个， 就会被查询到。
query.in('age', [30, 35, 50]);

// 查询名称不是 'Jet Lee' 和 'Jacky Chan'的人员
query.notIn('name', ['Jet Lee',  'Jacky Chan']);
```

针对数组类型的字段， 也可用上面的 `in` 及 `notIn` 方法。 如:

```javascript
// 只要某人员的 `likes`列表里有 'golang', 'python' 就会被查询到。
query.in('likes', ['golang', 'python']);

```

除了 `in` 及 `notIn`, 我们还提供了一个方法 `matchAll`。 用来查询匹配所有给出值的对象。 如:

```javascript
// 查询 `likes` 字段包含 'golang' 和 'python' 两个值的人员
query.matchAll('likes', ['golang', 'python']);
```

查询某个字段有值的对象。 如下面语句， 
```javascript
//将查询 `age` 这个字段有值的对象。
query.exist('age');

// 还未设置过 `age`字段的人员信息将被查询到。
query.notExist('age');
```

除了基本的查询条件外， 我们也提供了查询数量及排序等功能。

```javascript
// 先用名字排序， 再用年龄从大到小排。
query.orderBy('name').orderByDescending('age');

// 忽略60条记录， 再从第61个开始返回20条数据
query.skip(60).take(20);

```

上面的 `skip` 及 `take` 可以做分页。 但一般我做分页时都需知道查询到的数据总数。 此时我们可以设置 `count` 来返回总数。

```javascript
query.count(true);
```

当设置完所有条件后，我们用 `find` 方法来获取对象。此方法也可传入两个function, 第一个为查询成功后触发， 第二个是查询失败或出错时触发。

```javascript
query.find(function(data, count){
	// 在上面的步骤设置过 count 的话， 此处会返回 count 这个参数。
}, function(error){
	// do something
});
```

若只想返回一条记录， 可用 `first` 方法来获取， 调用方法和 `find` 相同。


## 用户
在SDK中专门提供了一个 `User`对象来操作用户。 用户对象继承自 `Object`对象， 所以多数操作相同。 只是增加了一些特殊的操作， 如 用户注册， 登录， 登出等。


### 创建

```javascript

// 创建新的user对象。
var user = sky.User();


// 操作已有的用户对象, 参数为用户Id.
var user = sky.User('54a923429760e37fd1000001');

```

### 注册
注册用户时， 可调用 `register` 方法。 可传入两个回掉function. 第一为注册成功时触发， 第二个为注册失败或出错时触发。

```javascript

user.register(function(data){
	// 注册成功后返回用户对象的 objectId。
}, function(error){
	// do something
});

```

### 登录
用户登录时， 可调用 `login`方法。 login方法需3个参数。 第一个为一个JSON对象。包含登录用户名及密码等信息。 第二个为成功时触发function, 第三个是失败或出错时触发function。

```javascript

// 用户名 + 密码 方式登录

user.lgoin({'username':'myusername', 'password': '123456'}, function(user){
	// 注册成功后返回用户信息
}, function(error){
	// do something
});

// 邮箱地址 + 密码 方式登录
user.lgoin({'email':'user@email.com', 'password': '123456'}, function(user){
	// 注册成功后返回用户信息
}, function(error){
	// do something
});

// 手机号码 + 密码 方式登录
user.lgoin({'phone':'18600000000', 'password': '123456'}, function(user){
	// 注册成功后返回用户信息
}, function(error){
	// do something
});


```

### 当前已登录用户
当用户登录后， 我们会在本地磁盘保存用户信息。 下次可不用再次登录即可获取已经登录过的用户信息。

```javascript

var user = sky.User().current();

```

### 删除用户
因 `User` 对象是继承自 `Object`, 所以删除时， 可直接用 `Object` 的 `delete` 方法。

```javascript
user.delete(function(data){
	// 删除成功
}, function(error){
	// do something
});
```


## 角色
我们不太建议在SDK中去操作角色， 因为关系到整个系统的权限问题。 所以您可以在管理后台操作或用 [Restful API](http://developer.skynology.com/restful-api.html#角色) 来操作角色。