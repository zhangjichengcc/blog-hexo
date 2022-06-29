---
title: JavaScript中的执行上下文和执行栈
date: 2022-06-29 11:48:25
tags: js基础
banner_img:
index_img:
categories:
---

<!-- more -->

## 什么是执行上下文

> 执行上下文是评估和执行 JavaScript 代码的环境的抽象概念。每当 Javascript 代码在运行的时候，它都是在执行上下文中运行。

## 执行上下文的类型

- **全局执行上下文（Global Execution Context 简称-GEC）**  

  这是默认或者说基础的上下文，任何不在函数内部的代码都在全局上下文中。它会执行两件事：
  1. 创建一个全局的 `window` 对象（浏览器的情况下）;
  2. 将 `this` 指向这个全局对象。 一个程序中只会有一个全局执行上下文。

  GEC存储内容：

  1. **全局对象（Global Object 简称-GO）**：这里面存储的是全局中声明的所有变量声明和函数声明。但是在 GO（全局对象） 中存储的变量声明与函数声明的存储形式，又是截然不同：

      - **变量声明**：在 GO（全局对象）中存储的变量声明，在声明的时候，全部声明为 undefined，在真正执行的时候，才会给每一个变量进行赋值操作。

      - **函数声明**：在 GO（全局对象）中存储的函数声明，在声明的时候，就已经为函数创造了其内存空间，在函数声明中引入的则是该内存空间的地址，该函数的内部代码将会存储在一个独立的内存空间当中。

  2. **执行代码**：就是在全局中将要执行的所有代码

- **函数执行上下文（Function Execution Context 简称-FEC）**  

  每当一个函数被调用时, 都会为该函数创建一个新的上下文。每个函数都有它自己的执行上下文，不过是在函数被调用时创建的。**函数上下文(FEC)** 可以有任意多个。每当一个新的执行上下文被创建，它会按定义的顺序（将在后文讨论）执行一系列步骤。

  FEC存储内容：
  
  1. **活动对象（Activation Object 简称-AO）**：这里面存储的是函数中声明的所有变量声明和函数声明。但是在 AO（活动对象） 中存储的变量声明与函数声明的存储形式，又是截然不同：

      - **变量声明**：在 AO（活动对象）中存储的变量声明，在声明的时候，全部声明为 undefined，在真正执行的时候，才会给每一个变量进行赋值操作。

      - **函数声明**：在 AO（活动对象）中存储的函数声明，在声明的时候，就已经为函数创造了其内存空间，在函数声明中引入的则是该内存空间的地址，该函数的内部代码将会存储在一个独立的内存空间当中。

  2. 执行代码：就是在全局中将要执行的所有代码

- **Eval 函数执行上下文**
  这个是比较特殊的，就是运行在 `Eval` 函数内会创建一个 `Eval` 函数执行上下文。

在不同的执行上下文中所存储的内容不尽相同，但是每一个执行上下文，都会存在一个 **变量对象（Variable Object 简称-VO）**，这个变量对象将会存储当前执行函数的变量声明等内容。

## 执行栈（Execution Context Stack 简称-ECS）

> 执行栈，也就是在其它编程语言中所说的“调用栈”，是一种拥有 **LIFO（后进先出）数据结构的栈**，被用来存储代码运行时创建的所有执行上下文。

当 JavaScript 引擎第一次遇到你的脚本时，它会创建一个 **全局执行上下文（Global Execution Context 简称-GEC）** 并且压入当前执行栈。每当引擎遇到一个函数调用，它会为该函数创建一个新的 **执行上下文** 并压入栈的顶部。

引擎会执行那些执行上下文位于栈顶的函数。当该函数执行结束时，执行上下文从栈中弹出，控制流程到达当前栈中的下一个上下文。

让我们通过下面的代码示例来理解：

``` javascript
let a = 'Hello World!';

function first() {
  console.log('Inside first function');
  second();
  console.log('Again inside first function');
}

function second() {
  console.log('Inside second function');
}

first();
console.log('Inside Global Execution Context');
```

![函数执行上下文](/images/posts/20220629_JavaScript中的执行上下文和执行栈/20220629_函数执行上下文.png)

当上述代码在浏览器加载时，JavaScript 引擎创建了一个全局执行上下文并把它压入当前执行栈。当遇到 `first()` 函数调用时，JavaScript 引擎为该函数创建一个新的执行上下文并把它压入当前执行栈的顶部。

当从 `first()` 函数内部调用 `second()` 函数时，JavaScript 引擎为 `second()` 函数创建了一个新的执行上下文并把它压入当前执行栈的顶部。当 `second()` 函数执行完毕，它的执行上下文会从当前栈弹出，并且控制流程到达下一个执行上下文，即 `first()` 函数的执行上下文。

当 `first()` 执行完毕，它的执行上下文从栈弹出，控制流程到达全局执行上下文。一旦所有代码执行完毕，JavaScript 引擎从当前栈中移除全局执行上下文。

## 怎么创建执行上下文？

创建执行上下文有两个阶段：**1) 创建阶段** 和 **2) 执行阶段**。

### 1. 创建执行上下文

1. this 值的决定，即我们所熟知的 This 绑定。

2. 创建词法环境组件。

3. 创建变量环境组件。

所以执行上下文在概念上表示如下：

``` bash
# 执行上下文
ExecutionContext = {
  ThisBinding = <this value>,
  # 词法环境
  LexicalEnvironment = { ... },
  # 变量环境
  VariableEnvironment = { ... },
}
```

#### 1. this 绑定

在全局执行上下文中，`this` 的值指向全局对象。(在浏览器中，`this` 引用  `Window` 对象)。

