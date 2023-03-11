
![Pasted Graphic.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/827c76365934453bb79387421c55a507~tplv-k3u1fbpfcp-watermark.image?)
# 有哪些操作到导致离屏渲染？
## 一、 添加光栅化

光栅化是一个缓存机制，如果开启了光栅化，它会将图片以一个`bitmap`位图的形式，保存起来，当下一次需要时候，`CPU`直接从缓存里拿出来交给`GPU`进行处理，这样`GPU`就不需要进行一些渲染的计算，光栅化就是一个能减少`GPU`计算的操作，光栅化是一个提高性能的操作。但是，日常使用光栅化的场景非常少，光栅化使用有限制，因为`bitmap`只能缓存`100ms`。


```js
//光栅化 
- (void)shouldRasterize {
    self.testImageView.layer.shouldRasterize = YES;
}
```
可以看到开启光栅化会触发离屏渲染：

![Pasted Graphic 1.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/57432907b8864b11b2e0dc56b82accba~tplv-k3u1fbpfcp-watermark.image?)

## 二、添加遮罩


```js
- (void)setMask {
    //添加到layer的上层
    CALayer *layer = [CALayer layer];
    layer.frame = CGRectMake(30, 30, self.testImageView.bounds.size.width, self.testImageView.bounds.size.height);
    layer.backgroundColor = [UIColor redColor].CGColor;
    self.testImageView.layer.mask = layer;
}
```
会触发离屏渲染：

![Pasted Graphic 2.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/c3afcb9ea3004780a33b0ef54a8e22ed~tplv-k3u1fbpfcp-watermark.image?)

>因为添加遮罩，创建出来的`layer`会被添加到原本图像的默认`layer`上面，而屏幕上的每一个像素点是通过多层`layer`由`GPU`混合计算出来的，多添加了一层`layer`，就是类似上篇文章讲的，层级变复杂了，这样`GPU`无法把需要呈现的图像一次绘制完毕，他只能用离屏渲染的方式来处理。

## 三、添加阴影


```js
//阴影
- (void)setShadows {
    self.testImageView.layer.shadowColor = [UIColor redColor].CGColor;
    self.testImageView.layer.shadowOffset = CGSizeMake(20, 20);
    self.testImageView.layer.shadowOpacity = 0.2;
    self.testImageView.layer.shadowRadius = 5;
    self.testImageView.layer.masksToBounds = NO;
}
```
会触发离屏渲染：

![Pasted Graphic 3.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/ff0f5f18f6794227aca688b0852b7134~tplv-k3u1fbpfcp-watermark.image?)

>这个阴影和遮罩`mask`是很像，只不过遮罩是添加到`layer`的上层，而阴影是添加到`layer`的下层，它的层级也比较复杂，所以也会触发离屏渲染。

## 四、使用贝塞尔曲线进行优化

```js
//阴影优化
- (void)setShadows2 {
    self.testImageView.layer.shadowColor = [UIColor redColor].CGColor;
    self.testImageView.layer.shadowOpacity = 0.2;
    self.testImageView.layer.shadowRadius = 5;
    self.testImageView.layer.masksToBounds = NO;
    
    //提前指定阴影的路径
    [self.testImageView.layer setShadowPath:[UIBezierPath bezierPathWithRect:CGRectMake(0, 0, self.testImageView.bounds.size.width + 20, self.testImageView.bounds.size.height + 20)].CGPath];
}
```
没有离屏渲染:

![Pasted Graphic 4.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/365597d42e754a629ab6445c6d6f690e~tplv-k3u1fbpfcp-watermark.image?)

>阴影是添加在`layer`的下层，阴影会优先会被渲染，在渲染阴影的时候依赖视图的大小，但视图本身还没有被渲染好，这个时候只能通过离屏渲染进行辅助处理，贝塞尔曲线就是提前指定阴影的路径，这个阴影的渲染就不需要依赖视图本体，这个阴影会被单独地进行渲染，不需要通过离屏渲染辅助合成图像。
## 五、抗锯齿

```js
//抗锯齿
- (void) setEdgeAnntialiasing {
    CGFloat angle = M_PI / 60.0;
    [self.testImageView.layer setTransform:CATransform3DRotate(self.testImageView.layer.transform, angle, 0.0, 0.0, 1.0)];
    self.testImageView.layer.allowsEdgeAntialiasing = YES;
} 
```
这个要分情况，在图片`Content Mode`是`Aspect Fill`的模式下：

![Pasted Graphic 5.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/e6870c72275c44c682b9b363b183b848~tplv-k3u1fbpfcp-watermark.image?)

会触发离屏渲染：

![Pasted Graphic 6.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/b1c305f4f8f34b8b8a04ca138573cd3e~tplv-k3u1fbpfcp-watermark.image?)

如果改为`Scale To Fill`或者`Aspect Fit`等不需要进行抗锯齿计算的模式就不会触发离屏渲染：

![Pasted Graphic 7.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/e69e1293a98b483fb666460c97a0ab53~tplv-k3u1fbpfcp-watermark.image?)

