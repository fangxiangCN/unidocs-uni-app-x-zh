# uts插件 - 标准模式组件开发

> HBuilderX4.31+ 支持app-android、app-ios，HBuilderX4.61+ 支持app-harmony。

本文重点在于讲述如何在app-android、app-ios、app-harmony上，使用vue组件开发规范封装原生UI封装为 uni-app x 项目使用的UTS组件，供使用者在uvue页面template中以组件的方式调用。

主要思路是将app平台的原生view关联内置 [native-view](../component/native-view.md) 组件，实现UTS组件的特定功能及UI展示。  

## 前置条件

继续阅读文档前，开发者需要了解以下前置条件：

- 了解 `标准模式` 和 `uni-app兼容模式` 的差异，详情参考[uts组件开发概述](./uts-component.md)
- 了解 [uts语法](/uts/) 和 [uts原生插件](uts-plugin.md)
- 了解 [vue组件](https://uniapp.dcloud.net.cn/tutorial/vue3-components.html)
- 了解 [native-view组件](../component/native-view.md)

## UTS插件-标准模式组件目录结构@dir

<pre v-pre="" data-lang="">
<code class="lang-" style="padding:0">
┌─components                  // vue组件代码
|	├─xxx                       // vue组件名称文件夹  xxx代替组件名称
|		├─xxx.uvue                // vue组件uts代码 xxx代替组件名称
├─static                      // 静态资源
├─utssdk
│	├─app-android               //Android平台目录
│	│	├─assets                  //Android原生assets资源目录，可选
│	│	├─libs                    //Android原生库目录，可选
│	│	├─res                     //Android原生res资源目录，可选
│	│	├─AndroidManifest.xml     //Android原生应用清单文件，可选
│	│	├─config.json             //Android原生配置文件
│	│	├─index.uts               //Android原生插件能力实现，可选
│	├─app-ios                   //iOS平台目录
│	│	├─Frameworks              //iOS原生依赖的第三方 framework 依赖库存放目录，可选
│	│	├─Resources               //iOS原生所依赖的资源文件存放目录，可选
│	│	├─info.plist              //iOS原生所需要添加到主 info.plist 文件中的配置文件，可选
│	│	├─UTS.entitlements        //iOS原生所需要添加到主工程 .entitlements 文件中的配置文件，可选
│	│	├─config.json             //iOS原生配置文件
│	│	├─index.uts               //iOS原生插件能力实现，可选
│	├─app-harmony               //鸿蒙平台
│	│	├─config.json             //鸿蒙平台配置，主要用于配置依赖
│	│	├─index.uts               //鸿蒙平台原生实现
│	│	├─libs                    //依赖的har包等存放位置
│	│	├─resources               //编译为鸿蒙原生模块后的resources目录
│	│	└─builder.ets             //鸿蒙原生builder存放位置
│	├─web                       //web平台目录
│	│	└─index.uts
│	└─index.uts                 // 跨平台插件能力实现，可选
└─package.json                // 插件清单文件
</code>
</pre>

如上所示，UTS插件-标准模式组件目录结构与UTS插件基本相同，差别在于components目录，vue组件代码存放components目录下。

其他目录文件详情可参考[UTS插件文档](https://doc.dcloud.net.cn/uni-app-x/plugin/uts-plugin.html#%E6%8F%92%E4%BB%B6%E7%9A%84%E7%9B%AE%E5%BD%95%E7%BB%93%E6%9E%84)

也就是说：**在components/xxx/xxx.uvue这个uvue组件文件中，定义组件的属性、事件、方法；然后这些属性、方法的具体实现，则调用在utssdk下的uts插件实现。**

## 开发UTS插件-标准模式组件

下面我们以一个例子来讲解标准模式组件的开发。

我们开发一个`native-button`组件，目标是把Android、iOS、harmonyOS 的原生button封装成uvue组件。

Android的原生button会带有水波纹效果。uni-app自带的button组件并没有这个效果。

该组件计划提供一个 text属性，用于显示按钮的文字；有一个buttonTap事件用于处理点击。

let's go!

### 新建标准模式组件

HBuilder X 选中你的项目，项目根目录选中`uni_modules`目录，右键选择新建`uni_modules`插件，弹窗后分类选择 “UTS插件-标准模式组件”，填写组件名称，以下均以 `native-button` 为例

![](https://web-ext-storage.dcloud.net.cn/doc/uts/uts_plugin/create-uts-vue-component.png)

创建完毕 HBuilder X 会自动创建一批模版文件，参考:[前述目录结构](#dir)

### 标准模式组件功能实现

#### 添加 native-view

上一步创建标准模式组件（名称为native-button）后，HBuilder X 会自动创建components/native-button/native-button.uvue文件。

这个文件是一个标准的uvue组件，符合easycom规范，在这个文件编写组件的属性、事件。

首先在该文件的template区添加 [native-view 组件](../component/native-view.md)

```html
<template>
	<native-view></native-view>
</template>
```

#### native-view 与 原生对象关联

native-view 组件的用途就是提供一个占位view，并且可以和原生的view进行绑定。

native-view 组件初始化会触发 @init 事件，如下代码在init时创建原生NativeButton对象，在其内部实现了view的绑定。

[NativeButton](#utscode)是在utssdk目录的app-android、app-ios、app-harmony目录下的index.uts中定义的原生对象。NativeButton对象内部处理原生view与native-view绑定关联业务。

native-button.uvue代码中用NativeButton对象调用插件相关的API。

native-view 组件在uts中对应着UniNativeViewElement对象，将 UniNativeViewElement 传递给NativeButton对象，进行关联绑定。


::: preview

> 组合式 API

```html
<template>
	<native-view @init="onviewinit"></native-view>
</template>
<script setup lang="uts">
	//引入 NativeButton 原生对象
	import { NativeButton } from "@/uni_modules/native-button";
	let button : NativeButton | null = null
	//native-view初始化时触发此方法
	function onviewinit(e : UniNativeViewInitEvent) {
		//获取UniNativeViewElement 传递给NativeButton对象
		button = new NativeButton(e.detail.element);
	}
</script>
```

> 选项式 API

```html
<template>
	<native-view @init="onviewinit"></native-view>
</template>
<script lang="uts">
	//引入 NativeButton 原生对象
	import { NativeButton } from "@/uni_modules/native-button";
	export default {
		data() {
			return {
				button: null as NativeButton | null
			}
		},
		methods: {
			//native-view初始化时触发此方法
			onviewinit(e : UniNativeViewInitEvent) {
				//获取UniNativeViewElement 实例化NativeButton将element以构造参数传递给NativeButton对象
				this.button = new NativeButton(e.detail.element);
			}
		}
	}
</script>
```

:::

#### 组件声明方法

vue中，为组件定义方法很简单，但分选项式和组合式。

选项式中，在 methods 节点中添加updateText方法；组合式中，直接定义function updateText。参考vue文档[页面调用vue组件方法](https://doc.dcloud.net.cn/uni-app-x/vue/component.html#page-call-component-method)

native-button组件使用者可调用该updateText方法中更新native-button文字。

但在vue组件的updateText方法中，需再次调用NativeButton的updateText方法，在原生插件中实现按钮文字更新。

::: preview

> 组合式 API

```ts
<script setup lang="uts">
	//引入 NativeButton 原生对象
	import { NativeButton } from "@/uni_modules/native-button";
	let button : NativeButton | null = null
	//声明方法
	function updateText(value : string) {
		button?.updateText(value)
	}
</script>
```

> 选项式 API

```ts
methods: {
	//对外函数
	updateText(value: string) {
		this.button?.updateText(value)
	}
}
```

:::

#### 组件声明属性props

native-button 声明组件属性props，例如native-button的文案信息text属性，按vue组件规范监听到text属性更新，通过NativeButton对象驱动更新原生view属性，在components/native-button/native-button.uvue编写如下代码，具体参考vue文档[vue组件Props规范](https://cn.vuejs.org/guide/typescript/composition-api#typing-component-props)

::: preview

> 组合式 API

```html
<script setup lang="uts">
	//声明属性
	const props = defineProps<{ text : string }>()
	//声明方法
	function updateText(value : string) {
		button?.updateText(value)
	}
	//监听属性变化
	watchEffect(() => {
		// console.log("watchEffect   "+props.text)
		const text = props.text
		updateText(text)
	})
</script>
```

> 选项式 API

```html
<script lang="uts">
	export default {
		props: {
			"text": {
				type: String,
				default: ''
			}
		},
		watch: {
			"text": {
				handler(newValue : string, oldValue : string) {
					this.value = newValue
					this.button?.updateText(this.value)
				},
				immediate: true
			},
		},
	}
</script>
```

:::

#### 组件声明事件

native-button 声明事件，例如原生组件触发点击事件@buttonTap, NativeButton对象通过 UniNativeViewElement 的 dispatchEvent 函数触发native-view的 @customClick 自定义事件。native-button.uvue监听native-view的 @customClick 自定义事件实现emit触发声明事件，具体参考[vue组件事件规范](https://cn.vuejs.org/guide/typescript/composition-api#typing-component-emits)

::: preview

> 组合式 API

```html
<template>
	<native-view @customClick="ontap"></native-view>
</template>
<script setup lang="uts">
	//声明事件
	const emit = defineEmits<{
		(e : "buttonTap", event : UniNativeViewEvent) : void
	}>()
	
	function ontap(e : UniNativeViewEvent) {
		emit("buttonTap", e)
	}
</script>
```

> 选项式 API

```html
<template>
	<native-view @customClick="ontap"></native-view>
</template>
<script lang="uts">
	export default {
		methods: {
			ontap(e: UniNativeViewEvent) {
				this.$emit("buttonTap", e)
			}
		}
	}
</script>
```

:::

**注意：**

目前自定义事件参数仅支持[UniNativeViewEvent](../component/native-view#uninativeviewevent)

native-button/components/native-button/native-button.uvue 最终代码如下：

::: preview

> 组合式 API

``` html
<template>
	<native-view @init="onviewinit" @customClick="ontap"></native-view>
</template>
<script setup lang="uts">
	import { NativeButton } from "@/uni_modules/native-button";
	let button : NativeButton | null = null
	
	//声明属性
	const props = defineProps<{ text : string }>()
	
	//声明事件
	const emit = defineEmits<{
		(e : "buttonTap", event : UniNativeViewEvent) : void
	}>()
	
	//声明方法
	function updateText(value : string) {
		button?.updateText(value)
	}
	
	//监听属性变化
	watchEffect(() => {
		const text = props.text
		updateText(text)
	})

	//native-view初始化时触发此方法
	function onviewinit(e : UniNativeViewInitEvent) {
		//获取UniNativeViewElement 传递给NativeButton对象
		button = new NativeButton(e.detail.element);
		updateText(props.text)
	}

	function ontap(e : UniNativeViewEvent) {
		emit("buttonTap", e)
	}

	function onUnmounted() {
		// iOS平台需要主动释放 uts 实例
		button?.destroy()
	}
</script>
```

> 选项式 API

```html
<template>
	<native-view @init="onviewinit" @customClick="ontap"></native-view>
</template>
<script lang="uts">
	import { NativeButton } from "@/uni_modules/native-button";
	export default {
		data() {
			return {
				button: null as NativeButton | null,
				value: ""
			}
		},
		props: {
			"text": {
				type: String,
				default: ''
			}
		},
		watch: {
			"text": {
				handler(newValue : string, oldValue : string) {
					this.value = newValue
					this.updateText(newValue)
				},
				immediate: true
			},
		},
		methods: {
			//native-view初始化时触发此方法
			onviewinit(e : UniNativeViewInitEvent) {
				//获取UniNativeViewElement 传递给NativeButton插件
				this.button = new NativeButton(e.detail.element);
				this.button?.updateText(this.value)
			},
			ontap(e: UniNativeViewEvent) {
				this.$emit("buttonTap", e)
			},
			updateText(value: string) {
				this.button?.updateText(value)
			}
		},
		unmounted() {
			// iOS平台需要主动释放 uts 实例
			this.button?.destroy()
		}
	}
</script>
```

:::

#### 实现NativeButton对象@utscode

utssdk目录实现不同平台的原生NativeButton对象，构造参数获取UniNativeViewElement对象与原生view绑定，封装原生view功能关联的API。

::: preview

> Android

```uts
import { Button } from "android.widget"

export class NativeButton {
	$element : UniNativeViewElement;

	constructor(element : UniNativeViewElement) {
		//接收传递过来的UniNativeViewElement
		this.$element = element;
		this.bindView();
	}

	button : Button | null = null;
	bindView() {
		//通过UniElement.getAndroidActivity()获取android平台activity 用于创建view的上下文
		this.button = new Button(this.$element.getAndroidActivity()!);  //构建原生view
		//限制原生Button 文案描述不自动大写
		this.button?.setAllCaps(false)
		//监听原生Button点击事件
		this.button?.setOnClickListener(_ => {
			const detail = {}
			//构建自定义UniNativeViewEvent返回对象
			const event = new UniNativeViewEvent("customClick", detail)
			//触发原生Button的点击事件
			this.$element.dispatchEvent(event)
		})
		//UniNativeViewEvent 绑定 安卓原生view
		this.$element.bindAndroidView(this.button!);
	}

	updateText(text: string) {
		//更新原生Button 文案描述
		this.button?.setText(text)
	}

	destroy(){
		//数据回收
	}
}
```

> iOS

```uts
import { UIButton, UIControl } from "UIKit"

export class NativeButton {

	element : UniNativeViewElement;
	button : UIButton | null;

	constructor(element : UniNativeViewElement) {
    // 接收组件传递过来的UniNativeViewElement
		this.element = element;
		super.init()
		this.bindView();
	}

	// element 绑定原生view
	bindView() {
    // 初始化原生 UIButton
    this.button = new UIButton(type=UIButton.ButtonType.system)
    // 构建方法选择器
    const method = Selector("buttonClickAction")
    // button 绑定点击回调方法
    button?.addTarget(this, action = method, for = UIControl.Event.touchUpInside)
    // UniNativeViewElement 绑定原生 view
		this.element.bindIOSView(this.button!);
	}

	updateText(text : string) {
    // 更新 button 显示文字
		this.button?.setTitle(text, for = UIControl.State.normal)
	}

	/**
	 * 按钮点击回调方法
	 * 在 swift 中，所有target-action (例如按钮的点击事件，NotificationCenter 的通知事件等)对应的 action 函数前面都要使用 @objc 进行标记。
	 */
	@objc buttonClickAction() {
    //构建自定义 UniNativeViewEvent 对象
		let event = new UniNativeViewEvent("customClick")
    //触发自定义事件
		this.element.dispatchEvent(event)
	}

	destroy() {
    // 释放 UTS 实例对象，避免内存泄露
		UTSiOS.destroyInstance(this)
	}
}
```

:::

HarmonyOS 平台，需要通过 ets 文件混编 build 构建函数来包装鸿蒙内置或三方原生组件：

- [@Builder装饰器：自定义构建函数](https://developer.huawei.com/consumer/cn/doc/harmonyos-guides-V5/arkts-builder-V5?ha_source=Dcloud&ha_sourceId=89000448)
- [build()函数的语法限制](https://developer.huawei.com/consumer/cn/doc/harmonyos-guides-V5/arkts-create-custom-components-V5#build函数?ha_source=Dcloud&ha_sourceId=89000448)

::: preview

> HarmonyOS - index.uts

```uts
import { BuilderNode } from "@kit.ArkUI"
// 导入混编实现的声明式UI构建函数
import { buildButton } from "./builder.ets"

import { INativeButtonContext } from "../interface.uts"
// 定义 buildButton 的参数类型
interface NativeButtonOptions {
    text : string
    onClick : () => void
}

export class NativeButton {
    private $element : UniNativeViewElement;
    private builder : BuilderNode<[NativeButtonOptions]> | null = null
    // 初始化 buildButton 默认参数
    private params : NativeButtonOptions = {
        text: '',
        onClick: () => {
            this.$element.dispatchEvent(new UniNativeViewEvent("customClick", {}))
        }
    }

    constructor(element : UniNativeViewElement) {
        // 绑定 wrapBuilder 函数
        this.builder = element.bindHarmonyWrappedBuilder(wrapBuilder<[NativeButtonOptions]>(buildButton), this.params)
        this.$element = element
        // 绑定当前实例为自定义的controller，方便其他地方通过 element 获取使用
        this.$element.bindHarmonyController(this)
    }

    updateText(text : string) {
        this.params.text = text
        // 调用 builder update 函数来更新 UI
        this.builder?.update(this.params)
    }
}


class NativeButtonContext implements INativeButtonContext {
    private controller : NativeButton
    constructor(element : UniNativeViewElement) {
        // 获取自定义的 controller
        this.controller = element.getHarmonyController<NativeButton>()!
    }
    updateText(text : string) {
        // 调用 controller 来更新文字
        this.controller?.updateText(text)
    }
}
/**
 * 递归查询
 */
function iterateElement(homeElement : UniElement) : UniNativeViewElement | null {
    if ("NATIVE-VIEW" == homeElement.nodeName) {
        return homeElement as UniNativeViewElement
    }
    for (const perChildEle of homeElement.children) {
        let findEle = iterateElement(perChildEle)
        if (findEle != null) {
            return findEle
        }
    }

    return null
}



export function createNativeButtonContext(id : string, ins : ComponentPublicInstance | null = null) : INativeButtonContext | null {
    if (ins == null) {
        const pages = getCurrentPages()
        if (pages.length > 0) {
            const page = pages[pages.length - 1]
            const rootViewElement = page.getElementById(id)
            if (rootViewElement != null) {
                /**
                 * 找到了root节点，递归检索目标 native-view
                 */
                const nativeViewElement = iterateElement(rootViewElement)
                if (nativeViewElement != null) {
                    return new NativeButtonContext(nativeViewElement)
                }
            }
        }
    } else {
        /**
         * 尝试迭代遍历
         */
        if (ins.$el != null) {
            const nativeViewElement = iterateElement(ins.$el as UniElement)
            if (nativeViewElement != null) {
                return new NativeButtonContext(nativeViewElement)
            }
        }
    }

    return null
}

```

> HarmonyOS - builder.ets

```uts
@Builder
export function buildButton(params: ESObject) {
  Button(params.text, { type: ButtonType.Normal, stateEffect: true })
    .borderRadius(8)
    .backgroundColor(0x317aff)
    .onClick(() => {
        params.onClick()
    })
    .attributeModifier(params.attributeUpdater)
}
```

:::

更多实现可参考 标准模式组件 [native-button](https://gitcode.net/dcloud/hello-uni-app-x/-/tree/dev/uni_modules/native-button)

此时一个简单的UTS插件-标准模式组件就完成了，

**注意:**
+ UTS插件-标准模式组件的 components 目录下的代码中不应该含有原生平台任何引用对象，这会导致vue原生组件无法跨平台，推荐与原生平台关联的代码都放在utssdk中
+ 绑定原生 view 方法（bindAndroidView、bindIOSView）仅支持调用一次，原生 view 一旦绑定后不支持再次绑定其他 view
+ ios平台需要vue组件主动释放 uts 实例，所以页面触发 unmounted 生命周期时需要调用 this.button?.destroy() 避免内存泄露
+ android平台 native-view 组件不支持border、background、box-shadow属性，可以使用view标签包裹native-view,在view标签设置以上属性
+ 鸿蒙原生应用中px是指物理像素，而在uni-app的css样式中px是指逻辑像素。上述@Builder内纯数值尺寸对应的是鸿蒙的vp单位（逻辑像素）等同于css样式中的px。鸿蒙原生单位文档[详见](https://developer.huawei.com/consumer/cn/doc/harmonyos-references/ts-pixel-units?ha_source=Dcloud&ha_sourceId=89000448)


### 页面引用UTS插件-标准模式组件@pagecode

以 native-button 为例, 创建标准模式组件的项目页面可以直接使用 native-button 标签，也可将native-button插件包放置其他项目的uni-modules文件夹中。项目页面即可使用 native-button 标签

```html
<template>
	<view style="flex:1">
		<native-button class="native-button" text="buttonText" @buttonTap="ontap"></native-button>
	</view>
</template>

<script>
	export default {
		methods: {
			ontap(e : UniNativeViewEvent) {
			  console.log("ontap----------"+e.type)
			}
		}
	}
</script>

<style>
	.native-button {
		height: 100px;
		width: 200px;
		margin: 25px auto 25px auto;
	}
</style>
```

在hello uni-app x中，有native-button的完整示例。集成native-button的页面在pages/component/native-view/native-view.uvue，native-button组件在uni_modules/native-button/components/native-button/中。


### 组件上下文

#### 组件上下文简介

某些情况下，我们的组件需要对外提供能力，比如：

 + Video组件需要提供控制视频播放状态的能力

 + Map组件需要提供添加/查询标记点的能力


这类场景下，我们建议开发者以上下文（Context）的形式对外提供能力，方便调用者使用和管理

下面我们以 `Native-Button` 为例，进行说明：

我们会给 `Native-Button` 组件创建一个 `NativeButtonContext`类型，组件的使用者可以通过使用`createNativeButtonContext` 得每个组件实例对应的上下文对象。后续开发者可以通过操作`NativeButtonContext`实现对 `Native-Button` 原生组件的操作和管理


#### 创建组件上下文

在 `uni_modules\utssdk\interface.uts` 文件中添加如下代码：

```uts
/**
 * 原生组件的上下文对象
 */
export interface INativeButtonContext {
	/**
	 * 更新文本示例
	 */
	updateText(text : string) : void
}

export type CreateNativeButtonContext = (
	id : string,
	component ?: ComponentPublicInstance | null
) => INativeButtonContext | null;
```

上面的代码中， 我们声明了一个`INativeButtonContext`接口，提供了更新文本内容的方法 `updateText`

接下来，我们在不同平台目录实现该接口定义

::: preview

> app-android/index.uts

```uts
import { INativeButtonContext } from "../interface.uts"

class NativeButtonContext implements INativeButtonContext {

	private button : Button | null = null
	constructor(button : Button) {
		this.button = button
	}
	updateText(text : string) {
		this.button?.setText(text)
	}
}
```

> app-ios/index.uts

```uts
import { INativeButtonContext, CreateNativeButtonContext } from "../interface.uts"

class NativeButtonContext implements INativeButtonContext {
	btn: UIButton | null 
	constructor(element : UniNativeViewElement) {
		let view = element.getIOSView()
		if (view != null) {
			this.btn = view as UIButton
		}
	}
	updateText(title: string) : void {
		this.btn?.setTitle(title, for = UIControl.State.normal)   
	}	  
}
```

> app-harmony/index.uts

```uts
import { INativeButtonContext } from "../interface.uts"


class NativeButtonContext implements INativeButtonContext {
    private controller : NativeButton
    constructor(element : UniNativeViewElement) {
        // 获取自定义的 controller
        this.controller = element.getHarmonyController<NativeButton>()!
    }
    updateText(text : string) {
        // 调用 controller 来更新文字
        this.controller?.updateText(text)
    }
}

```

:::

上面的代码创建了对应的实现类 `NativeButtonContext`，实现了更新文本内容的功能,需要注意在构建上下文对象时，要将具体待操作的原生对象作为入参传入。

#### 组件布局注意事项

需要特别注意： `native-button.uvue`文件的 <template> 节点下，仅建议存在一个节点：即 如果你的native-view 不存在兄弟节点，则直接以native-view为根节点

```vue
<template>
	<native-view  style="height: 100px;" @init="onviewinit" @customClick="ontap"></native-view>
</template>
```

如果 native-view 存在兄弟节点，则需要在外层包裹一个view 作为根节点

```vue
<template>
	<view >
		<native-view  style="height: 100px;" @init="onviewinit" @customClick="ontap"></native-view>
		<view style="width: 50%;height: 100px;">
			<button>测试按钮</button>
		</view>
	</view>
	
</template>
```


如果一定需要 native-view 存在兄弟节点 且不能包裹父容器，即 多个根节点的情况。 这种情况建议使用[v-bind 标签](https://vuejs.org/api/built-in-directives.html#v-bind) 解决


#### 对外提供上下文创建函数

::: preview

> app-android\index.uts

```uts

export function createNativeButtonContext(id : string, ins : ComponentPublicInstance | null = null) : INativeButtonContext | null {
	if (ins == null) {
		const pages = getCurrentPages()
		if (pages.length > 0) {
			const page = pages[pages.length - 1]
			const rootViewElement = page.getElementById(id)
			if (rootViewElement != null) {
				/**
				 * 找到了root节点，递归检索目标 native-view
				 */
				const nativeViewElement = iterateElement(rootViewElement)
				if (nativeViewElement != null) {
					return new NativeButtonContext(nativeViewElement.getAndroidView()! as Button)
				}
			}
		}
	} else {
		/**
		 * 尝试迭代遍历
		 */
		if (ins.$el != null) {
			const nativeViewElement = iterateElement(ins.$el as UniElement)
			if (nativeViewElement != null) {
				return new NativeButtonContext(nativeViewElement.getAndroidView()! as Button)
			}
		}
	}

	return null
}

/**
 * 递归查询
 */
function iterateElement(homeElement:UniElement):UniElement | null{
	if("NATIVE-VIEW" == homeElement.nodeName){
		return homeElement
	}
	for(perChildEle of homeElement.children){
		let findEle = iterateElement(perChildEle)
		if(findEle != null){
			return findEle
		}
	}
	
	return null
}

```

> app-ios\index.uts

```uts
export const createNativeButtonContext: CreateNativeButtonContext =  function (id : string, component ?: ComponentPublicInstance | null) : INativeButtonContext | null {
	let element : UniNativeViewElement | null = null;
	let e: UniElement | null = null;
	if (component == null) {  
		e = uni.getElementById(id)  
	} else { 
		e = component?.$el as UniElement | null;
	}
	if (e instanceof UniNativeViewElement) {   
		element = e as UniNativeViewElement | null
	}else {
		element = getNativeViewElemet(e)
	}

	if (element == null) return null;  
	return new NativeButtonContext(element!);
}  

/**
 * 递归查询
 */
function getNativeViewElemet(element: UniElement | null): UniNativeViewElement | null {
	if (element == null) {
		return null;
	}
	if (element instanceof UniNativeViewElement) {
		return element as UniNativeViewElement
	}
	for (item in element!.children) {
		let tmp = getNativeViewElemet(item)
		if (tmp != null) {
			return tmp
		}
	} 
	return null
}

```

> app-harmony\index.uts

```uts

export function createNativeButtonContext(id : string, ins : ComponentPublicInstance | null = null) : INativeButtonContext | null {
    if (ins == null) {
        const pages = getCurrentPages()
        if (pages.length > 0) {
            const page = pages[pages.length - 1]
            const rootViewElement = page.getElementById(id)
            if (rootViewElement != null) {
                /**
                 * 找到了root节点，递归检索目标 native-view
                 */
                const nativeViewElement = iterateElement(rootViewElement)
                if (nativeViewElement != null) {
                    return new NativeButtonContext(nativeViewElement)
                }
            }
        }
    } else {
        /**
         * 尝试迭代遍历
         */
        if (ins.$el != null) {
            const nativeViewElement = iterateElement(ins.$el as UniElement)
            if (nativeViewElement != null) {
                return new NativeButtonContext(nativeViewElement)
            }
        }
    }

    return null
}

/**
 * 递归查询
 */
function iterateElement(homeElement : UniElement) : UniNativeViewElement | null {
    if ("NATIVE-VIEW" == homeElement.nodeName) {
        return homeElement as UniNativeViewElement
    }
    for (const perChildEle of homeElement.children) {
        let findEle = iterateElement(perChildEle)
        if (findEle != null) {
            return findEle
        }
    }

    return null
}

```

:::

上面的代码中需要两个入参：

+ 组件id 也就是 native-button 设置的id属性，用来实现在 页面中定位当前组件，必填

+ ComponentPublicInstance 可选，在自定义组件中使用标准组件能力时需要传入此对象

> 特别注意：

> 上述 iOS 平台代码中 `component?.$el` 的用法需要 HBuilder X 4.44及以上版本才能使用。

#### 使用

在uvue页面中使用组件上下文

```vue

<template>
	<view>
		<page-head :title="title"></page-head>

		<view class="uni-btn-v uni-common-mt">
			<button type="primary" @tap="testCallMethod">调用组件方法</button>
			<native-button id="helloView" class="native-button" text="buttonText" @buttonTap="ontap"></native-button>
		</view>

	</view>

</template>
<script>
	import { createNativeButtonContext } from "../../uni_modules/native-button";

	export default {
		data() {
			return {
				title: '组件能力封装示例',
			}
		},
		methods: {
			ontap(e : UniNativeViewEvent) {
				console.log("ontap----------" + e.type)
			},

			testCallMethod: function () {
				let context = createNativeButtonContext("helloView")
				context?.updateText("test code")
			}
		}
	}
</script>

<style>
	.native-button {
		height: 100px;
		width: 200px;
		margin: 25px auto 25px auto;
	}
</style>
```

在自定义组件中使用组件上下文:

```vue
<template>
	<view>
		<button type="primary" @tap="testCallMethod">调用组件方法</button>
		<native-button id="helloView" class="native-button" text="自定义组件"></native-button>
	</view>
</template>

<script>
	import { createNativeButtonContext } from "../../uni_modules/native-button";

	export default {
		data() {
			return {
				title: '组件能力封装示例',
			}
		},

		methods: {
			ontap(e : UniNativeViewEvent) {
				console.log("ontap----------" + e.type)
			},

			testCallMethod: function () {
				let context = createNativeButtonContext("helloView", this)
				context?.updateText("test code")
			}

		}
	}
</script>

```

以上便是组件上下文相关的内容介绍





## 总结@sum

通过vue方式开发组件非常直观。它分为几个核心步骤：
1. 在uni_modules/xxx/components/xxx/xxx.uvue文件中，通过标准vue方式定义组件的属性、方式、事件
2. 在uni_modules/xxx/components/xxx/xxx.uvue文件中，提供native-view组件，用于和原生的view绑定。在native-view组件的init事件中，调用utssdk下封装的原生对象的new初始化，将native-view对应的UniNativeViewElement传入
3. 在uni_modules/xxx/utssdk/app-android等目录下的/index.uts文件中，通过uts的方式，定义原生对象，在构造时接收components/xxx/xxx.uvue组件传来的UniNativeViewElement，并创建一个真正的原生view，和UniNativeViewElement绑定。
4. 在uni_modules/xxx/utssdk/app-android等目录下的/index.uts文件中，定义原生对象的各种方法，供components/xxx/xxx.uvue组件在其属性变化和方法调用时调用。
