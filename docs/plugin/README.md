# uni-app x 插件生态

`uni-app x`积极拥抱社区，创建了开放、兼容的插件系统。

uni插件市场，[https://ext.dcloud.net.cn](https://ext.dcloud.net.cn)，是uni-app x官方插件生态集中地。支持前端组件、uts sdk、页面模板、项目模板、uts插件等多种类型。

请注意尽量在官方市场寻找插件，npm等三方市场没有uni-app兼容性描述，很容易下载到无法跨平台的、仅适配web的插件。

uni-app需要兼容多个平台，在统一和个性化之间，uni-app的设计原则一直是：把常用的部分uni化，不常用的各平台特色不限制使用，都可以在条件编译里调用。

- 编译到web时
	可调用浏览器的所有api，可混合使用js，可使用web生态的各种库，包括npm。

- 编译到小程序时
	可调用小程序的所有api，可混合使用js，小程序的自定义组件生态（如wxml组件），包括小程序的npm库。
	
- 编译到Android时
	可调用Android os的所有api，可混合使用kotlin、java，可使用所有适配Android的sdk，包括so库，可使用gradle等仓储。

- 编译到iOS时
	可调用iOS的所有api，可混合使用swift（object-c需封装为库后使用，不能直接使用oc源码），可使用所有适配iOS的sdk，包括cocoaPods。

- 编译到Harmony时
	可调用鸿蒙的所有api，可混合使用ArkTS，可使用所有适配鸿蒙的sdk，包括ohpm。

uni-app设计了[uni_modules](https://uniapp.dcloud.net.cn/plugin/uni_modules.html)，在`uni_modules`下，可以同时放置npm、gradle、cocoaPods、ohpm等库。方便封装跨平台的插件，统一给插件使用者的输出。

在[插件市场](https://ext.dcloud.net.cn)中，勾选搜索栏下方的 uni-app x 的 checkbox，即可浏览适配了uni-app x的插件。

在编译到不支持web的平台时，如需使用web生态的内容，有2种方式：
1. 在uni-app x的 web-view 组件里使用web库。uni-app x提供了web-view组件内的js和uts通信的机制。
2. 集成[uni小程序sdk](https://nativesupport.dcloud.net.cn/README)，或集成一个v8/quickjs等库来调用js生态的内容。
3. 把for web的ts库适配为uts库。目前npm上很多流行库已经是ts编写的了，稍加适配即可兼容uts。

插件市场的uts插件分类，适用于原生能力的封装。uts插件在uni-app和uni-app x中都支持，并且支持计费。