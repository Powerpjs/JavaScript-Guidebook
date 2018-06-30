# 理解作用域

> 作用域就是变量（标识符）适用范围，控制着变量的可见性。

《You don‘t know js》对作用域的定义：

> 使用一套严格的规则来分辨哪些标识符对那些语法有访问权限

《JavaScript权威指南》中对变量作用域的描述：

> 一个变量的作用域（scope）是程序源代码中定义这个变量的区域。全局变量拥有全局作用域，在JavaScript代码中的任何地方都是有定义的。然而在函数内声明的变量只在函数体内有定义。它们是局部变量，作用域是局部性的。函数参数也是局部变量，它们只是在函数体内有定义。



## 编译过程中的关键角色

- 引擎：从头到尾负责整个 JavaScript 程序的编译及执行过程。
- 编译器：负责语法分析及代码生成等步骤。
- 负责收集并维护由所有声明的标识符（变量）组成的一系列查询，并实施一套非常严格的规则，确定当前执行的代码对这些标识符的访问权限。

## 编译过程详解

在这里，我们会无数次用到作用域这个词，你完全可以按照之前的理解来阅读。这并不影响我们最终对作用域的理解。

还是 `var a = 2` 这行代码，通过上面的什么是编译部分我们可以知道，编译器首先会将这段代码分解成词法单元，然后将词法单元解构成一个树结构（AST），但是当编译器开始进行代码生成时，它对这段代码的处理方式会和预期的有所不同。

当我们看到这行代码，用伪代码进行跟别人进行概括时，可能会这样去表述：“为一个变量分配内存，并将其命名为 a，然后将值2保存到这个变量（内存）中。” 然而，这并不完全正确。

事实上编译器会进行如下操作：

1. 遇到 `var a`，编译器会询问作用域是否已经有一个该名称的变量存在于同一个作用域的集合中。如果是，编译器会忽略该声明，继续进行编译；否则它会要求作用域在当前作用域的集合中声明一个新的变量，并命名为a。
2. 接下来编译器会为引擎生成运行所需的代码，这些代码被用来处理`a = 2`这个赋值操作。引擎运行时会首先询问作用域，在当前的作用域集合中，是否存在一个叫作a的变量，如果是，引擎就会使用这个变量；如果否，引擎就会继续查找该变量。

总结起来就是：

1. 编译器在作用域声明变量（如果没有）；
2. 引擎在运行这些代码时查找该变量，如果有就进行赋值；

在上面的第二步中，引擎执行“运行时所需的代码”时，会通过查找变量 a 来判断它是否已经声明过。查找的过程由作用域进行协助，但是引擎执行怎么查找，会影响最终的查找结果。

还是 `var a = 2;` 这个例子，引擎会为变量 a 进行 LHS 查询。当然还有一种 RHS 查询。那么 LHS 和 RHS 查询是什么呢？这里的 L 代表左侧，R 代表右侧。通俗且不严谨的解释 LHS 和 RHS 的含义就是：当变量出现在赋值操作的左侧时进行 LHS 查询，出现在右侧时进行 RHS 查询。

那么描述的更准确的一点，*RHS 查询与简单的查找某个变量的值毫无二致，而 LHS 查询则是试图找到变量的容器本身，从而可以对其赋值。从这个角度说，RHS 并不是真正意义上的“赋值操作的右侧”，更准确的说是“非左侧”。所以，我们可以将 RHS 理解成 Retrieve his source value（取到它的源值）*，这意味着，“得到某某的值”。

那我们来看一段代码深入理解一下 LHS 与 RHS。

```js
function foo(a) {
  console.log(a)
}

foo(2)
```

从这段代码中，我们先看看: `console.log(a)`

其中 a 的引用是一个 RHS 引用，因为我们是取到 a 的值。并将这个值传递给 `console.log(…)` 方法。

相比之下，例如： `a = 2 // 调用foo(2)时，隐式的进行了赋值操作` 这里对 a 的引用就是 LHS 引用，因为我们实际上不关心当前的值时什么，只要想把 `=2` 这个赋值操作找到一个目标。 

LHS 和 RHS 的含义是“赋值操作的左侧或右侧”并不一定意味着就是 `=赋值操作符的左侧或右侧`。赋值操作还有其他几种形式，因此在概念上最好将其理解为“赋值操作的目标是谁（LHS）”以及“谁是赋值操作的源头（RHS）”

当然上面的程序并不只有一个 LHS 和 RHS 引用：

```js
function foo(a) {
  // 这里隐式的进行了对形参a的LHS引用。
  
  // 这里对log()方法进行了RHS引用，询问console对象上是否有log()方法。
  // 对log(a)方法内的a进行RHS引用，取到a的值。
  console.log(a)	// 2
}

// 此处调用foo()方法，需要调用对foo的RHS引用。意味着“去找foo这个值，并把它给我。”
foo(2)
```

需要注意的是：我们经常会将函数声明 `function foo(a) {...} ` 转化为普通的变量赋值 `var foo = function(a) {…}`，这样去理解的话，这个函数是 LHS 查询。但是有一个细微的差别，编译器可以在代码生成的同时处理声明和值的定义，比如引擎执行代码时，并不会有线程专门用来将一个函数值“分配给” `foo`，因此，*将函数声明理解成前面讨论的 LHS 查询和赋值的形式并不合适。*

到这里，是否对作用域的工作有了一个理解呢？但是它是什么，还是有些模糊，不知道该怎么去表述。先不管，先看看什么是作用域链。

## 总结

作用域是一套“标识符的查询规则”（注意这里的用的词是规则），根据查找的目的进行 LHS 与 RHS 查询。确定了在何处（当前作用域、上层作用域...全剧作用域）如何查找（LHS、RHS）