
---
title: typescript的声明实践
date:  2017-12-12
tags:  immutable.js, benchmark
---
> typescript最直观和方便的就是它的type，静态类型配合vscode往往比文档带来的体验都好

> 如无特殊情况，下文中所有tsconfig包含{moduleResolution: "node", module: "commonjs"}, tsc version >= 2

#### 主要内容
- 如何使用声明文件，声明文件的使用场景和typescript是如何寻找声明文件
- 一些typescript声明tips

#### 如何使用typescirpt的type
- 如何触发ts的compiler寻找声明文件
  - `import`某个模块才会触发
  - `require`不会触发，不管这个模块有没有声明文件

- 如何知道typescript寻找声明文件
  - 一般而言使用tsc会出现`Error TS7016: Could not find a declaration file for module xxx`, vscode也会有相应提示
 
  ![ts-no-declaration-error-hint](http://o9w5dyyws.bkt.clouddn.com/ts-no-declaration-hint.png) 
  - vscode会提示我们使用`npm install @types/xxx`安装该模块的声明文件，下文会介绍`@types`的使用场景

- 使用types的场景
  - ts项目使用js写的项目(指的是原有js项目没有声明文件)，有如下方法是该js项目支持typescript的声明
    1. 尝试安装@types/xxx, 如果没有就用第二种方法
    2. 在ts项目下对该模块做单独声明`declare moudle xxx`, 使用`paths`做映射
  - ts项目中使用ts写的项目，一般而言ts写的项目编译后会有声明文件，利用这一点ts可以直接关联到这些声明文件，无需其他操作

- 手动编写的声明文件引入其他声明文件
  - 引用nodejs的声明, `///<reference types=“node”/>`
  - 使用`import xxx from “xxx”`

#### typescript是如何寻找声明文件的
  - 影响typescript寻找声明文件的配置
    - moduleResolution, 默认是nodejs
    - paths, 模块和声明文件路径的映射
    - typeRoots, 针对[`/// <reference types="..." />`](https://github.com/Microsoft/TypeScript/issues/12222#issuecomment-260417733)
    - types, 是指加入编译的声明文件，和寻找路径无关

  - `tsc --traceResolution xxx`会把寻找声明文件的所有信息都输出

    ![typescript-resolve-types](http://o9w5dyyws.bkt.clouddn.com/typescript-resolve-types.png)

> nodejs项目需要手动安装@types/node, 才会有nodejs的提示

> scoped package如何使用@types，@yourscope/packagename -> @types/yourscope__packagename

#### 总结
- 自己能控制的项目，声明尽量放在项目内部
- 自己不能控制的项目，声明使用@types
