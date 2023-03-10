# 启动优化clang插桩(三)

## 一、获取符号
先把获取符号的代码写在`touchBegan`里面：

```js
-(void)touchesBegan:(NSSet<UITouch *> *)touches withEvent:(UIEvent *)event {
    //    NSLog(@"%s",__func__);
//    因为不知道有多少个，所有用while循环
    while(YES){
        //        将node取出来
        SYNode *node = OSAtomicDequeue(&symbolList, offsetof(SYNode, next));
        //        取到node为空退出当前循环
        if(node == NULL){
            break;
        }
//        打印拿到符号的信息
        Dl_info info;
        dladdr(node->pc,&info);
        printf("%s\n",info.dli_sname);
    }
} 
```
点击运行，会打印出一堆的`touchesBegan`。

```js
-[ViewController touchesBegan:withEvent:]
-[ViewController touchesBegan:withEvent:]
-[ViewController touchesBegan:withEvent:]
-[ViewController touchesBegan:withEvent:]
-[ViewController touchesBegan:withEvent:]
-[ViewController touchesBegan:withEvent:]
-[ViewController touchesBegan:withEvent:]
-[ViewController touchesBegan:withEvent:]
-[ViewController touchesBegan:withEvent:]
…
```
回到`Build setting`，将原来标记那里添加一个参数`func`：

<img width="691" alt="Pasted Graphic" src="https://user-images.githubusercontent.com/126937296/224032878-c2cfdab8-32e7-4ea2-854e-4b445f014a43.png">

再次运行，点击屏幕打印：

```js
-[ViewController touchesBegan:withEvent:]
-[SceneDelegate sceneDidBecomeActive:]
-[SceneDelegate sceneWillEnterForeground:]
-[ViewController viewDidLoad]
-[SceneDelegate window]
-[SceneDelegate scene:willConnectToSession:options:]
-[SceneDelegate window]
-[SceneDelegate setWindow:]
-[SceneDelegate window]
-[AppDelegate application:didFinishLaunchingWithOptions:]
main
```
这样就拿到了所有的符号。

## 二、处理符号
>因为队列是先进后出，所以我们需要做一个取反的操作，而且还有一些是重复的符号，我们需要去掉，处理完这些步骤之后的这些符号就是程序启动时候的顺序。

先给函数添加下划线：

```js
-(void)touchesBegan:(NSSet<UITouch *> *)touches withEvent:(UIEvent *)event {
    //    NSLog(@"%s",__func__);
    //    初始化一个数组来装载有顺序的数据
    NSMutableArray *symbolNames = [NSMutableArray array];
    
    //因为不知道有多少个，所有用while循环
    while(YES){
        //将node取出来
        SYNode *node = OSAtomicDequeue(&symbolList, offsetof(SYNode, next));
        //        取到node为空退出当前循环
        if(node == NULL){
            break;
        }
        //打印拿到符号的信息
        Dl_info info;
        dladdr(node->pc,&info);
        printf("%s\n",info.dli_sname);
        //转为OC字符串
        NSString *name = @(info.dli_sname );
        //判断是否是方法
        BOOL isMethod = [name hasPrefix:@"+["] ||
        [name hasPrefix:@"-["];
        //拿到处理后的符号
        NSString * symbolName = isMethod? name : [@“_” stringByAppendingString:name];
//        添加进数组
        [symbolNames addObject:symbolName];
    }
    NSLog(@"%@",symbolNames);
}
```
运行打印，得到：

```js
(
    "-[ViewController touchesBegan:withEvent:]",
    "-[SceneDelegate sceneDidBecomeActive:]",
    "-[SceneDelegate sceneWillEnterForeground:]",
    "-[SceneDelegate window]",
    "-[SceneDelegate scene:willConnectToSession:options:]",
    "-[SceneDelegate window]",
    "-[SceneDelegate setWindow:]",
    "-[SceneDelegate window]",
    "-[AppDelegate application:didFinishLaunchingWithOptions:]",
    "_main"
)
```

这样`main`函数就加上了下划线“_”
## 三、符号逆序

直接反向遍历：

```js
NSEnumerator *em = [symbolNames reverseObjectEnumerator];
NSLog(@"%@",em.allObjects);
```
运行打印，得到：
```js
 (
    "_main",
    "-[AppDelegate application:didFinishLaunchingWithOptions:]",
    "-[SceneDelegate window]",
    "-[SceneDelegate setWindow:]",
    "-[SceneDelegate window]",
    "-[SceneDelegate scene:willConnectToSession:options:]",
    "-[SceneDelegate window]",
    "-[ViewController viewDidLoad]",
    "-[SceneDelegate sceneWillEnterForeground:]",
    "-[SceneDelegate sceneDidBecomeActive:]",
    "-[ViewController touchesBegan:withEvent:]"
)
```
这就得到我们想要的顺序。
## 四、符号去重

```js
//   初始化去重后的数组
NSMutableArray *funcs = [NSMutableArray arrayWithCapacity:symbolNames.count];
//    定义表示
NSString *name;
//    判断是否数组里已经存在，不存在则添加
while (name = [em nextObject]) {
if(![funcs containsObject:name]){
[funcs addObject:name];
    }
}
NSLog(@"%@",funcs);
```
运行打印，得到：

```js
(
    "_main",
    "-[AppDelegate application:didFinishLaunchingWithOptions:]",
    "-[SceneDelegate window]",
    "-[SceneDelegate setWindow:]",
    "-[SceneDelegate scene:willConnectToSession:options:]",
    "-[ViewController viewDidLoad]",
    "-[SceneDelegate sceneWillEnterForeground:]",
    "-[SceneDelegate sceneDidBecomeActive:]",
    "-[ViewController touchesBegan:withEvent:]"
)
```
可以发现里面已经没有了重复的符号
## 五、生成`.order`文件

