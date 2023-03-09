# 启动优化clang插桩(二)
## 一、前言
上一篇的方法给到的是个数，但不是符号，个数并没有什么作用，甚至给了全部的符号也没什么用，因为二进制重排仅仅需要的是启动阶段所需要的符号，这就需要下面这个函数：
```js
void __sanitizer_cov_trace_pc_guard(uint32_t *guard) {
}
```
添加断点，点击箭头，可以看到绿色框中的函数调用栈：

<img width="411" alt="Pasted Graphic" src="https://user-images.githubusercontent.com/126937296/223900179-b6dc4493-2668-4753-9bee-8c7198724b46.png">

>函数调用栈跟之前讲到的`app名称.LinkMap-normal-arm64.txt`文件里面的数据格式一样。

把上面断点过掉，给`touchBegan`方法添加断点：：

```js
-(void)touchesBegan:(NSSet<UITouch *> *)touches withEvent:(UIEvent *)event {
///
}
```
点击上面的绿色箭头：

<img width="524" alt="Pasted Graphic 1" src="https://user-images.githubusercontent.com/126937296/223900215-1e9b9f68-19a4-4542-a677-d9418564ba98.png">

这里就出现了`touchesBegan`，那大概推出下面这个函数是系统每调用一个方法，都会调用这个`__sanitizer_cov_trace_pc_guard`函数：


```js
void __sanitizer_cov_trace_pc_guard(uint32_t *guard) {
}
```
## 二、调试验证
验证是否为真给下面两个函数添加打印方法：


```js
void test(void) {
    NSLog(@"%s",__func__);
}

void (^block) (void) = ^{
    NSLog(@"%s",__func__);
};
```
同时在`touchesBegan`方法里面为添加打印和调用两个函数：

```js
-(void)touchesBegan:(NSSet<UITouch *> *)touches withEvent:(UIEvent *)event {
    NSLog(@"%s",__func__);
    test();
    block();
}
```
这个追踪方法也添加打印：

```js
void __sanitizer_cov_trace_pc_guard(uint32_t *guard) {
    NSLog(@"%s",__func__);
}
```
`cmd + r`启动，先清空打印，点击屏幕：

```js
TraceDemo[20273:399962] __sanitizer_cov_trace_pc_guard
TraceDemo[20273:399962] -[ViewController touchesBegan:withEvent:]
TraceDemo[20273:399962] __sanitizer_cov_trace_pc_guard
TraceDemo[20273:399962] test
TraceDemo[20273:399962] __sanitizer_cov_trace_pc_guard
TraceDemo[20273:399962] block_block_invoke
```
可以看出先`__sanitizer_cov_trace_pc_guard` 再调用方法、函数、`block`，这说明不管是方法、函数、`block`它都会去回调这个函数，而且这个函数的调用是我们在调用这个函数之前，也就是这个函数拦截或者`hook`了所有的方法、函数包括`block`， 这就搞定了我们没有其他操作就能拦截到app启动时候调用了那些方法和函数

```js
void __sanitizer_cov_trace_pc_guard(uint32_t *guard) {
NSLog(@"%s",__func__);
}
```
我们在程序启动时候`__sanitizer_cov_trace_pc_guard`拦截到的方法函数：

<img width="297" alt="Pasted Graphic 3" src="https://user-images.githubusercontent.com/126937296/223900256-3f18f859-1690-4750-a3e2-170d673dbf32.png">

>把这些写入到`.order`文件里面这样二进制重排就搞定了

而`_sanitizer_cov_trace_pc_guard`，这个函数是如何做到这一点的？给`_sanitizer_cov_trace_pc_guard`添加断点，在Xcode的`Debug`选择`Debug WorkFlow`选择显示汇编，选择`main`：

<img width="283" alt="Pasted Graphic 4" src="https://user-images.githubusercontent.com/126937296/223900290-9c282a4b-21c3-4e75-aabd-262319999062.png">

可以看到在`main`之前，系统插入了`_sanitizer_cov_trace_pc_guard`这个符号

<img width="672" alt="Pasted Graphic 5" src="https://user-images.githubusercontent.com/126937296/223900319-d651f01a-1c29-4c8d-ae66-44e831b92c4e.png">

在`AppDelegate`页面也是插入了这个符号：

<img width="922" alt="Pasted Graphic 6" src="https://user-images.githubusercontent.com/126937296/223900794-67f29554-e2ee-491a-bfa3-77b98c34a81a.png">

在`SceneDelegate`同样如此：

<img width="948" alt="Pasted Graphic 7" src="https://user-images.githubusercontent.com/126937296/223900811-0068b754-13f2-4afb-a22c-cc0a1fa260b2.png">

也就是说，在编译器`clang`添加下面这个标记后，编译器会给函数方法前面都会调用`_sanitizer_cov_trace_pc_guard`这个函数：

<img width="614" alt="Pasted Graphic 8" src="https://user-images.githubusercontent.com/126937296/223900836-b2c12607-5ecd-4b00-8536-a370003a7c50.png">

这样，我们确实在打断点看到启动阶段的所有符号。

## 三、如何把符号都打印出来

先打开注释的代码：

```js
void __sanitizer_cov_trace_pc_guard(uint32_t *guard) {
    NSLog(@"%s",__func__);
    
    if (!*guard) return;
    void *PC = __builtin_return_address(0);
//    char PcDescr[1024];
//    printf("guard: %p %x PC %s\n", guard, *guard, PcDescr);
}
```
上面的`PC`指的是上一个函数的地址，有了这个函数地址，就可以拿到这个函数的符号，这里需要用到`dladdr`函数：

```js
#import <dlfcn.h>
```

`dladdr(const void *, Dl_info *)`，这个函数可以获得一个函数的名称以及地址，第一个参数传入PC，第二个参数定义一个变量 `Dl_info info`传进去：

```js
Dl_info info;
dladdr(PC, &info);
```
查看`Dl_info`，它是一个结构体：

```js
typedef struct dl_info {
        const char      *dli_fname;     /* Pathname of shared object */
        void            *dli_fbase;     /* Base address of shared object */
        const char      *dli_sname;     /* Name of nearest symbol */
        void            *dli_saddr;     /* Address of nearest symbol */
} Dl_info;
```
>`dli_sname`就是所需要符号

删除其他打印，把调试代码改为如下：


```js
void __sanitizer_cov_trace_pc_guard_init(uint32_t *start,
                                         uint32_t *stop) {
        static uint64_t N;
        if (start == stop || *start) return;
        printf("INIT: %p %p\n", start, stop);
        for (uint32_t *x = start; x < stop; x++)
            *x = ++N;
        NSLog(@"%d",N);
}
void __sanitizer_cov_trace_pc_guard(uint32_t *guard) {
    
    if (!*guard) return;
    void *PC = __builtin_return_address(0);
    Dl_info info;
    dladdr(PC, &info);
    printf("dli_sname -- %s\n",info.dli_sname);
}
```
运行：

```js
TraceDemo[21737:445410] 36
dli_sname -- main
dli_sname -- -[AppDelegate application:didFinishLaunchingWithOptions:]
dli_sname -- -[SceneDelegate window]
dli_sname -- -[SceneDelegate setWindow:]
dli_sname -- -[SceneDelegate window]
dli_sname -- -[SceneDelegate scene:willConnectToSession:options:]
dli_sname -- -[SceneDelegate window]
dli_sname -- -[ViewController viewDidLoad]
```

得到了启动时候所需要的所有符号和启动顺序。





