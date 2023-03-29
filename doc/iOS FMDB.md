# FMDB
`FMDB`是对`SQLite`的面向对象封装，主要包含 **3** 个类：

1. `FMDatabase` 表示单个`SQLite`数据库，用于执行`SQL`语句；
2. `FMResultSet` 表示对`FMDatabase`执行查询的结果；
3. `FMDatabaseQueue` 如果你想在多个线程上执行查询和更新，需要使用这个类。
 
## 一、数据库测试准备
### 1、创建或打开数据库

```js
/**创建或打开数据库
 */
- (void)openDB{
    
    if(self.db.isOpen){
        NSLog(@"数据库已经打开");
    }else{
        NSString *docsPath = NSSearchPathForDirectoriesInDomains(NSDocumentDirectory, 
        NSUserDomainMask, YES)[0];
        NSString *dbPath   = [docsPath stringByAppendingPathComponent:@"person_info.db"];
        NSLog(@"path == %@",dbPath);
        FMDatabase *db = [FMDatabase databaseWithPath:dbPath];
        _db = db;
        if (![self.db open]) {
            self.db = nil;
            NSLog(@"数据库打开失败");
            return;
        }else{
            if(self.db.goodConnection){
                NSLog(@"数据库连接成功");
                [self.db open];
                if(self.db.isOpen){
                    NSLog(@"数据库已经打开");
                    [self createTable];
                }
            }
            else{
                NSLog(@"数据库连接失败");
            }
        }
    }
}
```

### 2、创建表

```js
/**创建表
 */
- (void)createTable{
    NSString *sql = [NSString stringWithFormat:
                     @"CREATE TABLE IF NOT EXISTS 'person'("
                     "id INTEGER PRIMARY KEY AUTOINCREMENT NOT NULL,"
                     "name TEXT NOT NULL,"
                     "age INTEGER NOT NULL);"];
    BOOL success = [_db executeStatements:sql];
    if (success) {
        NSLog(@"创建或打开表成功");
    } else {
        NSLog(@"创建表或打开表失败 - %@",[_db lastErrorMessage]);
    }
}
```
### 3、准备好UI界面

![Pasted Graphic.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/1c3927966b9d4cd4b93527805f37e1a7~tplv-k3u1fbpfcp-watermark.image?)

点击模拟器`查全部`按钮，打开数据库和创建表：


```js
FMDBDemo[95834:1335897] path == /Users/xxxxx/Library/Developer/CoreSimulator/Devices/
.../data/Containers/Data/Application/…/Documents/person_info.db
FMDBDemo[95834:1335897] 数据库连接成功
FMDBDemo[95834:1335897] 数据库已经打开
FMDBDemo[95834:1335897] 创建或打开表成功
FMDBDemo[95834:1335897] 检索数据成功
----------------------------------
FMDBDemo[95834:1335897] person表没有数据
```
### 4、添加多条数据
在`FMDB`,任何类型的不是`SELECT`查询语句的 `SQL` 语句都可以使用更新`executeUpdate:`语句。这包括`CREATE`, `UPDATE`, `INSERT`, `ALTER`, `COMMIT`, `BEGIN`, `DETACH`, `DELETE`, `DROP`, `END`, `EXPLAIN`,`VACUUM`和`REPLACE`语句（以及更多）。基本上，如果你的`SQL`语句不以开头`SELECT`，它就是一个更新语句。

执行更新返回 `BOOL`值。返回值为`YES`表示更新已成功执行，返回值为`NO`表示遇到错误。可以调用`-lastErrorMessage`和`-lastErrorCode`方法来检索更多信息。



```js
- (IBAction)insertMultipleData:(UIButton *)sender {
    [self openDB];
    for (int i = 0; i < 10; i++) {
        NSString *name = [NSString stringWithFormat:@"张%d",i+1];
        int age = arc4random_uniform(20) + 10;
        // 拼接 sql 语句
        NSString *sql = [NSString stringWithFormat:@"INSERT INTO person (name,age) 
        VALUES ('%@',%d);",name,age];
        // 执行 sql 语句
        BOOL success = [_db executeUpdate:sql];
        if (success) {
            NSLog(@"插入数据成功 - %@",name);
        } else {
            NSLog(@"插入数据失败 - %@",[_db lastErrorMessage]);
        }
    }
    [_db close];
}
```
点击模拟器`新增多条`按钮：