在函数执行上下文中，`this` 的值取决于该函数是如何被调用的。如果它被一个**引用对象**调用，那么 `this` 会被设置成那个对象，否则 `this` 的值被设置为**全局对象**或者 **undefined（在严格模式下）**。例如：

``` javascript
let foo = {
  baz: function() {
    console.log(this);
  }
}

foo.baz();   // 'this' 引用 'foo', 因为 'baz' 被对象 'foo' 调用

let bar = foo.baz;

bar();       // 'this' 指向全局 window 对象，因为没有指定引用对象
```

#### 2. 词法环境

官方的 [ES6 文档](https://262.ecma-international.org/6.0/) 把词法环境定义为：

> **词法环境** 是一种规范类型，基于 ECMAScript 代码的词法嵌套结构来定义标识符和具体变量和函数的关联。一个词法环境由 **环境记录器** 和一个可能的 **引用外部词法环境** 的空值组成。

简单来说词法环境是一种持有 **标识符—变量** 映射的结构。（这里的标识符指的是 **变量/函数的名字**，而变量是对 **实际对象【包含函数类型对象】或原始数据的引用）**。

现在，在词法环境的内部有两个组件：**(1) 环境记录器** 和 **(2) 一个外部环境的引用**。

1. 环境记录器是存储变量和函数声明的实际位置。
2. 外部环境的引用意味着它可以访问其父级 **词法环境（作用域）**。

**词法环境** 有两种类型：

- **全局环境**（在全局执行上下文中）是没有外部环境引用的词法环境。全局环境的外部环境引用是 null。它拥有内建的 Object/Array/等、在环境记录器内的原型函数（关联全局对象，比如 window 对象）还有任何用户定义的全局变量，并且 this的值指向全局对象。
- 在 **函数环境** 中，函数内部用户定义的变量存储在环境记录器中。并且引用的外部环境可能是全局环境，或者任何包含此内部函数的外部函数。

**环境记录器** 也有两种类型：

- **声明式环境记录器** 存储变量、函数和参数。
- **对象环境记录器** 用来定义出现在全局上下文中的变量和函数的关系。

简而言之，

- 在全局环境中，环境记录器是对象环境记录器。
- 在函数环境中，环境记录器是声明式环境记录器。

**注意** — 对于函数环境，声明式环境记录器还包含了一个传递给函数的 `arguments` 对象（此对象存储索引和参数的映射）和传递给函数的参数的 `length`。

抽象地讲，词法环境在伪代码中看起来像这样：

``` bash
# 全局执行上下文
GlobalExectionContext = {
  # 词法环境
  LexicalEnvironment: {
    # 环境记录器
    EnvironmentRecord: {
      Type: "Object",
      # 在这里绑定标识符
    }
    outer: <null>
  }
}

# 方法执行上下文
FunctionExectionContext = {
  # 词法环境
  LexicalEnvironment: {
    # 环境记录器
    EnvironmentRecord: {
      Type: "Declarative",
      # 在这里绑定标识符
    }
    outer: <Global or outer function environment reference>
  }
}
```

#### 3. 变量环境

它同样是一个词法环境，其环境记录器持有 **变量声明语句** 在执行上下文中创建的绑定关系。

如上所述，变量环境也是一个词法环境，所以它有着上面定义的词法环境的所有属性。

在 ES6 中，词法环境组件和变量环境的一个不同就是前者被用来存储函数声明和变量（let 和 const）绑定，而后者只用来存储 var 变量绑定。

我们看点样例代码来理解上面的概念：

``` javascript
let a = 20;
const b = 30;
var c;

function multiply(e, f) {
 var g = 20;
 return e * f * g;
}

c = multiply(20, 30);
```

执行上下文看起来像这样：

``` bash
# 全局执行上下文
GlobalExectionContext = {

  ThisBinding: <Global Object>,

  # 词法环境
  LexicalEnvironment: {
    # 环境记录器
    EnvironmentRecord: {
      Type: "Object",
      # 在这里绑定标识符
      a: < uninitialized >,
      b: < uninitialized >,
      multiply: < func >
    }
    outer: <null>
  },

  # 变量环境
  VariableEnvironment: {
    # 环境记录器
    EnvironmentRecord: {
      Type: "Object",
      # 在这里绑定标识符
      c: undefined,
    }
    outer: <null>
  }
}

# 方法执行上下文
FunctionExectionContext = {
  ThisBinding: <Global Object>,

  # 词法环境
  LexicalEnvironment: {
    # 环境记录器
    EnvironmentRecord: {
      Type: "Declarative",
      # 在这里绑定标识符
      Arguments: {0: 20, 1: 30, length: 2},
    },
    outer: <GlobalLexicalEnvironment>
  },
  # 变量环境
  VariableEnvironment: {
    # 环境记录器
    EnvironmentRecord: {
      Type: "Declarative",
      # 在这里绑定标识符
      g: undefined
    },
    outer: <GlobalLexicalEnvironment>
  }
}
```

**注意** — 只有遇到调用函数 `multiply` 时，函数执行上下文才会被创建。

这是因为在创建阶段时，引擎检查代码找出变量和函数声明，虽然函数声明完全存储在环境中，但是变量最初设置为 `undefined`（`var` 情况下），或者未初始化（`let` 和 `const` 情况下）。

这就是为什么你可以在声明之前访问 `var` 定义的变量（虽然是 `undefined`），但是在声明之前访问 `let` 和 `const` 的变量会得到一个引用错误。

这就是我们说的变量声明提升。

### 2. 执行阶段

> 在此阶段，完成对所有这些变量的分配，最后执行代码。

**注意** — 在执行阶段，如果 JavaScript 引擎不能在源码中声明的实际位置找到 `let` 变量的值，它会被赋值为 `undefined`。
