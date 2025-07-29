## uts for Android

本文旨在帮助Android开发者快速上手UTS。

需要阅读者具备Android原生应用开发经验。


## 1 了解UTS插件是什么

`UTS插件`是`uni-app`新型插件形式，拥有跨平台，高效率，易调试等优点。[详情](./uts-plugin.md)

对于Android开发者来说，我们需要了解的是：

1. 编译时：当我们在保存`UTS`源码文件时，IDE会同步将其编译为对应的Kotlin代码。
2. 运行时：在真机运行/云打包时，这些编译后的kotlin源码也会成为apk的一部分

## 2 掌握UTS语法

### 2.1  对于掌握kotlin语言者

因为UTS语法与kotlin很类似，建议快速阅读后，在实践中掌握这UTS语法。[uts语法介绍](/uts/)。

### 2.2  对于仅掌握java语言者

与js相比，uts的语法和java更加类似。但是依然存在较大的差异，需要详细阅读2.3语法部分。

尽管开发UTS插件，并不要求一定掌握kotlin，但是鉴于`UTS`目前在android平台上，会编译为kotlin源码。学会kotlin语言，方便排查问题和复杂功能实现。

因此建议学习一下kotlin语法。

+ kotlin [https://kotlinlang.org/](https://kotlinlang.org/)

+ kotlin for android [https://developer.android.com/kotlin](https://developer.android.com/kotlin)

### 2.3 数据类型差异

虽然 UTS 和 koltin 在数据类型上基本保持了一致，但是在部分场景下，还是会有差异，在此特别说明

原则上：

**数据类型以UTS 内置的类型为准， 各原生平台都会对其自动适配。**

**但是 UTS本身是跨平台语言，当具体平台的api 有明确要求时，需要以对方明确要求的数据类型为准。**

-------------------------


#### 举例一： Int 和Number

默认情况下`UTS` 开发者可以使用 `Number` 覆盖`android` 平台上使用 `Int`的场景。

但是当开发者重写  `Service` 组件`onStartCommand` 方法时，`Android` API要求 明确要求后两个参数 必须为Int


原生开发环境中，应该这样写：

 ```kotlin
 override fun onStartCommand(intent: Intent, flags: Int, startId: Int): Int {
	return super.onStartCommand(intent, flags, startId);
 }
 ```


 标准的TS环境中，只有`Number`类型而没有`Int`类型

 为了适应这种情况，UTS 允许开发者使用原生平台的数据类型Int，来满足原生API对数据类型的要求：

```ts
 override onStartCommand(intent:Intent ,flags:Int ,startId:Int):Int {
	return super.onStartCommand(intent, flags, startId);
 }
```




#### 举例二：`MutableList`

`MutableList`是`android`平台 特有的数据类型，一般场景下，可以使用UTS中内置类型 `Array` 替代

但是在 调用`onAppActivityRequestPermissionsResult` 函数监听权限申请结果时，明确要求使用此类型的参数

在原生环境中，应该这样写：

```kotlin

onAppActivityRequestPermissionsResult(fun(requestCode: Number, permissions: MutableList<String>, grantResults: MutableList<Number>){

});
```


标准的TS环境中，没有`MutableList`类型，与之相近的数据类型是 `Array`

为了适应这种情况，UTS 允许开发者使用原生平台的数据类型`MutableList`，来满足原生平台API对数据类型的要求：

```ts
onAppActivityRequestPermissionsResult((requestCode: number,permissions: MutableList<string>,grantResults: MutableList<number>) => {

});

```

#### 举例三：`String[]`

部分三方sdk 使用java开发，要求继承/实现的方法参数为 string[]类型，这种情况比较特殊，需要将kotlin.Array先进行别名声明，再继续使用

```ts
import KotlinArray from 'kotlin.Array';


class XXX{
 	override onCaptureFinish(p0: KotlinArray<string>){
		// do sth
	};
}

```


### 2.4 线程环境差异 @thread-environment

UTS语言本身没有线程的概念。 但在具体的运行平台上会有线程环境差异：

+ uni-app 平台：默认代码执行在 `WeexJSBridgeThread`

+ uni-app x 平台：默认代码执行在 `main`线程


`Android`系统对线程操作存在较多的限制， UTS内置了[UTSAndroid.getDispatcher方法](https://doc.dcloud.net.cn/uni-app-x/uts/utsandroid.html#getdispatcher)  用来屏蔽大多数底层细节，一般来说开发者只需要关心两种特殊情况：

如果需要执行耗时任务：

```ts
UTSAndroid.getDispatcher("io").async(function(_){
	// 执行耗时任务
},null)
```

如果需要操作UI：

```ts
UTSAndroid.getDispatcher("main").async(function(_){
	// 执行界面修改，包括view添加移除等
},null)
```


## 3 Android原生环境配置

对于Android项目来说，除了源码之外，还会涉及依赖，资源，配置等常见问题

本章节将会介绍，UTS插件开发环境中如何配置这些属性

**注意**：

+ 1 本章节内的实例代码均取自Hello UTS [项目地址](https://gitcode.net/dcloud/hello-uts)
+ 2 本章节涉及的配置，均需自定义基座后才能生效

### 3.1 配置AndroidManifest.xml

以hello UTS中的native-page插件中的配置文件为例:

文件位置： ~\uni_modules\uts-nativepage\utssdk\app-android\AndroidManifest.xml

AndroidManifest.xml示例：

```xml
<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android" xmlns:tools="http://schemas.android.com/tools"
  // 注意：这里是插件的包名而不是应用的包名
  package="io.dcloud.uni_modules.utsNativepage">
   // 配置权限
   <!--创建前台服务权限-->
   <uses-permission android:name="android.permission.FOREGROUND_SERVICE" />

    <application>
	   // 配置service / activity
	   <service android:name="uts.sdk.modules.utsNativepage.ForeService"  />
       <activity android:name="uts.sdk.modules.utsNativepage.DemoActivity"></activity>
    </application>
</manifest>

```


UTS插件中的 `AndroidManifest.xml` 与 `android` 中的 `AndroidManifest.xml` 规则完全一致。


**特别提示**：

每一个UTS插件对应android项目中的一个 lib module.

与你在android studio中手动输入包名不同的是，如果你没有手动包名，HX会按照下面的规则默认生成一个：

```
uts插件默认包名规则：

如果是根目录utssdk下的uts插件
	包名：uts.sdk.(插件ID转驼峰)
如果是uni_modules目录下的uts插件
	包名：uts.sdk.modules.(插件ID转驼峰)


举例：
uni-getbatteryinfo -> uts.sdk.modules.uniGetbatteryinfo;
uts-nativepage  ->  uts.sdk.modules.utsNativepage
```

### 3.2 配置res资源

![](https://native-res.dcloud.net.cn/images/uts/forAndroid/uts_android_res_folder.jpg)

示例文件在hello uts中的位置：

~\uni_modules\uts-nativepage\utssdk\app-android\res

除了这里列出的layout、values目录外，还支持anim等所有android标准资源目录


**需要注意的是：如果res资源中使用了 android appcompat库内置的资源，需要在config.json中添加下面的配置：**

```json
{
	"dependencies": [
		"androidx.appcompat:appcompat:1.0.0"
	]
}


```



### 3.3 配置asset资源

以hello UTS中的uts-advance插件为例。

![目录结构](https://native-res.dcloud.net.cn/images/uts/forAndroid/uts_android_assets_folder.jpg)

关键代码:

```ts
// 获取asset管理器
let assetManager = getAppContext()!.getAssets();
// 加载free.mp3 资源
let afd = assetManager.openFd("free.mp3");
// 使用android 自带的媒体组件进行播放
let mediaPlayer = new MediaPlayer();
mediaPlayer.setDataSource(afd.getFileDescriptor(),afd.getStartOffset(), afd.getLength());
mediaPlayer.prepare();
mediaPlayer.start();
```

完整的代码在hello uts中的位置：

~\uni_modules\uts-advance\utssdk\app-android\assets

### 3.4 增加libs依赖资源

#### 远程依赖

远端插件可以通过配置 `config.json`添加依赖 ，下面是一个`config.json`示例

```json
{
	"dependencies": [
		"androidx.recyclerview:recyclerview:1.0.0"
	]
}


```


------

截止 HBuilder X 4.23 版本内置了以下依赖

开发者在使用列表中的依赖时，需要注意两点：

+  真机运行时，不需要添加列表中的依赖，即可直接引用相关类
+  请勿通过 手动添加jar/aar 等方式引入相同的依赖，否则会因依赖冲突导致云打包失败。



##### uni-app x

```gradle

implementation 'androidx.core:core-ktx:1.10.1'
implementation 'androidx.recyclerview:recyclerview:1.3.2'
implementation 'androidx.appcompat:appcompat:1.0.0'
implementation 'androidx.exifinterface:exifinterface:1.3.6'
implementation 'androidx.localbroadcastmanager:localbroadcastmanager:1.0.0@aar'
implementation 'androidx.constraintlayout:constraintlayout:2.1.4'
implementation 'com.google.android.material:material:1.4.0'
implementation 'androidx.viewpager2:viewpager2:1.1.0-beta02'
implementation 'com.alibaba:fastjson:1.2.83'
implementation 'com.facebook.fresco:fresco:3.1.3'
implementation 'com.facebook.fresco:middleware:3.1.3'
implementation 'com.facebook.fresco:animated-gif:3.1.3'
implementation 'com.facebook.fresco:webpsupport:3.1.3'
implementation 'com.facebook.fresco:animated-webp:3.1.3'
implementation 'com.github.bumptech.glide:glide:4.9.0'
implementation 'org.jetbrains.kotlinx:kotlinx-coroutines-core:1.6.4'
implementation 'org.jetbrains.kotlinx:kotlinx-coroutines-android:1.6.4'
implementation 'org.jetbrains.kotlin:kotlin-stdlib:1.8.10'
implementation 'org.jetbrains.kotlin:kotlin-reflect:1.8.10'
implementation 'org.jetbrains.kotlinx:kotlinx-serialization-json:1.4.1'
implementation 'com.squareup.okhttp3:okhttp:3.12.12'
implementation 'com.github.getActivity:XXPermissions:18.0'
implementation 'androidx.recyclerview:recyclerview:1.0.0'

```

##### uni-app

```gradle
implementation 'com.github.bumptech.glide:glide:4.9.0'
implementation 'androidx.localbroadcastmanager:localbroadcastmanager:1.0.0'
implementation 'androidx.core:core:1.1.0'
implementation 'androidx.fragment:fragment:1.1.0'
implementation 'androidx.recyclerview:recyclerview:1.1.0'
implementation 'androidx.appcompat:appcompat:1.1.0'
implementation 'com.alibaba:fastjson:1.2.83'
implementation 'androidx.webkit:webkit:1.3.0'
implementation 'com.squareup.okhttp3:okhttp:3.12.12'
implementation 'androidx.core:core-ktx:1.6.0'
implementation 'org.jetbrains.kotlin:kotlin-stdlib:1.8.10'
implementation 'org.jetbrains.kotlin:kotlin-reflect:1.6.0'
implementation 'org.jetbrains.kotlinx:kotlinx-coroutines-core:1.3.8'
implementation 'org.jetbrains.kotlinx:kotlinx-coroutines-android:1.3.8'
implementation 'com.github.getActivity:XXPermissions:18.0'
implementation 'com.facebook.fresco:fresco:2.5.0'
implementation 'com.facebook.fresco:animated-gif:2.5.0'
implementation 'com.facebook.fresco:webpsupport:2.5.0'
implementation 'com.facebook.fresco:animated-webp:2.5.0'
implementation 'androidx.recyclerview:recyclerview:1.0.0'
implementation 'androidx.appcompat:appcompat:1.0.0'
```

关于 `config.json` 的更多写法，可以参考[文档](https://nativesupport.dcloud.net.cn/NativePlugin/course/package.html#dependencies)


#### 本地依赖

jar/aar文件:

参考 [hello uts](https://gitcode.net/dcloud/hello-uts/-/tree/master/uni_modules/uts-nativepage/utssdk/app-android/libs) 将jar/aar 添加到 `utssdk/app-android/libs` 目录下即可使用

so文件:

目前暂不支持so文件直接本地调试.

HBuilderX 4.26版本之前, 推荐开发者将so封装为AAR，或者分别集成so和jar文件，自定义基座后再进行调试

HBuilderX 4.26版本之后，开发者可以使用[混编](https://doc.dcloud.net.cn/uni-app-x/plugin/uts-plugin-hybrid.html)kotlin代码的方式，只需要集成so文件后打包自定义基座就可以让so文件参与本地调试，省去了封装AAR和jar的环节。参考[hello uts](https://gitcode.net/dcloud/hello-uts/-/tree/alpha/uni_modules/uts-nativepage/utssdk/app-android/libs)

### 3.5 其他配置文件

uni-app x / uni-app 均支持打包时手动指定资源位置 [说明文档](https://uniapp.dcloud.net.cn/tutorial/app-nativeresource-android.html#nativeresources)

例如 接入`Firebase` 时需要将`google-services.json`文件放在 `app` 目录下，则可以通过如下的配置来实现：


<pre v-pre="" data-lang="">
	<code class="lang-" style="padding:0">
┌─pages                         //页面目录
│  └─[具体内容]
├─nativeResources               //配置文件目录
│  └─android
│  	└─google-services.json
├─ // 其他文件
└─App.vue
	</code>
</pre>

具体的打包示例，参考 [Hello UTS](https://gitcode.net/dcloud/hello-uts)

### 3.6 远程依赖仓库说明

目前云打包机支持下面的仓库：

```gradle


jcenter()
google()
// huawei
maven {url 'https://developer.huawei.com/repo/'}
// jitpack 远程仓库：https://jitpack.io
maven { url 'https://jitpack.io' }
// mavenCentral 默认支持
mavenCentral()

```

部分场景下，开发者可能需要将本地依赖，上传到远程仓库，避免wgt提交资源过大超出打包限制。

这种情况，推荐开发者上传到 jitpack.io  这也是目前android 原生开发主流的远程仓库。 [使用文档](https://docs.jitpack.io/)


### 3.7 Android 编译SDK版本说明

截止 HBuilderX 4.15 版本：

+ uni-app x 项目： 本地Android sdk 版本为34

+ uni-app   项目： 本地Android sdk 版本为31

开发者在使用系统api时，需要注意版本兼容性




## 4 Android内置库@iodcloudutsandroid

**在UTS语言中，所有的Android原生API都可以调用**

对于Android开发中高频使用的`application`/`activity`等系统能力、`uni-app`/`uni-app x` 运行时框架信息等，UTS通过内置对象`UTSAndroid` 进行了封装，以便开发者调用

下面列出了常见API的使用示例，完整的 `UTSAndroid` [API文档](https://doc.dcloud.net.cn/uni-app-x/uts/utsandroid.html)


#### getAppContext

```ts
import { UTSAndroid } from "io.dcloud.uts";
```

用法说明：获取当前应用Application上下文，对应android平台 Context.getApplicationContext 函数实现

Android开发场景中，调用应用级别的资源/能力，需要使用此上下文。更多用法，参考[Android官方文档](https://developer.android.google.cn/docs)


```ts
// [示例]获取asset下的音频，并且播放
let assetManager = UTSAndroid.getAppContext()!.getAssets();
let afd = assetManager.openFd("free.mp3");
let mediaPlayer = new MediaPlayer();
mediaPlayer.setDataSource(afd.getFileDescriptor(),afd.getStartOffset(), afd.getLength());
mediaPlayer.prepare();
mediaPlayer.start();
```

** 与Application的转换 **

UTSAndroid.getAppContext() 默认返回的是 实现了 `Context`抽象类的`Application` 对象

部分场景，明确需要 `Application` 对象，那么直接强制类型转换即可

```ts
import Application from 'android.app.Application'


let app = UTSAndroid.getAppContext() as Application
console.log(app)
```


#### getUniActivity

获取当前插件所属的activity实例，对应android平台 getActivity 函数实现

Android开发场景中，调用活动的级别的资源/能力，需要使用此上下文。更多用法，参考[Android官方文档](https://developer.android.google.cn/docs)

```ts
// [示例]获取当前activity顶层容器
let decorView = UTSAndroid.getUniActivity()!.window.decorView;
let frameContent = decorView.findViewById<FrameLayout>(android.R.id.content)
```


#### requestSystemPermission

HBuilder X 3.8.2版本之后支持

系统权限管理使用了 https://github.com/getActivity/XXPermissions 工具库

如果开发者使用了相同依赖，可能打包冲突。需要修改为 complileOnly 或者 修改为本章节内置API

请求系统权限,对应的两个参数：
1  请求的权限列表
2  请求结果回调

```ts
let permission = ["android.permission.ACCESS_FINE_LOCATION","android.permission.ACCESS_FINE_LOCATION"]
UTSAndroid.requestSystemPermission(UTSAndroid.getUniActivity()!,permission,function(allRight:boolean,grantedList:string[]){
		if(allRight){
			// 用户同意了全部权限
		}else{
			// 用户仅同意了 grantedList中的权限
		}
	},function(doNotAskAgain:boolean,grantedList:string[]){
		// 用户拒绝了部分权限，仅允许了grantedList中的权限
		if(doNotAskAgain){
			// 用户拒绝了权限，并且选择不再询问
		}
	})
```

--------------------------------


特别说明：

除了本章节列出的函数外，android环境下 application 其他上下文方法都可以通过 getAppContext()!.xxx()的方式实现

比如获取app缓存目录：

```ts
UTSAndroid.getAppContext()!.getExternalCacheDir()!.getPath()
```


activity 其他上下文方法都可以通过 getUniActivity()!.xxx()的方式实现

比如获取当前activity的顶层View容器

```ts
UTSAndroid.getUniActivity()!.getWindow().getDecorView();
```



## 5 隐私协议适配说明@iodcloudprivacy

UTS内置了隐私状态管理API，以支持开发者管理用户隐私协议状态配置的需求：

获取用户当前是否已同意隐私协议[isPrivacyAgree](https://doc.dcloud.net.cn/uni-app-x/uts/utsandroid.html#isprivacyagree-boolean)

更新用户对隐私协议的状态[setprivacyagree](https://doc.dcloud.net.cn/uni-app-x/uts/utsandroid.html#setprivacyagree-state-boolean-void)

重置隐私协议状态[resetPrivacyAgree](https://doc.dcloud.net.cn/uni-app-x/uts/utsandroid.html#resetprivacyagree-void)




## 6 Kotlin与UTS差异重点介绍 (持续更新)

通过上面的章节的阅读,至此我们认为你已经掌握了UTS语法，掌握了基本的Kotlin语法，掌握了UTS对于android资源的支持。

但是对于一个熟悉android开发的kotlin语言者来说，有很多常用的习惯发生了改变，我们会在这个章节特别指出，便于开发者加深认识。


### 语法差异

-------------------------------


#### let和var

`kotlin`中 可变变量修饰为 `var`、`val`。 区别在于 val 不可变，var可变。

`uts`中对应`var`的变量类型为 `var/let`

推荐使用`let` 因为只会在作用域内生效，需要慎用`var`，因为它具备有更大的作用范围


#### 方法定义

方法定义 `kotlin`里的方法只有一种定义方式

```kotlin
 fun startListener():void{

 }
```
uts中，需要区分全局方法、成员方法

```ts
 // 成员方法
 startListener():void{

 }
```

```ts
 // 全局方法
 function startListener():void{

 }
```

#### extends

`kotlin`中的: 继承操作符，需要用`extends`取代

|语法|kotlin|uts|
|---|-------|---|
|继承类|:|extends|
|实现type接口|:|extends|
|实现接口|:|implements|


```ts
// 使用UTS 实现 OnClickListener接口
class StartServiceListener extends OnClickListener{

    override onClick(v?: View):void{
		// 执行点击逻辑
    }
}

```





#### 非空断言

kotlin中的非空断言是`!!`，ts中是一个`!`

```ts
user!.sayHello();
```

```kotlin
user!!.sayHello();
```


#### 快速调用父类实现


```kotlin
//kotlin 中快速实现super
constructor() : super() {
}

```

```ts
//uts 中快速实现super
constructor (){
	super();
}
```


#### 匿名内部类

`kotlin`中可以使用匿名内部类

```kotlin
// kotlin 新建事件监听
user.setListener(Listener(){
	//todo
});
```

在UTS中这样使用：

```ts
const myListener = new (class implements Listener {
	//todo
})
user.setListener(myListener);
```

#### 可为空函数调用


有一种特殊场景，我们需要定义一些可为空的函数变量，比如下面的 success,fail：

```ts
type Option = {
	success?: (res: object) => void;
	fail?: (res: object) => void;
};

```


这个时候我们需要这样调用

```ts
options.success?.(res)
```

这样的调用方式在kotlin中是非法的，属于TS中的特有语法，需要特别注意。


#### 一个类只能有一个构造函数

在`Kotlin`/`java`中允许一个函数有多个构造器，但是UTS中是不被允许的



#### 界面跳转写法

android开发中场景的 intent跳转需要传入 目标界面的class对象，目前UTS中仅支持一种写法

```ts
let intent = new Intent(getUniActivity(),DemoActivity().javaClass);
getUniActivity()!.startActivity(intent);
```

#### 指定double数据类型

某些场景下开发者需要获得 指定double数据类型的数据

开发者下意识的写法可能是：
```ts
// 这样是错误的
let a:Int =3
let b:Int =4
let c:Double  = a/b
```

但是Android原生环境中，数据类型的精度是向下兼容的，如果想要获得一个double类型，必须要有一个double类型参与运算：

```ts
// 这样才是正确的
let a:Int =3
let b:Int =4
let c:Double  = a * 1.0 / b
```


---------------------------------

###  警告优化

下面的内容不会影响功能使用，但是在UTS环境中，有合适的解决办法

#### java lang包的引入问题

`kotlin` 或者`java` 中java.lang.*是被特殊处理的，可以直接使用而不需要引入。

```kotlin
// 获取当前时间戳
System.currentTimeMillis()
```


UTS环境中，lang包没有被特殊对待，需要手动引入。

```ts
// 手动引入lang包下的类
import System from 'java.lang.System';

// 获取当前时间戳
System.currentTimeMillis()
```


#### `UTS` 不建议使用 快捷构造

`kotlin`  中 支持通过()的方式，快速实现无参构造器的声明

```kotlin
// 获取当前时间戳
class ScreenReceiver extends BroadcastReceiver(){

}
```


UTS环境中，不建议这样做（虽然目前这样做不会影响编译），建议使用手动声明无参构造

```ts
class ScreenReceiver extends BroadcastReceiver{

	constructor (){
		super();
	}

}
```



## 7  常见问题(持续更新)

### 7.1 如何在UTS环境中，新建一个`activity`？

参考Hello UTS项目中的uts-nativepage插件


### 7.2 如何在UTS环境中，新建一个`service`？

参考Hello UTS项目中的uts-nativepage插件


### 7.3 如何在UTS环境中，新建一个`Thread`？

简单示例
```ts
class CustomThread extends Thread{

	constructor(){
		super();
	}

	override run(){
		Thread.sleep(1000)
		console.log("CustomThread = " + Thread.currentThread().getName())
	}
}
```

完整示例参考Hello UTS项目中的uts-nativepage插件

如果只是想要简单的开启一个异步任务，建议使用：

```ts
UTSAndroid.getDispatcher("io").async(function(_){

}）
```

[详细用法](https://doc.dcloud.net.cn/uni-app-x/uts/utsandroid.html#getdispatcher)



### 如何生成byte[]对象

在java平台中，二进制操作一般采用字节数组实现。

UTS在android平台编译后的语言为Kotlin,对应的语法对象是ByteArray.

使用这个类不需要额外引入包，直接运行即可

下面是一个简单的示例

```ts
let byteTest = new ByteArray(5)
console.log(byteTest)
```



### 编译报错：unresolved reference R （R资源无法识别）

UTS插件支持使用android的原生资源，比如动画，布局，字符串等。 [详细说明](https://uniapp.dcloud.net.cn/plugin/uts-for-android.html#_3-1-%E9%85%8D%E7%BD%AEandroidmanifest-xml)


如果提示 R资源无法找到:unresolved reference R

1 需要排查资源是否符合android原生格式

2 检查R资源引入的包名是否正确，参考hello uts nativepage插件

```ts
import R from 'io.dcloud.uni_modules.uts_nativepage.R';
```


### 7.4  如何实现一个接口

以HelloUTS nativepage插件 部分代码为例：

 ```ts
import OnClickListener from 'android.view.View.OnClickListener';
// 实现 OnClickListener 接口
class User {
	name:string = "name"
}

 class StartBroadcastListener extends User implements OnClickListener{

    override onClick(v?: View):void{

		let myReceiver = new ScreenReceiver();
		let filter = new IntentFilter();
		filter.addAction(Intent.ACTION_SCREEN_OFF);
		filter.addAction(Intent.ACTION_SCREEN_ON);
		UTSAndroid.getUniActivity()!.registerReceiver(myReceiver, filter);

		// 提示屏幕状态监听已经注册
		Toast.makeText(UTSAndroid.getAppContext(),"屏幕状态监听已注册，注意观察控制台日志",Toast.LENGTH_LONG).show();

    }
}


// 使用
let btn_start_screen_listen = this.findViewById<Button>(R.id.btn_start_screen_listen);
btn_start_screen_listen.setOnClickListener(new StartBroadcastListener());
 ```

如果要同时实现多个接口，采用的也是  implements 和 `,` 分隔来实现

```ts
class Person{
	name:string = ""
}
class User extends Person implements android.view.View.OnClickListener,Cloneable{
	constructor(){

	}

	override onClick(v?: android.view.View):void{
		console.log(v)
	}

	override equals(other?: any):boolean{
		return true
	}
}

```

编译后的kotlin代码

```kotlin
open class Person {
    open var name: String = "";
}
open class User : Person, android.view.View.OnClickListener, Cloneable {
    constructor(){}
    override fun onClick(v: android.view.View?): Unit {
        console.log(v, " at uni_modules/uts-helloworld/utssdk/index.uts:37");
    }
    override fun equals(other: Any?): Boolean {
        return true;
    }
}
```

其中需要注意的是

### UTS 如何访问静态实例方法

kotlin编译产出的AAR，会将访问方法修改为私有，不能以下面的方法访问

```ts
ScancodeConfig.setShowLine(false);
```

应该使用属性访问：

```ts
ScancodeConfig.showLine = false;
```

### 7.5 Android原生API过时警告处理

调用原生过时的API插件编译时产生警告，可以使用`@Suppress("DEPRECATION")`添加注解到使用的方法上忽略警告，例：
```js
@Suppress("DEPRECATION")
function getAppName(context : Context) : string {
	let appName = ""
	try {
		const packageManager = context.getPackageManager()
		const applicationInfo = packageManager.getApplicationInfo(context.getPackageName(), 0)
		appName = packageManager.getApplicationLabel(applicationInfo) as string
	} catch (e : Exception) {
		e.printStackTrace()
	}
	return appName
}
```

### 7.6 泛型传递丢失的问题 @lost-generics

如果在UTS中声明一个包含泛型声明的方法，可能会出现泛型丢失，原因是因为普通的kotlin 方法没有实现泛型的传递

错误的kt代码：

```kotlin
fun <T> getArtListByres(): A<T>? {
   var aRet = UTSAndroid.consoleDebugError(JSON.parse<A<T>>("{\"x\":111,\"y\":\"aaa\",\"t\":{\"name\":\"zhangsan\"}}"), " at pages/index/index.uvue:27");
   return aRet;
}
```

期望得到的正确的kt代码：

```ts
inline fun <reified T> getArtListByres(): A<T>? {
    var aRet = UTSAndroid.consoleDebugError(JSON.parse<A<T>>("{\"x\":111,\"y\":\"aaa\",\"t\":{\"name\":\"zhangsan\"}}"), " at pages/index/index.uvue:27");
    return aRet;
}
```

为了解决这种情况，我们可以在UTS中 添加android方法注解，来告诉编译器生成正确的代码：

```kotlin
@UTSAndroid.keyword("inline")
@UTSAndroid.keyword('reified')
export function request<T>(options : RequestOptions<T>) : RequestTask {
	//xxx
}
```


**注意：不要在`inline`方法中创建局部function，比如request的success回调、Promise的回调，原因是kotlin语言的限制（inline方法展开到内联位置，也会把局部方法展开过去，这是不允许的），由此把使用局部function的逻辑封装到非内联的方法中，绕过此限制。**

下面是可以完整的示例：

```
@UTSAndroid.keyword("inline")
@UTSAndroid.keyword("reified")
export function boxRequest<T>(url : string) : Promise<T> {
	return innerRequest<T>(url, UTSAndroid.getGenericClassName<T>(), UTSAndroid.getGenericType<T>())
}

function innerRequest<T>(url : string, clzName : string, type : Type) : Promise<T> {
	return new Promise<T>((resolve, reject) => {
		uni.request<string>({
			url: url,
			method: "GET",
			success: (e : RequestSuccess<string>) => {
				const result = JSON.parse<T>(e.data!, type)
				if (result != null) {
					resolve(result)
				} else if ("java.lang.Object".equals(clzName, true)) {// 解决泛型是any，但后端返回string的情况。
					resolve(e.data! as T)
				} else{
					reject("parsing failure")
				}
			},
			fail(e : RequestFail) {
				reject(e)
			},
		} as RequestOptions<string>)
	});
}
```
调用代码：
```
const respone = await boxRequest<CustomType>("xxxx")
```

此示例中，网络请求泛型为`string`在4.25版本以下会导致错误，此问题已在4.25进行修复 [issue](https://issues.dcloud.net.cn/pages/issues/detail?id=4010)




### 7.7 获取原生Class 对象

可以使用下面的代码获取指定class对象
```kotlin
// 根据 类名 获取 class
let getClassByName = Class.forName("io.dcloud.uts.UTSJSONObject")
console.log(getClassByName);
// 根据 实例 获取 class
let getClassByInstance = UTSAndroid.getJavaClass(UTSAndroid.getUniActivity()!)
console.log(getClassByInstance);
```

### 7.8 推断为XXX，但预期为Unit

`uts`中的 `Void` 对应 `kotlin`语言中的 `Unit`,当报错：预期为 Unit 时，实际上是期望`Void`


### 7.9 targetMethod error::java.lang.IllegalArgumentException: Callable expects 2 arguments, but 1 were provided.

目前uts的class实例不支持在vue2的data中定义。发生此类错误时，排查是否在data函数中定义了uts导出的class实例，如果定义了，移除该定义即可。

### 7.10 Qualified name must be a '.'-separated identifier list

排查导入的三方库包名中是否包含了kotlin的关键字，比如:
```ts
import GetObjectRequest from "com.tencent.cos.xml.model.object.GetObjectRequest" // 编译报错：Qualified name must be a '.'-separated identifier list
// 其中`object`是kotlin的关键字，需要使用`进行转义：
import GetObjectRequest from "com.tencent.cos.xml.model.`object`.GetObjectRequest" // 编译正常
```

## 已知待解决问题(持续更新)

### 结构入参 boolean 参数默认为true

当以type 结构体为参数时，其内部boolean字段 默认值为false，不支持指定。


### android原生资源文件，暂不支持三方库依赖

比如xml布局文件中暂时只支持 linearlayout等官方标签，不支持 appcompat等三方库标签。这个问题后续会被处理

### 不支持直接添加so文件到插件目录

截止HBuilder X 4.19 ：

UTS插件本地调试尚不支持直接使用so文件，需要将so文件和调用代码封装为AAR 或者分别集成 so和jar文件

AAR调用示例参考：[hello uts](https://gitcode.net/dcloud/hello-uts/-/tree/master/uni_modules/uts-toast)
