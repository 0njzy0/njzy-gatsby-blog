---
title: JavaScript中的作用域和闭包
date: '2019-08-10'
spoiler: JavaScript中的作用域和闭包
---

> 所有内容都是根据<< You Don't Know JS >>整理而来

## 作用域

* 作用域就是一套规则，通过这些规则JavaScript引擎可以很方便的找到需要的变量
* 作用域是可以嵌套的，嵌套的作用域又称为作用域链，在当前作用域无法找到某个变量时，js引擎就会在外层嵌套的作用域中继续查找，直到找到该变量或抵达最外层的作用域（也就是全局作用域）为止
* 作用域共有两种主要的工作模型：词法作用域和动态作用域。JavaScript采用的为词法作用域。简单地说，词法作用域是由你在写代码时将变量和块作用域写在哪里来决定的，因此当词法分析器处理代码时会保持作用域不变（排除一些欺骗词法作用域的方法是这样的）
* 除全局作用域外，JavaScript还存在函数作用域和块作用域。两者的行为表现是一样的，任何声明在某个作用域内的变量，都将附属于这个作用域
* 变量声明和函数声明会在当前的作用域中得到提升
* 函数声明和变量声明都会被提升，但是函数会首先被提升，其次是变量。如果遇到重复声明，后出现的声明如果为变量声明则被忽略，为函数声明则覆盖前者

### 1. LHS查询和RHS查询

js引擎会为需要的变量进行**LHS**查询和**RHS**查询。当要找到某个变量并为其执行赋值操作时则进行LHS查询，只是简单的获取某个变量则进行RHS查询

```javascript
function foo(b) {
  console.log(b)
}
var a = 2
foo(a)
```

上述代码的LHS查询有2次，RHS查询有4次。

LHS查询：

1. 执行var a = 2时，会对变量a进行赋值操作
2. 调用foo()时有一步隐含的赋值操作b=2

RHS查询：

1. 调用foo()时对foo有一次RHS查询
2. 调用foo()时要传递a参数，对a有一次RHS查询
3. 执行foo时会对console对象进行一次RHS查询
4. 执行console.log操作时要获取b

### 2. 查询异常

在变量还没有声明（在任何作用域中都无法找到该变量）时，进行LHS查询或RHS查询的行为是不同的

```javascript
function foo(b) {
  console.log(b + c)
}
var a = 2
foo(a)
```

执行上述代码，js引擎会抛出**ReferenceError**异常，这是因为在对变量c进行RHS查询时是无法找到该变量的

相较之下，当js引擎执行LHS查询时，如果在顶层（全局作用域）中也无法找到目标变量，就会在全局作用域中创建一个具有该名称的变量。如果RHS查询找到了一个变量但为其值进行了不合理的操作，那么js引擎会抛出**TypeError**异常

### 3.词法作用域

词法作用域是JavaScript采用的作用域工作模型

词法作用域就是定义在词法阶段（编译器的第一个工作阶段）的作用域。换句话说，词法作用域是由你在写代码时将变量和块作用域写在哪里决定的，因此当词法分析器处理代码时会保持词法作用域不变（排除一些欺骗词法作用域的方法是这样的）

见以下代码：

```javascript
function foo(a) {
  var b = a * 2
  function bar(c) {
    console.log(a,b,c)
  }
  bar(b*3)
}
foo(2)
```

这段代码有三个逐级嵌套的作用域。这里为了方便起见，我们称他们为作用域气泡，以下就是这三个气泡：

1. 包含着整个全局作用域，其中只有一个标识符：foo
2. 包含着foo所创建的作用域，其中有三个标识符：a、bar和b
3. 包含着bar所创建的作用域，其中只有一个标识符：c

当js引擎执行以上代码时，会按照作用域查询规则，由内到外查询，直到遇见第一个匹配的标识符为止。并且只会查找一级标识符。如果代码引用了foo.bar.baz 词法作用域只会试图找到foo标识符，找到这个变量后，对象属性访问原则会分别接管对bar和baz属性的访问

这里注意一点：**无论函数在哪里被调用，也无论它如何被调用，它的词法作用域都只由函数被声明时所处的位置决定**

### 4.函数作用域和块作用域

上面介绍的是JavaScript的作用域工作模型，下面介绍js作用域的几种类型。除全局作用域外，js中还存在函数作用域和块作用域

#### 4-1.函数作用域

函数作用域的含义指属于这个函数的全部变量都可以在整个函数的范围内使用及其复用，但无法在外部作用域中对函数内部的变量进行访问

```javascript
function foo() {
  var a = 2
  function bar() {
    var b = 3
    console.log(a + b)
  }
  bar()
}
foo()
```

在上面的代码片段中，函数foo创建了一个作用域气泡，其中包含了标识符a和bar。函数bar也创建了一个作用域气泡，其中包含了标识符b。全局作用域也有自己的作用域气泡，其中包含了标识符foo

由于标识符a、bar属于foo作用域气泡，所以无法从foo的外部（全局作用域）对他们进行访问。标识符b属于bar作用域气泡，所以在foo中也无法直接进行访问。但根据作用域嵌套查询规则，bar气泡嵌套在foo气泡中，所以可以在bar中访问a

#### 4-2.块作用域

虽然函数作用域是最常见的作用域单元，但其他类型的作用域单元也是存在的，并且通过其他类型的作用域单元甚至可以实现维护起来更加优秀、简洁的代码