>因此，`icon`的图片最好跟控件的比例或者尺寸是一样的，最大可能地减少离屏渲染的可能性。

## 六、不透明
不透明也要分两种情况：

```js
//不透明
- (void)allowsGroupOpacity {
  self.testImageView.alpha = 0.5;
  self.testImageView.layer.allowsGroupOpacity = YES;
}
```
#### 1. 如果是自己本身不透明，并不会触发离屏渲染：

![Pasted Graphic 8.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/c0d722a622b3475da5c8d4995f303996~tplv-k3u1fbpfcp-watermark.image?)
#### 2. 一种还有子视图的情况开启`allowsGroupOpacity`：

```js
- (void)allowGroupOpacity {
  UIView *view = [[UIView alloc] initWithFrame:CGRectMake(0, 0, 20, 20)];
  view.backgroundColor = [UIColor greenColor];
  [self.testImageView addSubview:view];
    
  self.testImageView.alpha = 0.5;
    //allowsGroupOpacity 设置视图的子视图在透明度上是否和俯视图一致
  self.testImageView.layer.allowsGroupOpacity = YES;
}
```
- 如果本身还有子视图，父视图不透明度小于`1`，开启允许组不透明就需要混合计算，这样就会触发离屏渲染：

![Pasted Graphic 9.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/9e838459bc6b48deb184dee89b890155~tplv-k3u1fbpfcp-watermark.image?)
- 如果本身还有子视图，父视图不透明度为`1`，开启允许组不透明就需要混合计算，这样不会触发离屏渲染：

![1__#$!@%!#__Pasted Graphic.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/8e37f0edac4141b99296bc09d5856f9f~tplv-k3u1fbpfcp-watermark.image?)

## 七、.圆角
#### 1. 设置背景颜色和圆角

```js
- (void)setRadius {
    self.testImageView.backgroundColor = [UIColor redColor];
    self.testImageView.layer.cornerRadius = 45;
}
```
会触发离屏渲染：

![1__#$!@%!#__Pasted Graphic 5.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/572f4aea42ea45b386d76a5cb88871a6~tplv-k3u1fbpfcp-watermark.image?)

#### 2. 只设置圆角
不会触发：

![1__#$!@%!#__Pasted Graphic 7.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/23b7cea00d724d94aa513c1d3129fa39~tplv-k3u1fbpfcp-watermark.image?)

#### 3. 给子视图标签添加圆角和颜色

```js
- (void)setRadius {
    self.testImageView.backgroundColor = [UIColor redColor];
    self.testImageView.layer.cornerRadius = 40;
    self.testLabel.backgroundColor = [UIColor greenColor];
    self.testLabel.layer.cornerRadius = 10;
}
```
发现绿色标签不会：

![1__#$!@%!#__Pasted Graphic 8.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/bdf534940ea749c280322d638774298f~tplv-k3u1fbpfcp-watermark.image?)

#### 4. 如果给label的layer层添加颜色

```js
- (void)setRadius {
    self.testImageView.backgroundColor = [UIColor redColor];
    self.testImageView.layer.cornerRadius = 40;
    self.testLabel.layer.backgroundColor = [UIColor greenColor].CGColor;
    self.testLabel.layer.cornerRadius = 10;
}
```
会触发离屏渲染：

![1__#$!@%!#__Pasted Graphic 9.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/2f5982c8d6154271863ac390597cc224~tplv-k3u1fbpfcp-watermark.image?)

>因为`label`的`layer`设置背景颜色，它实际是给`contents`设置颜色

图片设置圆角的时候，实际是设置给`border`和`backgroundColor`设置，不会设置`contents`，所以图片需要`Clips to Bounds`。

![Pasted Graphic 10.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/4b0ab7e58ce24efb9b2f72ab6e3e8d99~tplv-k3u1fbpfcp-watermark.image?)

>在`iOS9`之后给视图设置单纯设置圆角不会触发离屏渲染，但是如果同时操作`contents + backgroundColor/border`就会触发。

## 八、贝塞尔曲线

```js
//贝塞尔曲线
- (void)setBezier {
    //开始上下文 UIGaphicsBeginImageContextWithOptions(self.testImageView.bounds.size, NO, 0.0);
    [[UIBezierPath bezierPathWithRoundedRect:self.testImageView.bounds cornerRadius:40] addClip];
    [self.testImageView drawRect:self.testImageView.bounds];
    //当前上下文
    self.testImageView.image = UIGraphicsGetImageFromCurrentImageContext();
    // 结束上下文
    UIGraphicsEndImageContext();
}
```
不会触发离屏渲染：

![Pasted Graphic 11.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/1d480f7e655c4dec9aa84cf4fdbcf761~tplv-k3u1fbpfcp-watermark.image?)

## 九、drawRect使用

```js
-(void)drawRect:(CGRect)rect {
    //特殊的离屏渲染
    //baking store -- bitmap
}
```
视图本身就有一块画布，`drawRect`会重新增加一块画布，会重新生成`baking store`，增加内存消耗，虽然检测不到，但也会触发离屏渲染，是特殊的离屏渲染。








