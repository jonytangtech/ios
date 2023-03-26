# Sqlite
## 一、前言
>虽然平时开发不会直接使用`SQLite`，但是作为一个轻量级数据库，理解和掌握其基本原理和使用还是有必要的。`FMDB`就是对`SQLite`的封装，并且微信团队在自研自己的数据库前也是使用`SQLite`。

点击项目名称 -> `TAGETS` -> `Build Phases` -> `Link Binary With Libraries` 点击添加`libsqlite3.tbd`：

![Pasted Graphic.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/f46cfd6bc0674ddf8462942a32f4c57c~tplv-k3u1fbpfcp-watermark.image?)

## 二、代码准备


```js
@interface ViewController (){
    /**数据库*/
    sqlite3 *database;
    /**准备语句*/
    sqlite3_stmt *stmt;
}
@property(nonatomic, strong) NSString *databasePath;
/**id文本框*/
@property (weak, nonatomic) IBOutlet UITextField *idTF;
/**名字文本框*/
@property (weak, nonatomic) IBOutlet UITextField *nameTF;
/**年龄文本框*/
@property (weak, nonatomic) IBOutlet UITextField *ageTF;
```
模拟器先拉好UI：

![Pasted Graphic 1.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/7df48b1db6d34fae9f79b18f410b89aa~tplv-k3u1fbpfcp-watermark.image?)

### 1、生成路径

```js
- (NSString *)databasePath{
    if (!_databasePath) {
        /**获取Document路径*/
        NSArray *Paths = NSSearchPathForDirectoriesInDomains(NSDocumentDirectory, NSUserDomainMask, YES);
        NSString *DocumentPath = [Paths firstObject];
        /**拼接数据库位置*/
        _databasePath = [DocumentPath stringByAppendingPathComponent:@“person_info.sqlite"];
        NSLog(@"%@",_databasePath);
    }
    return _databasePath;
}
```
### 2、打开数据库

```js
/**打开数据库*/
- (void)openDatabase{
    /**创建或打开数据库*/
    int openFlag = sqlite3_open_v2(self.databasePath.UTF8String, &database, SQLITE_OPEN_CREATE | SQLITE_OPEN_READWRITE, NULL);
    /**判断是否成功*/
    if (openFlag != SQLITE_OK) {
        /**失败则关闭数据库*/
        sqlite3_finalize(stmt);
        sqlite3_close(self->database);
        NSLog(@"创建或打开数据库失败 --- %d",openFlag);
    }else{
        NSLog(@"创建或打开数据库成功");
        [self createForm];
    }
}
```
### 3、创建表

```js
/** 创建表*/
- (void)createForm{
    char *errMsg = NULL;
    /** 创建表
1、建表格式: create table if not exists 表名 (列名 类型,....)
2、如需生成默认增加的id: id integer primary key autoincrement
3、数据库名、表名、字段名使用小写，关键字、函数名称使用大写
4、每个sql语句最后都要加上”;“
*/
    NSString *sqlStr = [NSString stringWithFormat:
                        @"CREATE TABLE IF NOT EXISTS 'person'("
                        "id INTEGER PRIMARY KEY AUTOINCREMENT NOT NULL,"
                        "name TEXT NOT NULL,"
                        "age INTEGER NOT NULL);"];
/**
     第1个参数：数据库对象
     第2个参数：sql语句
     第3个参数：查询时候用到的一个结果集闭包
     第4个参数：用不到
     第5个参数：错误信息
     */
    int execRst = sqlite3_exec(database, sqlStr.UTF8String, NULL, NULL, &errMsg);
    if (execRst == SQLITE_OK) {
        NSLog(@"创建表成功");
    }else{
        NSLog(@"创建表失败:%s",errMsg);
    }
}
```
运行打印：

```js
Sqlite3Demo[46764:3296271] /Users/xxxx/Library/Developer/CoreSimulator/Devices/…/data/Containers/Data/Application/…/Documents/Person.sqlite
Sqlite3Demo[46764:3296271] 创建或打开数据库成功
Sqlite3Demo[46764:3296271] 创建表成功
```
打开生成路径，可以看到一个`person_info.sqlite`的文件：

