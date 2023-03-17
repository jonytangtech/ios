# YYCache（一）
## 前言：
我们一般对网络请求下来的比较大的数据做缓存，如果没有网络，或者是请求到的标识和之前的标识一致，表示数据没有变动，则可以使用缓存加载，不需要重新网络拉取数据，这里一般使用**YYCache**。

![Pasted Graphic 1.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/bfdcadb7a70445bb88df9747d5dce2ef~tplv-k3u1fbpfcp-watermark.image?)

从git上把YYCache pod下来

![Pasted Graphic.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/6be80c1e224446a19623643a74265622~tplv-k3u1fbpfcp-watermark.image?)

>可以看到YYCache的文件结构还是相对简单，除了YYCache这个对外使用的接口文件，还有YYDiskCache这个磁盘缓存，YYKVStorage这个元数据键值存储，还有YYMemoryCache这个内存缓存。
## 一、YYCache：
>／** ' YYCache '是一个线程安全的键值缓存。 它使用' YYMemoryCache '将对象存储在一个小而快速的内存缓存中， 并使用' YYDiskCache '将对象持久化到一个大而慢的磁盘缓存中。 * /

### 属性列表:

```js
属性就三个：
@interface YYCache : NSObject
/** 缓存名字，只读*/
@property (copy, readonly) NSString *name;
/** 内存缓存.*/
@property (strong, readonly) YYMemoryCache *memoryCache;
/** 磁盘缓存.*/
@property (strong, readonly) YYDiskCache *diskCache;
```
### 方法列表:
**初始化：**


```js
初始化：
/**
用指定的名称创建一个新实例。
 */
- (nullable instancetype)initWithName:(NSString *)name;
/**
用指定的路径创建一个新实例。
 */
- (nullable instancetype)initWithPath:(NSString *)path NS_DESIGNATED_INITIALIZER;

/**
便捷初始化，用指定的名称创建一个新实例。
 */
+ (nullable instancetype)cacheWithName:(NSString *)name;
/**
便捷初始化，用指定的路径创建一个新实例
 */
+ (nullable instancetype)cacheWithPath:(NSString *)path;
```
**访问方法**


```js

/**
返回一个布尔值，一个指定的键是否在缓存中，可能会阻塞线程直到文件读取完成。
 */
- (BOOL)containsObjectForKey:(NSString *)key;

/**
返回指定键相对应的值。
 */
- (nullable id<NSCoding>)objectForKey:(NSString *)key;

/**
设置缓存中指定键的值。
 */
- (void)setObject:(nullable id<NSCoding>)object forKey:(NSString *)key;

/**
删除缓存中指定键的值，如果为nil，则此方法无效。
 */
- (void)removeObjectForKey:(NSString *)key;
/**
清空缓存。
 */
- (void)removeAllObjects;
/**
用block清空缓存。可以通过参数得到进度。
 */
- (void)removeAllObjectsWithProgressBlock:(nullable void(^)(int removedCount, int totalCount))progress
                                 endBlock:(nullable void(^)(BOOL error))end;

```
**看.m文件**
看`.m`文件的初始化方法`initWithName`：

```js
- (instancetype)initWithName:(NSString *)name {
    if (name.length == 0) return nil;
    NSString *cacheFolder = [NSSearchPathForDirectoriesInDomains(NSCachesDirectory, NSUserDomainMask, YES) firstObject];
    NSString *path = [cacheFolder stringByAppendingPathComponent:name];
    return [self initWithPath:path];
}
```
发现`initWithName`会调用另一个初始化方法`initWithPath`和`YYMemoryCache`的初始化方法：

```js
- (instancetype)initWithPath:(NSString *)path {
    return [self initWithPath:path inlineThreshold:1024 * 20]; // 20KB
}
查看initWithPath:(NSString *)path
             inlineThreshold:(NSUInteger)threshold发现：

- (instancetype)initWithPath:(NSString *)path
             inlineThreshold:(NSUInteger)threshold {
	…
    
    YYKVStorageType type;
    if (threshold == 0) {
        type = YYKVStorageTypeFile;
    } else if (threshold == NSUIntegerMax) {
        type = YYKVStorageTypeSQLite;
    } else {
        type = YYKVStorageTypeMixed;
    }
	_inlineThreshold = threshold;
	…
}
```
>在`YYDiskCache`中可以看到内联阈值是`20KB`，`_inlineThreshold`被初始化为`20KB`。

点进YYKVStorageType看，发现有三种存储类型，一文件，二数据库，三自选：

