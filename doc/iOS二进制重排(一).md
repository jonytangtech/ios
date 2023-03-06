```js

```

` `


# 二进制重排（一）

对于app启动来说分为两个阶段，第一个阶段是在main函数之前，操作系统加载app可执行文件到内存，进行一系列的加载和链接工作，最后dyld调起main函数，这个过程叫pre-main；第二个阶段就是main函数开始到appdelegate的-didFinishLauching方法到展示首页的内容为止。

## 一、启动时间检测
在xcode13 & iOS 15之前，xcode已经为我们提供了便捷的方法，在项目里点击Edit Scheme：

<img width="415" alt="Pasted Graphic" src="https://user-images.githubusercontent.com/126937296/223014067-36f2d7b3-d7a5-4200-9222-a187dfebf974.png">

选择Run，环境变量填入：，Value值填：1：
<img width="762" alt="Pasted Graphic 1" src="https://user-images.githubusercontent.com/126937296/223014109-a9081b71-858b-484b-96dd-f08de5d13aba.png">

设置完环境变量之后，运行打印：</br>
<img width="639" alt="Pasted Graphic 5" src="https://user-images.githubusercontent.com/126937296/223014198-b3b15767-2300-4f3d-9e82-4c4ea56dcf43.png">

 可以看到main函数运行之前，pre-main阶段运行所消耗的时间，这是用一个老项目跑在真机上，差不多是120毫秒的消耗。
 可以看到time里面又分为四个阶段：
- dylib loading time
- rabase/binding time
- Objc setup time
- initializer time
这是行内代码`onCreate(Bundle savedInstanceState)`的例子。

<img width="644" alt="Pasted Graphic 6" src="https://user-images.githubusercontent.com/126937296/223016373-514b7064-6dc5-4da9-a17f-99ec9a35c585.png">

## 优化方法：
- 对于dylib loading time 加载时间的优化：可以减少动态库的数量，这里的动态库是指自己制作的动态库，而非系统的动态库 ，像苹果的libSystem.B.dylib或者Foundation等动态库已经在共享缓存里，系统已经做了最大优化，我们无需处理，而对于我们自己制作的动态库，苹果建议不要超过6个;
- 对于Objc setup time 加载时间的优化： 它做的事情是注册Objc所有的类，相当于在可执行文件Mac-o中读取所有的类，把他们放到一张表里面去，这些类就能被我们所使用，包括runtime动态添加一个类，它都需要register函数做注册操作。因此，我们所有使用到的类在系统加载时候都有一个注册的过程，所以，不管我们写的类最终有没有使用到，它都会有注册操作，这最终会影响到程序启动的时间。

## 如何检测项目中有哪些类没有被用到？
在终端输入：gem install fui 安装
查看是否成功： fui help
如何使用：
拿一个测试项目举例， 在项目点击到终端：

<img width="611" alt="Pasted Graphic 7" src="https://user-images.githubusercontent.com/126937296/223016594-619862b6-2cee-488a-bd87-b1c068f71ac8.png">

就可以找到项目中没有使用的类</br>
<img width="447" alt="Pasted Graphic 8" src="https://user-images.githubusercontent.com/126937296/223016642-e424794e-5276-40f4-b1b5-c9a50df8fd7f.png">

- 这样就可以根据需要删除一些一直没有使用的类文件（删之前请做好备份，主要针对老项目或者那些很多人接手的）;
- 还有一些方式可以查看哪些类是没被使用过，但是大部分类都是有用的，所以其实对于启动优化提升可能不是很明显，这就是setup阶段的优化;
- 另外，github有一个项目LSUnusedResources-master，可以清楚未使用的图片，减少包体积.

<img width="721" alt="Pasted Graphic 9" src="https://user-images.githubusercontent.com/126937296/223016866-e58dd1ce-7157-4d93-bb99-f34f1d02dad4.png">

运行完会看到软件界面</br>
<img width="660" alt="Pasted Graphic 11" src="https://user-images.githubusercontent.com/126937296/223016915-6e90123e-07d2-4a4c-9d71-3775b0a95b72.png">

选择项目路径，点击search会出现没有使用的图片资源

<img width="660" alt="Pasted Graphic 10" src="https://user-images.githubusercontent.com/126937296/223016981-30f75e93-483e-4052-be3f-f86022568652.png">
这对打包app体积优化有帮助，尤其是老项目

```js
在ObjC setup time这个阶段之后，还有一个initializer time，在这个阶段主要是各种初始化操作，比如初始化ObjC类的load函数，执行
C++的构造函数，load函数加载会在main执行之前，所以尽量不要在load里面做一些耗时操作，它都会影响app的启动时间。

```
再回过头来说一下rebase/binding time 
binding就是符号绑定，这里拿一个小的可执行文件举例









