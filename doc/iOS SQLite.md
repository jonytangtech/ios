# Sqlite
## 一、前言
>虽然平时开发不会直接使用`SQLite`，但是作为一个轻量级数据库，理解和掌握其基本原理和使用还是有必要的。`FMDB`就是对`SQLite`的封装，并且微信团队在自研自己的数据库前也是使用`SQLite`。

点击项目名称 -> `TAGETS` -> `Build Phases` -> `Link Binary With Libraries` 点击添加`libsqlite3.tbd`：

<img width="850" alt="Pasted Graphic" src="https://user-images.githubusercontent.com/126937296/227779824-9e4c138b-6aa8-493b-b8d5-e97c4694eac8.png">

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
模拟器先拉好**UI**：

<img width="336" alt="Pasted Graphic 1" src="https://user-images.githubusercontent.com/126937296/227779833-9caa14af-d7a6-4bf4-bee4-0842c807184f.png">

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

<img width="235" alt="Pasted Graphic 4" src="https://user-images.githubusercontent.com/126937296/227779861-f32f9d35-4645-492e-909a-57c00ee44038.png">

用`Navicat`打开`person_info.sqlite`，`person`的表已经创建好了：

<img width="553" alt="Pasted Graphic 3" src="https://user-images.githubusercontent.com/126937296/227779874-997f6192-8116-4a72-a79c-31ce28c443f0.png">

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

<img width="558" alt="1__#$!@%!#__Pasted Graphic 3" src="https://user-images.githubusercontent.com/126937296/227779906-bc97cc6f-ea1d-4587-a022-e4033e14ee6c.png">

还有一个`sqlite_sequence`，`seq`是**10**：

<img width="394" alt="1__#$!@%!#__Pasted Graphic 4" src="https://user-images.githubusercontent.com/126937296/227779922-46c6bfce-4f16-4329-917d-0a29291e143e.png">

>注：如果没有看到数据，点击左边下面的按钮刷新，右键刷新无效。

<img width="258" alt="1__#$!@%!#__Pasted Graphic 1" src="https://user-images.githubusercontent.com/126937296/227779930-bb7affd2-bcb7-4202-bd57-7c0161e89242.png">

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

<img width="335" alt="Pasted Graphic 5" src="https://user-images.githubusercontent.com/126937296/227779945-9d38bd28-71f9-456b-88ef-818890aa1023.png">

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

<img width="461" alt="Pasted Graphic 6" src="https://user-images.githubusercontent.com/126937296/227779957-66d9bfc0-82b7-4503-a0a5-1367fffaa2e9.png">

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

<img width="323" alt="Pasted Graphic 7" src="https://user-images.githubusercontent.com/126937296/227779963-cf0b952f-9fe0-4389-ad30-057f63440770.png">

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

<img width="457" alt="Pasted Graphic 8" src="https://user-images.githubusercontent.com/126937296/227779978-5445263b-1509-4fd6-b88c-9d5b43f20e72.png">

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

<img width="314" alt="Pasted Graphic 9" src="https://user-images.githubusercontent.com/126937296/227779994-a72a4451-c440-438b-9357-333a1dd70964.png">

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

<img width="333" alt="Pasted Graphic 10" src="https://user-images.githubusercontent.com/126937296/227780014-f2dfe3eb-78f7-4dc1-b845-247efdb1746f.png">

```js
------------------------------
查到的数据 id:4 name:张4 年龄:29
```
查年龄小于**25**的记录：

<img width="332" alt="Pasted Graphic 11" src="https://user-images.githubusercontent.com/126937296/227780027-c00fabd4-f708-4076-857e-65a8c13ca795.png">

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

<img width="436" alt="Pasted Graphic 12" src="https://user-images.githubusercontent.com/126937296/227780042-26e72690-d4e1-4535-bf29-c771cad685cf.png">

查看`squence`，点击刷新，`id`为**21**完全一致：

<img width="353" alt="Pasted Graphic 13" src="https://user-images.githubusercontent.com/126937296/227780055-44d2a29f-f36a-455e-914e-ff85f95d9a78.png">

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

<img width="458" alt="Pasted Graphic 14" src="https://user-images.githubusercontent.com/126937296/227780063-c53f5900-585f-4c84-8afd-7354461635fd.png">

这个时候`sqlite_sequence`还是**21**，没有归**0**：

<img width="394" alt="Pasted Graphic 15" src="https://user-images.githubusercontent.com/126937296/227780073-41401f4e-9eca-4a26-a4ae-e00d33f62347.png">

此时点击插入**10**条数据，发现`id`从**21**开始：

<img width="466" alt="Pasted Graphic 16" src="https://user-images.githubusercontent.com/126937296/227780094-0c9d86ae-5fbd-46b3-9efd-e89f7dd2252a.png">

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

<img width="387" alt="Pasted Graphic 17" src="https://user-images.githubusercontent.com/126937296/227780108-b63027cb-477c-4773-9253-b571e0f09eaf.png">

点击插入多条，`id`又从**1**开始：

<img width="464" alt="Pasted Graphic 18" src="https://user-images.githubusercontent.com/126937296/227780137-65e92401-e402-4216-8bc3-589e1aa38147.png">

## 四、总结
`SQLite`接口都是`C`语言，并不面向对象，所有操作需要自己手写`SQL`语句。

### 参考：
- [iOS-SQLite的使用](https://www.jianshu.com/p/4a0e6773c694)
- [iOS数据库的使用（二）：sqlite的基本使用](https://www.jianshu.com/p/f95c5a5b9af3)
- [iOS中sqlite3的增、删、改、查](https://www.jianshu.com/p/568cba50c2a6)
- [SQLite清空表并将自增列归零](http://www.360doc.com/content/22/0418/14/67767017_1027092464.shtml)
- [iOS 中使用 SQLite](https://www.jianshu.com/p/2333fad79f2f)

