# uvue css使用

uni-app x 在 app平台实现了 web css 的子集。

这个子集有时也被称为 ucss，但工程文件仍然是 css、less、scss 等后缀，style 节点的 lang 属性也没有特殊之处。

子集并不影响开发者开发出所需的界面，仅是写法上没有那么丰富。

当 uni-app x 编译到 web、小程序等平台时，可以支持 web 的全部 css。同时，**编译器会进行 css 重置**，保证 ucss 这个子集在各端效果的一致性。这也意味着使用uni-app x的web端，和直接在浏览器里写html在默认值处理上有一定的差异。

浏览器、Android原生、iOS原生，都有自己的布局和样式设置系统。

ucss这个子集，是抽取了web和app的共同能力，并通过web的css写法来设置原生布局和样式。

原生App的组件样式都是通过组件的属性设置的。父子组件的属性隔离，样式不会继承。

App端与web常见的区别是：
1. 仅支持**flex 布局**：这是web、iOS、Android均支持的布局方式
2. 选择器**只能用 class 选择器**，不能用tag、#id、[attr]等选择器，目前class选择器不支持除A-Z、a-z、0-9、_、- 之外的字符
3. **样式不继承**，即父元素样式不影响子元素

> 但以上仅是常见的区别，并非所有，需开发者继续阅读全文。

## 页面布局

