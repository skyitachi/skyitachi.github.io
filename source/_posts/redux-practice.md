---
title: react-redux-practice
date: 2016-08-18 11:35:09
tags: react, redux, best_practice
---

### 前言
  - 这里主要讲解redux和react-redux的一些实践。
  - ps:
    - 这里所讲的redux都是结合react使用的（react-redux)，container指的是有业务逻辑的的react-component（使用connect与redux store结合），ui component指的是纯ui组件（可能是针对业务的ui组件）

### 一些实践
  - 使用smart container处理几乎所有业务逻辑
    - 尽量使container的props包含所有的业务逻辑和ui state，少在container这一层使用setState
      - container一般都通过dispatch action来达到更新ui的目的，同时container级别的更新代价是相对大的，如果在一个event handle里同时使用setState和dispatch那么就会有两次更新，很明显setState的更新是可以通过将state放入props来省去的
    - ui component尽可能都是不涉及业务逻辑的，利用props和container通信
  - api数据加工应该怎么做
    - 一般而言fetch api放在action里面做，那么在action里需要做一些数据转化的东西其实也很顺其自然，不过redux有了reducer的概念以后，这些数据加工的东西放到reducer里做会好一点，毕竟reducer从语义上就是干这些活的。
    - 数据加工放到reducer里可以减轻action的逻辑
      - action几乎只负责api返回的数据结构
    - 数据加工的方法我们一般使用Array.prototype.forEach, map, filter, reduce，之所以没有用lodash这样的功能库，是因为这些方法加上es6的spread operator可以满足大部分需要

  - 通信问题
    - container间的通信
      - 主要是利用redux的single store来实现，比如containerA会update redux state中的某个变量，那么只要containerB connect这个变量到props，然后在wilReceiveProps做一些简单的判断就能捕获到这个更新了，表现上来看很像watch的功能，不过这里不需要像事件一样关注subscribe和publish的顺序, 如下图所示
        ![redux-communication](https://www.skyitachi.cn/oss/pic/reduex-container-communication.png)

      - 不过redux中实现类似watch的功能没有mvvm中的那么简洁
    - ui-component之间的通信
      - 利用action走父级的container的通信逻辑，相当于
        ![component-communication](https://www.skyitachi.cn/oss/pic/redux-component-communication.png)

  - boilerplate太多怎么办
    - 如果你讨厌在props中传递dispatch，那么使用bindActionCreator和mapDispatchToProps会解决这个问题，同时也减少了部分的boilerplate
	- 使用脚手架
  - 文件目录结构
    - 按照页面来组织action，container，reducer
      ```bash
      src
      ├── actions
      │   ├── home.js
      │   └── page1.js
      ├── components   # ui-components
      │   └── header
      │       ├── index.js
      │       └── index.less
      ├── containers  
      │   ├── home
      │   │   ├── index.js
      │   │   └── index.less
      │   └── page1
      │       ├── index.js
      │       └── index.less
      └── reducers
          ├── home
          │   └── index.js
          ├── index.js
          └── page1
              └── index.js
      ```
  - 针对单页应用的一些实践
    - react-router可以不使用`#`作为前端路由的分隔符，前端路由控制的url也可以像普通的url一样，只要加上[一点处理](https://github.com/reactjs/react-router/blob/master/docs/guides/Histories.md)，这样的实践应该在项目一开始就做，好处是在类似sso跳转的时候比较方便。
    - 回复初始状态的action（reset)

    ```javascript
    // actions
    function reset() {
      return {
        type: RESET
      }  
    }
    // reducer
    case RESET:
    	return initialState;
    // container
    componentWillUnmount() {
      dispatch(reset());
    }
    ```
  - 使用redux-dev-tools
    - 在找bug的过程中能带来很好的体验

### 存在的问题
- 一个container对应actions, reducers，components等多个文件
	- 写代码时需要来回不停地切换文件
	- 以页面的方式来组织逻辑导致很难按照功能来抽象，具体表现为能复用的只是ui-component, 一些相似的逻辑（比如点击某个button->fetchData->render)不方便统一
		- 以后可以不严格按照页面来组织代码，而是以功能来组织reducer和action，[这里](https://blog.maxleap.cn/archives/930)介绍了文件目录结构的相关实践

- container级别暂时没有抽象出一个基类来做一些通用逻辑，导致统一的逻辑加起来有点麻烦
  - 在项目初期最好就有这样的设计，后期改动成本较大

- 项目中redux middleware使用了redux-thunk这样较为简单的形式，导致action之间的几乎就是孤立的，不方便组合，如果使用redux-promise或者redux-saga这样的工具，在action的组合上会有点帮助，不过像redux-saga还是会带来比较大的使用成本，当然这也只是个人看法

- 由于redux采用了single store，那么state不可避免的就会非常大，所幸redux采用的shallow copy，不过当每个container中connect的props过多，对性能肯定也有一定影响。这就有一个问题，container的props是尽可能的少（层级深）还是扁平化（属性多），这里也希望大家能够给出自己的实践经验

- why not immutable
  - 在几乎所有有关react的性能文章里，immutablejs总是能出现身影，因为确实能解决问题
  - 我们不用的原因，正如上面所提到的，由于采用了更加扁平化的数据结构，这样不涉及较长的属性路径，同时我们使用了object spread operator（{…state, p1, p2}）,这样也能解决绝大多数问题
  - 不过，immutable还是很有用处的，以后也会慢慢用上，尤其是关注到性能问题的时候

### 总结

  - 学习和使用redux这个过程还是很愉悦的，因为你随时都有可能有[新认知](https://github.com/sorrycc/dva)
