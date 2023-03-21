# 对象序列化
## 一、NSCoding
有时需要对自定义的类对象进行数据持久化存储，但是`NSUserDefaults`只能对系统的数据类型进行存储，而且还有存储延迟的问题。`NSCoding`是使自定义对象能够被编码和解码以进行归档和分发的协议。

>NSCoding协议声明了一个类必须实现的两个方法，这样该类的实例才能被编码和解码。这种功能为归档(对象和其他结构存储在磁盘上)和分发(对象被复制到不同的地址空间)提供了基础。根据面向对象的设计原则，被编码或解码的对象负责对其实例变量进行编码和解码。编码器通过调用`encodeWithCoder:`或`initWithCoder:`来指示对象这样做。`encodeWithCoder:`指示对象将其实例变量编码到所提供的编码器;对象可以接收此方法任意次数。`initWithCoder:`指示对象从提供的编码器中的数据初始化自身;因此，它取代了任何其他初始化方法，并且每个对象只发送一次。任何可编码的对象类都必须采用`NSCoding`协议并实现其方法。

定义一个用户类：

```js
@interface User : NSObject<NSCoding>
@property (nonatomic, strong) NSString *registerName;
@property (nonatomic, strong) NSString *nickname;
@property (nonatomic, strong) NSString *phoneNumber;
@property (nonatomic, assign) BOOL isMember;
@property (nonatomic, assign) int balance;
@end
```
`.m`文件进行归档编码操作：

```js
@implementation User
- (void)encodeWithCoder:(nonnull NSCoder *)coder {
    [coder encodeObject:_registerName forKey:@"registerName"];
    [coder encodeObject:_nickname forKey:@"nickname"];
    [coder encodeObject:_phoneNumber forKey:@"phoneNumber"];
    [coder encodeBool:_isMember forKey:@"isMember"];
    [coder encodeInt:_balance forKey:@"balance"];
}
- (nullable instancetype)initWithCoder:(nonnull NSCoder *)coder {
    if(self = [super init]){
        if(coder){
            _registerName = [coder decodeObjectOfClass:[NSString class] forKey:@"registerName"];
            _nickname = [coder decodeObjectOfClass:[NSString class] forKey:@"nickname"];
            _phoneNumber = [coder decodeObjectOfClass:[NSString class] forKey:@"phoneNumber"];
            _isMember = [coder decodeBoolForKey:@"isMember"];
            _balance = [coder decodeIntForKey:@"balance"];
        }
    }
    return self;
}
@end
```
模拟器添加两个按钮，一个写入，一个读取：

<img width="264" alt="Pasted Graphic 1" src="https://user-images.githubusercontent.com/126937296/226541040-f560ed94-b362-450a-8b8a-b3067376062a.png">

点击把自定义数据存进本地：

```js
- (IBAction)insertData:(UIButton *)sender {
    
    User *user = [User new];
    user.registerName = @"孙悟空";
    user.nickname = @"猴子";
    user.phoneNumber = @"01234";
    user.isMember = YES;
    user.balance = 123;
    
    User *user2 = [User new];
    user2.registerName = @"庄周";
    user2.nickname = @"鱼";
    user2.phoneNumber = @"01245";
    user2.isMember = YES;
    user2.balance = 111;
    
    NSArray <User *>*userArr = [NSArray arrayWithObjects:user,user2, nil];
    NSData *perData = [NSKeyedArchiver archivedDataWithRootObject:userArr];
//    写入本地
    NSString *path = [NSHomeDirectory() stringByAppendingPathComponent:@"User.archiver"];
    BOOL result = [[NSFileManager defaultManager]createFileAtPath:path contents:nil attributes:nil];
    if (result){
        [perData writeToFile:path atomically:YES];
    }
}
```
点击把数写入本地的自定义数据取出来：

```js
- (IBAction)getData:(UIButton *)sender {
    NSString *path = [NSHomeDirectory() stringByAppendingPathComponent:@"User.archiver"];
    NSData *data = [NSData dataWithContentsOfFile:path];
    NSArray <User *>*userArr = [NSKeyedUnarchiver unarchiveObjectWithData:data];
    NSLog(@"userArr = %@",userArr);
    for (User *user in  userArr){
        NSLog(@"\n - 注册名称：%@\n - 昵称：%@\n - 号码：%@\n - 是否会员：%d\n - 账户余额：%d", user.registerName,
              user.nickname,
              user.phoneNumber,
              user.isMember,
              user.balance
              );
    }
}
```
点击写入数据，运行打印：

