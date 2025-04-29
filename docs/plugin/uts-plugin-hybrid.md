# UTS原生混编介绍

UTS插件中（具体是uni_modules下的utssdk目录），支持app-Android、app-iOS、app-harmony、web、mp-weixin等平台专用目录。
这些专用目录，可以直接放置原生的`kotlin`、`java`、`swift`、`ArkTS`代码，和`uts`代码混合使用，即 `UTS原生混编`(下文简称：原生混编)

UTS插件的入口仍然是uts代码，但在uts代码里，可以直接调用插件下的kotlin、swift代码中的函数、对象。

在[UTS插件](./uts-plugin.md)的编译流程中，`UTS`本身就会被编译为`Kotlin`/`swift`/`ArkTS` 源码。所以 `UTS` 调用原生代码的过程，**本质是同一语言内部，不同函数/对象之间的调用过程，无缝且不会有任何序列化等性能损耗**

同时在HBuilderX的真机运行中，可以直接改动uts或kotlin、swift、ArkTS代码、整体联编、直接运行，无需打包自定义基座。（java代码仍需打包自定义基座）

甚至在原生代码中，也可以使用console.log，把日志打印在HBuilderX的控制台中。

## UTS原生混编的优势和使用场景

`原生混编`出现之前，开发者只能使用[UTS语言](../uts/README.md) 来开发[UTS插件](./uts-plugin.md) 

不管是网上搜的还是历史存留的，当涉及到原生的kotlin、java、swift、ArkTS代码时，开发者要不把这些代码自行翻译成uts代码，要不把这些代码封装成aar、framework等包，再被uts引用。

有时会遇到uts还不支持的语法，只能使用原生语言，就必须封装库了。

如原生代码数量较多，则翻译比较吃力；如封装成aar等库，每次改动都需要打自定义基座，也很麻烦。

`UTS原生混编`的解决了上述问题：

开发者只需要把 `Kotlin`/`swift`/`java`/`ArkTS` 代码放在`UTS插件`目录中，就可以通过 `UTS`直接使用这些代码。

并且和`uts代码`一样，混编的原生代码（除java）可以直接真机运行，省去了手动集成`AAR`等三方库后打包自定义基座的环节，提升了开发效率。


下面我们以`内存监控`功能为例，分别拆解 `UTS原生混编`技术在`Android`和`ios`平台上的使用步骤


#### 前置条件

在开始使用 `UTS原生混编`之前，开发者需要确保两个前置条件：

1  `HBuidlerX` 4.25 以上版本

2  对[UTS插件](./uts-plugin.md)具备一定的了解和开发经验

## Android平台

在进行下一步的操作之前，你的目录应该是这样的：