![Pasted Graphic 4.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/0f906fcc5807471f915603c229feca76~tplv-k3u1fbpfcp-watermark.image?)

用`Navicat`打开`person_info.sqlite`，`person`的表已经创建好了：

![Pasted Graphic 3.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/9af7ffe74f40430489a6e46ee020838e~tplv-k3u1fbpfcp-watermark.image?)

### 4、插入多条测试数据

```js
/**插入多条数据*/
- (IBAction)insertMultipleData:(UIButton *)sender{
    [self openDatabase];
    for (int i = 0; i < 10; i++) {
        NSString *name = [NSString stringWithFormat:@"张%d",i+1];
        int age = arc4random_uniform(20) + 10;
        // 拼接 sql 语句
        NSString *sql = [NSString stringWithFormat:@"INSERT INTO person (name,age) VALUES ('%@',%d);",name,age];
        // 执行 sql 语句
        char *errMsg = NULL;
        int result = sqlite3_exec(database, sql.UTF8String, NULL, NULL, &errMsg);
        if (result == SQLITE_OK) {
            NSLog(@"插入数据成功 - %@",name);
        } else {
            NSLog(@"插入数据失败 - %s",errMsg);
        }
    }
}
```
点击模拟器增多条：

```js
Sqlite3Demo[47220:3310420] 插入数据成功 - 张1
Sqlite3Demo[47220:3310420] 插入数据成功 - 张2
Sqlite3Demo[47220:3310420] 插入数据成功 - 张3
Sqlite3Demo[47220:3310420] 插入数据成功 - 张4
Sqlite3Demo[47220:3310420] 插入数据成功 - 张5
Sqlite3Demo[47220:3310420] 插入数据成功 - 张6
Sqlite3Demo[47220:3310420] 插入数据成功 - 张7
Sqlite3Demo[47220:3310420] 插入数据成功 - 张8
Sqlite3Demo[47220:3310420] 插入数据成功 - 张9
Sqlite3Demo[47220:3310420] 插入数据成功 - 张10
```
### 5、添加查询

```js
/**查询所有记录*/
- (IBAction)retrieveAllData:(UIButton *)sender {
    [self openDatabase];
    NSString *searchSqlStr = @"SELECT * FROM person";
    [self operationData:searchSqlStr.UTF8String];
}
/**操作数据*/
- (void)operationData:(const char *)sql{
    int result = sqlite3_prepare_v2(database, sql, -1, &stmt, nil);
    if (result != SQLITE_OK) {
        NSLog(@"操作失败,%d",result);
    } else {
        /**打印操作后的数据，每调用一次sqlite3_step，stmt就会指向下一条记录*/
        printf("------------------------------\n");
        /**找到一条记录*/
        while (sqlite3_step(stmt) == SQLITE_ROW) {
            /**取出第0列字段的值*/
            int ID = sqlite3_column_int(stmt, 0);
            /**取出第1列字段的值*/
            const unsigned char *name = sqlite3_column_text(stmt, 1);
            /*取出第2列字段的值*/
            int age = sqlite3_column_int(stmt, 2);
            printf("查到的数据 id:%d name:%s 年龄:%d\n",ID,name,age);
            
        }
        /*关闭连接*/
        [self clearTextField];
        sqlite3_finalize(stmt);
        sqlite3_close(self->database);
    }
}
```
点击查全部：

```js
------------------------------
查到的数据 id:1 name:张1 年龄:29
查到的数据 id:2 name:张2 年龄:16
查到的数据 id:3 name:张3 年龄:26
查到的数据 id:4 name:张4 年龄:29
查到的数据 id:5 name:张5 年龄:18
查到的数据 id:6 name:张6 年龄:26
查到的数据 id:7 name:张7 年龄:20
查到的数据 id:8 name:张8 年龄:17
查到的数据 id:9 name:张9 年龄:14
查到的数据 id:10 name:张10 年龄:26
```
到`Navicat`里看一下，已经有**10**条记录：

