# 离屏渲染(二)

## 有哪些操作到导致离屏渲染

<img width="640" alt="Pasted Graphic" src="https://user-images.githubusercontent.com/126937296/224494746-a89b3876-6718-4b55-9395-76289ef4495a.png">

## 一、 添加光栅化

光栅化是一个缓存机制，如果开启了光栅化，它会将图片以一个bitmap位图的形式，保存起来，当下一次需要时候，CPU直接从缓存里拿出来交给GPU进行处理，这样GPU就不需要进行一些渲染的计算，光栅化就是一个能减少GPU计算的操作，光栅化是一个提高性能的操作。但是，日常使用光栅化的场景非常少，光栅化使用有限制，因为bitmap只能缓存100ms。

```js
//光栅化 
- (void)shouldRasterize {
    self.testImageView.layer.shouldRasterize = YES;
}
```
可以看到开启光栅化会触发离屏渲染：

<img width="333" alt="Pasted Graphic 1" src="https://user-images.githubusercontent.com/126937296/224494761-91ecc335-0aab-41f3-92c2-3cc3ee10271a.png">


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

<img width="339" alt="Pasted Graphic 2" src="https://user-images.githubusercontent.com/126937296/224494789-18bfefc4-d896-4efc-9533-4bb9d5d873c6.png">

>因为添加遮罩，创建出来的layer会被添加到原本图像的默认layer上面，而屏幕上的每一个像素点是通过多层layer由GPU混合计算出来的，多添加了一层layer，就是类似上篇文章讲的，层级变复杂了，这样GPU无法把需要呈现的图像一次绘制完毕，他只能用离屏渲染的方式来处理。

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

<img width="334" alt="Pasted Graphic 3" src="https://user-images.githubusercontent.com/126937296/224494801-debaf965-0455-4a5e-a865-67d1e38bb81d.png">

>这个阴影和遮罩mask是很像，只不过遮罩是添加到layer的上层，而阴影是添加到layer的下层，它的层级也比较复杂，所以也会触发离屏渲染。

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
没有了离屏渲染:

<img width="326" alt="Pasted Graphic 4" src="https://user-images.githubusercontent.com/126937296/224494814-89ee1686-e8a0-46d7-9f9e-0a49076dee63.png">

>阴影是添加在layer的下层，阴影会优先会被渲染，在渲染阴影的时候依赖视图的大小，但视图本身还没有被渲染好，这个时候只能通过离屏渲染进行辅助处理，贝塞尔曲线就是提前指定阴影的路径，这个阴影的渲染就不需要依赖视图本体，这个阴影会被单独地进行渲染，不需要通过离屏渲染辅助合成图像。
## 五、抗锯齿

```js
//抗锯齿
- (void) setEdgeAnntialiasing {
    CGFloat angle = M_PI / 60.0;
    [self.testImageView.layer setTransform:CATransform3DRotate(self.testImageView.layer.transform, angle, 0.0, 0.0, 1.0)];
    self.testImageView.layer.allowsEdgeAntialiasing = YES;
} 
```
这个要分情况，在图片Content Mode是Aspect Fill的模式下：

<img width="255" alt="Pasted Graphic 5" src="https://user-images.githubusercontent.com/126937296/224494826-0d5385eb-836b-4daf-8477-6d7a07b3465d.png">

会触发离屏渲染：

<img width="250" alt="Pasted Graphic 6" src="https://user-images.githubusercontent.com/126937296/224494831-57419274-d6d3-40a0-858c-89b9d1e8011e.png">

如果改为Scale To Fill或者Aspect Fit等不需要进行抗锯齿计算的模式就不会触发离屏渲染：

<img width="241" alt="Pasted Graphic 7" src="https://user-images.githubusercontent.com/126937296/224494844-12909008-a0f1-4cef-8d07-d5ea728b0f22.png">