```js
FMDBDemo[95834:1335897] 数据库连接成功
FMDBDemo[95834:1335897] 数据库已经打开
FMDBDemo[95834:1335897] 创建或打开表成功
FMDBDemo[95834:1335897] 插入数据成功 - 张1
FMDBDemo[95834:1335897] 插入数据成功 - 张2
FMDBDemo[95834:1335897] 插入数据成功 - 张3
FMDBDemo[95834:1335897] 插入数据成功 - 张4
FMDBDemo[95834:1335897] 插入数据成功 - 张5
FMDBDemo[95834:1335897] 插入数据成功 - 张6
FMDBDemo[95834:1335897] 插入数据成功 - 张7
FMDBDemo[95834:1335897] 插入数据成功 - 张8
FMDBDemo[95834:1335897] 插入数据成功 - 张9
FMDBDemo[95834:1335897] 插入数据成功 - 张10
```

### 5、查询全部数据
语句`SELECT`是一个查询，通过其中一种方法执行`-executeQuery`，执行查询`FMResultSet`成功则返回一个对象，`nil`失败则返回一个对象。您应该使用-`lastErrorMessage`和`-lastErrorCode`方法来确定查询失败的原因。为了遍历查询结果，您使用了一个`while()`循环。您还需要从一条记录“**步进**”到另一条记录。

```js
/**检索全部数据
 */
- (IBAction)retrieveAllData:(UIButton *)sender {
    [self openDB];
    NSString *searchSqlStr = @"SELECT * FROM person";
    BOOL success = [_db executeStatements:searchSqlStr];
    if (success) {
        NSLog(@"检索数据成功");
        FMResultSet *s = [_db executeQuery:searchSqlStr];
        int count = 0;
        printf("----------------------------------\n");
        while ([s next]) {
            printf("查到的数据 id:%s\t名字:%s\t\t年龄:%s\n",
            [s stringForColumn:[s columnNameForIndex:0]].UTF8String,
            [s stringForColumn:[s columnNameForIndex:1]].UTF8String,
            [s stringForColumn:[s columnNameForIndex:2]].UTF8String);
            count ++;
        }
        if(count == 0){
            NSLog(@"person表没有数据");
        }
       
    } else {
        NSLog(@"检索数据失败 - %@",[_db lastErrorMessage]);
    }
    [_db close];
}
```
点击模拟器`查全部`按钮：

```js
----------------------------------
查到的数据 id:1	名字:张1		年龄:22
查到的数据 id:2	名字:张2		年龄:10
查到的数据 id:3	名字:张3		年龄:28
查到的数据 id:4	名字:张4		年龄:25
查到的数据 id:5	名字:张5		年龄:23
查到的数据 id:6	名字:张6		年龄:25
查到的数据 id:7	名字:张7		年龄:19
查到的数据 id:8	名字:张8		年龄:16
查到的数据 id:9	名字:张9		年龄:14
查到的数据 id:10	名字:张10	年龄:10
```
在`Navicat`里查看：

![Pasted Graphic 2.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/0fb150511a104c00a1f27cac2a2a84b5~tplv-k3u1fbpfcp-watermark.image?)

### 6、清空表数据

```js
/**清空表数据
 */
- (IBAction)removeAllData:(UIButton *)sender {
    [self openDB];
    NSString *sql = @"DELETE FROM person";
    // 执行 sql 语句
    BOOL success = [_db executeUpdate:sql];
    if (success) {
        NSLog(@"清空表数据成功");
        NSString *setSeqSpl = @"UPDATE sqlite_sequence SET seq = 0 WHERE name = 'person'";
        //        BOOL success = [_db executeStatements:setSeqSpl];
        BOOL success = [_db executeUpdate:setSeqSpl];
        if (success) {
            NSLog(@"自增主键归零");
        }else{
            NSLog(@"自增主键归零失败 - %@",[_db lastErrorMessage]);
        }
    } else {
        NSLog(@"清空表数据失败 - %@",[_db lastErrorMessage]);
    }
    [_db close];
}
```
点击模拟器`清空全部`按钮：

