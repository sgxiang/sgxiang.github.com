---
layout: post
title: "从C声明谈到Objective-C块声明"
date: 2013-11-20 23:51
comments: true
categories: oc
keywords: oc,block
description: 在这篇文章中，我将从最基本的C声明符开始逐步过渡到复杂的Ojbective-C的blocks语法。
---
在这篇文章中，我将从最基本的C声明符开始逐步过渡到复杂的Ojbective-C的blocks语法。我花了一些时间去理解block语法，但是你一旦了解了它是如果组织和构成的，在你需要声明一个block的时候就不用无时无刻去google了。

##Declarators声明符

变量的声明在C(Objective-c)中用声明符。

一个声明符有2个规则：

* 指定变量的类型 
* 给变量一个名称让它在合适的范围里使用

从最基本的声明符开始：

```objc
int a;
```

`int`是基本类型，`a`是一个变量名称或者称为标识符。

你会从标识符开始然后从右阅读直到末尾，然后从变量左边开始。

在这里右边没有任何东西所以它直截了当：`a`是一个`int`。

一个声明只能有1个基本类型，它是最左边的字的说明符。 

说明符可以使用修饰符创建派生类型来修改基本类型。有4个修饰符：`*`,`[]`,`()`,`^`。

<!--more-->


##指针修饰符 *

```objc
int *a;
```

基本类型依然是`int`，它的名称依然是变量`a`。但是指针修饰符`*`告诉我们`a`是一个指向`int`的指针而不是一个`int`。

`*`修饰符通常出现在左边去修饰变量。

##数组修饰符 []

```objc
int a[];
```

在这里我们可以看到，数组修饰符`[]`告诉我们`a`现在是一个`int`数组而不是一个简单的`int`。我们还可以定义数组的个数例如`int a[10]`。

数组修饰符`[]`总是在右边修饰变量。

##函数修饰符 ()

```objc
int f();
```

函数修饰符`()`告诉我们`f`是一个函数它返回一个`int`。这个修饰符还可以指定函数的参数，例如`int f(long)`，这个函数带有一个`long`类型的参数以及返回一个`int`。

函数修饰符`()`总是在右边修饰变量。

##指针和数组

修饰符可以组合来创建复杂的变量类型。修饰符和算术运算符一样有优先级的区分。`[]`和`()`比`*`(`^`)拥有更高的优先级。2个高级的修饰符被正确的写入变量的时候，当我们阅读复制的声明的时候，__你总是从标识符开始然后向右阅读直到末尾或者一个括号的结尾然后才回到左边__。

```objc
int *a[];
```

我们可以通过加括号来提高可读性

```objc
int *(a[]);
```

这是一个指向`int`的指针的数组。

但是你可能会问，如果我想要有一个指向`int`数组的指针呢？因为`*`比`[]`的优先级低，所以你需要括号`[]`来提高它的优先级。

```objc
int (*a)[]
```

`a`现在是一个指针变量，指向一个`int`数组。

##数组和函数

你不可能有定义一个函数数组和返回一个数组或者函数。一个函数可以将数组作为参数。

```objc
int f(int [10]);
```

这个函数带有10个`int`类型的参数以及返回`int`。

##指针和函数

```objc
int *f();
int *(f());
```

这两种情况下，`f`是一个函数，返回一个指向`int`的指针。

如果想要一个指向函数的指针呢？__括号！__

```objc
int (*f)();
```

`f`是一个指向函数的指针，返回`int`。

##块指针修饰符 ^

苹果公司在其扩展的ANSI-C标准中提出第四个修饰符：`^`。这个修饰符称为块。块和指向函数的指针很类似。你声明块的方式类似与声明一个指针函数。

块指针修饰符只能应用到一个函数中（你不能这样写`int ^a`，这不是一个合法的定义）。

这就是为什么`int ^b()`是非法的，这会导致一个编译错误：当你用优先级的规则阅读这个声明的时候，`b`将是一个函数，返回一个块指针指向`int`。这就是为什么你声明块的时候你需要将`^`和标识符放在括号`()`中。

```objc
int (^b)()
```

`b`是一个块指针指向一个函数，返回`int`。或者缩写为块返回`int`。

当然，你可以在需要的时候指定其参数：

```objc
int (^b)(long)
```

这个块带有一个参数返回`int`。

这就是块声明的来源。

现在你已经了解了，在使用块的时候这里还有一些其他的语法需要你记住：一个用于定义一个块字面量，另一个是快递一个块给objective-c函数。

##抽象的声明符

一个声明符由两件事组成：一个抽象声明符在其中插入标识符。

C中使用抽象声明符的3个cases:

* 在这描述中：`int *a; long *b = (long *)a;`,`(long *)`是一个抽象声明符，是一个指向`long`的指针。
* sizeof()作为参数：`malloc(sizeof(long *));`
* 声明参数类型给函数：`int f(long *);`

objective-c可以在更多地方使用抽象声明符：声明方法参数或函数返回值。

```objc
-(long **)methodWithArgument:(int *)a;
```

`long **`和`int *`是抽象声明。

所以为了在objective-c中使用块作为参数或返回值，我们需要去找到这些块的抽象声明符。我们可以通过取得声明符和删除标识符来得到。

`int (^b)()`变成`int (^)()`。`int (^b)(long)`变为`int (^)(long)`。

例子：

```objc
-(void)methodWithArgument:(int(^)())block;
-(void)anotherMethodWithArgument:(void(^)(long arg1))block;
```

虽然你不需要参数的名字出现在块抽象的声明符中，当时这是一个很好的做法。它会给你一个很好的暗示当解析参数的时候，xcode会自动的完成它们来让你的在使用这个函数的时候变得更加方便。

##块字面量

当你这样写的时候：`int a=2;`，`a`是一个声明符，2是对于`int`的字面量。

`^`也被用作为一元运算符将一个函数变为一个块。你不需要指定返回类型，它会在返回语句中去推断，你也可以去添加它，这都一样。

为了实现这个块，你需要去命名它的参数。

所以对于块`int (^block)(long,long);`来说，其字面量是：

```objc
block = ^(long a,long b){
	int c = a + b;
	return c;
}
```

__译自：[http://nilsou.com/blog/2013/08/21/objective-c-blocks-syntax/](http://nilsou.com/blog/2013/08/21/objective-c-blocks-syntax/)__

__________

##怎样在objective-c中定义块？

作为本地变量：

```objc
returnType (^blockName)(parameterTypes) = ^returnType(parameters) {...};
```

作为属性：

```objc
@property (nonatomic, copy) returnType (^blockName)(parameterTypes);
```

作为函数参数：

```objc
- (void)someMethodThatTakesABlock:(returnType (^)(parameterTypes))blockName {...}
```

作为参数去响应函数：

```objc
[someObject someMethodThatTakesABlock: ^returnType (parameters) {...}];
```

作为typedef：

```objc
typedef returnType (^TypeName)(parameterTypes);
TypeName blockName = ^(parameters) {...}
```

__ 来自:http://fuckingblocksyntax.com __