>因此，icon的图片最好跟控件的比例或者尺寸是一样的，最大可能地减少离屏渲染的可能性。

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

<img width="250" alt="Pasted Graphic 8" src="https://user-images.githubusercontent.com/126937296/224494852-8285b074-3061-40cb-9a67-fd9886b578db.png">

#### 2. 一种还有子视图的情况开启allowsGroupOpacity：

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
- 如果本身还有子视图，父视图不透明度不为1，开启允许组不透明就需要混合计算，这样就会触发离屏渲染：

<img width="339" alt="Pasted Graphic 9" src="https://user-images.githubusercontent.com/126937296/224494863-38259f16-58ef-4e03-8167-5fbee00bf80e.png">

- 如果本身还有子视图，父视图不透明度为1，开启允许组不透明就需要混合计算，这样不会触发离屏渲染：
- 
<img width="249" alt="1__#$!@%!#__Pasted Graphic" src="https://user-images.githubusercontent.com/126937296/224494872-fd18e226-c0ce-41d3-9756-09ebae0e619d.png">

## 七、.圆角
#### 1. 设置背景颜色和圆角

```js
- (void)setRadius {
    self.testImageView.backgroundColor = [UIColor redColor];
    self.testImageView.layer.cornerRadius = 45;
}
```
会触发离屏渲染：

<img width="337" alt="1__#$!@%!#__Pasted Graphic 5" src="https://user-images.githubusercontent.com/126937296/224494889-8caf1af5-14bd-41f3-bd31-a5fffb2d93fe.png">

#### 2. 只设置圆角
不会触发：

<img width="219" alt="1__#$!@%!#__Pasted Graphic 7" src="https://user-images.githubusercontent.com/126937296/224494901-f2db3a2f-ee76-4b42-8d25-723e9a1fc77f.png">

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

<img width="233" alt="1__#$!@%!#__Pasted Graphic 8" src="https://user-images.githubusercontent.com/126937296/224494919-4c7a2609-b537-4586-8dff-935e9fa8ea2e.png">

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

<img width="224" alt="1__#$!@%!#__Pasted Graphic 9" src="https://user-images.githubusercontent.com/126937296/224494944-48af3ad5-fb35-4293-b211-cabd27f0383e.png">

>因为label的layer设置背景颜色，它实际是给contents设置颜色

图片设置圆角的时候，实际是设置给border和backgroundColor设置，不会设置contents，所以图片需要Clips to Bounds。

<img width="743" alt="Pasted Graphic 10" src="https://user-images.githubusercontent.com/126937296/224494962-030bc723-7d32-4abb-b34b-a29f01ca0eba.png">

>在iOS9之后给视图设置单纯设置圆角不会触发离屏渲染，但是如果同时操作contents + backgroundColor/border就会触发。

## 8、贝塞尔曲线

```js
//贝塞尔曲线
- (void)setBezier {
    //开始上下文 UIGraphicsBeginImageContextWithOptions(self.testImageView.bounds.size, NO, 0.0);
    [[UIBezierPath bezierPathWithRoundedRect:self.testImageView.bounds cornerRadius:40] addClip];
    [self.testImageView drawRect:self.testImageView.bounds];
    //当前上下文
    self.testImageView.image = UIGraphicsGetImageFromCurrentImageContext();
    // 结束上下文
    UIGraphicsEndImageContext();
}
```
不会触发离屏渲染：

<img width="164" alt="Pasted Graphic 11" src="https://user-images.githubusercontent.com/126937296/224494973-b6e08883-96e3-4f73-92c6-e7e8b15dbc74.png">

## 9、drawRect使用

```js
-(void)drawRect:(CGRect)rect {
    //特殊的离屏渲染
    //baking store -- bitmap
}
```
视图本身就有一块画布，drawRect会重新增加一块画布，会重新生成baking store，增加内存消耗，虽然检测不到，但也会触发离屏渲染，是特殊的离屏渲染。