```javascript
for(var i = 0;i<6;i++) {
  console.log(i)
}
console.log(i)
```

如果我们想让上述代码片段中的变量i只存在于for循环中，也就是说在全局作用域中无法对i进行访问，那么就需要块作用域

块作用域最常见的创建方式就是ES6中新引入的let关键字。出了let，我们还可以使用const、with、try/catch来创建，这里不做讨论

上述代码可以用let来改写：

```javascript
for(let i = 0;i<6;i++) {
  console.log(i)
}
```

这里注意一点：**for循环头部的let不仅将变量i绑定到了for循环的块中，事实上它将其重新绑定到了循环的每一个迭代中，确保使用上一个循环迭代结束时的值重新进行赋值**

下面是一段经典的代码片段：

```javascript
for(var i = 0;i<6;i++) {
  setTimeout(function(){console.log(i)},1000)
}
```

结果打印6个6

```javascript
for(let i = 0;i<6;i++) {
  setTimeout(function(){console.log(i)},1000)
}
```

结果打印0 1 2 3 4 5

### 5.声明提升

作用域同其中的变量声明出现的位置是存在某种联系的。看下面代码片段：

```javascript
console.log(a)
var a = 2
```

打印undefined 并不会由于对a进行RHS查询发现不存在报ReferenceError异常

上面的代码其实等同于下面：

```javascript
var a = undefined
console.log(a)
a = 2
```

由此可见**声明本身会被提升，而赋值或其他运行逻辑会保留在原地**

再看下面一段代码：

```javascript
foo()
function foo() {
  console.log(a)
  var a = 2
}
```

打印undefined 由此可见foo函数的声明被提升了。另外要注意的是，**每个作用域都会进行提升操作**。上述代码中的变量a的声明就是在所属作用域foo函数中进行提升的

**函数声明会被提升，但函数表达式（区分函数声明和表达式最简单的方法就是看function关键字出现在整个声明中的位置，如果是第一个词就是函数声明，否则就是一个函数表达式）不会**。代码如下：

```javascript
foo()
var foo = function bar() {
  console.log(a)
  var a = 2
}
```

报TypeError异常和ReferenceError异常

报TypeError异常是因为变量foo声明会被提升 此时foo为undefined 当对foo()进行RHS查询时会导致非法操作，所以报TypeError异常而不是ReferenceError

报ReferenceError异常是因为**函数表达式不会被提升，即使是具名的函数表达式**。所以对bar()进行RHS查询时是找不到标识符bar的，所以报ReferenceError

还有一点需要注意的是：**函数声明和变量声明都会被提升，但是函数会首先被提升，其次是变量。如果遇到重复声明，后出现的声明如果为变量声明则被忽略，为函数声明则覆盖前者**。可见下面代码：

```javascript
foo()
var foo = 2
function foo() {
  var a = 1
  console.log(a)
}
function foo() {
  var a = 100
  console.log(a)
}
```

打印100 var foo会被忽略 第二个foo的函数声明会覆盖第一个

## 闭包

* 闭包是基于词法作用域书写代码时所产生的自然结果
* 当函数可以记住并访问所在的词法作用域时就产生了闭包，即使函数是在当前词法作用域之外执行

以下代码清晰的展示了闭包：

```javascript
function foo() {
  var a = 2
  function bar() {
    console.log(a)
  }
  return bar
}
var baz = foo()
baz()
```

打印2 当foo执行以后并没有被垃圾回收机制处理，这是因为闭包阻止了。因为bar声明的位置，它拥有foo内部作用域的闭包，使得该作用域能够一直存活，以供bar在之后任何时间进行引用

无论以何种方式对函数类型的值进行传递，当函数在别处被调用时都可以观察到闭包：

```javascript
function foo() {
  var a = 2
  function bar() {
    console.log(a)
  }
  baz(bar)
}
function baz(fn) {
  fn()
}
foo()
```

把内部函数bar传递给baz，当调用这个内部函数时（现在叫fn），它涵盖的foo内部作用域的闭包就可以观察到了，因为它能够访问a

接下来我们看一段常见的代码：

```javascript
function foo(message) {
  setTimeout(function(){console.log(message)},1000)
}
foo('Hello World!')
```

打印Hello World！

下面来解释为什么能打印Hello World：

1. foo函数创建了一个函数作用域
2. foo作用域内部存在一个传递给setTimout的函数，我们称他为回调函数
3. 因为回调函数声明在foo内部作用域中，所以它能狗访问foo内部的标识符message，拥有了对foo内部作用域的闭包
4. 当执行foo后，闭包阻止了foo被垃圾回收机制处理，所以回调函数可以正常运行并访问标识符message

闭包在循环中也很常见，见以下代码：

```javascript
for(var i = 0;i<6;i++) {
  setTimeout(function(){console.log(i)},1000)
}
```

前面说过会打印6个6，并介绍了可以利用let来创建块作用域让它打印0 1 2 3 4 5。这里我们使用函数作用域和闭包来解决此问题：

```javascript
for(var i = 0;i<6;i++) {
  (function(i){
    setTimeout(function(){console.log(i)},1000)
  })(i)
}
```

我们在回过头看用let创建块作用域的代码：

```javascript
for(let i = 0;i<6;i++) {
  setTimeout(function(){console.log(i)},1000)
}
```

在仔细看一下，其实这就是利用块作用域和闭包共同解决的