![目录](https://web-ext-storage.dcloud.net.cn/doc/uts/uts_hybrid_plugin/bybrid_android_start.png)


#### 第一步 获取和验证原生代码

原生代码的获取有以下方式：

+ 1 [Android官方文档](https://developer.android.google.cn/?hl=zh-cn)

![获取代码](https://web-ext-storage.dcloud.net.cn/doc/uts/uts_hybrid_plugin/hybrid_android_getcode_2.png)

+ 2 搜索引擎/AI工具

![获取代码](https://web-ext-storage.dcloud.net.cn/doc/uts/uts_hybrid_plugin/hybrid_android_getcode.png)

AI工具或官方文档得到的代码并不总是准确的，我们需要去验证它。

目前`HBuilderX`并未提供原生代码的语法提示和校验，因此我们建议：

+ 如果编写大段原生代码，推荐在原生IDE(比如：`AndroidStudio`)中编写，再放入`UTS插件`混编联调

+ 如果是小的代码片段，可以直接放入`UTS插件`目录，依靠`HBuilderX`本地编译和打log功能来完成原生代码的校验


这里我们选择直接集成`UTS插件`, 使用`HBuilderX`来验证


#### 第二步 集成原生代码

`Kotlin`/`Java`语言中，存在[包名](https://kotlinlang.org/docs/packages.html) 的概念，类似`swift`的命名空间。为了让我们的原生代码可以被`UTS`使用，我们需要确保原生代码的包名是正确的:

大多数情况下，我们建议混编代码的包名与[UTS插件默认包名](https://doc.dcloud.net.cn/uni-app-x/plugin/uts-for-android.html#_3-1-%E9%85%8D%E7%BD%AEandroidmanifest-xml)保持一致，这样在UTS调用原生代码时，可以省去手动引入包名的步骤。

```kotlin
// 如 uts 插件目录为 uni-MemoryInfo 时默认混编代码的包名为
package uts.sdk.modules.uniMemoryInfo
```

如果混编代码的包名与`UTS插件默认包名`不一致，则需要像使用原生对象一样手动引入

```ts
// 如 kotlin 源码中指定包名为 uts.memoryinf.android，则按以下方式引用
import { MemoryInfoNative } from 'uts.memoryinf.android'
// 或使用以下方式引用
//import MemoryInfoNative from 'uts.memoryinf.android.MemoryInfoNative'
```


回到我们的示例，现在整理完的`Kotlin`代码是这样的：

```kotlin
// 设置原生包名  
package uts.memoryinf.android;


// 引用系统类  
import android.app.ActivityManager
import android.content.Context.ACTIVITY_SERVICE
// 引用 uts 基础库相关类
import io.dcloud.uts.UTSAndroid
import io.dcloud.uts.setInterval
import io.dcloud.uts.clearInterval
import io.dcloud.uts.console


/**
 * 原生 Kotlin 语言实现封装类  
 */
object MemoryInfoNative {

  /**
   * 获取内存信息
   */
  fun getMemInfoKotlin():Array<Number>{
    val activityManager = UTSAndroid.getUniActivity()?.getSystemService(ACTIVITY_SERVICE) as ActivityManager
    val memoryInfo = ActivityManager.MemoryInfo()
    activityManager.getMemoryInfo(memoryInfo)
    val availMem = memoryInfo.availMem / 1024 / 1024
    val totalMem = memoryInfo.totalMem / 1024 / 1024
    // availMem 可用内存，单位MB
    // totalMem 设备内存，单位MB
    // console.log(availMem,totalMem)
    return arrayOf(availMem,totalMem)
  }

}

```

上面的代码中，我们将获取内存的信息的功能以`Kotlin`静态方法`MemoryInfoNative.getMemInfoKotlin()` 的形式对外暴露

接下来，我们将整理好的原生代码添加到 在 `app-android` 目录下的 `MemoryInfoNative.kt` 中：  

```text
└── uni-MemoryInfo
    ├── package.json
    └── utssdk
        ├── app-android
        │   ├── config.json
        │   ├── index.uts
        │   └── MemoryInfoNative.kt
        ├── app-ios
        └── interface.uts
```

> 注意：java代码需要云打包自定义基座后生效，kotlin代码不需要打包，标准基座即可生效

是的，就是这样简单。如图所示，我们已经完成了对原生代码的集成。


#### 第三步 在原生代码中调用UTS内置对象

为了让原生代码中方便调用UTS的内置对象，尤其是用于数据类型转换。官方提供了在原生代码调用UTS对象的方法。

UTS的[内置对象](../uts/buildin-object-api/number.md)和[平台专用对象](../uts/utsandroid.md)均可以在原生环境使用，下面以kotlin中打印日志到HBuilder X 控制台为例说明：

第一步：手动导入对应的包名，包名规则为： `io.dcloud.uts.xxx` 。这里的 xxx 是具体的对象的类名 ：

```kotlin
import io.dcloud.uts.console // kt或java代码
```

第二步： 导入包名后，以原生方式使用内置对象

```kotlin
console.log("Hello World") // kt或java代码
```

这样就实现了在kt或java代码中打印日志到HBuilderX的控制台。

不过这个导入和使用过程将没有代码提示，输出的变量信息也不会包含变量所在的文件和代码行号等信息。

下面列出内置对象对应的类名，如果需要在原生环境和UTS环境/uvue环境中互传数据，建议转换为标准内置对象实现后再进行传递。


##### uts和kotlin对象映射表

|uts 内置对象		|编译成的原生类名		 
|:----		|:---						
|Array		|io.dcloud.uts.UTSArray		
|Number		|kotlin.Number	
|String		|kotlin.String				
|Set		|io.dcloud.uts.Set			
|Map		|io.dcloud.uts.Map			
|UTSJSONObject|io.dcloud.uts.UTSJSONObject
|JSON		|io.dcloud.uts.JSON			
|Date		|io.dcloud.uts.Date			
|Math		|io.dcloud.uts.Math			
|Promise	|io.dcloud.uts.UTSPromise	
|RegExp		|io.dcloud.uts.UTSRegExp	
|Error		|io.dcloud.uts.UTSError		
|console	|io.dcloud.uts.console		


在上面的示例中，已经实现了获取当前系统内存的功能，如果还想更进一步持续监控内存变化，并通过回调返回数据。

实现持续调用的方法有很多，这里我们为了演示在`Kotlin`代码中调用`UTS内置对象`的情况，选择采用[setInterval](../uts/buildin-object-api/timers.md#setinterval-handler-timeout-arguments)来实现这个功能:

使用 [UTS内置对象](../uts/buildin-object-api/number.md) 需要注意两点：

+ 正确引入类名：

  `UTS内置对象`在具体的平台会有一个对应的类名，举例： UTS内置的[Array](../uts/buildin-object-api/array.md) 对应 `Kotlin`中的`io.dcloud.uts.UTSArray`

+ 正确的处理原生对象和内置对象直接的转换

  当前示例中不涉及，但如果开发者可能遇到类似 kotlin.Array 转换 UTS内置Array的情况，这种情况可以通过查阅[内置对象文档](../uts/buildin-object-api/number.md)来解决。


> 完整的内置对象实现清单和与原生对象转换代码示例，大家都可以在内置对象文档的具体章节找到


原生`kotlin`代码的最终形态:

```kotlin
// 设置原生包名  
package uts.memoryinf.android;


// 引用系统类  
import android.app.ActivityManager
import android.content.Context.ACTIVITY_SERVICE
// 引用 uts 基础库相关类
import io.dcloud.uts.UTSAndroid
import io.dcloud.uts.setInterval
import io.dcloud.uts.clearInterval
import io.dcloud.uts.console


/**
 * 原生 Kotlin 语言实现封装类  
 */
object MemoryInfoNative {
  /**
   * 记录上一次的任务id
   */
  private var lastTaskId:Number = -1

  /**
   * 获取内存信息
   */
  fun getMemInfoKotlin():Array<Number>{
    val activityManager = UTSAndroid.getUniActivity()?.getSystemService(ACTIVITY_SERVICE) as ActivityManager
    val memoryInfo = ActivityManager.MemoryInfo()
    activityManager.getMemoryInfo(memoryInfo)
    val availMem = memoryInfo.availMem / 1024 / 1024
    val totalMem = memoryInfo.totalMem / 1024 / 1024
    // availMem 可用内存，单位MB
    // totalMem 设备内存，单位MB
    // console.log(availMem,totalMem)
    return arrayOf(availMem,totalMem)
  }

  /**
   * 开始监听内存信息变化
   */
  fun onMemoryInfoChangeKotlin(callback: (Array<Number>) -> Unit){
    if(lastTaskId != -1){
      // 避免重复开启
      clearInterval(lastTaskId)
    }

    // 延迟1000ms，每2000ms 获取一次内存
    lastTaskId = setInterval({
      val activityManager = UTSAndroid.getUniActivity()?.getSystemService(ACTIVITY_SERVICE) as ActivityManager
      val memoryInfo = ActivityManager.MemoryInfo()
      activityManager.getMemoryInfo(memoryInfo)
      val availMem = memoryInfo.availMem / 1024 / 1024
      val totalMem = memoryInfo.totalMem / 1024 / 1024
      // availMem 可用内存，单位MB
      // totalMem 设备内存，单位MB
      // console.log(availMem,totalMem)
      callback(arrayOf(availMem,totalMem))
    },1000,2000)
  }

  /**
   * 停止监听内存信息变化
   */
  fun offMemoryInfoChangeKotlin(){
    if(lastTaskId != -1){
      // 避免重复开启
      clearInterval(lastTaskId)
    }
  }

}
```

上面的代码中，我们将监听内存变化的功能以`Kotlin`静态方法`MemoryInfoNative.onMemoryInfoChangeKotlin(callback)` 的形式对外暴露。 

注意：这里的 `callback` 类型为 `(Array<Number>) -> Unit`, 其中参数为 Number 数组类型（注：这里的数组为`kotlin.Array`，与 uts 中的数组有区别），无返回值（类型为`Unit`）。

至此，内存监控功能的原生代码部分已经完全开发完毕。接下来，我们编写UTS代码来使用它。


#### 第四步 编写`UTS`调用原生代码

如我们在前文所讲，`UTS`是`Kotlin`语言的上游语言。所有`Kotlin`代码中的：`类`、`对象`、`函数`、`变量`，均可以在uts中直接使用。 

**但是反之则不行**。

因为`UTS`的编译器兼容了`Kotlin`的语法规则，所以`UTS`中调用`Kotlin`代码可以被很好的支持，即使升级HBuilderX版本也不会有什么问题。

但`UTS`从未保证过编译`Kotlin`的具体规则。所以虽然开发者可以通过一些取巧的方式实现 在`Kotlin`中调用`UTS`代码，但这是不安全的，`HBuilderX`升级后，类似的代码可能会失效/异常。


在示例里 `uni-MemoryInfo/utssdk/app-android/index.uts` 文件中，使用 uts 代码封装为插件导出 API：

```ts
import { GetMemoryInfo, OnMemoryInfoChange, OffMemoryInfoChange } from '../interface.uts'
import { MemoryInfoNative } from 'uts.memoryinf.android'


/**
 * 获取内存信息
 */
export const getMemoryInfo : GetMemoryInfo = function () : Array<number> {
  let kotlinArray = MemoryInfoNative.getMemInfoKotlin()
  // 将原生 Kotlin 语言的数组转换成 uts 语言中的数组
  return Array.fromNative(kotlinArray)
}


/**
 * 开始监听内存信息变化
 */
export const onMemoryInfoChange : OnMemoryInfoChange = function (callback : (res : Array<number>) => void) {
  //避免原生语言与uts语言的Array类型，这里需使用kotlin.Array
  MemoryInfoNative.onMemoryInfoChangeKotlin(function (res : kotlin.Array<number>) {
    // 将原生 Kotlin 语言的数组转换成 uts 语言中的数组
    callback(Array.fromNative(res))
  })
}


/**
 * 停止监听内存信息变化
 */
export const offMemoryInfoChange : OffMemoryInfoChange = function () {
  MemoryInfoNative.offMemoryInfoChangeKotlin()
}
```

上面的代码，我们在`UTS`中使用 `Array.fromNative(kotlinArray)` 将 kotlin 数组转换为 uts 数组。



## iOS平台

在进行下一步的操作之前，你的目录应该是这样的：

![目录](https://web-ext-storage.dcloud.net.cn/doc/uts/uts_hybrid_plugin/bybrid_ios_start.png)


#### 第一步 获取和验证原生代码

原生代码的获取有以下方式：

1 原生官方文档：
 + [swift官方文档](https://docs.swift.org/swift-book/)
 + [swift 中文版](https://swiftgg.gitbook.io/swift/)


2 搜索引擎/AI工具

AI工具或官方文档得到的代码并不总是准确的，我们需要去验证它。

目前`HBuilderX`并未提供原生代码的语法提示和校验，因此我们建议：

+ 如果编写大段原生代码，推荐在原生IDE(比如：`Xcode`)中编写验证，再放入`UTS插件`混编联调

+ 如果是小的代码片段，可以直接放入`UTS插件`目录，依靠`HBuilderX`本地编译和打log功能来完成原生代码的校验


这里我们选择直接集成`UTS插件`, 使用`HBuilderX`来验证

#### 第二步 集成原生代码

swift 文件默认会引入原生系统库 `Foundation`, 如果需要调用 UI 相关的代码，则需要引入 `UIKit`,

如果你需要使用 uts 内置对象，则需要引入 uts 基础库 `DCloudUTSFoundation`。

回到我们的示例，现在整理完的`swift`代码是这样的：

```swift
// 引用原生系统库  
import Foundation
// 引用 uts 基础库  
import DCloudUTSFoundation


/**
 * 原生 Swift 语言实现封装类  
 */
public class MemoryInfoNative {
  /**
   * 获取内测信息
   */
  static func getMemInfoSwift() -> [Int] {
    let freeMem = MemoryInfoNative.getFreeMemory()
    let totalMem = MemoryInfoNative.getTotalMemory()
    // console.log(freeMem, totalMem)
    return [freeMem, totalMem]
  }

  /**
   * 获取总内存大小（以MB为单位）
   * Returns: 设备总内存
   */
  static private func getTotalMemory() -> Int {
    return Int(ProcessInfo.processInfo.physicalMemory / 1024 / 1024)
  }

  /**
   * 获取可用内存大小（以MB为单位）
   * Returns: 设备可用内存
   */
  static private func getFreeMemory() -> Int {
    var vmStats = vm_statistics_data_t()
    var infoCount = mach_msg_type_number_t(MemoryLayout<vm_statistics_data_t>.size / MemoryLayout<integer_t>.size)
    let kernReturn = withUnsafeMutablePointer(to: &vmStats) {
      $0.withMemoryRebound(to: integer_t.self, capacity: Int(infoCount)) {
        host_statistics(mach_host_self(), HOST_VM_INFO, $0, &infoCount)
      }
    }

    if kernReturn != KERN_SUCCESS {
      return 0
    }

    let vmPageSize = vm_page_size
    let freeMemorySize = Int(vmPageSize) * Int(vmStats.free_count + vmStats.inactive_count)
    return freeMemorySize / 1024 / 1024
  }

}
```

上面的代码中，我们将获取内存的信息的功能以`swift`静态方法`MemoryInfoNative.getMemInfoSwift()` 的形式对外暴露。 而获取内存信息具体功能的实现，则是由两个原生方法`getTotalMemory`、`getFreeMemory`实现。

接下来，我们将整理好的原生代码添加到 在 `app-ios` 目录下的 `MemoryInfoNative.swift` 中：  

```text
└── uni-MemoryInfo
    ├── package.json
    └── utssdk
        ├── app-android
        ├── app-ios
        │   ├── config.json
        │   ├── index.uts
        │   └── MemoryInfoNative.swift
        └── interface.uts
```


是的，就是这样简单。如图所示，我们已经完成了对原生代码的集成。


#### 第三步 在原生代码中调用UTS内置对象

UTS的[内置对象](../uts/buildin-object-api/number.md)和[平台专用对象](../uts/utsios.md)均可以在原生环境使用，
但是在使用前需要导入基础库 `DCloudUTSFoundation`。

##### 原生代码使用 console 向 HX 控制台输出打印日志			

首先将基础库 `DCloudUTSFoundation` 导入到 swift 源码文件中，不过这个导入和使用过程将没有代码提示，输出的变量信息也不会包含变量所在的文件和代码行号等信息。

示例如下：

```swift

import DCloudUTSFoundation;

func test1() -> String {
    console.log("this is in swift file")
    return "123"
}
```

##### 原生代码使用 UTSiOS 对象

如果你想在 swift 代码中使用 `UTSiOS` 对象提供的能力，你需要先导入基础库 `DCloudUniappRuntime`.

示例如下：

```swift

import DCloudUniappRuntime;

func getKeyWindow() -> UIWindow {
    return UTSiOS.getKeyWindow()
}
```


##### uts和Swift对象映射表
我们知道在 uts 中使用的 uts 内置对象会被编成原生类型，那么在混编的 swift 文件中要想使用 uts 内置对象，就要直接使用其编译后的原生类型。
下面列出 uts 内置对象对应的 swift 原生类名

|uts 内置对象		|编译成的原生类名		  			
|:----			|:---						
|Array			|Array						
|Number			|NSNumber 					
|String			|String						
|Set			|UTSSet						
|Map			|Map						
|UTSJSONObject	|UTSJSONObject				
|JSON			|JSON						
|Date			|Date						
|Math			|Math										
|RegExp			|UTSRegExp					
|Error			|UTSError					
|console		|console					

在上面的示例中，已经实现了获取当前系统内存的功能，如果还想更进一步持续监控内存变化，并通过回调返回数据。

实现持续调用的方法有很多,这里我们为了演示在`swift`代码中调用`UTS内置对象`的情况，选择采用[setInterval](../uts/buildin-object-api/timers.md#setinterval-handler-timeout-arguments)来实现这个功能:


使用 [UTS内置对象](../uts/buildin-object-api/number.md) 需要注意两点：

1 正确引入类名：

`UTS内置对象`在具体的平台会有一个对应的类名，举例： UTS内置的[Set](../uts/buildin-object-api/array.md) 对应 `swift`中的`UTSSet`

2 正确的处理原生对象和内置对象直接的转换

下面的示例代码中涉及到 uts 内置对象 `Number` 转换成原生类型 `Int` 的例子，实现方式为 `toInt()`, 具体见示例代码。如果你遇到其他类型的转换，请查阅内置对象文档来确定转换方法。


> 完整的内置对象实现清单和与原生对象转换代码示例，大家都可以在内置对象文档的具体章节找到


原生`swift`代码的最终形态:

```swift
// 引用原生系统库  
import Foundation
// 引用 uts 基础库  
import DCloudUTSFoundation


/**
 * 原生 Swift 语言实现封装类  
 */
public class MemoryInfoNative {
  // 记录上一次的任务id
  static private var lastTaskId = -1

  /**
   * 获取内测信息
   */
  static func getMemInfoSwift() -> [Int] {
    let freeMem = MemoryInfoNative.getFreeMemory()
    let totalMem = MemoryInfoNative.getTotalMemory()
    // console.log(freeMem, totalMem)
    return [freeMem, totalMem]
  }


  /**
   * 开始监听内存信息变化
   */
  static func onMemoryInfoChangeSwift(_ callback: @escaping (_ res: [Int]) -> Void) {
    if lastTaskId != -1 {
      // 避免重复开启
      clearInterval(NSNumber.from(lastTaskId))
    }
        
    lastTaskId = setInterval({ 
      let freeMem = MemoryInfoNative.getFreeMemory()
      let totalMem = MemoryInfoNative.getTotalMemory()
      // console.log(freeMem, totalMem)
      callback([freeMem, totalMem])
    }, 2000).toInt()
  }

  /**
   * 停止监听内存信息变化
   */
  static func offMemoryInfoChangeSwift() {
    if lastTaskId != -1 {
      clearInterval(NSNumber.from(lastTaskId))
      lastTaskId = -1
    }
  }

  /**
   * 获取总内存大小（以MB为单位）
   * Returns: 设备总内存
   */
  static private func getTotalMemory() -> Int {
    return Int(ProcessInfo.processInfo.physicalMemory / 1024 / 1024)
  }

  /**
   * 获取可用内存大小（以MB为单位）
   * Returns: 设备可用内存
   */
  static private func getFreeMemory() -> Int {
    var vmStats = vm_statistics_data_t()
    var infoCount = mach_msg_type_number_t(MemoryLayout<vm_statistics_data_t>.size / MemoryLayout<integer_t>.size)
    let kernReturn = withUnsafeMutablePointer(to: &vmStats) {
      $0.withMemoryRebound(to: integer_t.self, capacity: Int(infoCount)) {
        host_statistics(mach_host_self(), HOST_VM_INFO, $0, &infoCount)
      }
    }

    if kernReturn != KERN_SUCCESS {
      return 0
    }

    let vmPageSize = vm_page_size
    let freeMemorySize = Int(vmPageSize) * Int(vmStats.free_count + vmStats.inactive_count)
    return freeMemorySize / 1024 / 1024
  }

}
```

上面的代码中，我们将获取内存的信息的功能以`swift`静态方法`NativeCode.startMemMonitor(callback)` 的形式对外暴露。 

这里的 `callback`参数是一个 参数为`[Int]`(即 `Array<Int>`) 类型的 `swift`函数，对应`UTS`中一个参数为`Array`的`function`对象

至此，内存监控功能的原生代码部分已经完全开发完毕。接下来，我们编写UTS代码来使用它。


#### 第四步 编写`UTS`调用原生代码

如我们在前文所讲，`UTS`是`swift`语言的上游语言。所有`swift`代码中的：`类`、`对象`、`函数`、`变量`，均可以在uts中直接使用。 

**但是反之则不行**。

因为`UTS`的编译器兼容了`swift`的语法规则，所以`UTS`中调用`swift`代码可以被很好的支持，即使升级 HBuilderX 版本也不会有什么问题。

但`UTS`从未保证过编译对应的`swift`的具体规则。所以虽然开发者可以通过一些取巧的方式实现：swift 中调用 UTS 代码，但这是不安全的。HBuilderX 版本升级后，类似代码可能会失效或者异常。


在我们的示例中, UTS 中的调用的代码是这样的：

```ts
import { GetMemoryInfo, OnMemoryInfoChange, OffMemoryInfoChange } from '../interface.uts'


/**
 * 获取内存信息
 */
export const getMemoryInfo : GetMemoryInfo = function () : Array<number> {
  // 将原生 swift 语言的 Int 数组转换成 uts 语言中的 number数组
  let numberArray = MemoryInfoNative.getMemInfoSwift().map((value : Int, index : number) : number => {
    // 将 Int 数据类型转换成 number
    return Number.from(value);
  })
  return numberArray;
}


/**
 * 开始监听内存信息变化
 */
export const onMemoryInfoChange : OnMemoryInfoChange = function (callback : (res : Array<number>) => void) {
  MemoryInfoNative.onMemoryInfoChangeSwift((res : Array<Int>) => {
    // 将原生 swift 语言的 Int 数组转换成 uts 语言中的 number数组
    let numberArray = res.map((value : Int, index : number) : number => {
      // 将 Int 数据类型转换成 number
      return Number.from(value);
    })
    callback(numberArray)
  })
}


/**
 * 停止监听内存信息变化
 */
export const offMemoryInfoChange : OffMemoryInfoChange = function () {
  MemoryInfoNative.offMemoryInfoChangeSwift()
}
```

上面的代码，我们在`UTS`中使用 `Number.from(value)` 将 swift 中 Int 转换为 uts 中的 number 类型。


#### 注意事项

- UTSiOSHookProxy 因为涉及到自动注册的问题，在 swift 代码中直接使用将不生效。
- 目前仅支持 Swift 源码混编，OC 源码即使添加也不会参与编译
- Swift 源码文件中定义的函数、全局变量、类 等符号名称不要和 uts 文件中的符号名相同，否则会因为符号冲突导致编译不过



## harmonyos平台

> uts、ets文件依赖js新增于HBuilderX 4.61版本

### uts与ArkTS对象映射表

|uts 内置对象	|编译成的原生对象名																						|
|---					|---																												|
|Array				|	Array																											|
|Number				|	Number																										|
|String				|	String																										|
|Set					|	Set																												|
|Map					|	Map																												|
|UTSJSONObject|	import {UTSJSONObject} from '@dcloudio/uni-app-x-runtime'	|
|JSON					|	JSON																											|
|Date					|	Date																											|
|Math					|	Math																											|
|Promise			|	Promise																										|
|RegExp				|	RegExp																										|
|Error				|	Error																											|
|console			|	console																										|

### ets内使用uts特有对象

#### UTSJSONObject

示例：

```ts
import {UTSJSONObject} from '@dcloudio/uni-app-x-runtime'
const obj = new UTSJSONObject({
  a: 1
})
obj.get('a') as number // 1
obj.getNumber('a') // 1
```

UTSJSONObject不会对对象字面量进行深层转换

```ts
import {UTSJSONObject} from '@dcloudio/uni-app-x-runtime'
const obj = new UTSJSONObject({
  a: {
    b: 1
  }
})
obj.get('a') // 返回一个ESObject类型对象，并非UTSJSONObject
```

### harmony平台示例

如下示例使用的uni_module目录结构如下

```text
└── uni-MemoryInfo
    ├── package.json
    └── utssdk
        ├── app-android
        ├── app-harmony
        │   ├── index.uts
        │   └── MemoryInfoNative.ets
        ├── app-ios
        └── interface.uts
```

### 第一步 获取、编写原生代码

鸿蒙平台使用uts插件时，uts文件可以依赖ets文件、js文件、本地har包以及ohpm上的包。

### 第二步 集成原生代码

```ts
// MemoryInfoNative.ets
import { hidebug } from '@kit.PerformanceAnalysisKit';

let systemMemInfo: hidebug.SystemMemInfo = hidebug.getSystemMemInfo();

type MemoryInfoChangeCallback = (memInfo: number[]) => void

export class MemoryInfoNative {
  private static _memoryInfoChangeCallbacks: MemoryInfoChangeCallback[] = []
  private static _interval: number = -1
  static onMemoryInfoChangeArkts(callback: MemoryInfoChangeCallback) {
    MemoryInfoNative._memoryInfoChangeCallbacks.push(callback)
    if(MemoryInfoNative._interval = -1) {
      MemoryInfoNative._interval = setInterval(() => {
        const memInfo = MemoryInfoNative.getMemInfoArkts()
        MemoryInfoNative._memoryInfoChangeCallbacks.forEach(callback => {
          callback(memInfo)
        })
      }, 2000)
    }
  }
  static offMemoryInfoChangeArkts() {
    MemoryInfoNative._memoryInfoChangeCallbacks = []
    clearInterval(MemoryInfoNative._interval)
    MemoryInfoNative._interval = -1
  }
  static getMemInfoArkts(): number[] {
    let systemMemInfo: hidebug.SystemMemInfo = hidebug.getSystemMemInfo();
    return [Number(systemMemInfo.freeMem / 1024n), Number(systemMemInfo.totalMem / 1024n)]
  }
}
```

### 第三步 UTS调用原生代码

```ts
// index.uts
import { OnMemoryInfoChange, OffMemoryInfoChange, GetMemoryInfo } from '../interface.uts'
import { MemoryInfoNative } from "./MemoryInfoNative.ets"

type MemoryInfoChangeCallback = (memInfo : number[]) => void

// 开启内存监听
export const onMemoryInfoChange : OnMemoryInfoChange = function (callback : MemoryInfoChangeCallback) {
  MemoryInfoNative.onMemoryInfoChangeArkts(callback)
}

// 结束内存监听
export const offMemoryInfoChange : OffMemoryInfoChange = function () {
  MemoryInfoNative.offMemoryInfoChangeArkts()
}

// 同步获取内存信息
export const getMemoryInfo : GetMemoryInfo = function () : Array<number> {
  return MemoryInfoNative.getMemInfoArkts();
}
```

### 注意事项

需要注意的是ets文件不能引用uts文件，js文件不能引用ets文件及uts文件。如果有ets引用uts内的对象的需求，可以通过一个js文件进行中转。示例如下：

```ts
// deps.js
export const utsExports = {}
```

```ts
// index.uts
import {
  getAppVMMemoryInfo as getAppVMMemoryInfoOrigin
} from './mem.ets'
import { utsExports } from './deps.js'

function hello() {
  return 'hello from uts'
}
utsExports.hello = hello

export function getAppVMMemoryInfo() {
  return getAppVMMemoryInfoOrigin()
}
```

```ts
// mem.ets
import { hidebug } from '@kit.PerformanceAnalysisKit';
import { utsExports } from './deps.js'

export function getAppVMMemoryInfo() {
  console.log(utsExports.hello())
  return hidebug.getAppVMMemoryInfo();
}
```


## uvue页面使用uts插件

`UTS`与`uvue`之间的相互调用，属于[UTS插件开发](../plugin/uts-plugin.md)的相关内容，这里不展开叙述，开发者可以查阅相关文档掌握这部分知识。

下面列出了uvue调用的示例代码：

```vue
<template>
	<view class="content">
		<button @tap="utsGetMemory">获取内存(同步)</button>
		<button @tap="utsStartMemoryWatch">开始监听内存变化</button>
		<button @tap="utsStopMemoryWatch">停止监听内存变化</button>
	</view>
  <view class="content">
		<text style="color: red;">{{memInfo}}</text>
  </view>
</template>


<script>
import {getMemoryInfo,onMemoryInfoChange,offMemoryInfoChange} from "@/uni_modules/uni-MemoryInfo";
 
export default {
  data() {
    return {
      memInfo: '-'
    }
  },
  onLoad() {
  },
  methods: {
    utsGetMemory(){
      let array = getMemoryInfo()
      this.memInfo = "可用内存:" + array[0] + "MB \n整体内存:" + array[1] + "MB"
      console.log('getMemoryInfo', array)
    },
    utsStartMemoryWatch(){
      onMemoryInfoChange((res: Array<number>) => {
        this.memInfo = "可用内存:" + res[0] + "MB \n整体内存:" + res[1] + "MB"
        console.log('onMemoryInfoChange', res)
      })
    },
    utsStopMemoryWatch(){
      offMemoryInfoChange()
      this.memInfo = "已停止监听"
      console.log('offMemoryInfoChange')
    },
  }
}
</script>


<style>
.content {
  margin: 12px;
}
</style>
```



## Web和小程序平台js混编

在uts编译为js时，uts和js可以任意混编，就像ts和js可以互相引用一样。

在js中可直接调用uts代码，反之亦然。

```ts
// test-uts.uts
export function testUts() {
	const obj = {
		name: "uts"
	}
	console.log(obj) // UTSJSONObject
}
```

```ts
// test-js.js
import { testUts } from './test-uts.uts'
export function testJs() {
	const obj = {
		name: "js"
	}
	console.log(obj) // js object
	testUts()
}
```

## 混编注意事项

+ `index`是保留文件名，原生代码不能命名为 index.kt/index.java/index.swift
 
+ HBuilder X 暂不支持原生代码的语法提示、转到定义。仅支持高亮和格式化。

+ 插件市场暂不支持原生代码的加密

+ 混编需要使用[条件编译](https://uniapp.dcloud.net.cn/tutorial/platform.html#%E6%9D%A1%E4%BB%B6%E7%BC%96%E8%AF%91%E5%A4%84%E7%90%86%E5%A4%9A%E7%AB%AF%E5%B7%AE%E5%BC%82)限制编译入口

```uts
// 下面的代码只会在Android平台编译
// #ifdef APP-ANDROID
export function callKotlinMethodGetInfo():String {
	return NativeCode.getPhoneInfo()
}
export function callJavaMethodGetInfo():String {
	return new JavaUser("jack",12).name
}
// #endif
```



[完整的uts混编插件示例](https://ext.dcloud.net.cn/plugin?name=uni-MemoryInfo)

