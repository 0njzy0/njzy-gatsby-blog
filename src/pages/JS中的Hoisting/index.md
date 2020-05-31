---
title: JS中的Hoisting
date: 2020-05-30
spoiler: 详解JS中的声明提升
---

之前写过一篇博客 [JS中的作用域和闭包](/JS中的作用域和闭包/#5声明提升)，里面简单的介绍过 JS 中的声明提升（Hoisting），但是最近发现 Hoisting 并没有表面看起来那么简单，有很多细节当时也没有介绍清楚，所以这里准备从以下几个问题入手，重新梳理一遍：

1. 何为 Hoisting
2. 为什么需要 Hoisting
3. 变量声明和函数声明提升的区别
4. 同名的变量声明和函数声明谁的优先级高
5. 函数参数会得到声明提升吗
6. let 和 const 存不存在 Hosting
7. 什么是 TDZ
8. Hoisting 发生在什么阶段

## 何为 Hoisting

我们先从以下代码开始：

```js
console.log(a)
```

执行结果为 `ReferenceError: a is not defined`，这里很好理解，因为变量a并不存在

```js
var a = 1
console.log(a)
```

执行结果为：`1`，因为打印 a 时，变量 a 已存在并且值为1

```js
console.log(a)
var a = 1
```

执行结果为：`undefined`，这里可能会有疑问，打印 a 时 a 并不存在啊，不应该报错吗，为啥会显示`undefined`

其实上面的代码可以等价于这个：

```js
var a
console.log(a)
a = 1
```

可见，变量的声明被提升了，这就是我们要说的 Hoisting

除了变量声明会被提升，函数声明也会

```js
foo()
function foo(){
  console.log("foo")
}
```

执行结果为：`foo`

## 为什么需要 Hoisting

至于为啥需要 Hoisting，大家可以看下这篇文章 [Two words about “hoisting”.](http://dmitrysoshnikov.com/notes/note-4-two-words-about-hoisting/)

文章的结论是，使用 Hoisting 有以下两个好处：

1. mutual recursion 可以相互递归
2. optimization 性能提升

这里我们看一下相互递归的例子：

```js
function loop(n) {
  if (n < 10) {
    logEvenOrOdd(++n);
  }
}

function logEvenOrOdd(n) {
  console.log(n, n % 2 ? "Odd" : "Even");
  loop(n);
}

loop(0);
```

可见这里foo和bar进行了相互递归调用，如果不存在 Hoisting，它们之前的相互调用是根本没办法实现的

## 变量声明和函数声明提升的区别

看以下代码：

```js
console.log(a)
var a = 3
foo()
bar()
function foo(){
  console.log("foo")
}
var bar = function() {
  console.log("bar")
}
```

输出结果为：

```bash
undefined
foo
TypeError: bar is not a function
```

由输出结果我们可以得出以下结论：

- 变量和函数声明都会得到提升
- 变量声明提升时不包括赋值
- 函数表达式不存在函数声明提升，仅仅是变量声明提升

## 同名的变量声明和函数声明谁的优先级高

那如果变量名和函数明同名会怎么样，看这个例子：

```js
a();
var a = 1;
console.log(a);
function a() {
  console.log("a1");
}
function a() {
  console.log("a2");
}
```

输出结果为：

```bash
a2
1
```

由输出结果我们可以得出以下结论：

- 函数声明提升优先级比变量声明提升高，会覆盖变量声明提升
- 同名的函数声明提升会被覆盖

## 函数参数会得到声明提升吗

```js
foo(1);
function foo(a) {
  console.log(a);
  var a = 10;
}
```

输出结果为：`1`

在foo(1)执行时，foo内部等价于这个：

```js
function foo() {
  var a = 1
  var a
  console.log(a)
  a = 10
}
```

那么这里不是重复声明了a吗，不应该是undefined吗，我们单独来看一个例子：

```js
var a = 1
var a
console.log(a)
```

输出结果为：`1`，代码等价于这个：

```js
var a
var a
a = 1
console.log(a)
```

## let 和 const 存不存在 Hosting

之前我一直认为用 let 和 const 是不存在 Hoisting 的，直到看到这段代码：

```js
var a = 1;
function foo() {
  console.log(a);
  let a = 10;
}
foo();
```

这段代码的实际执行结果不是 `1`，而是执行报错了：`ReferenceError: Cannot access 'a' before initialization`

可见let生命的变量实际上是得到了提升，只不过表现跟var不同，这也就是接下来要说到的问题: TDZ

## 什么是 TDZ

TDZ全称为：Temporal Dead Zone，一句话概括就是，在提升之后和赋值之前这段时间就被成为TDZ，在这段时间内是无法访问变量的

```js
var a = 1;
function foo() {
  console.log(a); // a的TDZ开始
  let a = 10; // a的TDZ结束
}
foo();
```

我们再看一个例子：

```js
foo();
let b = 10;
function foo() {
  console.log(b);
}
```

这里我们要注意一个点，TDZ跟时间有关，跟空间无关，也就是说虽然b在foo之前被赋值了，但是时间上，foo实际执行时，b还没有被赋值，也就是还处于TDZ时间范围内，所以上面代码执行时会报错：

```bash
ReferenceError: Cannot access 'b' before initialization
```

## Hoisting 发生在什么阶段

那么 Hoisting 是在代码实际执行时进行的吗，我们来看一个例子：

```js
console.log(a)
if (false) {
  var a = 1
}
```

执行结果为：`undefined`，可见虽然if内的代码没有执行，变量a的声明还是被提升到了当前作用域的最顶部

再看一个例子：

```js
function foo() {
  bar();
  function bar() {
    console.log(a);
  }
  if (false) {
    var a = 1;
  }
}

foo();
```

代码执行结果同样为：`undefined`

通过上面两个例子我们基本可以得出一个结论：**声明提升是发生在代码实际运行之前的**

其实这里的运行之前，我们可以理解为编译阶段，因为js引擎在实际执行代码之前确实存在编译阶段，在这个阶段中，我们的代码会被转换成AST（抽象语法树）

## 参考链接

[我知道你懂 hoisting，可是你了解到多深？](https://blog.techbridge.cc/2018/11/10/javascript-hoisting/)

[MDN](https://developer.mozilla.org/en-US/docs/Glossary/Hoisting)
