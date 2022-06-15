---
title: 函数式编程（Functional Programming）
catalog: true
date: 2022-6-15 16:04
lang: cn
header-img: /img/header_img/lml_bg.jpg
tags:
- javascript 
- Functional Programming
categories:
- javascript
---

# 函数式编程（Functional Programming）

## **1 什么是函数式编程**

### **1.1 历史**

函数式编程并不是什么新颖的概念，它最早出现在数学领域，在计算机发展的历史上也一直伴随发展。

在1950年，随着Lisp语言的创建，函数式编程逐渐出现在大众的视野里。较为现代的语言例子的话有包括Haskell、Clean、[Erlang](https://www.infoq.cn/article/rkr4ree9aov6vrcyardw)和Miranda。

### **1.2 定义**

函数式编程本质上是一种起源于范畴轮的数学分支的数学运算方法，在计算机科学中被当做一种编程范式（programming paradigm）或者一种编程方法。

简单得介绍下范畴论，范畴是指存在某种关系的事务、概念。范畴论是抽象地处理数学结构以及结构之间联系的一门数学理论，以抽象的方法来处理数学概念，将这些概念形式化成一组组的“对象”及“态射”。

对应到计算机科学中便是一种抽象数据和数据间的映射关系的方法。

x → f(映射) → y

![https://alidocs.oss-cn-zhangjiakou.aliyuncs.com/res/YpLdn5My3AwBno83/img/36b6c35a-d85c-42a3-aada-5bcb1aa3ed8c.png](https://alidocs.oss-cn-zhangjiakou.aliyuncs.com/res/YpLdn5My3AwBno83/img/36b6c35a-d85c-42a3-aada-5bcb1aa3ed8c.png)

![https://alidocs.oss-cn-zhangjiakou.aliyuncs.com/res/YpLdn5My3AwBno83/img/1f06b524-c067-459f-bb37-9bd42970c893.png](https://alidocs.oss-cn-zhangjiakou.aliyuncs.com/res/YpLdn5My3AwBno83/img/1f06b524-c067-459f-bb37-9bd42970c893.png)

### **1.3** JavaScript **是函数式语言？**

在JavaScript的世界中万物皆可对象，给我们的第一反应这是一门面向对象（Object-oriented programming）语言。事实上，JavaScript 是基于原型（prototype-based）的多范式编程语言。也就是说面向对象只是 JavaScript 支持的其中一种范式而已，由于 JavaScript 的函数是一等公民，它也支持函数式编程范式。

## **2. 函数式编程的特性**

1. 纯函数（pure functions）
- **不依赖外部状态**：函数的运行结果不依赖当前函数作用域外的变量
- **没有副作用：**指当调用函数时，除了返回函数值之外，还对主调用函数产生附加的影响
1. 不可变性（immutable），不可变性指的是不直接修改数据，而是使用新的数据替换旧的数据
2. 引用透明和可置换性，如果一个函数对于相同的输入始终产生相同的结果，那么它就是引用透明的
3. 声明式（declarative）编程 ，函数式编程属于声明式（Imperative）编程范式，只关心数据的映射，与之对应的就是命令式编程范式。命令式编程是面向计算机硬件的抽象，有变量、赋值语句、表达式和控制语句等，而函数式编程是面向数学的抽象，将计算描述为一种表达式来求解。

```jsx
// 命令式
const foo = [1, 2, 3, 4, 5, 6];
for (let i = 0; i < foo.length; i++) {
	foo[i] = i + 1;
}

// 声明式
foo.map(number => number + 1);

```

## **3. 函数式编程的实践**

在讲实践之前，顺便讲一下编程范式和设计模式的区别。

编程范式可以理解为一种类型的编程风格，侧面反应程序员的哲学观和世界观。而设计模式则是一种实际生产开发中总结出的一套方法论。

既然它不是一种方法论，那我们应该如何进行函数式编程呢？

在过往的岁月中，前人在函数式编程范式的“战略”思想的指导下，总结归并了一些最佳实践：

组合、柯理化、函子等等。

### **3.1 组合（compose）**

函数组合：咋听之下是个名词，其实是一个动词，将一些函数组合起来，变成拥有复杂功能的函数，用于实现某些特定的功能。

下面看个例子：

某个商品售卖要计算价格（现实中不会放前端计算）：

1. 先要计算折扣
2. 优惠券
3. 再计算汇率

```jsx
const calcDiscount = (originPrice) => {
	// calc something
  return price;
}

const calcCoupon = (originPrice) => {
	// calc something
  return price;
}

const calcExchangeRate = (originPrice) => {
	// calc something
  return price;
}

const finalPrice = calcExchangeRate(calcCoupon(calcDiscount(price)))
```

如果这时候要求添加一个计算满减、添加一个活动优惠烦人的括号，以及不能一眼看清的顺序；

这时候你需要compose

先来看一个简单的compose实现：

```jsx
const compose = (f,g) => (x) => f(g(x));
// compose(f, g)(x) === f(g(x))
```

在lodash等库中已经将compose封装好了，_.flow和_.flowRight。

在这里实名吐槽他的api名称设计，一般现在业界公认的从左往右叫pipe，从右往左叫compose

再回到上面计算价格的例子

```jsx
const calcFinalPrice = compose(calcExchangeRate, calcCoupon, calcDiscount);
const price = calcFinalPrice(20);
// 使用lodash的api
const calcFinalPrice = _.flowRight(calcExchangeRate, calcCoupon, calcDiscount);
const price = calcFinalPrice(20);
```

### **3.1.1 pointfree**

pointfree 模式指的是，永远不必说出你的数据。即在你的方法定义中不要出现你数据。

还是拿计算价格这个例子，可能有的哥们会写成

```jsx
const calcFinalPrice = (price) => compose(calcExchangeRate, calcCoupon, calcDiscount)(price);
const price = calcFinalPrice(20);
```

出现了参数price这就非常不pointfree

pointfree是一种函数式编程的风格，虽然不是必须的，但可以用此来检验我们的方法组织是否合理

## **3.2 柯**理化(currying**)**

### **3.2.1 柯理化定义**

柯里化是一种函数的转换，是把接受多个参数的函数变换成接受一个单一参数（最初函数的第一个参数）的函数，并且返回接受余下的参数而且返回结果的新函数的技术。

它是指将一个函数从可调用的 f(a, b, c) 转换为可调用的 f(a)(b)(c)。

柯里化不会调用函数。它只是对函数进行转换。

### **3.2.2 为什么要柯理化**

- 参数复用
- 延迟计算 / 运行

继续拿计算价格的来举例子

如果现在折扣是从外部各种参数输入动态计算的

```jsx
const calcDiscount = (discount, params1, params2, ..., originPrice) => {
	// calc something
  return price;
}

const calcCoupon = (originPrice) => {
	// calc something
  return price;
}

const calcExchangeRate = (originPrice) => {
	// calc something
  return price;
}
// 不能compose了
```

这时就轮到curry出场了，柯理化在lodash中也有实现：_.curry

先看个小例子方便快速建立理解：

```jsx
const add = (a, b) => a + b;
const curryAdd = _.curry(add);
const addTen = curryAdd(10);
const addThree = curryAdd(3)
console.log(addTen(2)); // 12
console.log(addThree(5)) // 8

curryAdd(3)(10) === add(3, 10)
```

再回到我们的价格计算，那么我们可以

```jsx
const calcDiscountWithParams = curryCalcDiscount(calcDiscount)(discount, params1, params2, ..., )
const calcFinalPrice = compose(calcExchangeRate, calcCoupon, calcDiscountWithParams);
```

## **3.3 函子（Functor）**

### **3.3.1 函子定义**

函子也是范畴论中的一个概念：

范畴再抽象化一次，范畴自身亦为数学结构的一种，因此可以寻找在某一意义下会保持其结构的“过程”；此一过程即称之为函子。

大白话：

```jsx
// lodash
_(data).map(item => item + 1).filter(item => item > 1).values();

new Promise((resolve, reject) => {}).then(() => {}).then(() => {}).catch(() => {});

[data].map(item => item + 1).filter(item => item > 1)

Rx.Observable.fromEvent($input, 'keyup')
  .map(e => e.target.value)
  .filter(text => text.length > 0)
  .debounceTime(100)

```

根据以上场景可以得出：**值被容器化之后具有一条标准协议规范的数据类型或者数据容器。**

![https://alidocs.oss-cn-zhangjiakou.aliyuncs.com/res/YpLdn5My3AwBno83/img/521d904b-5adb-415c-a8e7-69716125815a.png](https://alidocs.oss-cn-zhangjiakou.aliyuncs.com/res/YpLdn5My3AwBno83/img/521d904b-5adb-415c-a8e7-69716125815a.png)

如果仍理解困难的话，可以途中的计算作为例子：

```jsx
const Box = x => ({
  map: f => Box(f(x)),
  fold: f => f(x),
  inspect: () => `Box(${x})`
})

Box(2)
  .map(x => x + 3)
  .fold(x => x)  // => 5
```

Box即为一个简单的函子，当然这么简单的函子在实际生产中的作用是有限的，实际生产中我们要处理各种副作用，比如IO副作用；逻辑分支处理；异常处理；

### **3.3.2 MayBe函子**

MayBy函子的存在意义是为了处理在链式调用中，开始或者中途出现空值的情况做处理。

```jsx
class MayBe {
  static of (value) {
    return new MayBe(value)
  }
  constructor (value) {
    this._value = value
  }

  map(fn) {
    // 判断一下value的值是不是null和undefined，如果是就返回一个value为null的函子，如果不是就执行函数
    return this.isNothing() ? MayBe.of(null) : MayBe.of(fn(this._value))
  }

 // 定义一个判断是不是null或者undefined的函数，返回true/false
  isNothing() {
    return this._value === null || this._value === undefined
  }
}

/**************** 实际使用 ******************/
const foo => str => MayBe.of(str)
  .map(x => x.toUpperCase())

console.log(foo('hello word!')) // MayBe { _value: 'HELLO WORLD' }
console.log(foo(null)) // MayBe { _value: null }
```

### **3.3.3 Either函子**

- Either 两者中的任何一个，类似于if...else...的分支处理
- 异常会让函数变得不纯，Either函子可以用来做异常处理

```jsx
class Left {
  static of (value) {
    return new Left(value)
  }

  constructor (value) {
    this._value = value
  }

  // 左倾是错误的
  map (fn) {
    return this
  }
}

class Right {
  static of (value) {
    return new Right(value)
  }

  constructor (value) {
    this._value = value
  }

  map (fn) {
    return Right.of(fn(this._value))
  }
}

function parseJSON (str) {
    try {
        return Right.of(JSON.parse(str))
    } catch (e) {
        return Left.of({ error: e.message})
    }
}

let r = parseJSON('{ name: zs}')
console.log(r); // // Left { _value: { error: 'Unexpected token n in JSON at position 1' } }
let r1 = parseJSON(JSON.stringify({ name:'zs'} ))
console.log(r); // Right { _value: { name: 'zs' } }

```

### **3.3.4 IO 函子**

在前端工程中是不可能做到所谓的完全的纯函数，总有一些副作用，这时就需要使用IO函子。

IO 函子和其他函子非常特殊，因为他是用来处理“不纯”的操作：

- IO函子中的_value是一个函数，这里是把函数作为值来处理，这个函数往往是“不纯”的操作，比如异步请求、读取文件等等
- IO函子可以把不纯的动作存储到_value中，延迟执行这个不纯的操作（惰性执行），包装当前的纯操作
- 把不纯的操作交给调用者来处理

```jsx
class IO {
  // 这里的value是一个不纯函数
  static of (value) {
    return new IO(function () {
      return value
    })
  }
  constructor(fn) {
    this._value = fn
  }
  map(fn) {
    return new IO(flowRight(fn, this._value))
  }
}

let r = IO.of(process).map(p => p.execPath)._value();
console.warn(r); // /usr/local/bin/node
```

### **3.3.5 其他函子**

Task函子、Pointed函子、Monad函子等等等等。。。功能各不相同

## **4. 优缺点**

### 优点

- 更好的管理状态：因为它的宗旨是无状态，或者说更少的状态，能最大化的减少这些未知、优化代码、减少出错情况
- 更简单的复用：固定输入->固定输出，没有其他外部变量影响，并且无副作用。这样代码复用时，完全不需要考虑它的内部实现和外部影响
- 更优雅的组合：往大的说，网页是由各个组件组成的。往小的说，一个函数也可能是由多个小函数组成的。更强的复用性，带来更强大的组合性
- 隐性好处。减少代码量，提高维护性

### 缺点：

- 性能：函数式编程相对于指令式编程，性能绝对是一个短板，因为它往往会对一个方法进行过度包装，从而产生上下文切换的性能开销
- 资源占用：在 JS 中为了实现对象状态的不可变，往往会创建新的对象，因此，它对垃圾回收所产生的压力远远超过其他编程方式
- 递归陷阱：在函数式编程中，为了实现迭代，通常会采用递归操作

文献参考：

[函数式编程-维基百科](https://zh.wikipedia.org/wiki/%E5%87%BD%E6%95%B0%E5%BC%8F%E7%BC%96%E7%A8%8B)

[副作用-维基百科](https://zh.wikipedia.org/wiki/%E5%89%AF%E4%BD%9C%E7%94%A8_(%E8%AE%A1%E7%AE%97%E6%9C%BA%E7%A7%91%E5%AD%A6))

[范畴论（数学分支）-百度百科](https://baike.baidu.com/item/%E8%8C%83%E7%95%B4%E8%AE%BA/8281114)

[clojure flavored javascript](https://link.juejin.cn/?target=https%3A%2F%2Foyanglul.us%2Fclojure-flavored-javascript%2Fzh%2F)

[函数式编程初探-阮一峰](https://www.ruanyifeng.com/blog/2012/04/functional_programming.html)

[函数式编程指北](https://llh911001.gitbooks.io/mostly-adequate-guide-chinese/content/)

[函数式编程进阶：杰克船长的黑珍珠号-网易云音乐技术团队](https://juejin.cn/post/6844904034260910094)