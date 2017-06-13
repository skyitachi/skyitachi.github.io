---
title: 浅谈 this
date:  2017-06-13
tags:  javascript
---

#### 本文宗旨
- 说明`this`在各种上下文中是什么
- 介绍改变`this`的方法
> 本文代码均是在node.js v8.0.0上实验的, v8 version: 5.8.283.41, 本文也不考虑eval，with的动态函数, 同样在也不会考虑vm.runInNewContext等相关函数
> 所有代码均在**非严格模式**下运行, 一般来讲严格模式会影响隐式绑定

#### 各种情况下的this
- 全局直接使用this
```javascript
// index.js
console.log(this);
// {}, this === module.exports
```
> 在nodejs环境下，单文件top-level的this指向的是module.exports, 而浏览器中指向的是window对象

- function中使用`this`
```javascript
// 函数声明
function foo() {
	console.log(this === global);
}
foo();
// true
// 函数表达式
const bar = function () {
	console.log(this === global);
};
bar();
// true
```
> 在这种调用方式下，this就是nodejs的top-level的global对象

- 构造函数中的this
```javascript
function Foo() {
  console.log(this);
}
const f = new Foo();
// Foo {}
```
> 调用new时会创建一个新对象，并把新对象绑定到相关调用函数上

- Object中使用`this`
```javascript
const obj = {
  foo: function () {
    console.log(this);
  }
};
obj.foo();
// { foo: [Function: foo] }
```
> 一般而言，在调用object的属性方法时，this指向的是object本身

#### 如何改变`this`
> javascript提供了bind，call，apply等机制方便改变函数的上下文（this）

- `apply` and `call`
```javascript
const sandbox = { name: "sandbox" };
function foo() {
	console.log(this);
}
foo.apply(sandbox);
// { name: 'sandbox' }
foo.apply(null);
// global object
foo.call(sandbox);
foo.call(null);
// same as above
```

> Function.prototype.apply(null), 并不是将this改成null， 而是不改变原有this的意思, 同理call也是这样

- `bind`
```javascript
const sandbox = { name: "sandbox" };
const sandbox2 = { name: "anthoer sandbox" };
function foo() {
	console.log(this);
}
foo.bind(sandbox)();
// { name: 'sandbox' }
foo.bind(null)();
// global object
foo.bind(sandbox).bind(sandbox2)();
// { name: 'sandbox' }
foo.bind(null).bind(sandbox)();
// 仍然是global objet
foo.bind(sandbox).call(sandbox2);
foo.bind(sandbox).apply(sandbox2);
// { name: 'sandbox' }
```
> 可以见在bind(null)的情况下，和apply，call的情况一致。bind只会起作用一次，即第一次调用的情况下。与call和apply一起用的时候也是如此。

- arrow function
> es2015中提供了arrow function方便bind当前this
```javascript
const foo = () => {
	console.log(this === module.exports);
};
foo();
// true

const sandbox = { name: "sandbox" };
const obj = {
  foo: () => {
    console.log(this === module.exports);
  }
};
obj.foo();
// true
obj.foo.bind(sandbox)();
// true
```
> arrow function绑定了当前scope中this的值，同时这种绑定也不能改变，类似bind

#### 总结
- 一般来讲this的绑定分为显式和隐式
  - apply，call，bind和new一般称为显式, 其他称为隐式
- 值得注意的是bind的行为，`this`也不是想改就能改的
- 个人推荐尽量使用arrow function，避免使用直接使用this（纯粹个人偏好）

> 本文是笔者在看《You don't know js》上卷中的this章节的总结，尽量从实用的角度出发，舍弃了this绑定优先级的介绍，事实上书上讲了很多语言细节和很多特殊的case，喜欢研究细节的同学可以看书的对应章节。同时本文也是笔者在看完该章节之后过了2个月左右之后写的，主要是看哪些知识给我留下了深刻的印象。
