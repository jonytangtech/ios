# YYCache(二)

## 代码测试：
初始化`YYCache`实例：

``` java
#import <YYCache/YYCache.h>
#import "ViewController.h"
@interface ViewController ()
@property (nonatomic, strong) YYCache *contactsCache;
@end
@implementation ViewController
- (void)viewDidLoad {
    [super viewDidLoad];
    self.contactsCache = [YYCache cacheWithName:@"Contacts”];
}
```
添加一个通讯录模型：
```js
@interface ContactsModel : NSObject
@property (nonatomic, strong) NSString *name;
@property (nonatomic, strong) NSString *phoneNumber;
@end
```
添加`10`条数据：

```js
for (int i = 0; i < 10; i ++) {
     ContactsModel *model = [ContactsModel new];
     model.name = [NSString stringWithFormat:@"张%d",i];
     model.phoneNumber = [NSString stringWithFormat:@"1588889999%d",i];
     [self.contactsCache setObject:model forKey:
     [NSString stringWithFormat:@"kContacts_%d",i]];
 }
```
提示`Sending 'ContactsModel *__strong' to parameter of incompatible type 'id<NSCoding> _Nullable`：
给`ContactsModel`添加`<NSCoding>`协议：

```js
@interface ContactsModel : NSObject<NSCoding>

//通讯类内部的两个属性变量分别转码
- (void)encodeWithCoder:(nonnull NSCoder *)coder {
    [coder encodeObject:_name forKey:@"name"];
    [coder encodeObject:_phoneNumber forKey:@"phoneNumber"];
}
//分别把两个属性变量根据关键字进行逆转码，最后返回一个Contacts类的对象
- (nullable instancetype)initWithCoder:(nonnull NSCoder *)coder {
    if (self = [super init]) {
        if (coder) {
            _name = [coder decodeObjectOfClass:[NSString class] 
            forKey:@"name"];
            _phoneNumber = [coder decodeObjectOfClass:[NSString class] 
            forKey:@"phoneNumber"];
        }
    }
    return self;
}
```
提示消失，用循环把值取出来：


```js
for (int i = 0; i < 10; i++) {
    ContactsModel *model = (ContactsModel *)[self.contactsCache
    objectForKey:[NSString stringWithFormat:@"kContacts_%d",i]];
    NSLog(@"name = %@ phoneNumber = %@",model.name, model.phoneNumber);
}
```
运行打印：

```js
YYCacheDemo[3304:73220] name = 张0 phoneNumber = 15888899990
YYCacheDemo[3304:73220] name = 张1 phoneNumber = 15888899991
YYCacheDemo[3304:73220] name = 张2 phoneNumber = 15888899992
YYCacheDemo[3304:73220] name = 张3 phoneNumber = 15888899993
YYCacheDemo[3304:73220] name = 张4 phoneNumber = 15888899994
YYCacheDemo[3304:73220] name = 张5 phoneNumber = 15888899995
YYCacheDemo[3304:73220] name = 张6 phoneNumber = 15888899996
YYCacheDemo[3304:73220] name = 张7 phoneNumber = 15888899997
YYCacheDemo[3304:73220] name = 张8 phoneNumber = 15888899998
YYCacheDemo[3304:73220] name = 张9 phoneNumber = 15888899999
```
>打印self.contactsCache.memoryCache和self.contactsCache.diskCache，发现数据一样

## 主要功能：
一、添加限制</br>
二、数据修剪

**YYMemoryCache**：
![Pasted Graphic.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/3a7faf016e9341ca8484b8b7dcce0c0d~tplv-k3u1fbpfcp-watermark.image?)
**YYDiskCach**：

![Pasted Graphic 1.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/a967fec96057468c9b5f685fd4a5dfef~tplv-k3u1fbpfcp-watermark.image?)

测试给内存添加数量限制：

```js
self.contactsCache.memoryCache.countLimit = 5;
```

```js
YYCacheDemo[4272:102918] name = (null) phoneNumber = (null)
YYCacheDemo[4272:102918] name = (null) phoneNumber = (null)
YYCacheDemo[4272:102918] name = (null) phoneNumber = (null)
YYCacheDemo[4272:102918] name = (null) phoneNumber = (null)
YYCacheDemo[4272:102918] name = (null) phoneNumber = (null)
YYCacheDemo[4272:102918] name = 张5 phoneNumber = 15888899995
YYCacheDemo[4272:102918] name = 张6 phoneNumber = 15888899996
YYCacheDemo[4272:102918] name = 张7 phoneNumber = 15888899997
YYCacheDemo[4272:102918] name = 张8 phoneNumber = 15888899998
YYCacheDemo[4272:102918] name = 张9 phoneNumber = 15888899999
```
>发现前面的`5`个数据都被移除了

修剪最大个数为`8`个：
```js
[self.contactsCache.memoryCache trimToCount:8];
```
运行：
```js
YYCacheDemo[4461:108978] name = (null) phoneNumber = (null)
YYCacheDemo[4461:108978] name = (null) phoneNumber = (null)
YYCacheDemo[4461:108978] name = 张2 phoneNumber = 15888899992
YYCacheDemo[4461:108978] name = 张3 phoneNumber = 15888899993
YYCacheDemo[4461:108978] name = 张4 phoneNumber = 15888899994
YYCacheDemo[4461:108978] name = 张5 phoneNumber = 15888899995
YYCacheDemo[4461:108978] name = 张6 phoneNumber = 15888899996
YYCacheDemo[4461:108978] name = 张7 phoneNumber = 15888899997
YYCacheDemo[4461:108978] name = 张8 phoneNumber = 15888899998
YYCacheDemo[4461:108978] name = 张9 phoneNumber = 15888899999
```
清空所有缓存：

