---
title: 安全地删除require.cache
date:  2017-04-11
tags:  Node.js, memory, profiling, require, Module
---

#### 前言

> 大家都知道Node.js中`require`是有缓存的，很多人在通过删除cache已达到`热更新`的目的，笔者在自己实现的基于react和koa的同构渲染插件时，采用了这种方式热更新编译好的模板文件，一开始都很美好，但随着watch的时间变长，我发现web app出现gc的频率越来越高直至崩溃，于是开始了接下来的追踪。
>
> 本文主要探讨的是直接删除require.cache[filename]导致的内存泄漏问题

#### 如何发现有内存泄漏

<del>全靠崩溃log</del>

一般来讲使用`process.memoryUsage()`就能够发现一些问题, 笔者在一开始就是利用该方法发现问题的，你也可以使用一些可视化工具，不过这些不是本文重点
> ps: memory增长不一定意味着内存泄漏，因为v8是有gc的，需要确认经过gc后内存仍然没有得到释放，才更据有说服力
> 本文中使用node的--expose-gc，暴露出global.gc方法手动gc

#### 如何memory profiling

这里笔者使用`node-heapdump`输出内存快照，然后加载到`chrome-dev-tools`中，主要还是基于chrome的dev-tool来做分析

#### 实验的场景

```bash
├── index.bundle.js #重复require的模块
├── leak.js #重现场景的代码
```

- 产生内存泄漏

```javascript
function leak(dpath) {
  for(let i = 0; i < 100; i++){ // 重复require 100次(已经能明显的观察到内存问题了)
    const test = require(dpath);
    const libPath = require.resolve(dpath);
    // const children = module.children;
    // const depends = children.filter(mod => {
    //   return mod.filename !== libPath;
    // });
    // module.children = depends;
    delete require.cache[libPath];
  }
}

// before leak
heapdump.writeSnapshot("./test-no-leak.heapsnapshot");
leak("./index.bundle.js");
// after leak
heapdump.writeSnapshot("./test-leak.heapsnapshot");
```

- 使用chrome-dev-tool查看内存快照

  - 初步印象

    ![触目惊心](https://www.skyitachi.cn/oss/pic/memory_leak/first.png)

  - 深入分析

    ![内存对象概览](https://www.skyitachi.cn/oss/pic/memory_leak/no_leak_constructor.png)

  - 然后。。。<del>这么多对象，我怎么知道哪个泄漏了</del>

    理性分析：上图是按constructor来分析对象的 ，基于我们的场景，**显然**，require导致的泄漏，很有可能是`Module`（Node.js 用于构造模块的constructor，不是本文重点，先忽略），那么接下来的重点就是会在Module下面

  - leak前和leak后Module对象的对比
    - before leak
    ![before_leak](https://www.skyitachi.cn/oss/pic/memory_leak/module_before_leak.png)

    - after leak
    ![after_leak](https://www.skyitachi.cn/oss/pic/memory_leak/module_leak.png)

  - 很明显，内存泄漏的罪魁祸首就是这些未被释放掉的Module对象，虽然在require.cache中删除了，这些Module仍然存在，说明在其他地方仍然有所引用，那么到底哪里有引用了，接下来看下详细的Module对象

    ![final](https://www.skyitachi.cn/oss/pic/memory_leak/final.png)
  - 上图说明，Module对象会存在与某个Moudle的.children中，**结合require的原理来看，依赖会被加入到当前模块的children中**（这也不是本文重点），于是问题得解

    ```javascript
    function leak(dpath) {
      for(let i = 0; i < 100; i++){ // 重复require 100次(已经能明显的观察到内存问题了)
        const test = require(dpath);
        const libPath = require.resolve(dpath);
        const children = module.children;
        const depends = children.filter(mod => {
          return mod.filename !== libPath;
        });
        module.children = depends;  // 将当前模块的children中依赖删除掉
        delete require.cache[libPath];
      }
    }
    ```

  - 解决之后的heap中的Module对象分布
  ![safe](https://www.skyitachi.cn/oss/pic/memory_leak/safe.png)

   终于得解

#### 总结
- 定位一个内存泄漏问题最主要的还是要善于利用工具，从发现问题，确认问题，分析问题，解决问题，所用工具都不太一样，而且也有多种解决方法。
- 需要从原理层面了解问题，如果了解require的原理，很快就能定位出Module对象的泄漏导致的问题，解决方案也很简单，
当然也可通过heapdump的对比发现一些问题

#### 参考资料
- [关于require.cache的讨论](https://github.com/nodejs/node-v0.x-archive/issues/6176)，关闭该issue的理由值得细读
- [find js memory leak](https://www.alexkras.com/simple-guide-to-finding-a-javascript-memory-leak-in-node-js/), 很好的get start
- 还有就是node的module源码
