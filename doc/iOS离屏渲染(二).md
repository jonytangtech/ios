
<img width="640" alt="Pasted Graphic" src="https://user-images.githubusercontent.com/126937296/224495422-d5173d0f-44c9-4d87-ac20-7ae11cab5301.png">

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

<img width="333" alt="Pasted Graphic 1" src="https://user-images.githubusercontent.com/126937296/224495429-24db9fad-70d7-4155-afb3-c61459066d4a.png">

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

<img width="339" alt="Pasted Graphic 2" src="https://user-images.githubusercontent.com/126937296/224495439-44dd88e0-2ecf-4387-9870-65cf149881cd.png">

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

<img width="334" alt="Pasted Graphic 3" src="https://user-images.githubusercontent.com/126937296/224495451-065ace36-ae06-4680-a5bf-499921c9191a.png">

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
    [self.testImageView.layer setShadowPath:[UIBezierPath bezierPathWithRect:CGRectMake(0, 0,       self.testImageView.bounds.size.width + 20, self.testImageView.bounds.size.height + 20)].CGPath];
}
```
没有离屏渲染:

<img width="326" alt="Pasted Graphic 4" src="https://user-images.githubusercontent.com/126937296/224495476-3adf5b8a-3c42-4135-871a-7c954a4d63c5.png">

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

<img width="255" alt="Pasted Graphic 5" src="https://user-images.githubusercontent.com/126937296/224495498-718c6c3e-8258-40ea-9274-8175be2b8b93.png">

会触发离屏渲染：

<img width="250" alt="Pasted Graphic 6" src="https://user-images.githubusercontent.com/126937296/224495506-4ea0cc18-a55e-4e00-970b-519b4113c56b.png">

如果改为`Scale To Fill`或者`Aspect Fit`等不需要进行抗锯齿计算的模式就不会触发离屏渲染：

<img width="241" alt="Pasted Graphic 7" src="https://user-images.githubusercontent.com/126937296/224495515-306bd6c6-1dcf-4d67-a22a-e1ea6ea46bf4.png">

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

<img width="250" alt="Pasted Graphic 8" src="https://user-images.githubusercontent.com/126937296/224495525-1667237b-4861-458b-bf81-a110b47b4326.png">

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

<img width="339" alt="Pasted Graphic 9" src="https://user-images.githubusercontent.com/126937296/224495545-20034a2a-3ded-452d-976a-646f51278fd6.png">

- 如果本身还有子视图，父视图不透明度为`1`，开启允许组不透明就需要混合计算，这样不会触发离屏渲染：

<img width="249" alt="1__#$!@%!#__Pasted Graphic" src="https://user-images.githubusercontent.com/126937296/224495555-5758a8b7-ce76-415c-9dd2-cc0113772be4.png">

## 七、.圆角
#### 1. 设置背景颜色和圆角

```js
- (void)setRadius {
    self.testImageView.backgroundColor = [UIColor redColor];
    self.testImageView.layer.cornerRadius = 45;
}
```
会触发离屏渲染：

<img width="337" alt="1__#$!@%!#__Pasted Graphic 5" src="https://user-images.githubusercontent.com/126937296/224495571-908a4ba7-3373-4de9-b93c-a5c510b19fe0.png">

#### 2. 只设置圆角
不会触发：

<img width="219" alt="1__#$!@%!#__Pasted Graphic 7" src="https://user-images.githubusercontent.com/126937296/224495577-fe3d63ad-2c23-4d87-9ca0-c6aff9eaed67.png">

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

<img width="233" alt="1__#$!@%!#__Pasted Graphic 8" src="https://user-images.githubusercontent.com/126937296/224495603-841754f1-1181-4b8c-8be1-696c96edfb96.png">

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

<img width="224" alt="1__#$!@%!#__Pasted Graphic 9" src="https://user-images.githubusercontent.com/126937296/224495612-b57f4422-2d59-43f3-a4b2-fb470a2d1c04.png">

>因为`label`的`layer`设置背景颜色，它实际是给`contents`设置颜色

图片设置圆角的时候，实际是设置给`border`和`backgroundColor`设置，不会设置`contents`，所以图片需要`Clips to Bounds`。

<img width="743" alt="Pasted Graphic 10" src="https://user-images.githubusercontent.com/126937296/224495618-deb24a77-af9c-4bb4-bde1-376741be0b99.png">

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

<img width="164" alt="Pasted Graphic 11" src="https://user-images.githubusercontent.com/126937296/224495632-0bb16e22-9961-4195-9a6e-8142f01db551.png">


## 九、drawRect使用

```js
-(void)drawRect:(CGRect)rect {
    //特殊的离屏渲染
    //baking store -- bitmap
}
```
视图本身就有一块画布，`drawRect`会重新增加一块画布，会重新生成`baking store`，增加内存消耗，虽然检测不到，但也会触发离屏渲染，是特殊的离屏渲染。








