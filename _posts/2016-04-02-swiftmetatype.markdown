---
layout: post
title: Swift中的类型和元类型
published: true
category: translation
tags:
    - iOS
    - Swift
---

# Swift中的类型和元类型
原文链接: [Types and meta types in swift](https://medium.com/@NSomar/types-and-meta-types-in-swift-9cd59ba92295#.i2ez3m55j)

在siwft中`类型`和`元类型`并不显得像Objc那样重要. 在Objc中不需要类型名也能实例化一个对象或者在运行中修改类,然而在纯siwft中这些都是不允许的.

事实上siwft是一个支持泛型的语言, 并且几乎没有反射能力(reflection capabilities), 获取和处理类型和元类型在swift中并不常见.

然而, 有时候我们必须或者很想知道一个实例(或者类)的类型, swift提供了两种方法获取类型;
用*dynamicType*获得实例的类型, 用*self*获取类或结构体的类型.

## Type.self
对一个类型调用`self`方法可以获取到该类型的元类型. 返回值可能是一个*Class metatype*, *Struct metatype*, 或者是*Protocol metatype*.

![](/assets/14586202648339/14586252657255.jpg)￼

尽管上面的例子看起来都很相似, 然而实际上并不是. *NSString.self*返回的值看起来很像Objective-C类.
实际上它(NSString.self 返回的类)还包含了一些用seift扩展定义的其他方法, 例如调用`NSString.self.className()`会得到“NSString”.

调用*MyClass.self*得到的对象其实就是*MyClass*的swift元类型(强调swift以区分objc中的元类概念).
定义在swift对象离的`init`方法和所有定在这个类中的所有`curried method`都会对外暴露出来([Instance Methods are Curried Functions in Swift](http://oleb.net/blog/2014/07/swift-instance-methods-curried-functions/))

如果我们把*MyClass.self* 传到*NSStringFromClass*方法中,会得到一个奇怪的类名.
![](/assets/14586202648339/14586999399589.jpg)￼

*String.self*调用返回值似乎没什么问题. 有一点小小的区别值得提一提,如果把*String.self*传到*NSStringFromClass*会得到一个编译错误. 不要太吃惊, *String*是一个结构体不是一个类.

## dynamicType

如果对Type调用`.self`会得到它的`metatype`, 获取一个实例的Type可以`dynamicType`方法. 对一个类调用`.self`和实例对象调用`dynamicType`的返回结果是一样的都是`metatype`.
![](/assets/14586202648339/14587014529347.jpg)￼

然而, 这个规则对String, Int和Double无效.

![](/assets/14586202648339/14587015195991.jpg)￼

作者认为, 这是因为String, Int和Double在swift中是特殊类型. 实际上swift能无缝的转换成Foundation类型,直接对其元类型进行比较是不可能的.

更新: Joe Groff, swift team成员之一在twitter上透露,这个或许是一个bug.
![Screen Shot 2016-03-23 at 10.59.01 A](/assets/14586202648339/Screen%20Shot%202016-03-23%20at%2010.59.01%20AM.png)￼


所以 *dynamicType*和*Type.self*进行比较成为可能:
![](/assets/14586202648339/14587023646397.jpg)￼

类型比较并不是一个主意最好不要那么做. 检查一个swift的类型的一个安全方式是使用swift的`is`语法. 如下面比较Double的方式(请忽略警告)
![](/assets/14586202648339/14587027988433.jpg)￼

我们可以用Type实例化一个对象. 比如下面我们用 *MyClass*的类型实例化一个对象:
![](/assets/14586202648339/14587029669096.jpg)￼

用 *dynamicType*初始化却不能:
![](/assets/14586202648339/14587038586002.jpg)￼

通过错误提示描述,给MyClass添加*required init*方法:
![](/assets/14586202648339/14587039230735.jpg)￼

## Holding a metatype references

当用一个变量存*"".dynamicType*或者*String.self*的返回值,其引用的类型为*String.Type*.*String.Type*通常用于声明元类型变量:
![](/assets/14586202648339/14587877197899.jpg)￼

用*String.Type*或者任何*Struct.Type*声明的变量且仅可以存结构体的meta-type.编译器不允许对同一个变量赋于不同的meta-type:
![](/assets/14586202648339/14587879503238.jpg)￼

------

对于类(区别于结构体), 因为其能子类化, 其行为不同于上面的例子. 一个*BaseClass.Type*能引用*SubClass.Type*.

下面我们用*NSDictionary.self*和*NSMutableDictionary.self*给同一个变量赋值:
![](/assets/14586202648339/14587882025616.jpg)￼

值得注意的事上面的例子反过来不成立,如果声明的是*NSMutableDictionary.Type*类型变量,编译器不允许引用*NSDictionary.self*.

------

毫无疑问,  *Protocol.Type*变量能引用所有实现了其协议的类型:
![](/assets/14586202648339/14588043900385.jpg)￼

这里要注意,没有关联*type*或者*Self*的协议不能用于声明变量.有关联*type*或者*Self*协议也仅仅用于泛型约束上.

![](/assets/14586202648339/14595852009740.jpg)￼

最后要注意的是, 声明为 *Any.Type*的变量可以引用任何类型, 无论是类还是结构体.
![](/assets/14586202648339/14595852784627.jpg)￼


*AnyObject.Type*在另一方面, 可以引用任何类的元类型.

![](/assets/14586202648339/14595852893747.jpg)￼
