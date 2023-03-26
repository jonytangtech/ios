# 离屏渲染(一)

## 前言

在iOS中图形图像的渲染流程，在平时开发中最常使用的开发框架是`UIKit`，我们会使用`UIKit`的框架来绘制界面，而`UIKit`实际可以看成是集成`CoreAnimation`和`CoreGraphics`这两个框架，来方便开发者使用，设置`UIKit`的一些布局和相关属性来绘制界面。

<img width="853" alt="Pasted Graphic" src="https://user-images.githubusercontent.com/126937296/224248160-6199508a-f79c-4c42-bd57-80f6fb4719a4.png">

- 如果界面如果需要动画效果就需要`CoreAnimation`这个框架来实现，而`CoreAnimation`又依赖于`OpenGL ES/Metal` 来做`GPU`的渲染。
- `CoreGraphics`框架是一个高级的绘图引擎，它的主要作用是运行的时候去绘制图像。我们可以使用这个框架来做一些绘图、转换、离屏渲染、阴影、图像的创建等等。
- `CoreGraphics`是在`CPU`是执行的 ，`CoreImage`和`CoreGraphics`是相反的，`CoreImage`是处理创建图像运行前的操作，`CoreImage`拥有一套现成的图像过滤性，它能对一些已经存在的图像进行高效的处理，这个框架既能在`CPU`上执行，也能在`GPU`上执行，我们APP就会使用`CoreGraphics`、`CoreAnimation`和`CoreImage`来绘制一些可视化的内容，这些框架都需要再通过`OpenGL ES/Metal`来做`GPU`的绘制，最终再将图像显示到屏幕上面。

## 一、图像图像渲染流程

<img width="935" alt="Pasted Graphic 7" src="https://user-images.githubusercontent.com/126937296/224248188-2ca96d01-a0f5-44cc-8a28-8c1e16f05fa6.png">
 
### 上面这张图像就告诉我们图形图像在渲染的时候`CPU`和`GPU`分别做了哪些事情：
- 首先，在`Application`阶段，是由`CPU`来处理，`CPU`会创建我们的视图，它会计算视图的一些数据，进行编解码，绘制纹理等等的操作后再交给`GPU`，`GPU`在第一阶段会通过顶点着色器去确定图像在硬件上具体的显示位置；
- 然后，通过片源着色器来计算每个像素点的颜色值，最后进行光栅化，这个光栅化会找到图形上像素的点的范围，然后把一个一个的像素点的颜色显示上去，最终会把图形转化为一个个的实际的屏幕像素，当这些操作都处理完之后，就会把渲染完成之后的数据放到帧缓存区里面；
- 最后，再交由显示系统将帧缓存区里的数据给读取出来进行显示 。
   
 ## 二、图像显示流程图

<img width="925" alt="Pasted Graphic 9" src="https://user-images.githubusercontent.com/126937296/224248229-94f8d663-6ac5-44f9-85ca-084f09024450.png">

上面就是图像显示流程图，当一个图像在`CPU`和`GPU`处理完之后，它就会被存放到`FrameBuffer`里面，`FrameBuffer`被称为帧缓存区，然后视频控制器就会往`FrameBuffer`去读取数据，读取的数据就会交给显示器显示，我们就能看到屏幕上一帧一帧的画面。

 ## 三、屏幕扫描

<img width="843" alt="Pasted Graphic 12" src="https://user-images.githubusercontent.com/126937296/224248254-620a1478-3de7-4bfb-b8af-94dc732325b0.png">

它是通过屏幕扫描的方式，会通过`CRT`电子枪从上到下逐行扫描，这个扫描的过程就是读取帧缓存区里的数据，当它扫描完的时候，它就会显示一帧的画面， 当显示完一帧画面后，`CRT`电子枪又回到原来的位置继续从上到下扫描，这样就会显示下一帧的画面，如此循环往复，这就是我们能看到手机屏幕上的一些内容。

当电子枪扫描完一行要换到下面一行的时候，这个时候显示器会发出水平同步信号`HSync`；当电子扫描完一帧，要回到原来位置，这个时候显示器就会发出垂直同步信号`VSync`。这个显示器通常都是按照固定的频率来刷新的，这个刷新的频率就是垂直同步信号`VSync`的频率。苹果手机（除了新出的高刷）的刷新频率是每秒`60`次，也就是`60FPS`，这就是我们在进行屏幕卡顿监测或者卡顿优化的时候，会用`FPS`来作为衡量的指标。

