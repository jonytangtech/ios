# 单元测试

单元测试（unit testing），是指对软件中的最小可测试单元进行检查和验证。对于单元测试中单元的含义，一般来说，要根据实际情况去判定其具体含义，总的来说，单元就是人为规定的最小的被测功能模块。单元测试是在软件开发过程中要进行的最低级别的测试活动，软件的独立单元将在与程序的其他部分相隔离的情况下进行测试。
>单元测试（模块测试）是开发者编写的一小段代码，用于检验被测代码的一个很小的、很明确的功能是否正确。通常而言，一个单元测试是用于判断某个特定条件（或者场景）下某个特定函数的行为。例如，你可能把一个很大的值放入一个有序list 中去，然后确认该值出现在list 的尾部。或者，你可能会从字符串中删除匹配某种模式的字符，然后确认字符串确实不再包含这些字符了。

## 单元测试使用前后对比：

<img width="861" alt="Pasted Graphic" src="https://user-images.githubusercontent.com/126937296/225666271-df3c1800-0a47-43ff-a4a3-5fe101326369.png">

### 好处：
- 单元测试可以不启动整个工程调试某个功能
- 单元测试容易定位问题
- 单元测试保证修改代码后容易发现问题
- 可以从测试角度开始思考代码设计
- 保证代码的质量

创建项目，勾选`Include Tests`

<img width="730" alt="Pasted Graphic" src="https://user-images.githubusercontent.com/126937296/225662472-0c356bf3-f5df-49ae-81e5-d3b03003c4c2.png">


创建JTMathTool文件

```js
#import <Foundation/Foundation.h>
@interface JTMathTool : NSObject
//加
- (int)addNum1:(int)num1 num2:(int)num2;
//减
- (int)subNum1:(int)num1 num2:(int)num2;
//乘
- (int)mulNum1:(int)num1 num2:(int)num2;
//除
- (int)divNum1:(int)num1 num2:(int)num2;
@end
```
.m文件

```js
#import "JTMathTool.h"

@implementation JTMathTool

- (int)addNum1:(int)num1 num2:(int)num2 {
    return num1 + num2;
}
- (int)subNum1:(int)num1 num2:(int)num2 {
    return num1 - num2;
}
- (int)mulNum1:(int)num1 num2:(int)num2 {
    return num1 * num2;
}
- (int)divNum1:(int)num1 num2:(int)num2 {
    return num1 / num2;
}
@end

<img width="730" alt="Pasted Graphic 1" src="https://user-images.githubusercontent.com/126937296/225662634-5429d3f4-78e7-42de-b9a1-8d3351b4b382.png">


```
在`JTUnitTestDemoTests`文件新建`JTMathToolTests`文件
>单元测试文件名字要测试类的名字 + Tests

JTMathToolTests会自动生成几个方法：
提供在调用测试用例中的每个测试方法之前重置状态的机会

```js
- (void)setUp {
/把设置代码放在这里。在调用类中的每个测试方法之前调用此方法。
}
```

提供在测试用例中的每个测试方法结束后执行清理的机会：

```js
- (void)tearDown {
}
```
写测试用例

```js
- (void)testExample {
//这是一个功能测试用例。
//使用XCTAssert和相关函数来验证您的测试产生正确的结果。
```
测试性能

```js
- (void)testPerformanceExample {
//这是一个性能测试用例。
[self measureBlock: ^ {
//把你想测量时间的代码放在这里。
});
｝
```
导入要测试的类，定义属性：

```js
#import <XCTest/XCTest.h>
#import "JTMathTool.h"

@interface JTMathToolTests : XCTestCase
@property (nonatomic, strong) JTMathTool *tool;
@end
```
定义好初始化和置空：

```js
- (void)setUp {
    self.tool = [JTMathTool new];
}

- (void)tearDown {
    self.tool = nil;
}
```
使用断言:

```js
//生成一个失败的测试； 
XCTFail(format…) 
//为空判断，a1为空时通过，反之不通过； 
XCTAssertNil(a1, format...)
//不为空判断，a1不为空时通过，反之不通过；
XCTAssertNotNil(a1, format…)
//当expression求值为TRUE时通过；
XCTAssert(expression, format...) 
//当expression求值为TRUE时通过； 
XCTAssertTrue(expression, format...)
//当expression求值为False时通过；
XCTAssertFalse(expression, format...) 
//判断相等，[a1 isEqual:a2]值为TRUE时通过，其中一个不为空时，不通过；
XCTAssertEqualObjects(a1, a2, format...)
//判断不等，[a1 isEqual:a2]值为False时通过；
XCTAssertNotEqualObjects(a1, a2, format...)
//判断相等（当a1和a2是 C语言标量、结构体或联合体时使用,实际测试发现NSString也可以）； 
XCTAssertEqual(a1, a2, format...)
//判断不等（当a1和a2是 C语言标量、结构体或联合体时使用）；
XCTAssertNotEqual(a1, a2, format...)
//判断相等，（double或float类型）提供一个误差范围，当在误差范围（+/-accuracy）以内相等时通过测试； 
XCTAssertEqualWithAccuracy(a1, a2, accuracy, format...)
//判断不等，（double或float类型）提供一个误差范围，当在误差范围以内不等时通过测试；
XCTAssertNotEqualWithAccuracy(a1, a2, accuracy, format...)  
//异常测试，当expression发生异常时通过；反之不通过；
XCTAssertThrows(expression, format...)
//异常测试，当expression发生specificException异常时通过；反之发生其他异常或不发生异常均不通过； 
XCTAssertThrowsSpecific(expression, specificException, format...) 
//异常测试，当expression发生具体异常、具体异常名称的异常时通过测试，反之不通过；
XCTAssertThrowsSpecificNamed(expression, specificException, exception_name, format...) 
//异常测试，当expression没有发生异常时通过测试；
XCTAssertNoThrow(expression, format…)
//异常测试，当expression没有发生具体异常、具体异常名称的异常时通过测试，反之不通过；
XCTAssertNoThrowSpecific(expression, specificException, format...) 
//异常测试，当expression没有发生具体异常、具体异常名称的异常时通过测试，反之不通过，特别注意下XCTAssertEqualObjects和XCTAssertEqual。
XCTAssertNoThrowSpecificNamed(expression, specificException, exception_name, format...)
//判断条件是[a1 isEqual:a2]是否返回一个YES，a1 == a2是否返回一个YES,对于后者，如果a1和a2都是基本数据类型变量，那么只有a1 == a2才会返回YES.
XCTAssertEqualObjects(a1, a2, format...)
```
先用最简单的XCTAssert：