```js
 [self.contactsCache removeAllObjects];
```
内存缓存：
```js
YYCacheDemo[6819:162854] memory name = (null) phoneNumber = (null)
YYCacheDemo[6819:162854] memory name = (null) phoneNumber = (null)
YYCacheDemo[6819:162854] memory name = (null) phoneNumber = (null)
YYCacheDemo[6819:162854] memory name = (null) phoneNumber = (null)
YYCacheDemo[6819:162854] memory name = (null) phoneNumber = (null)
YYCacheDemo[6819:162854] memory name = (null) phoneNumber = (null)
YYCacheDemo[6819:162854] memory name = (null) phoneNumber = (null)
YYCacheDemo[6819:162854] memory name = (null) phoneNumber = (null)
YYCacheDemo[6819:162854] memory name = (null) phoneNumber = (null)
YYCacheDemo[6819:162854] memory name = (null) phoneNumber = (null)
```
磁盘缓存：

```js
YYCacheDemo[6819:162854] disk name = (null) phoneNumber = (null)
YYCacheDemo[6819:162854] disk name = (null) phoneNumber = (null)
YYCacheDemo[6819:162854] disk name = (null) phoneNumber = (null)
YYCacheDemo[6819:162854] disk name = (null) phoneNumber = (null)
YYCacheDemo[6819:162854] disk name = (null) phoneNumber = (null)
YYCacheDemo[6819:162854] disk name = (null) phoneNumber = (null)
YYCacheDemo[6819:162854] disk name = (null) phoneNumber = (null)
YYCacheDemo[6819:162854] disk name = (null) phoneNumber = (null)
YYCacheDemo[6819:162854] disk name = (null) phoneNumber = (null)
YYCacheDemo[6819:162854] disk name = (null) phoneNumber = (null)
```
## 源码：
`YYMemoryCache`的初始化：

```js
- (instancetype)init {
    self = super.init;
    pthread_mutex_init(&_lock, NULL);
    _lru = [_YYLinkedMap new];
    _queue = dispatch_queue_create("com.ibireme.cache.memory", DISPATCH_QUEUE_SERIAL);
    _countLimit = NSUIntegerMax;
    _costLimit = NSUIntegerMax;
    _ageLimit = DBL_MAX;
    _autoTrimInterval = 5.0;
    _shouldRemoveAllObjectsOnMemoryWarning = YES;
    _shouldRemoveAllObjectsWhenEnteringBackground = YES;
    [[NSNotificationCenter defaultCenter] addObserver:self selector:@selector(_appDidReceiveMemoryWarningNotification) name:UIApplicationDidReceiveMemoryWarningNotification object:nil];
    [[NSNotificationCenter defaultCenter] addObserver:self selector:@selector(_appDidEnterBackgroundNotification) name:UIApplicationDidEnterBackgroundNotification object:nil];
    [self _trimRecursively];
    return self;
}
```
一个`_YYLinkedMap`的实例，查看`_YYLinkedMap`：

```js
@interface _YYLinkedMap : NSObject {
    @package
    CFMutableDictionaryRef _dic; // do not set object directly
    NSUInteger _totalCost;
    NSUInteger _totalCount;
    _YYLinkedMapNode *_head; // MRU, do not change it directly
    _YYLinkedMapNode *_tail; // LRU, do not change it directly
    BOOL _releaseOnMainThread;
    BOOL _releaseAsynchronously;
}
```
发现`_YYLinkedMap`是一个双向链表，有两个`_YYLinkedMapNode`类型节点，还有一个`CFMutableDictionaryRef`的字典`_dic`，`_dic`是真正存放数据的地方。

递归修剪：

```js
- (void)_trimRecursively {
    __weak typeof(self) _self = self;
    dispatch_after(dispatch_time(DISPATCH_TIME_NOW, (int64_t)(_autoTrimInterval * NSEC_PER_SEC)), dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_LOW, 0), ^{
        __strong typeof(_self) self = _self;
        if (!self) return;
        [self _trimInBackground];
        [self _trimRecursively];
    });
}
- (void)_trimInBackground {
    dispatch_async(_queue, ^{
        [self _trimToCost:self->_costLimit];
        [self _trimToCount:self->_countLimit];
        [self _trimToAge:self->_ageLimit];
    });
}
```
>每隔`_autoTrimInterval`秒就自动调用修整内存数据，`_autoTrimInterval`默认是`5`秒。

添加数据：

```js
- (void)setObject:(id)object forKey:(id)key withCost:(NSUInteger)cost {
    ….
    _YYLinkedMapNode *node = CFDictionaryGetValue(_lru->_dic, (__bridge const void *)(key));
    NSTimeInterval now = CACurrentMediaTime();
    if (node) {
        _lru->_totalCost -= node->_cost;
        _lru->_totalCost += cost;
        node->_cost = cost;
        node->_time = now;
        node->_value = object;
        [_lru bringNodeToHead:node];
    } else {
        node = [_YYLinkedMapNode new];
        node->_cost = cost;
        node->_time = now;
        node->_key = key;
        node->_value = object;
        [_lru insertNodeAtHead:node];
    }
   …
    pthread_mutex_unlock(&_lock);
}
```
由于`_lru = [_YYLinkedMap new];` ，可以看到就是操作`_YYLinkedMap`双链表，使用的是`pthread_mutex`锁。
