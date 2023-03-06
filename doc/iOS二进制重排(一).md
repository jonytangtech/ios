
# 二进制重排（一）

对于`app`启动来说分为两个阶段：
- 第一个阶段是在`main`函数之前，操作系统加载app可执行文件到内存，进行一系列的加载和链接工作，最后`dyld`调起`main`函数，这个过程叫`pre-main`；
- 第二个阶段就是`main`函数开始到`appdelegate`的`-didFinishLauching`方法到展示首页的内容为止。

## 启动时间检测
在 Xcode 13 & iOS 15 之前，Xcode 已经为我们提供了便捷的方法，在项目里点击`Edit Scheme`：

<img width="415" alt="Pasted Graphic" src="https://user-images.githubusercontent.com/126937296/223014067-36f2d7b3-d7a5-4200-9222-a187dfebf974.png">

选择`Run`，环境变量填入：`Value`值填：`1`：

<img width="762" alt="Pasted Graphic 1" src="https://user-images.githubusercontent.com/126937296/223014109-a9081b71-858b-484b-96dd-f08de5d13aba.png">

在设置完环境变量之后运行打印：

<img width="639" alt="Pasted Graphic 5" src="https://user-images.githubusercontent.com/126937296/223014198-b3b15767-2300-4f3d-9e82-4c4ea56dcf43.png">

可以看到`main`函数运行之前，`pre-main`阶段运行所消耗的时间，这是用一个老项目跑在真机上，差不多是`120`毫秒的消耗。
### time 里面又分为4个阶段：

- dylib loading time
- rabase/binding time
- Objc setup time
- initializer time

<img width="644" alt="Pasted Graphic 6" src="https://user-images.githubusercontent.com/126937296/223016373-514b7064-6dc5-4da9-a17f-99ec9a35c585.png">

## 优化方法：
- 对于`dylib loading time` 加载时间的优化：可以减少动态库的数量，这里的动态库是指自己制作的动态库，而非系统的动态库 ，像苹果的`libSystem.B.dylib`或者`Foundation`等动态库已经在共享缓存里，系统已经做了最大优化，我们无需处理，而对于我们自己制作的动态库，苹果建议不要超过`6`个;
- 对于`Objc setup time` 加载时间的优化： 它做的事情是注册`Objc`所有的类，相当于在可执行文件`Mac-o`中读取所有的类，把他们放到一张表里面去，这些类就能被我们所使用，包括`runtime`动态添加一个类，它都需要`register`函数做注册操作。

>因此，我们所有使用到的类在系统加载时候都有一个注册的过程，所以，不管我们写的类最终有没有使用到，它都会有注册操作，这最终会影响到
程序启动的时间。
>

### 如何检测项目中有哪些类没有被用到？
- 打开终端，运行下面的命令：
```swift
gem install fui
```
- 查看是否成功： 
```swift
fui help
```
### 如何使用？
拿一个测试项目举例， 在项目点击到终端：


<img width="611" alt="Pasted Graphic 7" src="https://user-images.githubusercontent.com/126937296/223016594-619862b6-2cee-488a-bd87-b1c068f71ac8.png">

就可以找到项目中没有使用的类

<img width="447" alt="Pasted Graphic 8" src="https://user-images.githubusercontent.com/126937296/223016642-e424794e-5276-40f4-b1b5-c9a50df8fd7f.png">

- 这样就可以根据需要删除一些一直没有使用的类文件（删之前请做好备份，主要针对老项目或者那些很多人接手的）；
- 还有一些方式可以查看哪些类是没被使用过，但是大部分类都是有用的，所以其实对于启动优化提升可能不是很明显，这就是`setup`阶段的优化；
- 另外，`github`有一个项目`LSUnusedResources-master`，可以清楚未使用的图片，减少包体积。

<img width="721" alt="Pasted Graphic 9" src="https://user-images.githubusercontent.com/126937296/223016866-e58dd1ce-7157-4d93-bb99-f34f1d02dad4.png">

运行完会看到软件界面：

<img width="660" alt="Pasted Graphic 11" src="https://user-images.githubusercontent.com/126937296/223016915-6e90123e-07d2-4a4c-9d71-3775b0a95b72.png">

选择项目路径，点击`search`会出现没有使用的图片资源

<img width="660" alt="Pasted Graphic 10" src="https://user-images.githubusercontent.com/126937296/223016981-30f75e93-483e-4052-be3f-f86022568652.png">

这对打包`app`体积优化有帮助，尤其是老项目

```js
在ObjC setup time这个阶段之后，还有一个initializer time，在这个阶段主要是各种初始化操作，比如初始化ObjC类的load函数，执行
C++的构造函数，load函数加载会在main执行之前，所以尽量不要在load里面做一些耗时操作，它都会影响app的启动时间。

```
### 符号绑定
再回过头来说一下`rebase/binding time binding`就是符号绑定，这里拿一个小的可执行文件举例：

<img width="159" alt="Pasted Graphic 12" src="https://user-images.githubusercontent.com/126937296/223039589-b1382fc7-61f7-4cae-8d6d-c9802750a265.png">

在终端执行：

```swift
xcrun nm -nm main
```
得到：

<img width="651" alt="Pasted Graphic 13" src="https://user-images.githubusercontent.com/126937296/223039993-5dc586c2-c2f7-48ff-b370-4a0e4e3a0b57.png">

最上面两个`(undefined)` 未定义里面的 `binder`就是用来符号绑定的，在进行静态链接的时候，`printf`函数会被标记来自`libSystem`动态库， 但是只知道哪个库是没有用的，需要知道函数的切确地址，一旦执行了`_printf`函数`binder`就会把`libStystem`里面函数真正实现的地址和`_printf`这个符号进行绑定，调用`_printf`能够真正的去执行函数的实现，这个过程就是`binding`符号绑定。

### 虚拟内存
另一个`rebase`，他是做指针的修复（翻译），这里涉及到虚拟内存的概念。

- 虚拟内存是由物理内存发展得来，在早期操作系统使用的是物理内存，就是直接把程序直接加载到物理内存里面，就是内存的地址和它真是的地址是一模一样，当加载太多程序的 时候就会出现内存不足的情况，这个时候需要关掉一些程序才能打开现在要打开的程序，而且软件的发展速度远远快于硬件的升级速度，这样直接使用物理内存的缺陷非常明显；
- 同时由于程序上访问的内存地址是真实的地址，这样可以通过内存指针偏移去访问到其他的数据，这样就存在一定的安全风险；
- 另外程序加载后，有许多的内容并不是马上需要用到，只需要加载所需的内容，所以就把内存分为一块一块，加载需要的数据进内存，就设计出了懒加载，当你需要使用的时候才加载进内存里面， 这样就节省了内存，但是这样就衍生出一个新的问题，用户切换不同程序的时候，用户使用这个程序需要把这个程序加载到内存，用另一个程序又需要把程序加到内存，这样就会造成一种现象，一个程序的地址在内存里面是不连续的，这又出现了一个新的情况，系统每次分配空闲内存是随机的，并不知道哪块内存一定对应哪个程序，这样程序运行效率很低，因为系统需要大量计算来确定内存属于哪个程序，系统就会显示出卡顿的现象。

因此，虚拟内存出现了。