```js
- (void)testExample {
    int addResult = [self.tool addNum1:20 num2:10];
    XCTAssert( addResult == 30, "加法错误");
}
```
点击绿色框位置

<img width="595" alt="1__#$!@%!#__Pasted Graphic" src="https://user-images.githubusercontent.com/126937296/225662759-ffc74eb0-d167-4871-bdfd-3c6544b7125d.png">

测试通过会显示绿色箭头：

<img width="575" alt="1__#$!@%!#__Pasted Graphic 1" src="https://user-images.githubusercontent.com/126937296/225662826-12ec1938-ba70-445a-ba6e-8e84195dd2d6.png">

改期望结果为29，显示错误：

<img width="557" alt="Pasted Graphic 2" src="https://user-images.githubusercontent.com/126937296/225662885-ebeb4547-68d7-4b3f-8412-ad409937f691.png">

数值比较的还可以用XCTAssertEqual：

<img width="565" alt="Pasted Graphic 4" src="https://user-images.githubusercontent.com/126937296/225663155-5c9397ba-a491-4a60-ba97-16e4bc57c6cc.png">

改为29：

<img width="532" alt="Pasted Graphic 3" src="https://user-images.githubusercontent.com/126937296/225663250-fedb2341-f2a5-4af3-9745-fcdf1b070b3d.png">

把加减乘除表达式都做测试：

<img width="651" alt="Pasted Graphic 5" src="https://user-images.githubusercontent.com/126937296/225663324-8ee3462d-efd3-40e2-a941-7ed8036ff188.png">

>但单元测试是最小单位的测试，不建议方法都写在一起

独立的测试方法以test开头：

```js
- (void)testAddMath{
    XCTAssertEqual( [self.tool addNum1:20 num2:10], 30);
}
- (void)testsubMath{
    XCTAssertEqual( [self.tool subNum1:20 num2:10], 10);
}
- (void)testMulMath{
    XCTAssertEqual( [self.tool mulNum1:20 num2:10], 200);
}
- (void)testdivMath{
    XCTAssertEqual( [self.tool divNum1:20 num2:10], 2);
}
```
按 cmd + u 运行

<img width="648" alt="Pasted Graphic 6" src="https://user-images.githubusercontent.com/126937296/225663392-83e0bb93-2378-486b-b811-bc6d4ddc6093.png">

点击红色框位置，就会对所有运行单元测试，全部通过
既可准备打包：

<img width="289" alt="Pasted Graphic 7" src="https://user-images.githubusercontent.com/126937296/225663471-5fc912eb-709e-4ff3-a429-6d07270e5c8a.png">
## 总结
1、iOS单元测试有4个方法，setUp、tearDown、testExample、testPerformanceExample；
2、初始化的值写在setUp里面，tearDown里面把值置空，testExample写测试例子，testPerformanceExample写测试性能的东西
3、自己写的测试方法必须以test开头
4、执行顺序是setUp、test开头的测试用例、tearDown，多个测试用例会重复执行这一顺序
5、性能测试必须写在mesure里面，OC叫mesureBlock
6、测试异步，需要用到期望，初始化一个期望值，然后异步执行完成，调用实现期望，最后调用等待期望和超时做比较。
7、swift使用的时候，必须导入@testable import target名称
8、每个单元测试都是独立的，可以独立运行
9、单元测试设计不要太复杂，多个复杂的单元测试会耗费太多时间
10、尽可能地覆盖边界条件，提升代码安全性
11、定期维护，随着项目迭代，可能部分单元测试已经过时，不能再准确测试函数功能，需要及时维护单元测试，确保测试的有效性
12、不是所有的代码都需要写单元测试，但是单元测试主要覆盖核心代码
13、每次提交合并前，需要跑一次全量的单元测试


## 参考：
- [iOS开发之单元测试](https://www.jianshu.com/p/6f4f9fe5f1e1)
- [iOS单元测试简介和使用](https://www.jianshu.com/p/eeed010fc889)
- [iOS 单元测试](https://www.jianshu.com/p/0146d9b3d0b2)
- [iOS单元测试](https://www.jianshu.com/p/72395afd7c96)
- [iOS单元测试](https://blog.csdn.net/qq_14920635/article/details/121997083)
- [ios单元测试及自动化测试(只看这篇就够了)](https://www.freesion.com/article/84021421264/)
