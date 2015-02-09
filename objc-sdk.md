# 上空云服务 IOS SDK 使用文档
上空云服务 IOS SDK是针对我们的 [Restful API](http://developer.skynology.com/restful-api.html)进行的一个包装。 以方便大家写IOS程序时调用。 整个类库是用 `Objective-C` 语言来编写。 并以开源方式放到我们的 [github库](https://github.com/skynology/objc-sdk) 中。 若大家使用当中有什么问题或建议， 请联系我们。若对类库有贡献代码， 请直接 `Fork` 并且 push 到 `develop` 分支下。


## 安装

### 使用 Cocoapods 安装
[Cocoapods](http://cocoapods.org/) 为IOS平台上较流行的类库管理工具。 安装上空云IOS SDK时， 可在您的 `Podfile` 中增加如下使命。

```
pod 'Skynology'
```

在保存后， 在您的 `Podfile` 对应的目录执行下面命令即可开始使用我们的IOS SDK了。

```
pod install
```

> 在国内使用 `pod install` 或 `pod update` 之类的命令时， 往往会很慢。一般表现为： 卡在 `Analyzing dependencies`步骤上。
> 此时可先把源换成 **淘宝** 提供的源。 再执行命令时， 增加参数 `--no-repo-update`。 详细说明可参考 segmentfault 上对应讨论: [cocoapods慢如何解决](http://segmentfault.com/q/1010000000434890) 。

### 手动安装
因我们的各类SDK是以开源方式来提供的， 所以您当然可以直接下载源码到您的项目中。

人点击链接 : [上空云IOS SDK源码包](https://githubc.com/skynology/objc-sdk)来下载， 解压缩下载包后， 将其中的`Skynology`文件夹复制到您的项目文件中即可使用了。


### 初始化SDK

当安装完成后， 需要在您的 `AppDelegate` 中的 引用 `SkyClient` 文件。

```objectivec
#import <Skynology/SkyClient.h>;
```

并且在 `application: didFinishLaunchingWithOptions:` 设置 `applicationId` 以及 `applicationKey`。 如:

```objectivec
SkyClient *client = [SkyClient sharedClient];
[client setApplicationId:@"xxx" applicationKey:@"xxxxx"];

```

> 如果还不知道 `applicationId` 及 `applicationKey`， 可先阅读 [基础概念](http://developer.skynology.com/getting-started.html) 以及 [控制台使用说明](http://developer.skynology.com/console-tutorial.html)。


## 对象

`SkyObject` 类是用来操作 `对象`。 `SkyObject` 所操作的资源类， 需在 [管理后台](https://console.skynology.com) 中已经创建过的。 若未在后台创建的类， 在保存，更新， 删除等操作时， 将直接返回错误码。

如同 `基础概念` 中讲的一样， 对象有几个默认字段。 在 `SkyObject` 中有一些默认属性。如下:

* `objectId`: 对象Id字段。
* `acl`: 对象权限字段。
* `createdAt` 及 `updatedAt`: 对象创建时间及最后更新时间。
* `resourceName`: 当前对象对应的资源类名。(一般是您初始化`SkyObject`时传入的。) 

### 初始化对象
使用 `SkyObject` 类时， 需在你当前的Controller中引用 `SkyObject.h` 文件。如:

```objectivec
#import <Skynology/SkyObject.h>;
```

在您需要使用 `SkyObject` 的地方， 需先初始类。 用如下代码: 

```objectivec
SkyObject *object = [[SkyObject allow] initWithResourceName: @"MyUser"];
```

当需要操作已经存在对象，如直接做删除或更新时， 可用资源类名及对应的 `objectId` 来初始化。

```objectivec
SkyObject *object = [[SkyObject allow] initWithResourceName: @"MyUser" objectId:@"54a923429760e37fd1000002"];
```


### 设置字段
当初始化以后， 可以设置类的字段值，字段需在后台设置过， 并且字段类型同后台设置的类型匹配。

```objectivec
[object setValue:@"Jacky Chen" forField:@"name"];
[object setValue:[NSNumber numberWithInt:30] forField:@"age"];
[object setValue:[NSNumber numberWithBool:true] forField:@"isMarried"];
[object setValue:@[@"golang", @"python", @"javascript"] forField:@"likes"];

```
我们支持众多数据类型， 可参考[数据类型](#数据类型)。

### 更新字段
更新字段同 [设置字段](#设置字段) 基本上是相同的， 普通更新直接使用上面的 `[object setValue:@"Jacky Chen" forField:@"name"];` 这种方式即可，我们会SDK中计算出您设置了哪些字段， 并且只更新您设置过的字段。

还有以下几种特殊处理更新字段的方式。

#### 计数器
我们除了在创建资源类时设置字段为 **自动更新** 以外， 也可以在SDK中**递增**或**递减** `数值` 类型字段的值。 您也可以直接设置字段值， 但是若有多人同时做了相同的更新值操作， 将会发生覆盖先更新的值的情况。您使用 `计数器` 方式去更新时， 将不会发覆盖的情况。

您可用下面的方式去**递增**或**递减**值:

```objectivec
[object increment:@"followCount"];
```

基中 `followCount` 为您要操作的字段名。 当设置上面命令，并保存后， 系统会自动对当前字段值增加一个数。 我们也提供了更自由操作的方法。

```objectivec
[object increment:@"followCount" amount:[NSNumber numberWithInt:5]];
```

此时， 将会对字段的原有值增加5个数。 如果您要做递减操作时， 可用上面的方法， 并且给 `amount` 参数传入一个负数即可。 如您要递减一个数， 传入 `-1`, 若要递减5个数， 就传入 `-5`。

#### 数组字段
当您需要更新数组字段时， 除了上面的直接设置 `setValue`方式去覆盖外， 也可以用下面提供的几个方法去更新数组内的值。

用 `addValueToArray` 方法去追加一个值到数组字段内。 此方法不检查唯一性， 也就是你所要增加的值已经存在于数组字段内，也会把值成功追加进去。

```objectivec
[object addValueToArray:@"java" forField:@"likes"];
```

用 `addUniqueValueToArray` 方法。 此方法会先检查值的唯一性， 若数组字段中已经存在当前值， 将不会重复增加。

```objectivec
[object addUniqueValueToArray:@"python" forField:@"likes"];
```

用 `removeValueFromArray` 方法从数组字段中删除一个值。如下面命令将会从 `likes` 字段中删除 `golang` 这个值。

```objectivec
[object removeValueFromArray:@"golang" forField:@"likes"];
```

#### ACL字段
因 **ACL** 字段为一个特殊的JSON Object类型。 所以我们在IOS SDK中提供了几个常用的方法来设置此字段。

##### 使用 `用户Id` 来设置ACL。

```objectivec
[object setReadAccess:true forUser:@"54a923429760e37fd1000002"];
[object setWriteAccess:false forUser:@"54a923429760e37fd1000002"];
```

上面设置代表用户 `54a923429760e37fd1000002` 对当前对象有查看权限， 而无更新或删除权限。
除了普通用户Id外， 也支持特殊通配符。有以下通配符:

* 星号(*): 型号代表所有人。 包括未匿名用户。 当未设置对象ACL时， 默认就是所有人(\*)都有读写权限。
* 井号(#): 井号代表已经登录用户。 需先使用 [用户登录功能](#用户) 登录过才能查看或编辑此对象。

##### 用角色名来设置。 

```objectivec
[object setReadAccess:true forRole:@"admin"];
[object setWriteAccess:true forRole:@"admin"];
```
上面设置代码名称为 `admin` 的角色对当前对象有读写权限。 角色相关概念， 请参考文档 [基础概念 - 角色](http://developer.skynology.com/)。


### 保存
当设置完类的字段后， 可以使用 `saveWithBlock` 方法来保存当前的对象。 保存会有两种情况， 一种是完全**新增**； 另一种是**更新**。 不管是哪一种， 我们都是用同一个方法去保存。 SDK后台会自动分辨出是属于哪一种。

```objectivec
[object saveWithBlock:^(BOOL successed, NSError *error){
	// successed: 代表保存是否成功的标记。 若为 true 则代表保存成功。 若为 false 代表保存失败。
	// error: 当保存成功时， 值为nil, 保存失败后 error 将会有失败的明细信息。
	
}];

```
另保存成功后。SDK会更新 `object` 的字段值。 


### 删除对象
删除对象时， 可调用 `SkyObject` 的 `removeWithBlock` 方法。

```objectivec
[object removeWithBlock:^(BOOL successed, NSError *error) {
    // 相关参数同保存时相同。 successed 代表删除是否成功。 
    // 若删除失败 error 将会返回失败信息。
}];
```


## 查询
查询相对应的类为 `SkyQuery`。 可用此类查询单个或多个 `SkyObject` 对象。

### 查询单个对象
查询单个对象时， 可用如下方法:

```objectivec
[SkyQuery getObjectWithBlock:@"MyUser" objectId:@"54a923429760e37fd1000002" completion:^(SkyObject *object, NSError *error) {
	// 若查询成功， object 值是相对应的对象。 error 为 nil。
	// 若未找到或出错， object 值为 nil。 error 为错误明细信息。
}];
```

### 普通查询
当查询多条记录时， 可用下面方法去初始化 `SkyQuery` 对象。

```objectivec 
// 初始化 SkyQuery对象
SkyQuery *query = [[SkyQuery alloc]initWithResource:@"MyUser"];

// 此处设置各种查询条件等
[query where:@"name" isEqualTo:@"Jacky Chen"];

// 返回结果
[query findObjectsWithBlock:^(NSArray *objects, NSNumber *count, NSError *error) {
	// objects 为 SkyObject 数组。 
	// 查询出错时， error 将返回错误明细
}];

```

### 查询条件
当初始化完成后。 我们可以设置各种查询条件来拿到您想拿的数据。

匹配值

```objectivec
[query where:@"age" isEqualTo:[NSNumber numberWithInt:30]];
[query where:@"name" isEqualTo:@"Jacky Chen"];
```

上面条件将会从 `MyUser` 资源中查找叫 `Jacky Chen`， 并且年龄为 `30` 的人员。
> 需注意您设置的字段类型必须同管理平台设置的字段类型匹配。

不等于

```objectivec
[query where:@"name" isNotEqualTo:@"Jacky Chen"];
```

针对数值类型， 可以使用比较方法。

```objectivec

// 小于: age < 30
[query where:@"age" isLessThan:[NSNumber numberWithInt:30]];

// 小于等于: age <= 30
[query where:@"age" isLessThanOrEqualTo:[NSNumber numberWithInt:30]];

// 大于: age > 30
[query where:@"age" isGreaterThan:[NSNumber numberWithInt:30]];

// 大于等于: age >= 30
[query where:@"age" isGreaterThanOrEqualTo:[NSNumber numberWithInt:30]];
```

针对字段串类型的字段。 提供了下面的方法。

```objectivec

// 代表查询所有名字为 'Jacky' 开头的人员。
[query where:@"name" startWith:@"Jack"];


// 代表查询所有名字为 'Lee' 结尾的人员， 如 'Bruce Lee' 将被搜索到。
[query where:@"name" endWith:@"Lee"];


// 代表查询所有名字中包含 'Mike' 的人员。如 'Mike Jordan', 'Mike Jackson'。
[query where:@"name" containValue:@"Mike"];


```

可以用 `in` 或 `notIn` 方法来查询匹配给出数组中的值。

```objectivec

// 只要年龄为 30, 35, 50 中的任何一个， 就会被查询到。
[query where:@"age" in:@[@30, @35, @50]];


// 查询名称不是 'Bruce Lee' 和 'Jacky Chan'的人员
[query where:@"name" notIn:@[@"Bruce Lee", @"Jacky Chan"]];

```


针对数组类型的字段， 也可用上面的 `in` 及 `notIn` 方法。 如:

```objectivec

// 只要某人员的 `likes`列表里有 'golang', 'python' 就会被查询到。
[query where:@"likes" in:@[@"golang", @"python"]];
```

除了 `in` 及 `notIn`, 我们还提供了一个方法 `matchAll`。 用来查询匹配所有给出值的对象。 如:

```objectivec
// 查询 `likes` 字段包含 'golang' 和 'python' 两个值的人员
[query where:@"likes" matchAll:@[@"golang", @"python"]];

```


查询某个字段有值的对象。 如下面语句， 将查询 `age` 这个字段有值的对象。

```objectivec
[query where:@"age" exists:true];
```

除了基本的查询条件外， 我们也提供了查询数量及排序等功能。

```objectivec

// 忽略60条记录
query.skip = 60;

// 返回20条记录
query.take = 20;

```

上面的设置， 将等于一个分页操作。 取第4页(也就是从61到80)。 一般我们做分页时， 也需要知道查询到的**总数**。 我们可以设置 `count`方法来同时返回总数。

```objectivec
query.count = true;

```

当设置完所有选项后，我们用 `findObjectsWithBlock` 方法来获取对象。

```objectivec
[query findObjectsWithBlock:^(NSArray *objects, NSNumber *count, NSError *error) {
	// objects 内默认是返回 'SkyObject' 类型。
}];
```

上面的查询方法内的 `objects` 默认是返回 `SkyObject`对象类型。 但我可以设置 query 的 `resultType` 来拿到想拿的类型。
比如要返回 `SkyUser` 类型时， 需设置:

```objectivec
query.resultType = SkyQueryResultTypeUser;

```




## 用户
在SDK中专门提供了一个类 `SkyUser` 来操作用户。 `SkyUser` 继承自 `SkyObject`， 所以包含 `SkyObject` 的全部功能。 另我们在 `SkyUser` 中增加了一些特殊属性或方法。

### 字段
在`SkyUser` 中除了 `SkyObject` 默认的 `objectId`, `acl`, `createdAt`, `updatedAt` 外， 另外增加了 `username`, `email`, `phone` 以及 `sessionToken` 等字段。


### 注册用户
注册用户时， 我们至少要求输入 `username` 以及 `password` 两个字段。

```objectivec

SkyUser *user = [SkyUser newUser];
user.username = @"Jet Lee";
user.password = @"qr4hFHEG4k1z5suHLOuc";
user.email = @"jetlee@email.com";
user.phone = @"18600000000";
[user setValue:[NSNumber numberWithInt:40] forField:@"age"];
[user setValue:@[@"javascript", @"bash", @"c#"] forField:@"likes"];

[user registerWithBlock:^(NSDictionary *object, NSError *error) {
	// 保存成功后， object 对象将会返回 'objectId' 以及 'createdAt' 两个字段。
	// 若保存失败， 将在 error 中看到详细错误信息。
        
}];

```

### 用户登录
我们提供了如几个方法来做用户登录。

1. 用户名 + 密码 来登录

```objectivec
[SkyUser loginWithUserName:@"jetlee" password:@"qr4hFHEG4k1z5suHLOuc" completion:^(SkyUser *user, NSError *error) {
	
}];
```

2. 邮箱地址 + 密码 来登录

```objectivec
[SkyUser loginWithEmail:@"jetlee@email.com" password:@"qr4hFHEG4k1z5suHLOuc" completion:^(SkyUser *user, NSError *error) {
		
}];
```
3. 手机号码 + 密码 来登录

```objectivec
[SkyUser loginWithPhone:@"18600000000" password:@"qr4hFHEG4k1z5suHLOuc" completion:^(SkyUser *user, NSError *error) {
	
}];
```

### 当前已登录用户
为了避免用户每次打开APP后都做登录操作， 我们在用上面的几个方法登录成功后， 会将用户信息保存到磁盘。 以后可直接用下面的方法来获取已登录的用户信息。

```objectivec

// 若已经有登录过， 将会返回登录的用户。
SkyUser *user = [SkyUser currentUser];

```

### 删除用户
因 `SkyUser` 是继承自 `SkyObject`, 所以删除时， 可直接用 `SkyObject` 的 `removeWithBlock` 方法。

```objectivec
[user removeWithBlock:^(BOOL successed, NSError *error) {
    // 相关参数同保存时相同。 successed 代表删除是否成功。 
    // 若删除失败 error 将会返回失败信息。
}];
```


## 角色
我们不太建议在SDK中去操作角色， 因为关系到整个系统的权限问题。 所以您可以在管理后台操作或用 [Restful API](http://developer.skynology.com/restful-api.html#角色) 来操作角色。