```js
//    拼接成一个字符串
    NSString *funcsStr = [funcs componentsJoinedByString:@"\n"];
//    文件路径
    NSString *filePath = [NSTemporaryDirectory() stringByAppendingString:@"TraceDemo.order"];
//    文件的内容
    NSData *file = [funcsStr dataUsingEncoding:NSUTF8StringEncoding];
//    写入文件
    [[NSFileManager defaultManager] createFileAtPath:filePath contents:file attributes:nil];
//    打印路径
    NSLog(@"%@",NSHomeDirectory());
```
运行打印：

```js
TraceDemo[31577:752540] /Users/xxxx/Library/Developer/CoreSimulator/Devices/876D0DEB-7AC9-4B67-A877-DB2BC4B5BD10/data/Containers/Data/Application/702BBFFB-D619-4B19-814C-0C9CXXXXX
Tmp文件下可以看到一个.order文件
```
<img width="137" alt="Pasted Graphic 3" src="https://user-images.githubusercontent.com/126937296/224033029-199ef4f6-7549-499a-9470-7fdf5a5563d1.png">

打开文件：

<img width="469" alt="Pasted Graphic 4" src="https://user-images.githubusercontent.com/126937296/224033775-361989a9-ded5-43e7-95bf-65725f2681fd.png">

写入的内容就是我们想要的内容，这样就可以把`.order`文件复制进项目里。

<img width="401" alt="Pasted Graphic 2" src="https://user-images.githubusercontent.com/126937296/224033094-d44b4246-3425-4a6d-815f-d405f5d5867a.png">

Order File添加文件位置：

<img width="603" alt="Pasted Graphic 1" src="https://user-images.githubusercontent.com/126937296/224033165-58e4132c-ca16-45d6-b179-f1cd6ee84ef5.png">

`Link Map File`打开：

<img width="746" alt="1__#$!@%!#__Pasted Graphic 3" src="https://user-images.githubusercontent.com/126937296/224033217-e1adcf27-7560-4928-88ed-178daa3fbdc2.png">

运行，然后找到这个`LinkMap`文件：

<img width="141" alt="1__#$!@%!#__Pasted Graphic 4" src="https://user-images.githubusercontent.com/126937296/224033312-367346a5-439e-4da1-a9d4-ed57dbd83123.png">

打开和`.order`文件对比：

<img width="821" alt="Pasted Graphic 5" src="https://user-images.githubusercontent.com/126937296/224033348-eb42bad3-27dc-4ba7-be09-67f19f9b3bb3.png">

发现完全一致。

>重排之后减少多少时间，就需要用`Instruments`工具的`System Trace`去做具体对比。
## 六、使用swift情况
如果项目使用`swift`的话，跟重排和使用`OC`类似。创建一个`swift`文件：

```js
import Foundation
class SwiftPage: NSObject{
@objc class public func swiftFunc(){
print("我是swift")
    }
}
```
导入头文件：

```js
#import "TraceDemo-Swift.h"
```
添加方法：

```js
- (void)viewDidLoad {
    [super viewDidLoad];
    [SwiftPage swiftFunc];
}
```
运行：

```js
我是swift
```
点击屏幕打印：

```js
(
    "_main",
    "-[AppDelegate application:didFinishLaunchingWithOptions:]",
    "-[SceneDelegate window]",
    "-[SceneDelegate setWindow:]",
    "-[SceneDelegate scene:willConnectToSession:options:]",
    "-[ViewController viewDidLoad]",
    "-[SceneDelegate sceneWillEnterForeground:]",
    "-[SceneDelegate sceneDidBecomeActive:]",
    "-[ViewController touchesBegan:withEvent:]"
)
```
发现并没有打印`swift`方法，因为`swift`并不是`clang`编译的，`clang`插桩只能编译`C`、`C++`和`OC`，这里就需要用在`Other Swift Flags`添加两个标记：`-sanitize-coverage=func`、`-sanitize=undefined`。

<img width="681" alt="1__#$!@%!#__Pasted Graphic" src="https://user-images.githubusercontent.com/126937296/224033425-c1165631-35fc-4503-8a8b-ee36755bbb91.png">

再次运行：


```js
(
    "_main",
    "-[AppDelegate application:didFinishLaunchingWithOptions:]",
    "-[SceneDelegate window]",
    "-[SceneDelegate setWindow:]",
    "-[SceneDelegate scene:willConnectToSession:options:]",
    "-[ViewController viewDidLoad]",
    "_$s9TraceDemo9SwiftPageC9swiftFuncyyFZTo",
    "_$s9TraceDemo9SwiftPageC9swiftFuncyyFZ",
    "_$ss27_finalizeUninitializedArrayySayxGABnlF",
    "_$sSa12_endMutationyyF",
    "_$ss5print_9separator10terminatoryypd_S2StFfA0_",
    "_$ss5print_9separator10terminatoryypd_S2StFfA1_",
    "-[SceneDelegate sceneWillEnterForeground:]",
    "-[SceneDelegate sceneDidBecomeActive:]",
    "-[ViewController touchesBegan:withEvent:]"
)
```
可以看到`swift`方法，因为`swift`方法自带混淆，这里`swift`也捕获到了，这里就完成了`OC`和`swift`的二进制重排。在项目需要上线的时候，删除一开始的标记`-fsanitize-coverage=func,trace-pc-guard`和其他测试代码。