```js
NSCodingDemo[48440:1221308] 写入数据成功
点击读取数据：
NSCodingDemo[48440:1221308] userArr = (
    "<User: 0x600002c89950>",
    "<User: 0x600002c885a0>"
)
NSCodingDemo[48440:1221308] 
 - 注册名称：孙悟空
 - 昵称：猴子
 - 号码：01234
 - 是否会员：1
 - 账户余额：123
NSCodingDemo[48440:1221308] 
 - 注册名称：庄周
 - 昵称：鱼
 - 号码：01245
 - 是否会员：1
 - 账户余额：111
```
## 二、废弃提醒

存档方法废弃提醒：

<img width="777" alt="Pasted Graphic 3" src="https://user-images.githubusercontent.com/126937296/226541090-a0a5e8aa-d8b1-42d0-a289-a2d84804bd6f.png">

修改存档方法：

```js
//    NSData *perData = [NSKeyedArchiver archivedDataWithRootObject:userArr];
NSError *error = nil;
NSData *perData = [NSKeyedArchiver archivedDataWithRootObject:userArr requiringSecureCoding:YES error:&error];
```
解档方法废弃提醒：

<img width="788" alt="Pasted Graphic 4" src="https://user-images.githubusercontent.com/126937296/226541114-a46bf9ab-ae29-4102-b6bd-d8a5705db509.png">

修改废弃提醒：

```js
//    NSArray <User *>*userArr = [NSKeyedUnarchiver unarchiveObjectWithData:data];
NSError *error = nil;
NSArray <User *>*userArr = [NSKeyedUnarchiver unarchivedArrayOfObjectsOfClass:[User class] fromData:data error:&error];
```
重新运行：

```js
NSCodingDemo[48863:1233999] 写入数据成功
NSCodingDemo[48863:1233999] userArr = (null)
```
没能读取数据：
>研究了一下是要改成`NSSecureCoding`，由于这种技术可能是不安全的，因为当您可以验证类类型时，对象已经构造好了，并且如果它是集合类的一部分，则可能插入到对象图中。为了符合`NSSecureCoding`: 不重写`initWithCoder:`的对象可以不做任何更改地符合`NSSecureCoding`(假设它是另一个符合`NSSecureCoding`的类的子类)。覆盖`initWithCoder:`的对象必须使用`decodeObjectOfClass:forKey:`方法解码任何包含的对象。例如:`id obj = [decoder decodeObjectOfClass:[MyClass类] forKey: @“myKey”);` 此外，该类必须重写其`supportsSecureCoding`属性的`getter`以返回`YES`。

## 三、NSSecureCoding
一种能够以一种健壮的方式对对象替换攻击进行编码和解码的协议。
把`User`遵循协议改成`NSSecureCoding`：

```js
@interface User : NSObject<NSSecureCoding>
```
添加支持安全编码：

```js
+ (BOOL)supportsSecureCoding{
    return YES;
}
```
运行测试：

```js
userArr = (
    "<User: 0x600003f6ebe0>",
    "<User: 0x600003f6ea30>"
)
NSCodingDemo[54390:1372398] 
 - 注册名称：孙悟空
 - 昵称：猴子
 - 号码：01234
 - 是否会员：1
 - 账户余额：123
NSCodingDemo[54390:1372398] 
 - 注册名称：庄周
 - 昵称：鱼
 - 号码：01245
 - 是否会员：1
 - 账户余额：111
```
## 四、YYCacheDemo适配问题
由于`YYCache`很久不更新，在`YYDiskCache`里面，`- (id<NSSecureCoding>)objectForKey:(NSString *)key`方法里面，也是提示方法废弃：

<img width="682" alt="Pasted Graphic" src="https://user-images.githubusercontent.com/126937296/226541167-df029bd3-485d-40f4-92c3-96f857c9e138.png">

```js
object = [NSKeyedUnarchiver unarchiveObjectWithData:item.value];
```
下面的`- (void)setObject:(id<NSSecureCoding>)object forKey:(NSString *)key`方法里面，也是提示方法废弃：

