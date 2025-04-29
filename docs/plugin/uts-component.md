# uts插件 - 组件开发

uts插件的组件开发，简称`uts组件`，是uts插件的一个分支。uts插件提供了原生API的扩展，而uts组件提供了原生UI组件的开发模式。

组件是一种独立、可复用的UI单元，用于单独封装和承担一定的代码逻辑。

组件与API的区别在于：
- 组件：以标签的形式，对外提供UI封装，可以在页面template中使用，内嵌在页面中成为一个区域；
- API：有的不涉及UI，也有涉及UI的。但涉及UI时，也是全屏的UI，不内嵌在文档流中。调用相关API后弹出全屏界面，无法以标签方式在页面模板中引用。

比如`<video>`是组件；`uni.showModal` 或 `uni.createRewardedVideoAd` 虽然有UI，但属于API。

组件一般适用于封装非全屏的场景，即在页面中内嵌一个区域。如果需要你需要封装的UI是全屏界面，也可以通过API来弹出全屏的activity或view controller，但弹出这种原生窗体后，需要注意和UniPage的页面栈的关系。uts通过API方式开发原生页面的示例：[uts开发原生页面示例](https://gitcode.net/dcloud/hello-uts/-/tree/master/uni_modules/uts-nativepage)

当然uts组件是多端的，一个uts组件作为一个`uni_modules`，可同时支持app-Android、app-iOS、app-Harmony、web、以及各家小程序组件。

## uts组件开发模式

app平台如何将原生UI扩展成一个组件，嵌入在页面中？有以下开发模式：
- 标准模式（HBuilderX4.31 及以上版本支持）  
- uni-app兼容模式（HBuilderX3.6.18 及以上版本支持）  

**开发模式对比表** 

|								|uni-app兼容模式													|标准模式																		|
|--							|--																			|--																				|
|支持平台				|uni-app的app-nvue和uni-app x的app-uvue。不支持鸿蒙	|仅支持uni-app x，但uni-app x中app、web、小程序全平台支持	|
|组件规范				|参考vue的模仿，是子集、也有扩充定制，不支持组合式API	|完全vue组件规范，标准easycom	|
|使用native-view|否																			|是																				|
|学习成本				|低																			|无																				|

一般来讲，我们推荐开发者使用`标准模式`。这种模式就是写一个标准的vue组件，没有学习和认知门槛。

因为uni-app下只有nvue页面才能放置原生扩展的组件，而nvue已经不再维护，我们推荐开发者在uni-app上使用web组件。除非你的组件无法使用web实现，可以接受在uni-app的app-nvue中使用，且不需要鸿蒙支持，才使用`uni-app兼容模式`。



## 标准模式组件介绍  

从HBuilderX4.31起，uni-app x内置组件里新增了 [native-view](../component/native-view.md) 组件，它就是为了扩展原生组件而提供的一个内置组件。

首先在页面或组件中，放置 native-view 组件，它作为内置组件，可以正常的参与排版布局。

在uts插件中，可以通过 [uni.getElementById](../api/get-element-by-id.md) 方法获取到 native-view 的 [UniElement](../dom/unielement.md) 对象，再通过 [UniElement](../dom/unielement.md) 的 [getAndroidView](../dom/unielement.md#getandroidview)/[getIOSView](../dom/unielement.md#getiosview)方法，可以获取到 uvue 界面上的一个 native-view 组件的原生view。

当插件开发者拿到一个原生的Android或iOS的view后，就可以使用UTS或原生混编代码随意操作这个原生view。比如绘制地图、绘制摄像头内容。

如果开发者想要对外封装为一个组件，暴露组件的属性、方法、事件，只需要按照标准vue组件的做法，使用easycom的方式，把 native-view 组件包在一个vue组件中。

完整开发教程参考[uts组件 - 标准模式组件开发](./uts-component-vue.md)。


## uni-app兼容模式组件介绍  

uni-app x项目是HBuilderX4.31起才新增了 [native-view](../component/native-view.md) 组件，且 uni-app 项目不支持 native-view 组件。

在没有 native-view 组件的情况下，uts提供了另一种uts组件的开发模式，即`uni-app兼容模式`。

为了让原生开发的ui组件可以在页面上作为一个组件来嵌入，DCloud参考vue规范定义了一个特殊组件规范，部分实现了vue组件规范、且又扩充了一些规范（如生命周期NVBeforeLoad、NVLoad、NVLoaded、NVLayouted、NVBeforeUnload、NVUnloaded等）。

`uni-app兼容模式`的好处是即可以在uni-app的nvue页面使用，也可以在uni-app x的uvue页面使用。

如果您的插件仅服务uni-app x，那么没必要使用`uni-app兼容模式`。

如果您确定需要使用`uni-app兼容模式`来开发uts组件，那么需要了解这种模式的开发规范，详情参考[uts组件 - uni-app兼容模式组件开发](./uts-component-compatible.md)。