![1__#$!@%!#__Pasted Graphic 3.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/fc43c40de824444d8eb9e7bd666105b1~tplv-k3u1fbpfcp-watermark.image?)

还有一个`sqlite_sequence`，`seq`是**10**：

![1__#$!@%!#__Pasted Graphic 4.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/3ffcffee618e46a0857accd840656b65~tplv-k3u1fbpfcp-watermark.image?)

>注：如果没有看到数据，点击左边下面的按钮刷新，右键刷新无效。

![1__#$!@%!#__Pasted Graphic 1.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/0aeb6f4b3f844a86909c428f7196a025~tplv-k3u1fbpfcp-watermark.image?)

## 三、CRUD增删改查操作

### 1、增加一条数据

```js
/**插入一条记录*/
- (IBAction)insertData:(UIButton *)sender {
    [self openDatabase];
    NSString *insertSplStr = [NSString stringWithFormat:@"INSERT INTO person (name,age) VALUES ('%@',%@);",_nameTF.text,_ageTF.text];
    [self updateDataWithSql:insertSplStr.UTF8String success:^{
        [self clearTextField];
        NSLog(@"插入数据成功");
    }];
}
```
`name`填入夏洛特，`age`填**24**，点击增：

![Pasted Graphic 5.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/274f3c33ba214f10b9077aa8be7cf812~tplv-k3u1fbpfcp-watermark.image?)

```js
Sqlite3Demo[47564:3319611] 插入数据成功
```
点击查全部，发现已经插入了一条数据：

```js
------------------------------
查到的数据 id:1 name:张1 年龄:29
查到的数据 id:2 name:张2 年龄:16
查到的数据 id:3 name:张3 年龄:26
查到的数据 id:4 name:张4 年龄:29
查到的数据 id:5 name:张5 年龄:18
查到的数据 id:6 name:张6 年龄:26
查到的数据 id:7 name:张7 年龄:20
查到的数据 id:8 name:张8 年龄:17
查到的数据 id:9 name:张9 年龄:14
查到的数据 id:10 name:张10 年龄:26
查到的数据 id:11 name:夏洛特 年龄:24
```
回到`Navicat`的`person`表点击刷新，可以看到新增的一条记录：

![Pasted Graphic 6.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/4d67ae1f7a354697b330c7762b80e6ed~tplv-k3u1fbpfcp-watermark.image?)

### 2、删除一条数据

```js
/**删除一条记录*/
- (IBAction)deleteData:(UIButton *)sender {
    [self openDatabase];
    NSString *deleteSplStr = [NSString stringWithFormat:@"DELETE FROM person where id = %@;",self.idTF.text];
    [self updateDataWithSql:deleteSplStr.UTF8String success:^{
        [self clearTextField];
        NSLog(@"删除数据成功");
    }];
}
```
把`id = 5` 的数据删除掉：

![Pasted Graphic 7.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/062ac1e18f1f4fc9bbff5d7cc346e730~tplv-k3u1fbpfcp-watermark.image?)
点击删除：

```js
Sqlite3Demo[47564:3319611] 删除数据成功
```
查看全部数据，发现`id`为**5**的数据已经没有了：

```js
------------------------------
查到的数据 id:1 name:张1 年龄:29
查到的数据 id:2 name:张2 年龄:16
查到的数据 id:3 name:张3 年龄:26
查到的数据 id:4 name:张4 年龄:29
查到的数据 id:6 name:张6 年龄:26
查到的数据 id:7 name:张7 年龄:20
查到的数据 id:8 name:张8 年龄:17
查到的数据 id:9 name:张9 年龄:14
查到的数据 id:10 name:张10 年龄:26
查到的数据 id:11 name:夏洛特 年龄:24
```
在`Navicate`里查看：

![Pasted Graphic 8.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/ca471c71ad6c4fe4a80fce62b041ffbb~tplv-k3u1fbpfcp-watermark.image?)

### 3、修改一条数据

```js
/**更新一条记录*/
- (IBAction)updateData:(UIButton *)sender {
    [self openDatabase];
    NSString *changeSqlStr = [NSString stringWithFormat:@"UPDATE person SET name = '%@' WHERE id = '%@';",self.nameTF.text,self.idTF.text];
    [self updateDataWithSql:changeSqlStr.UTF8String success:^{
        [self clearTextField];
        NSLog(@"修改成功");
    }];
}
/**更新记录操作*/
- (void)updateDataWithSql:(const char *)sql success:(void(^)(void))successBlock {
    /**
     第1个参数：一个已经打开的数据库对象
     第2个参数：sql语句
     第3个参数：参数2中取出多少字节的长度，-1 自动计算，\0停止取出
     第4个参数：准备语句
     第5个参数：通过参数3，取出参数2的长度字节之后，剩下的字符串
     */
    int result = sqlite3_prepare_v2(database, sql, -1, &stmt, nil);
    if (result != SQLITE_OK) {
        NSLog(@"操作失败,%d",result);
        sqlite3_finalize(stmt);
        sqlite3_close(self->database);
    } else {
        sqlite3_step(stmt);
        successBlock();
    }
}
```
把`id = 10`的名字改为赵云：

![Pasted Graphic 9.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/5ab74a851f1841db8d4fd605f369aae7~tplv-k3u1fbpfcp-watermark.image?)

查看全部：

```js
------------------------------
查到的数据 id:1 name:张1 年龄:29
查到的数据 id:2 name:张2 年龄:16
查到的数据 id:3 name:张3 年龄:26
查到的数据 id:4 name:张4 年龄:29
查到的数据 id:6 name:张6 年龄:26
查到的数据 id:7 name:张7 年龄:20
查到的数据 id:8 name:张8 年龄:17
查到的数据 id:9 name:张9 年龄:14
查到的数据 id:10 name:赵云 年龄:26
查到的数据 id:11 name:夏洛特 年龄:24
```
### 4、查找检索数据

```js
/**查询记录*/
- (IBAction)retrieveData:(UIButton *)sender {
    [self openDatabase];
    /**
     查询age < 25也可使用sql= "select * from person where age < 25"
     */
    NSString *sqlStr;
    if(_idTF.text.length > 0){
        sqlStr = [NSString stringWithFormat:@"SELECT id, name, age FROM person WHERE id = '%@';",_idTF.text];
    }
    else if(_nameTF.text.length > 0){
        sqlStr = [NSString stringWithFormat:@"SELECT id, name, age FROM person WHERE name = '%@';",_nameTF.text];
    }
    else if(_ageTF.text.length > 0){
        sqlStr = [NSString stringWithFormat:@"SELECT id,name,age FROM person WHERE age %@;",_ageTF.text];
        NSLog(@"sqlStr == %@",sqlStr);
    }
    [self operationData:sqlStr.UTF8String];
}
```
查`id = 4`的记录：

![Pasted Graphic 10.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/e075820b56094f279f247ea0eb2dbd6d~tplv-k3u1fbpfcp-watermark.image?)

```js
------------------------------
查到的数据 id:4 name:张4 年龄:29
```
查年龄小于**25**的记录：

![Pasted Graphic 11.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/c87a21b882f0412e8447fd4d511e143d~tplv-k3u1fbpfcp-watermark.image?)

点击查按钮：
```js
------------------------------
查到的数据 id:2 name:张2 年龄:16
查到的数据 id:7 name:张7 年龄:20
查到的数据 id:8 name:张8 年龄:17
查到的数据 id:9 name:张9 年龄:14
查到的数据 id:11 name:夏洛特 年龄:24
```
查找名字赵云：

```js
------------------------------
查到的数据 id:10 name:赵云 年龄:26
```
点击增加多条数据测试，再查看全部数据：

```js
------------------------------
查到的数据 id:1 name:张1 年龄:29
查到的数据 id:2 name:张2 年龄:16
查到的数据 id:3 name:张3 年龄:26
查到的数据 id:4 name:张4 年龄:29
查到的数据 id:6 name:张6 年龄:26
查到的数据 id:7 name:张7 年龄:20
查到的数据 id:8 name:张8 年龄:17
查到的数据 id:9 name:张9 年龄:14
查到的数据 id:10 name:赵云 年龄:26
查到的数据 id:11 name:夏洛特 年龄:24
查到的数据 id:12 name:张1 年龄:21
查到的数据 id:13 name:张2 年龄:11
查到的数据 id:14 name:张3 年龄:20
查到的数据 id:15 name:张4 年龄:28
查到的数据 id:16 name:张5 年龄:25
查到的数据 id:17 name:张6 年龄:24
查到的数据 id:18 name:张7 年龄:22
查到的数据 id:19 name:张8 年龄:29
查到的数据 id:20 name:张9 年龄:14
查到的数据 id:21 name:张10 年龄:26
```
查看`Navicat`的`person`：

![Pasted Graphic 12.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/832c3f084f084cecae4263857fef866b~tplv-k3u1fbpfcp-watermark.image?)

查看`squence`，点击刷新，`id`为**21**完全一致：

![Pasted Graphic 13.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/5e9f83eda02a4dc98bd506f5fc8bca6e~tplv-k3u1fbpfcp-watermark.image?)

点击清空：

```js
/**清空表中数据和自增排序*/
- (IBAction)removeAllData:(UIButton *)sender {
    [self openDatabase];
    const char *deleteSpl = "DELETE FROM person";
    [self updateDataWithSql:deleteSpl success:^{
        NSLog(@"清空表数据成功");
    }];
}
```
表数据已经被清空：

![Pasted Graphic 14.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/500b91b7ca0f496a9c4fac5aec36de33~tplv-k3u1fbpfcp-watermark.image?)

这个时候`sqlite_sequence`还是**21**，没有归**0**：


![Pasted Graphic 15.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/6c90cd3774c84afb8bb541528fd43338~tplv-k3u1fbpfcp-watermark.image?)

此时点击插入**10**条数据，发现`id`从**21**开始：

![Pasted Graphic 16.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/bc82079c9d2b449a9fd65edd05c2f384~tplv-k3u1fbpfcp-watermark.image?)

并没有真正的清空，需要给数据添加`seq`归**0**操作：
>sqlite3 没有truncate关键字：

```js
/**清空表中数据和自增排序*/
- (IBAction)removeAllData:(UIButton *)sender {
    [self openDatabase];
    const char *deleteSpl = "DELETE FROM person";
    [self updateDataWithSql:deleteSpl success:^{
        NSLog(@"清空表数据成功");
        const char *setSeqSpl = "UPDATE sqlite_sequence SET seq = 0 WHERE name = 'person'";
        [self updateDataWithSql:setSeqSpl success:^{
            NSLog(@"自增主键归零");
        }];
    }];
}
```
再点击清空：

![Pasted Graphic 17.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/764f3b6068364e58bc144533ecc8ee09~tplv-k3u1fbpfcp-watermark.image?)

点击插入多条，`id`又从**1**开始：

![Pasted Graphic 18.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/6c721c7319844881ba47773c31242c5a~tplv-k3u1fbpfcp-watermark.image?)

## 四、总结
`SQLite`接口都是`C`语言，并不面向对象，所有操作需要自己手写`SQL`语句。

### 参考：
- [iOS-SQLite的使用](https://www.jianshu.com/p/4a0e6773c694)
- [iOS数据库的使用（二）：sqlite的基本使用](https://www.jianshu.com/p/f95c5a5b9af3)
- [iOS中sqlite3的增、删、改、查](https://www.jianshu.com/p/568cba50c2a6)
- [SQLite清空表并将自增列归零](http://www.360doc.com/content/22/0418/14/67767017_1027092464.shtml)
- [iOS 中使用 SQLite](https://www.jianshu.com/p/2333fad79f2f)

