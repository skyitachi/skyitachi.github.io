---
title: GMTC见闻
date:  2017-06-14
tags:  GMTC，javascript，大前端
---

> 笔者有幸参加了2017GMTC大会，在两天会议中主要关注了移动AI和大前端分享，在此和大家分享相关见闻.

#### [PWA: 移动Web的现在与未来](http://ppt.geekbang.org/slide/show/837)

作者从"用户花80%的时间在top3的app"上来说明，移动web还是有机会的，只要提供Reliable，Fast，Engaging的体验就有可能，作者主要从PWA的提供的能力上来阐述移动Web未来的趋势。

同时简单介绍了ServiceWorker的实践和利用[workbox](https://workboxjs.org/)优化开发体验。

从谷歌对PWA的宣传程度上来讲，至少在Android上PWA的体验是很好的，不过iOS相关的支持可能还得等上一段时间了，但是拥抱标准是肯定的。

ps: PWA 在近一年来不断的被提及，是时候提升自己的武器库了

#### [Lean app:Instagram Architecture at Scale](http://ppt.geekbang.org/slide/show/838)

作者对Instagram Android app在Binary size，Build time和Java method count三个方面进行了优化，总结了在工程实践方面的经验，比如：

Keep lean app with measurable goals.

Do Simple Things First!

ps: 我个人感触较大的地方是对代码质量的重视和大约20%-40%的时间花在mentorship上，也就是说在团队快速扩张的同时（指导新人）又能保持业务的快速迭代，这其中肯定是有方法论的。虽然我不是客户端开发，但是这些方法论是通用的，关键还是看实践和团队文化本身。

#### [Re:移动开发的未来——来自一个微信移动开发者的自白](http://ppt.geekbang.org/slide/show/842)

作者开场就抛出了“原生App开发者要失业了吗？”这样一个问题，结合这两年移动开发的趋势（大量使用web技术），比如react-native，小程序和**weex**等，来阐明原生App开发者所面临的问题。

当然作为android开发者作者也对新业务热点比如VR/AR, AI, IOT等方向上表明原生App开发者的新机遇。同时也肯定了原生App在体验上有着无法被超越的优势。当然作者还是强调了AI的影响力。

作者最后对原生App开发者提出了保持好奇心，了解新的对手，审慎而勇敢的做出改变这样的观点。

ps: 说实话，作为前端开发看到原生App开发者具有如此的危机意识还是很钦佩的，虽然web技术有些天然的优势，但是短板很明显，比如体验上和利用系统的能力上都有着巨大的劣势，在AI时代貌似能做的就更少了，我个人认为前端同样也面临危机，不要给自己设限，共勉。

#### [利用CNN实现无需联网的智能图像处理](http://ppt.geekbang.org/slide/show/847)
作者主要介绍了百度图搜在端上是如何利用CNN做到分类和单个框选的，介绍了机器学习基本原理，卷积，池化等优化手段

整体架构上采用了服务端训练和客户端识别，以及阐述了客户端识别的优势（网络payload多大影响体验，服务端识别不能有效利用用户的计算能力等）。

技术上在客户端移植了GoogLeNet模型，移植caffe等。

移动端的挑战有内存，耗电量，模型大小，加密问题等。

通过精简模型, 使用uint代替float和压缩等方法解决了模型大小问题，使用陀螺仪避免无效识别（跑一次神经网络耗电巨大）等。

作者主要介绍了在Android端的优化实践，抛弃低端CPU，与ARM深度合作，共同研发ARMComputeLibrary, 使用NEON指令，使用汇编，inline和循环展开等优化手段。

作者还介绍了SIFT模型，由于SIFT特征量较小可实现流式识别。

ps: 开眼界系列。同时也可看出从理论到实践之间还是有段距离的。

#### [使用TensorFlow搭建智能开发系统，自动生成App UI代码](http://ppt.geekbang.org/slide/show/848)
作者从利用Tensorflow识别设计稿或者app截屏并自动生成UI代码的角度阐述了人工智能技术如何辅助开发者开发，帮助开发者提升效率。

原理上利用CNN，softmax实现组件级别的分类识别，同时利用React-native生成代码。在识别效果不是很好的情况下，加入人工决策，形成反馈。

ps: 目前从设计开始就有机会直接生成代码了，[react-sketchapp](http://airbnb.io/react-sketchapp/)，或许从设计稿生成的角度会有更多的发现

#### [一站式短视频技术架构的新解读](http://ppt.geekbang.org/slide/show/851)
作者主要介绍了UCloud在视频存储，处理等方面的解决方案，提到了UCloud在数据安全，数据获取上做的一些优化实践。

比如：不同数据中心的备份，利用cdn做视频访问加速等。

ps: 虽然技术上都是通用的方案，但是从上传，存储，处理，分发每个环节都有相应的解决方案并形成完整的链路，还是很不错的。

#### [Conversation as a platform创新交互方式](http://ppt.geekbang.org/slide/show/856)
作者从消息类app占据了用户大量时间这一事实出发，介绍了微软在对话服务上做的一些产品，比如Cortana.

同时介绍了[聊天机器人](https://dev.botframework.com/)的一整套开发框架以及相关的服务

ps: 虽然也是硬广，但是讲师的气质好啊。。。

#### [移动互联网时代的VR技术之路](http://ppt.geekbang.org/slide/show/865)
作者从全行业的角度，讲述几乎所有的巨头都在发力VR/AI，提出了信息交换正在向着广度，深度和密度方向发展。

同时也指出了国内从业人员目前在知识体系和心态上的不足，VR/AI需要更多的数学知识，同时也要有足够的编码能力。

作者指出人工智能热潮是由于机器学习+数据ready的方向明确性导致的，也指出了还有很多技术要突破。

ps: 道阻且长，对于已经工作的程序员来说系统的学习一些新东西还是很有挑战的，共勉。

#### [Vue.js在前端服务化上的探索与实践 ](http://ppt.geekbang.org/slide/show/860)
作者讲述了饿了么使用Vue.js的一些经验。

利用Vue Component实现了营销页面搭建的自动化，使用json配置，自动生成动态化的内容。使用App Builder加速页面搭建。

介绍了饿了么DAS系统，方便生成快速搭建Banner。

作者总结了在做前端服务化的经验，比如主动熟悉业务流程，发现可优化的点，多方沟通，确保可以落地，不断迭代等，
同时指出了不应最求功能多而全，前端服务化也会有客服成本，过于追求灵活，忽视用户使用门槛等。

作者还讲述了在做编辑器的一些探索，使用command模式实现undo/redo等。

ps: 主动熟悉业务流程

#### [React在大型后台管理项目中的工程实践](http://ppt.geekbang.org/slide/show/861)
作者提出了今日头条在前端工程化上设计，webpack作为依赖管理，middleware提供前端功能增强，express/koa作为基础web服务框架，yarn/npm作为资源管理工具，nvm用于管理多个node版本。

使用不同版本的配置文件，实现各个环境配置的隔离。

前后端分离的实践，本地server，通过node-http-proxy代理本地请求。

使用webpack的commonChunks plugin和webpack 1的require.ensure对spa做分片加载和动态加载，提升了spa的首次加载性能。

使用ducks-modular-redux对redux的项目结构做了改善，即将reducer，action creator放到一个文件中。

使用redux-saga解决redux-thunk 在控制流上较弱的问题。

使用husky做代码检查。

ps: 熟悉的味道。。。适合项目的就是好实践。

#### [ReactNative框架在京东无线端的实践](http://ppt.geekbang.org/slide/show/869)
作者介绍了使用react-native的原因，以及在京东的实践，实现了react-native的三端统一，自己实现了到web端转换

对listview的内存优化，自己实现了类似react-native的flatlist，针对低端机器的做的加载性能优化（拆分js bundle），防止白屏

性能优化上，使用了yoga引擎，复杂交互原生化，尽可能多使用Antimated类实现动画

基础设施建设，包括热更新，灰度，容灾，预加载等，同时也对react-native进行了深度定制

暂未有开源计划

ps: 友商的实践还是挺有深度的，整体完成度比较高。

#### [PWA在饿了么的实践经验](http://ppt.geekbang.org/slide/show/870)
作者列举了PWA的优缺点，以及为什么选择PWA，拥抱标准，和UC，X5团队深度合作

列举了PWA可能带来的重要问题

底层劫持，可能导致全站白屏；清不掉的缓存；前端也会踩到P0事故

做了充足的准备工作，灰度/降级方案，监控系统，数据统计

饿了么多页应用的实践，基于业务场景而选择多页应用，并非一味的追求单页，同时也遇到了一些坑，比如useragent在worker线程中不正确，
http header导致跨域请求失败，UC 301跳转等

在实践过程中和腾讯X5，阿里UC和Google的Michael Young建立了良好的合作关系

ps：在合作用推动web标准的落地，实力和勇气兼具

#### 总结
热点总是在变化，需要审慎而勇敢的做出改变
