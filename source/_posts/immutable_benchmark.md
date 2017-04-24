---
title: vanila javascript vs immutable.js
date:  2016-10-14
tags:  immutable.js, benchmark
---

#### 主要内容
- immutable.js和原生js的一些benchmark，这里原生js也包括babel转义后的代码, 比如测试object的spread operator就需要用babel转义

### Array.prototype.push vs List.prototype.push
- 测试代码
``` javascript
suite
   .add("Array.push", function () {
     const tmp = [].concat(arr);
     tmp.push(1);
   })
   .add("List.push", function () {
     arrIm.push(1);
   })
```
- 为了达到immutable push会生成新的数组对象的效果，原生js必须先复制出一个新array，然后再push，所以很明显在这个比对上immutable.list会完胜
- ![benchmark 结果](https://www.skyitachi.cn/oss/pic/benchmark/benchmark-push.png)

### Array.prototype.concat vs List.prototype.push
- 测试代码
```javascript
suite
    .add("Array.concat", function () {
      const arrNext = arr.concat([1]);
    })
    .add("List.push", function () {
      arrIm.push(1);
    });
```
- 由于concat的会生成新数组，功能上来讲和List.push类似, 所以做了点比较, 结果很有意思
![benchmark 结果](https://www.skyitachi.cn/oss/pic/benchmark/benchmark-concat.png)

### Object.assign vs Immutable.Map
> 虽然该测试看上去有点诡异，这里主要是想测试生成一个新对象，对于Immutable.Map来说set操作就可以返回一个新对象, 只测试一层属性原因在于Object本身也是shallow copy, Immutable.Map有setIn这种操作，这是原生Object不是太好模拟的（如果要模拟，性能肯定会比较差，也是先生成一个新对象, 然后在新对象上做一些set，所以只要比较如何生成新对象就能反应问题了）

- 测试代码
```javascript
// obj是一个只有一层属性的对象
const obj = {};
// ...
let imObj = new Immutable.Map(obj);
suite
   .add("Object.assign", function () {
     const nextObj = Object.assign({}, obj);
   })
   .add("Immuatble.Map", function () {
     let nextObj = imObj.set(keys[0], obj[keys[0]])
   }
```
- 我个人感觉也是Immutable生成新对象的方式效率更高，当然还不了解其实现原理，只能做一些benchmark

 ![benchmark 结果](https://www.skyitachi.cn/oss/pic/benchmark/object_and_assign_benchmark.png)

### Object.assign vs Object Spread Operator
> 由于Object Spread Operator只是Object.assign的语法糖, 两者性能基本接近

- 测试代码
```javascript
const obj = {};
// 生成len长度的属性
for(let i = 0; i < len; i++) {
  const p1 = Math.random().toString().substr(2, 10);   
  obj[p1] = Math.random();
}
const suite = new Benchmark.Suite();
suite
  .add("Object.assign", function () {
    const nextObj = Object.assign({}, obj);
  })
  .add("Object Spread Opeartor", function () {
    const nextObj = {
      ...obj
    };
  })
```
![benchmark结果](https://www.skyitachi.cn/oss/pic/benchmark/object_and_spread.png)

### 总结
- 这里的性能测试还不是很完善，有很多场景没有覆盖到，主要想覆盖的场景是**根据已有对象生成新对象**, 也就是在redux中主要使用的场景
