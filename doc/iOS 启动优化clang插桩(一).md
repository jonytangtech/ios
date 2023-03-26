# 启动优化clang插桩（一）
## 一、了解Clang
首先到`Clang`地址：[Clang Documentation](https://clang.llvm.org/docs/SanitizerCoverage.html)

<img width="563" alt="Pasted Graphic" src="https://user-images.githubusercontent.com/126937296/223763011-8967fa51-142d-4b84-a22c-dc31d1be5b7b.png">

`PCs`指的是`CPU`的寄存器，用来存储将要执行的下一条指令的地址，`Tracing PCs`就是跟踪`CPU`将要执行的代码。
## 二、如何使用
网页下拉有个`Example`

<img width="615" alt="Pasted Graphic 1" src="https://user-images.githubusercontent.com/126937296/223763137-ff634161-e64b-4975-ae18-a0d5c715bbc2.png">

使用之前要在工程添加标记：

<img width="764" alt="Pasted Graphic 2" src="https://user-images.githubusercontent.com/126937296/223763200-8892f026-5e69-422f-99b1-ffe5689e9d53.png">

>编译器就会在每一行代码的边缘插入这一段函数：`__sanitizer_cov_trace_pc_guard(&guard_variable)`

打开实例`demo`，在`Build Settings` 搜索 `Other c Flag` 填入 `-fsanitize-coverage=trace-pc-guard`

<img width="715" alt="1__#$!@%!#__Pasted Graphic 1" src="https://user-images.githubusercontent.com/126937296/223940172-dcf22058-548c-48a0-9083-b220551d8cd3.png">


项目会报未定义符号的错：

<img width="263" alt="Pasted Graphic 7" src="https://user-images.githubusercontent.com/126937296/223763439-9c67af0c-47a0-4b31-9298-b0d61f285a8c.png">

这就需要去定义这两个符号，先把这两个函数复制过来：

<img width="669" alt="Pasted Graphic 5" src="https://user-images.githubusercontent.com/126937296/223763486-f0f909bc-d39d-4011-b502-cd3f2a3a1eea.png">

先把代码复制进`ViewController`

```
extern "C" void __sanitizer_cov_trace_pc_guard_init(uint32_t *start,
                                                    uint32_t *stop) {
  static uint64_t N;  // Counter for the guards.
  if (start == stop || *start) return;  // Initialize only once.
  printf("INIT: %p %p\n", start, stop);
  for (uint32_t *x = start; x < stop; x++)
    *x = ++N;  // Guards should start from 1.
}
// This callback is inserted by the compiler on every edge in the
// control flow (some optimizations apply).
// Typically, the compiler will emit the code like this:
//    if(*guard)
//      __sanitizer_cov_trace_pc_guard(guard);
// But for large functions it will emit a simple call:
//    __sanitizer_cov_trace_pc_guard(guard);
extern "C" void __sanitizer_cov_trace_pc_guard(uint32_t *guard) {
  if (!*guard) return;  // Duplicate the guard check.
  // If you set *guard to 0 this code will not be called again for this edge.
  // Now you can get the PC and do whatever you want:
  //   store it somewhere or symbolize it and print right away.
  // The values of `*guard` are as you set them in
  // __sanitizer_cov_trace_pc_guard_init and so you can make them consecutive
  // and use them to dereference an array or a bit vector.
  void *PC = __builtin_return_address(0);
  char PcDescr[1024];
  // This function is a part of the sanitizer run-time.
  // To use it, link with AddressSanitizer or other sanitizer.
  __sanitizer_symbolize_pc(PC, "%p %F %L", PcDescr, sizeof(PcDescr));
  printf("guard: %p %x PC %s\n", guard, *guard, PcDescr);
}
```
把头文件也粘贴进来：

```
#include <stdint.h>
#include <stdio.h>
#include <sanitizer/coverage_interface.h>
```
>两个方法里面都有`extern “C”`，`extern “C”`的主要作用是为了能够正确实现`C++`去调用其他C语言的代码，加上`extern “C”`就会指示作用域内的代码按照C语言区编译，而不是`C++`，这个`extern “C”`在OC项目里没什么用，直接删除

此时还会包一个错误：

<img width="230" alt="Pasted Graphic 8" src="https://user-images.githubusercontent.com/126937296/223763562-f03dd76e-7746-4a6e-a0b0-82c26b6c87c9.png">

这个`__sanitizer_symbolize_pc(PC, "%p %F %L", PcDescr, sizeof(PcDescr));`函数没有什么作用，直接删除即可。

## 三、代码调试

`cmd + r`运行，此时终端会打印一些信息：

<img width="504" alt="Pasted Graphic 9" src="https://user-images.githubusercontent.com/126937296/223763607-72680a88-cee1-444e-a89f-aff567d22b5f.png">

删除两个函数里面的注释，先注释第二个的内容，然后运行

`INIT: 0x1025c5478 0x1025c54f0`

这是运行打印得到的地址，就是函数`(uint32_t *start, uint32_t *stop)`的`start`和`stop`两个指针的地址

`stop`存储的就是我们工程里面符号的个数

```swift
for (uint32_t *x = start; x < stop; x++)
        *x = ++N;
```
>看一下这个`for`循环，`start`会先复制给`*x`，`x++`就是内存平移，按照`uint32_t`的大小去平移，而`uint32_t`的定义是`typedef unsigned int uint32_t;` 是无符号整型，占`4`个字节，所以每次按`4`个字节平移。

`start`和`stop`里面存的是什么，打断点调试：

<img width="514" alt="Pasted Graphic 10" src="https://user-images.githubusercontent.com/126937296/223763645-c2c12c2f-79db-44f0-9c71-baa17a06c813.png">

先看`start`:

```swift
INIT: 0x1042a5278 0x1042a52e0
(lldb) x 0x1042a5278
0x1042a5278: 01 00 00 00 02 00 00 00 03 00 00 00 04 00 00 00  ................
0x1042a5288: 05 00 00 00 06 00 00 00 07 00 00 00 08 00 00 00  ................
(lldb)
```
由于`uint32_t`按`4`个字节来存储发现`start`就是 `0 1 2 3 4…`，再看`stop`，由于`stop`的已经是结束位置，读取的数据是在`start`和`stop`之间的数据，所以需要向前平移`4`个字节得到其真实数据。

```swift
(lldb) x (0x1042a52e0-4)
0x1042a52dc: 1a 00 00 00 00 00 00 00 00 00 00 00 fe f1 29 04  ..............).
0x1042a52ec: 01 00 00 00 00 00 00 00 00 00 00 00 90 40 2a 04  .............@*.
(lldb)
```
可以得到`1a` 就是`26`,也可以循环外面打印结果：

<img width="474" alt="Pasted Graphic 11" src="https://user-images.githubusercontent.com/126937296/223763710-6b3eacbe-cee6-4565-a1ba-0847a9f4a730.png">

可以得到:
```swift
TraceDemo[16814:301325] 26
```
也是`26`个符号。
## 四、测试验证方法
可以验证一下，添加一个函数：
```
void test(void) {
    NSLog(@"%s",__func__);
}
```
符号变成`27`：
```swift
TraceDemo[16911:304537] 27
```
再添加一个`block`：
```swift
void (^block) (void) = ^{
    NSLog(@"%s",__func__);
};
```
符号变成`28`：
```swift
TraceDemo[16933:305465] 28
```
添加一个数据类型属性:
```swift
@property (nonatomic ,assign) int age;
```
>由于系统自动生成getter、setter方法，符号变成30
```swift
TraceDemo[16975:306816] 30
```
添加一个对象属性:

```js
@property (nonatomic ,copy) NSString *str;
```
符号变成`33`：

```js
TraceDemo[17041:308780] 33
```
>对象属性由于ARC,系统自动除了生成getter、setter方法外还生成了cxx_destruct()析构函数

添加一个方法:

```js
- (void)test{
}
```
符号变成`34`：

```js
TraceDemo[17114:311256] 34
```
在其他类`AppDelegate`类中添加一个属性：

```js
@interface AppDelegate : UIResponder <UIApplicationDelegate>
@property (nonatomic, strong) NSString *name;
@end
```
符号变成`37`:

```js
TraceDemo[17266:316294] 37
```
符号变成`37`，

## 结论
这就说明了通过这个方法整个项目里的符号，它都能捕获到。