```js
typedef NS_ENUM(NSUInteger, YYKVStorageType) {
    /// The `value` is stored as a file in file system.
    YYKVStorageTypeFile = 0,
    
    /// The `value` is stored in sqlite with blob type.
    YYKVStorageTypeSQLite = 1,
    
    /// The `value` is stored in file system or sqlite based on your choice.
    YYKVStorageTypeMixed = 2,
};
```
再看赋值：

```js
- (void)setObject:(id<NSCoding>)object forKey:(NSString *)key {
	…
    NSData *value = nil;

    if (!value) return;
    NSString *filename = nil;
    if (_kv.type != YYKVStorageTypeSQLite) {
        if (value.length > _inlineThreshold) {
            filename = [self _filenameForKey:key];
        }
    }
    Lock();
    [_kv saveItemWithKey:key value:value filename:filename extendedData:extendedData];
    Unlock();
}
```
可以看到`value.length` >` _inlineThreshold`，`filename`会被赋值，点击`saveItemWithKey`方法：

```js
- (BOOL)saveItemWithKey:(NSString *)key value:(NSData *)value filename:(NSString *)filename extendedData:(NSData *)extendedData {

	…
    if (filename.length) {
        if (![self _fileWriteWithName:filename data:value]) {
            return NO;
        }
        if (![self _dbSaveWithKey:key value:value fileName:filename extendedData:extendedData]) {
            [self _fileDeleteWithName:filename];
            return NO;
        }
        return YES;
    } else {
        if (_type != YYKVStorageTypeSQLite) {
            NSString *filename = [self _dbGetFilenameWithKey:key];
            if (filename) {
                [self _fileDeleteWithName:filename];
            }
        }
        return [self _dbSaveWithKey:key value:value fileName:nil extendedData:extendedData];
    }
}
```
可以看到`filename.length`如果存在值，就直接写成文件，如果不大于`20KB`，使用`sqlite`写入。
>**作者**：NSURLCache、FBDiskCache 都是基于 SQLite 数据库的。基于数据库的缓存可以很好的支持元数据、扩展方便、数据统计速度快，也很容易实现 LRU 或其他淘汰算法，唯一不确定的就是数据库读写的性能，为此我评测了一下 SQLite 在真机上的表现。iPhone 6 64G 下，SQLite 写入性能比直接写文件要高，但读取性能取决于数据大小：当单条数据小于 20K 时，数据越小 SQLite 读取性能越高；单条数据大于 20K 时，直接写为文件速度会更快一些。

## 二、YYDiskCache：
（那些相同的参数和方法就不重新写）
### 属性列表：

```js

/**
如果对象的数据大小(以字节为单位)大于此值，则对象将
存储为文件，否则对象将存储在sqlite中。
0表示所有对象将存储为分开的文件，NSUIntegerMax表示所有对象
对象将存储在sqlite中。
默认值为20480 (20KB)。
 */
@property (readonly) NSUInteger inlineThreshold;
/**
如果这个块不是nil，那么这个块将被用来存档对象
NSKeyedArchiver。您可以使用此块来支持不支持的对象
遵守' NSCoding '协议。
默认值为空。
 */
@property (nullable, copy) NSData *(^customArchiveBlock)(id object);
/**
如果这个块不是nil，那么这个块将被用来解存档对象
NSKeyedUnarchiver。您可以使用此块来支持不支持的对象
遵守' NSCoding '协议。
默认值为空。
 */
@property (nullable, copy) id (^customUnarchiveBlock)(NSData *data);
/**
当需要将对象保存为文件时，将调用此块来生成
指定键的文件名。如果块为空，缓存使用md5(key)作为默认文件名。默认值为空。
 */
@property (nullable, copy) NSString *(^customFileNameBlock)(NSString *key);
#pragma mark - Limit
/**
缓存应容纳的最大对象数。
默认值为NSUIntegerMax，即不限制。
这不是一个严格的限制-如果缓存超过限制，缓存中的一些对象
缓存可以稍后在后台队列中被清除。
 */
@property NSUInteger countLimit;
/**
在开始清除对象之前，缓存可以保留的最大总开销。
默认值为NSUIntegerMax，即不限制。
这不是一个严格的限制-如果缓存超过限制，缓存中的一些对象
缓存可以稍后在后台队列中被清除。
 */
@property NSUInteger costLimit;
/**
缓存中对象的最大过期时间。

>值为DBL_MAX，即无限制。
这不是一个严格的限制-如果一个对象超过了限制，对象可以
稍后在后台队列中被驱逐。
 */
@property NSTimeInterval ageLimit;
/**
缓存应保留的最小空闲磁盘空间(以字节为单位)。

>默认值为0，表示不限制。
如果可用磁盘空间低于此值，缓存将删除对象
释放一些磁盘空间。这不是一个严格的限制——如果空闲磁盘空间没有了
超过限制，对象可能稍后在后台队列中被清除。
 */
@property NSUInteger freeDiskSpaceLimit;
/**
自动修整检查时间间隔以秒为单位。默认值是60(1分钟)。
缓存保存一个内部计时器来检查缓存是否达到
它的极限，一旦达到极限，它就开始驱逐物体。
 */
@property NSTimeInterval autoTrimInterval;
/**
设置“YES”为调试启用错误日志。
 */
@property BOOL errorLogsEnabled;
```
### 方法列表：

