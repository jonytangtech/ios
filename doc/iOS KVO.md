KVO

---
theme: github
---

# KVO

`KVO` 是 `NSKeyValueObserving` 缩写，对象采用的一种非正式协议，监听对象指定的属性发生变化时得到通知。

## 一、概述

你可以观察任何对象属性，包括简单属性、一对关系和多对多关系。对多关系的观察者被告知所进行的更改的类型——以及哪些对象涉及到更改。
`NSObject` 实现了 `NSKeyValueObserving` 协议，该协议提供了自动观察所有对象的能力。您可以通过禁用自动观察者通知和使用本协议中的方法实现手动通知来进一步完善通知。

## 二、主要方法：

### 1、注册观察者

注册观察者对象去接收KVO通知：

```js
- addObserver:forKeyPath:options:context:
```

### 2、变化通知

当被观察者对象的值已经发生变化时，通知观察者对象：

```js
- observeValueForKeyPath:ofObject:change:context:
```

### 3、停止观察

停止观察者对象接收变化的通知：

```js
- removeObserver:forKeyPath:
```

## 三、内部实现方法

通知观察对象给定的值即将更改：

```js
- willChangeValueForKey:
```

通知观察对象给定的值已经更改：

```js
- didChangeValueForKey:
```

### 1、使用

可以使用点语法、set方法，也可以使用KVC触发

```js
//通过属性的点语法间接调用
objc.name = @"";

// 直接调用set方法
[objc setName:@"Savings"];

// 使用KVC的setValue:forKey:方法
[objc setValue:@"Savings" forKey:@"name"];

// 使用KVC的setValue:forKeyPath:方法
[objc setValue:@"Savings" forKeyPath:@"account.name"];
```

### 2、集合类使用KVO

```js
- mutableArrayValueForKey
```
监听集合对象变化是，需要通过`KVC`的`mutableArrayValueForKey`的方法获得代理对象，并使用代理对象进行操作，当代理对象内部发生改变时，会触发`KVO`的监听方法。
> 集合对象包括NSArray和NSSet。

## 四、实现原理

添加观察后，`KVO` 会调用私有方法 `_NSSetxxxValueAndNotify`
在运行时，会动态生成一个原来类的子类 ，并动态修改原来对象的 `isa` 指针指向这个中间类，派生类的指针指向了原来的类对象，新类的类名是`NSKVONotifying_原来类名`。

这个类会重写被观察属性的`set`方法、`class`方法、`dealloc`方法和一个私有的 `_isKVOA`方法。
新类重写了`set`方法：

```js
- willChangeValueForKey:
属性 = 新值
- didChangeValueForKey:
```

> 此时，代理方法接收到值已经改变的通知

新类重写了`class`方法，如果此时执行`[类名 class]`还是返回了原来的类，苹果屏蔽了具体的实现细节，不让你去操纵动态生成的类。