```js
FMDBDemo[95834:1335897] 数据库连接成功
FMDBDemo[95834:1335897] 数据库已经打开
FMDBDemo[95834:1335897] 创建或打开表成功
FMDBDemo[95834:1335897] 清空表数据成功
FMDBDemo[95834:1335897] 自增主键归零
```
`Navicat`里查看表，数据确实被清空：

![Pasted Graphic 3.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/a56a9579adbd475d8f98098a960d5f99~tplv-k3u1fbpfcp-watermark.image?)
`sqlite_sequenece`也变为 **0** ：

![Pasted Graphic 4.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/af6c3c8c32114cdd8afc97b8ababec70~tplv-k3u1fbpfcp-watermark.image?)

## 二、增删改查 操作
### 1、新增数据

```js
/**增
 */
- (IBAction)insertData:(UIButton *)sender {
    [self openDB];
    NSString *insertSplStr = [NSString stringWithFormat:@"INSERT INTO person (name,age) 
    VALUES ('%@',%@);",_nameTF.text,_ageTF.text];
    BOOL success = [_db executeUpdate:insertSplStr];
    if (success) {
        NSLog(@"插入数据成功");
    } else {
        NSLog(@"插入数据失败 - %@",[_db lastErrorMessage]);
    }
    [self clearTextField];
}
```
模拟器添加数据，点击`增`按钮：

![Pasted Graphic 5.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/56ca06f50d0f4f3ca216e30c5a30455f~tplv-k3u1fbpfcp-watermark.image?)

运行：
```js
FMDBDemo[95834:1335897] 数据库连接成功
FMDBDemo[95834:1335897] 数据库已经打开
FMDBDemo[95834:1335897] 创建或打开表成功
FMDBDemo[95834:1335897] 插入数据成功
```
查看全部，就有一条数据：

```js
----------------------------------
查到的数据 id:1	名字:鲁班	年龄:26
```
点击新增多条数据：
```js
----------------------------------
查到的数据 id:1	名字:鲁班	年龄:26
查到的数据 id:2	名字:张1		年龄:21
查到的数据 id:3	名字:张2		年龄:16
查到的数据 id:4	名字:张3		年龄:21
查到的数据 id:5	名字:张4		年龄:21
查到的数据 id:6	名字:张5		年龄:13
查到的数据 id:7	名字:张6		年龄:17
查到的数据 id:8	名字:张7		年龄:24
查到的数据 id:9	名字:张8		年龄:24
查到的数据 id:10	名字:张9		年龄:13
查到的数据 id:11	名字:张10	年龄:28
```

### 2、删除数据

```js
/**删
 */
- (IBAction)deleteData:(UIButton *)sender {
    [self openDB];
    NSString *deleteSplStr = [NSString stringWithFormat:@"DELETE FROM person where id = %@;",
    self.idTF.text];
    BOOL success = [_db executeUpdate:deleteSplStr];
    if (success) {
        NSLog(@"删除数据成功");
    } else {
        NSLog(@"删除数据失败 - %@",[_db lastErrorMessage]);
    }
    [self clearTextField];
}
```
删除`id`为 **5** 的数据：

![Pasted Graphic 6.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/aa7ac727005645f687ce87f11638d865~tplv-k3u1fbpfcp-watermark.image?)

点击`删`按钮，查看打印数据,`id`为 **5** 的数据已被删除：

```js
----------------------------------
查到的数据 id:1	名字:鲁班	年龄:26
查到的数据 id:2	名字:张1		年龄:21
查到的数据 id:3	名字:张2		年龄:16
查到的数据 id:4	名字:张3		年龄:21
查到的数据 id:6	名字:张5		年龄:13
查到的数据 id:7	名字:张6		年龄:17
查到的数据 id:8	名字:张7		年龄:24
查到的数据 id:9	名字:张8		年龄:24
查到的数据 id:10	名字:张9		年龄:13
查到的数据 id:11	名字:张10	年龄:28
```
### 3、修改数据