```js
/**
指定的初始化式。
threshold数据存储内联阈值，单位为字节。如果对象的数据
Size(以字节为单位)大于此值，则对象将存储为
文件，否则对象将存储在sqlite中。0表示所有对象都会
NSUIntegerMax表示所有对象都将被存储
sqlite。如果您不知道对象的大小，20480是一个不错的选择。
在第一次初始化后，您不应该更改指定路径的这个值。
如果指定路径的缓存实例在内存中已经存在，
该方法将直接返回该实例，而不是创建一个新实例。
 */
- (nullable instancetype)initWithPath:(NSString *)path inlineThreshold:(NSUInteger)threshold NS_DESIGNATED_INITIALIZER;
/**
返回此缓存中的对象数量。
 */
- (NSInteger)totalCount;
/**
以字节为单位的对象开销总数。
 */
- (NSInteger)totalCost;
#pragma mark - 修剪
/**
使用LRU从缓存中移除对象，直到' totalCount '低于指定值。
 */
- (void)trimToCount:(NSUInteger)count;
/**
使用LRU从缓存中移除对象，直到' totalCost '低于指定值，完成后调用回调。
 */
- (void)trimToCost:(NSUInteger)cost;
/**
使用LRU从缓存中删除对象，直到所有过期对象被指定值删除为止。
 */
- (void)trimToAge:(NSTimeInterval)age;
/**
从对象中获取扩展数据。
 */
+ (nullable NSData *)getExtendedDataFromObject:(id)object;
/**
将扩展数据设置为一个对象。
 */
+ (void)setExtendedData:(nullable NSData *)extendedData toObject:(id)object;

@end
```
## 三、YYMemoryCache：
### 属性列表：

```js
/** 存中的对象数量(只读)*/
@property (readonly) NSUInteger totalCount;

/** 缓存中对象的总开销(只读)*/
@property (readonly) NSUInteger totalCost;

/**
自动修整检查时间间隔以秒为单位。默认是5.0。
 */
@property NSTimeInterval autoTrimInterval;
/**
如果' YES '，当应用程序收到内存警告时，缓存将删除所有对象。
 */
@property BOOL shouldRemoveAllObjectsOnMemoryWarning;
/**
如果是，当应用程序进入后台时，缓存将删除所有对象。
 */
@property BOOL shouldRemoveAllObjectsWhenEnteringBackground;
/**
当应用程序收到内存警告时要执行的块。
 */
@property (nullable, copy) void(^didReceiveMemoryWarningBlock)(YYMemoryCache *cache);
/**
当应用程序进入后台时执行的一个块。
 */
@property (nullable, copy) void(^didEnterBackgroundBlock)(YYMemoryCache *cache);
/**
如果' YES '，键值对将在主线程上释放，否则在后台线程上释放。默认为NO。。
 */
@property BOOL releaseOnMainThread;
/**
如果' YES '，键值对将在主线程上释放，否则在后台线程上释放。默认为NO。
 */
@property BOOL releaseAsynchronously;
```
## 总结：
### 对比一下NSCache：

```js
@interface NSCache <KeyType, ObjectType> : NSObject
@property (copy) NSString *name;
@property (nullable, assign) id<NSCacheDelegate> delegate;
- (nullable ObjectType)objectForKey:(KeyType)key;
- (void)setObject:(ObjectType)obj forKey:(KeyType)key; // 0 cost
- (void)setObject:(ObjectType)obj forKey:(KeyType)key cost:(NSUInteger)g;
- (void)removeObjectForKey:(KeyType)key;
- (void)removeAllObjects;
@property NSUInteger totalCostLimit;	// limits are imprecise/not strict
@property NSUInteger countLimit;	// limits are imprecise/not strict
@property BOOL evictsObjectsWithDiscardedContent;
@end
@protocol NSCacheDelegate <NSObject>
@optional
- (void)cache:(NSCache *)cache willEvictObject:(id)obj;
@end
```
会发现`NSCache`相对简单很多，`YYCache`对内存和磁盘缓存给了很多个接口去准备控制缓存的数量和生命周期。

