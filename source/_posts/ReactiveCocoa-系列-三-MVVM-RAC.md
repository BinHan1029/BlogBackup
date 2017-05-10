---
title: ReactiveCocoa 系列(三) MVVM + RAC
date: 2016-05-15 21:53:45
tags:
- 移动开发
- iOS
---
对于一个开发者来说, MVC 设计模式被广泛应用于各种语言的开发中，这在 iOS 开发中体现的尤为多，比如我们的控制器 Controller，视图 View，模型 Model，从名称中就可以体现出各自的作用及分工，Apple 也是推荐我们使用 MVC 模式进行程序设计及开发。其实绝大部分时候 MVC 模式基本是可以满足我们的业务需求的，可是随着业务的膨胀，开发人员变动等因素，MVC 模式也暴露了其弊端，当控制器层变得越来越臃肿的时候，业务过于集中到控制器层，就会为为接下来的开发带来不便及代码的可维护性变低。

所以这时候为了解决刚才的问题，MVVM 模式就“应孕而生”了，它不仅仅可以对原有 MVC 模式进行代码上的优化瘦身，更重要的一点是兼容 MVC 模式，我们可以对原有项目必要的地方进行 MVVM 模式的升级。

<!-- more -->
 
### 关于 MVVM 的理解
 
![Alt text](/assets/blogImg/mvvm_1.jpeg)

在理解 MVVM 前，视图 View、模型 Model 是 MVC 中已经有的概念。在大部分开发的时候，table 可以说是我们使用最多的一个视图了，它里面的一个个 cell 视图的展示都需要设置一个模型 Model，通常会让 model 会作为 cell 的一个属性而存在，所以如果我们暂时不考虑 model 是从哪里冒出来的话，就会发现 model 和 cell 其实是一一对应的，而 cell 中展示的内容全部来自于 model 对象。现在我们在来看控制器 Controller，其实每一个 Controller 都会都应一个视图 View，那么这个视图的数据来源哪里呢，没错来自 Controller 本身，这也是造成 MVC  模式臃肿的根本原因，所以能不能对这个视图 View 也对应一个唯一的 model 呢？所以就产生了 ViewModel，这也就是 MVVM 模式的基础模型。当然这里每个人的理解可能会有不同。关于 ViewModel 的作用就是拆分业务后，将逻辑代码、网络请求等集中在 ViewModel 中进行处理。而之前的 Controller 层依旧负责自己视图的创建，然后将视图对应的 ViewModel 进行 bind 就好了。

### MVVM 结合 ReactiveCocoa

这里又提到一个很重要的概念 bind。其实单单把业务通过 MVVM 拆分成更小更单一的模块时，那么 View 和 ViewModel 两者之间的数据的同步就可以交给函数式响应式编程框架 [ReactiveCocoa](https://github.com/ReactiveCocoa/ReactiveCocoa)，当然从某方面将，做数据的时时同步也有其他的方式，但是无论使用 delegate 或者 block，总是让我觉得写出来的代码不如通过 [ReactiveCocoa](https://github.com/ReactiveCocoa/ReactiveCocoa) bind 的方式显得优雅。因为系统中的 KVO、UIKit event、delegate、selector 等都增加了 RAC 支持，所以都不用去做很多跨函数的事，提供了一个单一的、统一的方法去处理异步的行为。


### 关于MVVM的优缺点：

优点：

* 相对于 MVC 控制器层臃肿的逻辑与视图，是很难对某一块逻辑进行测试的，而剥离出来的 ViewModel 是一个独立的模块，所以对于逻辑部分可以进行单元测试。
* 正因为 ViewModel 的独立性，所以代码的可复用性更高，在某两个模块只要业务逻辑相同，那么完全可以粘贴过来将需要更改的接口部分进行修改，即可。这里面可以使用类模板减少重复的代码书写。

缺点：

* 项目中类(.h/.m)文件会增多，这样可能会让代码编译速度和程序启动速度变慢，当然这我觉得这个效率其实是可以忽略的。
* 从某方面讲，其实随着业务的逐渐庞大，是很难从根本上绝对避免功能上的耦合的，所以 ViewModel 的剥离的层级越多，也会相对增加项目结构的复杂度。比如一个 table 有 headerView，对应的 headerView 也会有一个 ViewModel，但是如果 headerView 的模块足够复杂的话，可能需要进一步的细分 headerView，那么就会增加更多的 ViewModel。
* 结合 RAC，排错困难。

最后，撸了一个 MVVM + RAC 的 demo，效果如下，通过搜索然后展示列表，当搜索结果为空时展示空数据视图。代码可以在Github中[BlogDemo](https://github.com/binhandev/BlogDemo)进行查看。


<video width="308" height="570" autoplay controls>
	<source src="http://binhandev.github.io/assets/blogImg/mvvm.mov">
</video>






