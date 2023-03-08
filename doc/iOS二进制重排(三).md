#二进制重排(三)
## 二进制重排
>二进制重排为何能优化启动速度？主要跟缺页中断page fault有关， 因为我们的app在冷启动的时候有大量的类和大量的函数，它需要被加载和执行，这个时候产生的page fault所带来的时间消耗是比较大的，虽然一次page fault带来的时间消耗是毫秒级别，但是如果有几百个几千个这样的消耗，那这个时间加起来的消耗就很多。
### 一、缺页分析
首先打开自己的app，启动跑到真机上，然后按`cmd + ctrl + i`，`启动Instruments`

<img width="790" alt="Pasted Graphic" src="https://user-images.githubusercontent.com/126937296/223635445-aecb235c-410b-4f10-8f7b-7c04552c02d2.png">

选择`System Trace` 点击`Choose`，打开后点击箭头指向位置：

<img width="433" alt="Pasted Graphic 1" src="https://user-images.githubusercontent.com/126937296/223634875-08ea75de-2846-430a-90ec-d2d72584065e.png">


就会开始分析启动情况：

<img width="900" alt="Pasted Graphic 2" src="https://user-images.githubusercontent.com/126937296/223634928-f9d51be1-cb3b-4935-b50c-a31bd1e9e8ce.png">


分析完毕，再次点击箭头位置结束：

<img width="900" alt="Pasted Graphic 3" src="https://user-images.githubusercontent.com/126937296/223634968-f8792316-5ae5-4761-a490-c3eb058dd884.png">

在搜索框输入`main`：

<img width="443" alt="Pasted Graphic 4" src="https://user-images.githubusercontent.com/126937296/223635003-831697e5-bc65-4a52-a6f0-befb95139b12.png">

`Narrative`选择 `Summary`: `Virtual Memory`：

<img width="246" alt="Pasted Graphic 6" src="https://user-images.githubusercontent.com/126937296/223635034-e33778de-feb4-43cd-b142-16aa8ccbcbdc.png">

点击要优化app的`Main Thread`，可以看到`File Backed Page In` 这个就是`page fault` 缺页异常，总共是`2200`个，冷启动总耗时是`441ms`，占总耗时`454ms`非常大的比例

<img width="384" alt="Pasted Graphic 5" src="https://user-images.githubusercontent.com/126937296/223635069-be004f2f-34ba-4521-8f10-66f2a777936b.png">

这个地方就可以去优化，去减少`page fault`的次数

### 二、启动分析

把工程点击`Build Settings`，搜素`link map`，把`Wirte Link Map File` 改为`Yes`

<img width="719" alt="Pasted Graphic 9" src="https://user-images.githubusercontent.com/126937296/223635099-57ef9f26-997c-49dd-a6f1-5c9f96792428.png">

按`cmd + b`编译一下，点击`Product`，选择`show Buld Folder In Finder`

>按照 —> Intermediates.noindex —> app名称.build —> Debug-iphoneos —>  app名称.build  —> app名称-LinkMap-normal-arm64.txt

<img width="118" alt="Pasted Graphic 11" src="https://user-images.githubusercontent.com/126937296/223635134-8eafe11e-6405-49b6-a521-ec074a55d5fe.png">


用Xcode打开，搜索`# Address`可以看到函数名称和地址

<img width="748" alt="1__#$!@%!#__Pasted Graphic 5" src="https://user-images.githubusercontent.com/126937296/223635157-9b4f3114-2bcb-43b7-9e5a-3aede6204cb8.png">


这里是编译过后的地址，虽然是虚拟内存的地址，还没有加上`ASLR`，还需要加上一步`rebase`之后，才是真的虚拟内存的地址。这里的地址相当于第二篇说的`P2`、`P3`

<img width="495" alt="1__#$!@%!#__Pasted Graphic" src="https://user-images.githubusercontent.com/126937296/223635195-68f3f2ab-a702-4ec0-ab1a-d9e72ec5e84b.png">


而函数的排列，这是代码的编译顺序

<img width="456" alt="1__#$!@%!#__Pasted Graphic 6" src="https://user-images.githubusercontent.com/126937296/223635222-6d37107e-c743-48f0-9d6d-21d55c126c14.png">


如果在排列这些方法的时候，app所有启动需要调用到的数据都排列在最前面得`page`位置，这样就能最大程度的启动时候的`page`加载情况，并且减少大大减少p`age fault`缺页中断，最大程度的优化启动速度，这个行为就叫二进制重排，即重新排列程序的二进制文件

### 三、启动顺序调整

这个时候就需要一个`.order`文件，给编译器使用，这里先简单用一个例子举例

<img width="235" alt="1__#$!@%!#__Pasted Graphic 1" src="https://user-images.githubusercontent.com/126937296/223635247-1f00b4c6-bf96-4395-bc81-077e8f6603ce.png">


在`Demo1` 目录下的终端输入

```js
touch jt.order
```
打开` jt.order`

输入

```js
_main
_66666666666666
+[AppDelegate load]
+[ViewController load]
```

<img width="186" alt="1__#$!@%!#__Pasted Graphic 2" src="https://user-images.githubusercontent.com/126937296/223635271-31208a19-6851-46a9-a792-09427c9e96b7.png">

关闭保存

在`Bulid Settings` 输入 `order` 选择`Order File` 添加`./jt.order`

<img width="621" alt="1__#$!@%!#__Pasted Graphic 3" src="https://user-images.githubusercontent.com/126937296/223635309-e4622502-6e25-4204-9b45-d2c7addd013d.png">


按`cmd + b`编译一下，点击`Product`，选择`show Buld Folder In Finder`

<img width="700" alt="1__#$!@%!#__Pasted Graphic 4" src="https://user-images.githubusercontent.com/126937296/223635334-5739575b-c730-4e38-b082-b16055881678.png">

> 还是打开 `Intermediates.noindex` —> `app名称.build` —> `Debug-iphoneos` —>  `app名称.build`  —> `app名称-LinkMap-normal-arm64.txt`

### 总结
可以看到，现在编译的顺序就是我们`.order`文件里面写的顺序，而且无关输入被忽略，这样就可以根据我们设置方法的启动顺序，那怎样才能知道一个程序启动他需要哪些函数，哪些函数这就需要用到`clang`插桩。