<img width="933" alt="Pasted Graphic 13" src="https://user-images.githubusercontent.com/126937296/224248280-9268d0ca-9104-4e59-9232-b8bad955fc8f.png">

如果屏幕刷新频率不是接近固定刷新频率，那就是出现了掉帧。当一个垂直同步信号过来的时候，如果说`CPU`和`GPU`还没有完成渲染的结果去做提交，也就是没有把数据放到`FrameBuffer`里面，这种情况，未过来提交过来这一帧的画面就会被丢弃，然后等待下一次垂直同步信号过来，再来显示新的画面，这个过程被称为掉帧。 掉帧的时候，屏幕刷新的`FPS`频率就会减少，这也是界面显示真正卡顿的原因。

>接下来是可能造成屏幕卡顿的离屏渲染.

## 四、离屏渲染

<img width="819" alt="Pasted Graphic 14" src="https://user-images.githubusercontent.com/126937296/224248329-a2c000f6-bdaa-47f5-88cc-b25e45b8ee36.png">

比如我们给图像添加了遮罩，设置了某些圆角，`CPU`和`GPU`没有办法把渲染的数据放到`FrameBuffer`上，它会把渲染的数据先放到`FrameBuffer`之外的一块缓冲区，在这块区域进行合成渲染，渲染到我们最终想要的画面，再把数据放到`FrameBuffer`里面，这个过程就是**离屏渲染**。

<img width="926" alt="Pasted Graphic 15" src="https://user-images.githubusercontent.com/126937296/224248369-2f4cf8dc-1221-4940-a4f2-e82a12b96519.png">

先看`UIView`和`CALayer`的关系，`UIView`是基于`CALayer`的封装，一个`View`他本身是不能显示的，它想要显示，需要通过内部图层的`CALayer`来显示，`UIView`有`CALayer`的只读属性和遵循`CALayer`的代理。当`View`显示什么内容，都要绘制在它内部图层`Layer`上面，这个`Layer`就是负责显示，而`View`负责处理响应的事件。因此，我们看到的界面是`View`里面的`layer`层所呈现出来的。

<img width="623" alt="Pasted Graphic 16" src="https://user-images.githubusercontent.com/126937296/224248402-015674e9-6cae-449f-a1ad-b09595a3c9b5.png">

>下图可以看出，`layer`主要包括三个部分`backgroundColor`、`contents`和`borderColor`。

<img width="425" alt="Pasted Graphic 17" src="https://user-images.githubusercontent.com/126937296/224248434-93b98bb6-2aca-4992-9402-256bae2bdeff.png">

- 当视图层级比较复杂的时候，`GPU`无法直接把渲染数据放到`FrameBuffer`。比如下面的图例，第三张图是我们最终要看到的画面，它比较复杂，由于`CPU`和`GPU`是硬件，它有性能瓶颈，它去读取画面数据需要一定的算法，这个时候由于`GPU`没有办法通过一次遍历就能拿到一帧的完整数据，这个时候它就会遵循画家算法。
- 要画下面这张图，`GPU`它是一个机器，它就一块画布，第一次遍历扫描到图片上的山，这样第一次遍历结束后第二次遍历开始的时候，需要一块新的画布，又扫描出了草地， 以此类推，第三次遍历在新的画布上加上了树。
- 但是，无论是树还是草地或者树都不是我们最终想要的画面效果，这样就不能把任一画布单独放进`FrameBuffer`里面，因为画面不够完整，这样就需要开启一块额外的内存缓冲区，然后山、草地和树就在这里面合成，合成最终的效果再把合成的数据存到帧缓存区，这就是离屏渲染的整个流程。

<img width="611" alt="Pasted Graphic 19" src="https://user-images.githubusercontent.com/126937296/224248483-caee649a-4f30-431e-8026-fb532b3bf847.png">

> 离屏渲染实质就是硬件的瓶颈，有的图形，它没有办法做到一次渲染完成，于是通过离屏渲染的方式来处理，这就是离屏渲染产生的原因。


