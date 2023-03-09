# 启动优化clang插桩(二)
## 一、前言
上一篇的方法给到的是个数，但不是符号，个数并没有什么作用，甚至给了全部的符号也没什么用，因为二进制重排仅仅需要的是启动阶段所需要的符号，这就需要下面这个函数：
```js
void __sanitizer_cov_trace_pc_guard(uint32_t *guard) {
}
```
添加断点，点击箭头，可以看到绿色框中的函数调用栈：

![Pasted Graphic.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/da117b3c5b9142828920fa421551cda6~tplv-k3u1fbpfcp-watermark.image?)

>函数调用栈跟之前讲到的`app名称.LinkMap-normal-arm64.txt`文件里面的数据格式一样。

把上面断点过掉，给`touchBegan`方法添加断点：：

```js
-(void)touchesBegan:(NSSet<UITouch *> *)touches withEvent:(UIEvent *)event {
///
}
```
点击上面的绿色箭头：

![Pasted Graphic 1.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/1a745e75c83b47458c08037768527054~tplv-k3u1fbpfcp-watermark.image?)

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


![Pasted Graphic 3.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/10b77628acc048b5b306a7711e36324b~tplv-k3u1fbpfcp-watermark.image?)

>把这些写入到`.order`文件里面这样二进制重排就搞定了

而`_sanitizer_cov_trace_pc_guard`，这个函数是如何做到这一点的？给`_sanitizer_cov_trace_pc_guard`添加断点，在Xcode的`Debug`选择`Debug WorkFlow`选择显示汇编，选择`main`：

![Pasted Graphic 5.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/4d1b0666cf1b4ad8828f09784843b537~tplv-k3u1fbpfcp-watermark.image?)

在`AppDelegate`页面也是插入了这个符号：

![Pasted Graphic 6.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/f10bca67f7dd45ff83ebf41b2378f6e8~tplv-k3u1fbpfcp-watermark.image?)

在`SceneDelegate`同样如此：

![Pasted Graphic 7.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/4c39426e962143d8b6bf94f51eda8599~tplv-k3u1fbpfcp-watermark.image?)

也就是说，在编译器`clang`添加下面这个标记后，编译器会给函数方法前面都会调用`_sanitizer_cov_trace_pc_guard`这个函数：

![Pasted Graphic 8.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/38fa589830d54d9386403cdf1ebc9c62~tplv-k3u1fbpfcp-watermark.image?)

这样，我们确实在打断点看到启动阶段的所有符号：

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