> uni-app x 使用flex布局。\
> 这是一种清晰易用、全平台支持的布局。不管web、Android、iOS、微信skyline、快应用，均支持flex布局。\
> 页面布局有2个注意事项，[flex方向](#flex-direction) 和 [页面级滚动](#pagescroll)。

### flex方向 @flex-direction

在W3C规范中，flex 默认是`横向`的，但uni-app x里全平台的flex方向默认值都是`纵向`的。

因为W3C规范中，默认布局是block，当使用flex时更高频率是用它的横排能力。而在flex是第一布局模型的手机端，大量布局都是竖排的，此时要求开发者大量编写`style="flex-direction:column"`很不友好。

所以在uni-app x中默认是竖排（之前nvue也默认是竖排）。同时在[manifest.json](../collocation/manifest.md)中提供了配置项，可以修改flex方向为横排。

```html
<view style="flex-direction:row">
	<!-- 这里的组件们都是横排 -->
</view>
<view style="flex-direction:column">
	<!-- 这里的组件们都是竖排 -->
</view>
<view>
	<!-- 这里的组件们在uni-app x里全平台都是竖排。而W3C的规范中这里是横排 -->
</view>
```

一般在`app.uvue`的style里编写全局样式，在页面里使用class引用更为方便。
```html
<style>
.uni-row{
	flex-direction: row;
}
.uni-column{
	flex-direction: column;
}
</style>
```

```vue
<template>
	<scroll-view class="uni-column" style="flex:1">
		<view class="uni-row">
			<text>左</text>
			<text>右</text>
			<!-- 2个text组件横排 -->
		</view>
		<view class="uni-column">
			<text>上</text>
			<text>下</text>
			<!-- 2个text组件竖排 -->
		</view>
	</scroll-view>
</template>
```

::: info 其实flex布局概念非常简单清晰。解释下上面的代码。

1. 页面根节点是一个纵向的（class="uni-column"）、全屏的（style="flex:1"）scroll-view，那么它的子view们就是纵向。
2. 二级view里的第一个view，设为横向，里面又放了2个三级组件text，那么这2个三级组件text是横向，即左右2个字。
:::

### 页面级滚动@pagescroll

`web开发`中，页面是必然可以滚动的。当然也可以给某些div设局部滚动。

而`原生开发`中，**页面不能滚动**。如果你需要某个地方滚动，那么要在相应位置放scroll-view或list-view等可滚动组件，在这些组件内部滚动。

如果你想要整页滚动，那么可以在页面最外层套一个scroll-view，看起来就和web开发的页面滚动一样了。

在老版nvue中，如果开发者顶层不是scroll-view，编译器会自动在外面套一层scroll-view，来变相实现页面滚动。\
但在uvue中，废弃了这个策略。因为开发者的页面情况较复杂，而且vue3支持多个一级组件，之前的策略可能会多给页面套一层不必要的scroll-view。\
在追求高性能时，多一层scroll-view是不能忍受的。

`uvue的策略`：在新建页面时，提供一个选项，让开发者选择是否需要页面级滚动。如需要则自动在页面代码里template的根节点加一个全屏的scroll-view。如下

> 如果开发者不需要，随时可以自己修改代码。

```html
<template>
	<!-- #ifdef APP -->
	<scroll-view style="flex:1">
	<!-- #endif -->

	<!-- #ifdef APP -->
	</scroll-view>
	<!-- #endif -->
</template>
```

因为web平台和基于webview的小程序使用页面滚动更方便，所以自动套在页面顶层的scroll-view写在了[条件编译](https://uniapp.dcloud.net.cn/tutorial/platform.html)里。

> 当然如果你只做app，可以不写条件编译。

这样在web浏览器里就无需多套一层scroll-view，自然的使用浏览器的页面滚动就好了。

尤其在Android webview中，scroll-view其实是可区域滚动的div，滚动区变长后，性能远不如页面滚动。

上述代码中给scroll-view的style设为`flex:1`，意思是铺满剩余空间。设在顶层节点上，意味着铺满屏幕。

当然，如果页面的pages.json里配置使用了原生导航栏，那么页面区整体是在原生导航栏下面。

#### 自定义导航栏

如果开发者想要自定义导航栏，首先在pages.json里对应页面的style里设置`"navigationStyle": "custom"`，关闭原生导航栏。\
然后编写自定义的导航栏组件[\<uni-navbar-lite>](https://ext.dcloud.net.cn/plugin?id=14618)，那么推荐的页面代码结构为：

```html
<template>
	<uni-navbar-lite title="自定义导航栏标题">
	</uni-navbar-lite>
	<!-- #ifdef APP -->
	<scroll-view style="flex:1">
	<!-- #endif -->

	<!-- #ifdef APP -->
	</scroll-view>
	<!-- #endif -->
</template>
```

> 注：这里的“原生导航栏”是一个历史沿袭叫法，指配置在pages.json里的导航栏，不属于页面代码区。事实上在uni-app x的app平台里所有界面都是原生的。

#### 页面滚动相关的生命周期、api

在uni-app的规范中，页面滚动有一批相关的生命周期、api，比如：`onPageScroll`、`onReachBottom`、`uni.pageScrollTo()`

在app端，会判断页面根节点是否为scroll-view（不认list-view等其他滚动容器）。

* 如果是，页面滚动相关的生命周期和API继续生效，效果如前。
* 如果不是scroll-view，全部失效。

> 如果根节点使用了list-view，它也有自己的滚动相关的API和监听事件。详见[list-view](../component/list-view.md)的文档。

#### 页面滚动引起的差异

`uni-app-x` App端无页面滚动，且其根节点高度为从导航栏底部到tabBar顶部。如果在页面根节点的子元素使用`position: absolute;`，页面内部scroll-view滚动时不会改变此元素位置。其他端有页面滚动，如果在页面根节点的子元素使用`position: absolute;`页面滚动会改变此元素的位置。如果有不随页面滚动变化位置的需求建议使用`position: fixed`。

注意：web端需要使用[css变量](common/function.md)使元素不覆盖在navigationBar和tabBar上。

## 样式不继承@stylenoextends

web的样式继承，主要是文字样式继承。web的css属性众多，规范比较松散，随便一个div都可以写文字相关的样式。这样其实不严谨、性能也不好。

在原生等严谨的应用开发方案中，均是组件搭配该组件的专有属性。容器组件和文本组件分离，属性各自隔离，不可能在容器组件里写文本组件的样式。

在uni-app x中也是，文本必须使用`<text>`组件，`<view>`组件就是容器组件，它的style里不应用使用与文本修饰相关的样式，比如文字颜色、大小等。

如下代码，在web浏览器渲染时，父view的style会影响子text，所以123是红色。

但是在app-uvue中，样式不继承，123的颜色仍然是默认颜色黑色。
```html
<template>
	<view style="color:red">
		<text>123</text>
	</view>
</template>
```

在app-uvue中，如需修改123的样式，需写在text组件中
```html
<template>
	<view>
		<text style="color:red">123</text>
	</view>
</template>
```

uvue中文字都是要使用text组件的。

虽然把文字直接写在view的text区也能运行，其实是因为编译器自动套了一层text组件。
```html
<template>
	<view>123</view>
</template>
```

上述代码，在编译后其实变成了：
```html
<template>
	<view>
		<text>123</text>
	</view>
</template>
```

那么，就意味着直接写在view的text区，虽然可以用，但将无法修改文字的样式。因为被自动套一层的text组件上没有任何样式。

也就是下面的代码是不能改变文字颜色的。
```html
<template>
	<view style="color:red">123</view> <!-- 文字颜色不会变，因为被套了一层text组件后，父view的样式并不能被新套的text组件继承-->
</template>
```

app-uvue的css的“样式不继承”规则，虽然与web有差异，其实只是更严谨。

一般情况下，开发者遵循仅在text组件下写文字有关的样式，就可以编译到全端而保持界面正常。

由于app平台样式不继承，那么在web中继承相关的关键字`inherit`、`unset`在app中也不支持。

由于web的css中某些属性的默认值就是`inherit`，即继承父，此时在ucss中会修改为一个独立而具体的默认值。

## 样式冲突、优先级@css-specificity

在web中，因为有多种样式的设置方式、父子样式也有继承关系，这导致很多样式冲突。既然有冲突，就需要明确优先级规则。

实际上web的样式优先级规则非常非常复杂，可参考[MDN文档](https://developer.mozilla.org/zh-CN/docs/Web/CSS/Specificity)

app中，设置样式只有内联样式即style属性和class属性这两种方式。它们只遵循1个简单规则：`内联样式(即style属性)优先级高于class选择器、class选择器之间的优先级遵循web规范`

在web中，还存在`!important`例外规则，可参考[MDN文档](https://developer.mozilla.org/zh-CN/docs/Web/CSS/Specificity#!important_%E4%BE%8B%E5%A4%96%E8%A7%84%E5%88%99)

在app中，`!important`仅支持在class选择器（可以覆盖内联style中的样式）中书写，style内联属性不支持。

除了`!important`外，后设的样式总是覆盖先设的样式。如果通过响应式变量绑定或DOM API操作时，要注意这一点。

## 样式作用范围

在 `uni-app x` 中，不支持 `css scoped`，样式的作用范围遵循以下规则：

* `App.uvue` 中的样式作用于全局。
* 页面的样式作用于当前页面及其子组件。
* 组件的样式仅作用于当前组件。

## 层级

App仅对`同层的兄弟节点`之间支持`z-index`来调节层级。不支持脱离dom树任意调节层级。

## css模块

|模块				|App支持情况	|备注									|
|:-:				|:-:		|:-:									|
|背景与边框			|√			|不支持背景图							|
|盒子模型			|√			|										|
|Flex 布局			|√			|										|
|Inline 布局			|×			|										|
|Inline-Block 布局	|×			|										|
|Block 布局			|×			|										|
|字体				|√			|[详见](font-family.md)	|
|Positioned 布局		|√			|										|
|CSS Animation		|x			|										|
|CSS Transition		|√			|										|
|CSS Variable		|×			|										|
|媒体查询			|×			|										|

## css方法 @css-function

目前支持url()、rgb()、rgba()、var()、env()。[详见](./common/function.md)

<!-- CSSJSON.readmeTable.name -->

<!-- CSSJSON.readmeTable.compatibility -->

## Bug

css相关bug[详见](https://issues.dcloud.net.cn/?mid=css)