```js
/**改
 */
- (IBAction)updateData:(UIButton *)sender {
    [self openDB];
    
    NSString *changeSqlStr = [NSString stringWithFormat:@"UPDATE person SET name = '%@' 
    WHERE id = '%@';",self.nameTF.text,self.idTF.text];
    BOOL success = [_db executeUpdate:changeSqlStr];
    if (success) {
        NSLog(@"修改数据成功");
    } else {
        NSLog(@"修改数据失败 - %@",[_db lastErrorMessage]);
    }
    [self clearTextField];
}
```
将`id`为 **10** 的名字改为妲己：

![Pasted Graphic 7.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/6fe20774b96b4bef888195eb89898de0~tplv-k3u1fbpfcp-watermark.image?)
点击`改`按钮，查看打印数据：

```js
----------------------------------
查到的数据 id:1	名字:鲁班	年龄:26
查到的数据 id:2	名字:张1		年龄:21
查到的数据 id:3	名字:张2		年龄:16
查到的数据 id:4	名字:张3		年龄:21
查到的数据 id:6	名字:张5		年龄:13
查到的数据 id:7	名字:张6		年龄:17
查到的数据 id:8	名字:张7		年龄:24
查到的数据 id:9	名字:张8		年龄:24
查到的数据 id:10	名字:妲己	年龄:13
查到的数据 id:11	名字:张10	年龄:28
```
### 4、查询数据

```js
- (IBAction)retrieveData:(UIButton *)sender {
    [self openDB];
    
    NSString *sqlStr;
    if(_idTF.text.length > 0){
        sqlStr = [NSString stringWithFormat:@"SELECT id, name, age FROM person 
        WHERE id = '%@';",_idTF.text];
    }
    else if(_nameTF.text.length > 0){
        sqlStr = [NSString stringWithFormat:@"SELECT id, name, age FROM person 
        WHERE name = '%@';",_nameTF.text];
    }
    else if(_ageTF.text.length > 0){
        sqlStr = [NSString stringWithFormat:@"SELECT id,name,age FROM person 
        WHERE age %@;",_ageTF.text];
        NSLog(@"sqlStr == %@",sqlStr);
    }
    BOOL success = [_db executeStatements:sqlStr];
    if (success) {
        FMResultSet *s = [_db executeQuery:sqlStr];
        while ([s next]) {
            printf("查到的数据 id:%s\t名字:%s\t\t年龄:%s\n",
            [s stringForColumn:[s columnNameForIndex:0]].UTF8String,
            [s stringForColumn:[s columnNameForIndex:1]].UTF8String,
            [s stringForColumn:[s columnNameForIndex:2]].UTF8String);
        }
    } else {
        NSLog(@"检索数据失败 - %@",[_db lastErrorMessage]);
    }
    [self clearTextField];
}

/**清空输入查询
 */
- (void)clearTextField{
    self.idTF.text = @"";
    self.nameTF.text = @"";
    self.ageTF.text = @"";
    [_db close];
}
```
查询年龄大于 **25** 的数据：

![Pasted Graphic 8.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/a503da9294194fb9abe95a152059832e~tplv-k3u1fbpfcp-watermark.image?)

点击`查`按钮，发现 **2** 条数据符合：

```js
----------------------------------
查到的数据 id:1	名字:鲁班	年龄:26
查到的数据 id:11	名字:张10	年龄:28
```

>注意，当您完成对数据库的查询和更新后，您应该`-close`建立`FMDatabase`连接，以便`SQLite`放弃它在操作过程中获得的任何资源。

## 三、总结：
`FMDB`是对`SQLite`的一层包装，相对于`SQLite`会更加面向对象，但仍然需要自己手写`SQL`语句。`FMDB`支持多条语句和批处理，可以使用`FMDatabase`的 `executeStatements:withResultBlock:` 在字符串中执行多个语句。
如果需要线程安全，可以使用`FMDatabaseQueue`，支持跨多个线程访问。并且，支持事务处理，如果事务，如果事务失败会回滚。
### 参考
- [ccgus/fmdb](https://github.com/ccgus/fmdb)

