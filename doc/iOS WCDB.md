# WCDB

# WCDB
## 前言：
iOS 中常用的数据库有 `CoreData` 、 `SQLite` 和 `FMDB` 等等，其中 `CoreData` 和 Xcode 深度结合，易用度较差； `SQLite` 本身就是C语言，使用需要了解C语言接口； `FMDB` 是对 `SQLite` 的一层封装，很多胶水代码，仍然自己需要写 `SQL` 语句，而 `WCDB` 是微信团队开发的一个易用、高效、完整的移动数据库框架，它基于 `SQLite` 和 `SQLCipher` 开发，支持加密、损坏检测、数据备份、和数据修复，在微信中应用广泛，且支持在  `C++` 、 `Swift` 、 `Objc` 三种语言环境中使用。

![CodeStructure](https://user-images.githubusercontent.com/126937296/229737809-f4cd8aac-14e9-4403-9513-2292c88d8256.png)

## WCDB 的最基础的调用过程大致分为三个步骤：
1. 模型绑定
2. 创建数据库与表
3. 操作数据

## 一、模型绑定
假设存在一个`Person`类：

```js
class Person {
    var identifier: Int = 0
    var name: String? = nil
    var age: Int = 0
}
```
要把该类的数据存储进数据库，可以采用`WCDB`的文件模板：
### 1、文件与代码模版

如果没有获取`WCDB`的`Github`仓库，可以终端执行命令：

```js
curl https://raw.githubusercontent.com/Tencent/wcdb/master
/tools/templates/install.sh -s | sh
```

打开`cmd + n`拉到最下面：

<img width="730" alt="Pasted Graphic" src="https://user-images.githubusercontent.com/126937296/229737926-005e948c-16c5-434a-8b51-72adf18b233e.png">

>选swift创建，由于模板是旧代码，需要把`import WCDB`改成`import WCDBSwift`

### 2、WCDB Swift 的模型绑定分为五个部分：
1. 字段映射
2. 字段约束
3. 索引
4. 表约束
5. 虚拟表映射

#### 1）字段映射
- 在类内定义 `CodingKeys` 的枚举类，遵循 `String` 和 `CodingTableKey` ；
- 把想要存储的变量写到枚举 `CodingKeys` 里面的 `case` 后面，表示绑定到了数据库表中的字段；
- 把需要在数据库重命名的字段，进行别名映射， `case identifier = "id"`  表示把 `identifier` 在数据库表中重命名为`id`；
- 如果使用的字段与 `SQLite` 字段关键字相同，也需要做别名映射。
#### 2）字段约束

字段约束，它用于针对单个字段的约束，例如主键约束、非空约束、唯一约束，默认值等等，可以根据自己的需求选择实现或者不实现。方法是 `columnConstraintBindings:`，`Github` 上和 `TableCodable` 模板默认生成的是以前的代码，新代码如下：

```js
static var columnConstraintBindings: [CodingKeys:
ColumnConstraintBinding]? {
    return [
        identifier: ColumnConstraintBinding(isPrimary: true),
        name: ColumnConstraintBinding(isNotNull: true, defaultTo:"空"),
        age: ColumnConstraintBinding(isNotNull: true, defaultTo: 0)
    ]
}
```
#### 3）自增属性

`isAutoIncrement` 表示是否自增，约束定义 `isPrimary` 为 `true`，支持自增，但是仍然可以支持非自增方式插入。

当需要自增插入，需要设置 `isAutoIncrement` 为 `true`，数据库会将主键最大值 `+ 1` 作为新的最大主键值。

>索引，表约束，虚拟表映射相对复杂，一般表用不到，这里就不写。

#### 4）swift6 错误警告
由于 `Github上WCDB` 的是 `swift4.0` 的代码，现在使用有些需要修改，会提示：

<img width="528" alt="Pasted Graphic 1" src="https://user-images.githubusercontent.com/126937296/229738025-c862ed2e-0027-4e89-8df4-4768ce7c2b09.png">

在 `class Person` 前面添加 `final` 消除警告

**最终模型绑定代码**：

```js
final class Person: TableCodable {
    var identifier: Int = 0
    var name: String? = nil
    var age: Int = 0
    enum CodingKeys: String, CodingTableKey {
        typealias Root = Person
        
        case identifier = "id"
        case name
        case age
        static let objectRelationalMapping = 
        TableBinding(CodingKeys.self)
        static var columnConstraintBindings: 
        [CodingKeys: ColumnConstraintBinding]? {
            return [
                identifier: ColumnConstraintBinding
                (isPrimary: true),
                name: ColumnConstraintBinding
                (isNotNull: true, defaultTo: "空"),
                age: ColumnConstraintBinding
                (isNotNull: true, defaultTo: 0)
            ]
        }
    }
    var isAutoIncrement: Bool = true
}
```
## 二、创建数据库与表

### 1、创建数据库

```js
var database = Database(withPath: NSSearchPathForDirectoriesInDomains(
            .documentDirectory,
            .userDomainMask,
            true).last!+”/Person/person.db")
```
### 2、创建表
一行代码就创建表：
```js
try database?.create(table: "personTable", of: Person.self)
```
由于 `WCDB` 推荐用表操作数据，所以可以获取表对象：

```js
var personTable = try database.getTable(named: "personTable")
```

## 三、操作数据

### 1、插入操作
向表中插入一条数据，`id` 前面已经定义了自增：
```js
 try database?.insert(objects: p, intoTable: "personTable")
```
`WCDB` 推荐操作表，因为操作的对象更明确，更简洁，后面代码都是表操作：

```js
 try personTable?.insert(objects: p)
```
### 2、删除操作
示例代码，删除 `id` 为 **2** 的数据：

```js
try personTable?.delete(where: Person.Properties.identifier == 2)
```

### 3、更新操作
示例代码，更新 `id` 为 **2** 的数据的 `name` 字段：

```js
try personTable?.update(on: Person.Properties.name, with: p, 
where:  Person.Properties.identifier == 2 )
```
 
### 4、查找操作
示例代码，查找年龄大于 **25** 的数据：

```js
 let persons: [Person]! = try personTable?.getObjects(where: 
 Person.Properties.age > 25)
```

>主要功能代码都是一行代码搞定，并且让增删改查的语法一致，使用非常方便。

## 四、数据库、表
### 1、打开数据库
由于 `WCDB` 是采用延迟初始化，使用时候才会创建并且初始化，所以不需要手动调用 `open`，但可以使用 `database.canOpen` 测试数据是否可以正常打开，另外 `database.isOpened` 需要创建表之后才会 `true` 。
### 2、关闭数据库
`WCDB` 一般情况下不需要开发者手动调用关闭，如果控制器被回收，数据库会自动关闭，并且自动回收内存。当然，也可以手动调用，一般都是基于文件操作，比如移动文件影响到了数据库的数据，才需要手动关闭，接口是：

```js
try database.close(onClosed: {
    try database.moveFiles(toDirectory: otherDirectory)
})
```
### 3、表
通过 `getTable` 接口获取数据库中的一个表

```js
let table = database.getTable(named: "sampleTable", of: Sample.self) 
```
`WCDB` 中 `Table` 具备了 `database` 的所有增删改查接口，并且更简洁，以表为单位来管理数据读写逻辑更合理方便，所以尽量使用 `Table` 来进行数据读写操作，上面代码已经演示。

## 五、事务
事务一般用来提升性能和保证操作的原子性，通过 `transaction` 控制事务。

### 1、性能

假设给数据库插入 **100** 万条数据，看耗费时间和使用事务的优化情况，先准备 **100** 万条数据：

```js
print("startTime ------------------" + startTime)
var persons:[Person] = [];
for i in 1...1000000{
    let p = Person()
    p.age = Int(arc4random_uniform(20)) + 10;
    p.name = String(format: "张%d", i)
    persons.append(p)
}

```
#### 1）普通插入操作
```js
// 单独插入，效率很差
for p in persons {
    do{
        try personTable!.insert(objects: p)
    }catch let error{
        debugPrint("插入数据失败 \(error.localizedDescription)")
    }
}
let endTime = Self.getCurrentTime(timeFormat: TimeFormat.HHMMSS)
print("endTime ------------------" + endTime)
```
数据库表中如下：

<img width="772" alt="1__#$!@%!#__Pasted Graphic 1" src="https://user-images.githubusercontent.com/126937296/229738154-a7a0f8f3-9b51-4135-b243-3f993c2fbf79.png">

运行打印：
```js
startTime ------------------ 12:05:43
endTime   ------------------ 12:06:23
```
普通操作，插入 **100** 万条数据，耗时差不多 **41** 秒。

#### 2）事务插入操作

```js
// 事务插入，性能较好
do{
    try database.run(transaction: {
        for object in persons {
            try personTable?.insert(objects: object)
        }
    })}catch let error{
        debugPrint("插入数据失败 \(error.localizedDescription)")
    }
let endTime = Self.getCurrentTime(timeFormat: TimeFormat.HHMMSS)
print("endTime ------------------" + endTime)
```


```js
运行打印：
startTime ------------------ 12:14:42
endTime   ------------------ 12:14:55
```
事务操作，插入 **100** 万条数据，耗时差不多 **13** 秒，性能有明显提升。


#### 3）内置事务插入操作
`insert(objects:)`接口内置了事务，并对批量数据做了针对性的优化，性能更好

```js
do{
    try personTable?.insert(objects: persons)
}catch let error{
    debugPrint("插入数据失败 \(error.localizedDescription)")
}
let endTime = Self.getCurrentTime(timeFormat: TimeFormat.HHMMSS)
print("endTime ------------------" + endTime)
```
运行打印：
```js
startTime ------------------ 12:23:05
endTime   ------------------ 12:23:08
```
可以看出，内置事务接口的优化非常明显，插入 **100** 万条数据只使用了 **3** 秒。

### 2、原子性
在多线程下，删除数据，同时插入一条数据，操作在瞬间，很难确实哪个先执行：
#### 1）非事务操作

```js
DispatchQueue(label: "other thread").async {
    do{
        try self.personTable?.delete()
    }catch let error{
        debugPrint("事务操作失败 \(error.localizedDescription)")
    }
}
do{
    
    let p = Person()
    p.age = Int(arc4random_uniform(20)) + 10;
    p.name = "马可bro"
    try personTable?.insert(objects: p)
    let objects = try personTable?.getObjects()
    print(objects?.count ?? "出错") // 可能输出 0 或 1
}catch let error{
    debugPrint("事务操作失败 \(error.localizedDescription)")
}
```
结果：可能输出 **0** 或 **1** 或 **2** 。

#### 2）事务操作

```js
DispatchQueue(label: "other thread").async {
    do{
        try self.personTable?.delete()
    }catch let error{
        debugPrint("事务操作失败 \(error.localizedDescription)"
    }
}
do {
    try database.run(transaction: {
        let p = Person()
        p.age = Int(arc4random_uniform(20)) + 10;
        p.name = "大小姐"
        try personTable?.insert(objects: p)
        let objects = try personTable?.getObjects()
        print(objects?.count ?? "出错") // 输出 1
    })
}catch let error{
    debugPrint("事务操作失败 \(error.localizedDescription)")
}
```
结果：只会输出 **1**。

## 参考
- [Tencent/wcdb](https://github.com/Tencent/wcdb)
- [微信移动端数据库组件WCDB系列
](https://mp.weixin.qq.com/s/1XxcrsR2HKam9ytNk8vmGw)



