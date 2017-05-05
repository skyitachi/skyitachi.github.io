---
title: 浅谈 javascript 作用域
date:  2017-04-24
tags:  javascript
---

#### 前言
- 本文将主要介绍javascript中作用域相关的问题，尽可能多的使用代码举例说明，尽量少涉及动态作用域相关

#### 词法作用域(核心)
> javascript的作用域是词法作用域（静态作用域）, 不过像`eval`，`with`这些具有动态改变作用域的能力, 本文重点在于词法作用域

```javascript
let a = 1;

function testLexicalScope() {
  console.log(a); // 由于当前scope中a是由最外层定义的，所以此处的a只能访问到最外层的a
}

function b() {
  let a = 2;
  testLexicalScope();
}

b();

// 另一个例子
let sameVar1 = 1;
let sameVar2 = 1;

function outerScope() {
  let sameVar1 = 2;
  function innerScope() {
    console.log("current scope: sameVar1 is ", sameVar1); // 当前的scope中最近的sameVar1值是2
    console.log("current scope: sameVar2 is ", sameVar2);
  }
  innerScope();
}

outerScope();
// current scope: sameVar1 is  2
// current scope: sameVar2 is  1

//另一个例子
const f1 = function () {
  console.log(outVar);
};

let outVar = 1;
f1(); // 1 why

```
> 只有在函数声明的时候才会遵循lexical scope的规则, 如果是函数表达式则取决于调用的时机

#### 不同类型的作用域(如何创建scope)
- 函数作用域

> 属于这个函数的全部变量都可以在整个函数的范围内使用及复用
> javascript 每个函数都会创建一个scope

```javascript
function fScope() {
  var aStr = "function";
  console.log(aStr);
}

fScope();
console.log(aStr); // ReferenceError: aStr is not defined

```

- 块级作用域({...})

```javascript
var foo = true;
{ // 这里是使用let，将foo绑定到了{}这个块作用域中
  let foo = false;
  let bar = "cannot seen";
  console.log(foo); // false
}
console.log(foo); // true
console.log(bar); // ReferenceError
```

#### `var`
- 函数作用域中的`var`仍然遵循函数作用域相关的
- `var`中没有块级作用域

```javascript
var foo = true;
{
  var foo = false;
  var bar = true;
}
console.log(foo); // false
console.log(bar); // bar
```

#### 变量提升
> 变量和函数的所有声明多会在任何代码被执行前首先被处理（编译器找到这些变量与合适的作用域关联）

```javascript
function hoisting() {
  console.log(a);
  var a = 1;
  console.log(a);
}
hoisting();

```
**又是let**

```javascript
function hoisting() {
    console.log(a); // 第一个let之上的区域叫做`temporal dead zone`
    let a = 1;
    console.log(a);
}
hoisting(); // ReferenceError
```
**函数声明的优先级会高于变量声明**

```javascript
foo(); // in the function

var foo = 1;

function foo() {
  console.log("in the function");
}
```

#### 总结
- 关于块级作用域, 可用try{} catch(err) {/*这里是块级作用域*/}模拟，更多参考:《你不知道的javascript》上卷中3.4.2节
- 使用`let`，`const`是最佳实践
- 需要区分函数声明和函数表达式
- 尽量不要写有提升的代码（声明尽量提前）