<img width="677" alt="1__#$!@%!#__Pasted Graphic 1" src="https://user-images.githubusercontent.com/126937296/226541185-fe16b7be-d509-4790-971a-b5e89ac15e20.png">

```js
value = [NSKeyedArchiver archivedDataWithRootObject:object requiringSecureCoding:YES error:&error];
```
解档添加版本适配：

```js
if (@available(iOS 12.0, *)) {
    NSError *error = nil;
    value = [NSKeyedArchiver archivedDataWithRootObject:object requiringSecureCoding:YES
error:&error];
} else {
    // 消除方法弃用(过时)的警告
    #pragma clang diagnostic push
    #pragma clang diagnostic ignored "-Wdeprecated-declarations"
    // 要消除警告的代码
    value = [NSKeyedArchiver archivedDataWithRootObject:object];
    #pragma clang diagnostic pop
}
```
归档添加版本适配：

```js
if (@available(iOS 12.0, *)) {
    NSError *error = nil;
    object = [NSKeyedUnarchiver unarchivedObjectOfClasses:[NSSet
setWithArray:@[NSObject.class]] fromData:item.value error:&error];
}
else {
    // 消除方法弃用(过时)的警告
    #pragma clang diagnostic push
    #pragma clang diagnostic ignored "-Wdeprecated-declarations"
    // 要消除警告的代码
    object = [NSKeyedUnarchiver unarchiveObjectWithData:item.value];
    #pragma clang diagnostic pop
}
```
把`YYCache`里面所有的`NSCoding`改成`NSSecureCoding`，在自己自定义的类里面也添加支持安全编码：

```js
+ (BOOL)supportsSecureCoding{
    return YES;
}
```
运行测试，发现找不到自定义的类：

<img width="937" alt="Pasted Graphic 2" src="https://user-images.githubusercontent.com/126937296/226541237-8eb4c84b-e1c0-41ad-9ebf-5bf968c4a0fe.png">

就是需要指定一个具体的类名，让他做解码操作，由于`YYCache`是`pod`下来的，不能直接导入文件，为了避免相互引用，只用`runtime`获取类：

```js
object = [NSKeyedUnarchiver unarchivedObjectOfClasses:[NSSet setWithArray:@[objc_getClass("ContactsModel")]] fromData:item.value error:&error]
```
运行测试：

```js
YYCacheDemo[57594:1453699] disk name = 张0 phoneNumber = 15888899990
YYCacheDemo[57594:1453699] disk name = 张1 phoneNumber = 15888899991
YYCacheDemo[57594:1453699] disk name = 张2 phoneNumber = 15888899992
YYCacheDemo[57594:1453699] disk name = 张3 phoneNumber = 15888899993
YYCacheDemo[57594:1453699] disk name = 张4 phoneNumber = 15888899994
YYCacheDemo[57594:1453699] disk name = 张5 phoneNumber = 15888899995
YYCacheDemo[57594:1453699] disk name = 张6 phoneNumber = 15888899996
YYCacheDemo[57594:1453699] disk name = 张7 phoneNumber = 15888899997
YYCacheDemo[57594:1453699] disk name = 张8 phoneNumber = 15888899998
YYCacheDemo[57594:1453699] disk name = 张9 phoneNumber = 15888899999
```
没有警告，运行正常。
## 五、优化代码
如果后面需要添加新的`model`，就需要继续给`NSKeyedUnarchiver unarchivedObjectOfClasses:`方法添加解档归档类名，需要经常修改原始框架代码，这样对于维护不利，因为重新添加一个类，专门处理添加解档归档`model`类：

```js
#import "YYModelSet.h"
#import <objc/runtime.h>
@implementation YYModelSet
+ (YYModelSet *)getClasses{
    return (YYModelSet *)[NSSet setWithArray:@[objc_getClass("ContactsModel")]];
}
```
`YYDiskCache`的改为：

```js
object = [NSKeyedUnarchiver unarchivedObjectOfClasses:[YYModelSet getClasses] fromData:item.value error:&error];
```
这样，后面如果新增需要解档，归档的类，只需要修改自己新增`YYModelSet`类方法即可，不动原来`YYCache`的代码。
