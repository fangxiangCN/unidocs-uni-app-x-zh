## uts for harmonyOS

鸿蒙系统有很多原生API，这些API通过ArkTS来调用。

由于uts可以编译为ArkTS，所以uts可以调用鸿蒙的所有原生API。

如需在uni-app中使用，就需要把鸿蒙的原生API封装为uts插件，然后在uni-app中使用。

这些uts插件，是同时兼容uni-app和uni-app x的。

uni-app的普通页面代码是编译为js，js无法直接调用鸿蒙原生API。而uts插件是编译为ets文件，所以可以调用鸿蒙原生API。（ArkTS的文件后缀为.ets）

uni-app x的页面和uts插件，都运行在ArkTS引擎下，不管在普通页面还是在uts插件中均可调用鸿蒙原生API。但普通页面目前会编译js文件，运行在arkts引擎下的js不能调用@kit的库，不能使用多线程能力。完善的ets能力，需要在uts插件中才能使用。

只有uts插件才支持混编ets。

## 了解 UTS 插件是什么

UTS 插件是 uni-app 扩展API的标准插件形式 [详情](./uts-plugin.md)

uts插件在编译到harmonyOS端时会被编译成ArkTS代码。因此编写代码时应注意遵循 `uts规范` 和 `ets规范`。

## 掌握UTS语法及ArkTS语法

无论是uts还是ArkTS都是在ts的语法基础上进行扩展来的。建议先阅读如下文档

- [typescript官方文档](https://www.typescriptlang.org/zh/docs/)
- [uts语法](https://doc.dcloud.net.cn/uni-app-x/uts/)
- [ArkTs约束](https://developer.huawei.com/consumer/cn/doc/harmonyos-guides/typescript-to-arkts-migration-guide-0000001820879565?ha_source=Dcloud&ha_sourceId=89000448)

## harmonyOS 原生环境配置

- 参考：[uni-app 开发鸿蒙应用](https://uniapp.dcloud.net.cn/tutorial/harmony/dev.html)

## ArkTS 与 UTS 差异重点介绍 (持续更新)

### 类型差异

#### any类型

ArkTS内不能使用any类型，但是uts内any用处比较多，因此在编译为ArkTS时，any类型被转为了Object类型。

#### 对象字面量

ArkTS不允许无类型的对象字面量，编写代码时应注意在需要类型时为对象字面量指定类型。如未指定类型，uts会将此对象字面量编译成`as UTSJSONObject`的形式。

```ts
// 源码
const obj = {
  a: 1
}

// 编译结果
const obj = {
  a: 1
} as UTSJSONObject
```

```ts
// 源码
interface Obj {
  a: number
}
const obj: Obj = {
  a: 1
}
//或
const obj = {
  a: 1
} as Obj

// 编译结果
class Obj { ... }
const obj: Obj = {
  a: 1
}
//或
const obj = {
  a: 1
} as Obj
```

## 配置uts插件依赖

鸿蒙的库管理工具是ohpm。类似于js的npm，Android的仓储。

鸿蒙的三方sdk封装文件为`.har`，类似于Android的`.aar`

uts插件的`utssdk/app-harmony/config.json`文件内可以配置依赖使用鸿蒙的三方库，配置方式如下：

```json
{
  "dependencies": {
    "@cashier_alipay/cashiersdk": "15.8.26", // ohpm依赖
    "local-deps": "./libs/local-deps.har" // 本地依赖
  }
}
```

**注意**

- config.json文件不能含注释。
- 本地依赖相对路径是相对于config.json文件所在目录的路径，例如上面的示例中local-deps.har文件位于`utssdk/app-harmony/libs/local-deps.har`。

## 使用resources

uts 插件内包含了一个resources目录，用于存放插件的资源文件，如图片、字体等，关于resources的更多信息请参考：[鸿蒙资源分类与访问](https://developer.huawei.com/consumer/cn/doc/harmonyos-guides-V5/resource-categories-and-access-V5?ha_source=Dcloud&ha_sourceId=89000448)。此目录位于`utssdk/app-harmony/resources`。

## module.json5

开发者可以在uts插件内配置module.json5文件，用于配置插件的一些信息，如模块名、支持的设备类型、请求的权限等。module.json5内配置权限请参考鸿蒙官方文档：[声明权限](https://developer.huawei.com/consumer/cn/doc/harmonyos-guides-V5/declare-permissions-V5?ha_source=Dcloud&ha_sourceId=89000448)

module.json5文件内容示例

```json
{
  "module": { // 模块信息 name、type、deviceTypes可以省略，编译时会自动生成默认值
    "name": "uni_modules__test_mylocation", // 鸿蒙模块名，默认值生成规则见下文
    "type": "har", // 固定为har
    "deviceTypes": [ // 支持的设备类型
      "default",
      "tablet",
      "2in1"
    ],
    "requestPermissions": [ // 配置需要请求的权限
      {
        "name": "ohos.permission.LOCATION",
        "usedScene": {
          "when": "inuse"
        },
        "reason": "$string:permission_location_reason" // 本地resources内的字符串
      }
    ]
  }
}
```

鸿蒙模块名生成规则：

对于一个名称为 uni-getBatteryInfo 的 uni_module，它的 moduleName 为uni_modules__uni_getbatteryinfo，packageName 为@uni_modules/uni-getbatteryinfo。

packageName（引用模块名）规则较为简单，给 uni_module 名称前加上@uni_modules前缀然后转为全小写。

moduleName（鸿蒙模块名）是在 packageName 的基础上生成的，移除@符号，将/替换为两个下划线，将-替换为一个下划线

## 特殊文件拷贝

uts插件编译到鸿蒙时会将整个插件编译为一个鸿蒙的module。如下文件会拷贝到鸿蒙module内的对应位置。其中module.json5文件可以配置依赖的权限等信息。

```text
utssdk/app-harmony/module.json5  -->  src/main/module.json5
utssdk/app-harmony/resources  -->  src/main/resources
utssdk/app-harmony/*.ets  -->  utssdk/app-harmony/*.ets
utssdk/app-harmony/*.har  -->  utssdk/app-harmony/*.har
```

## 使用ets文件

uts插件内的ets文件会原样拷贝到产物内，如果需要开发arkui声明式界面可以在ets文件内编写，uts文件内引用。用法可参考文档：[uts插件-实现NativeButton对象](https://doc.dcloud.net.cn/uni-app-x/plugin/uts-component-vue.html#utscode)

## 常见问题

### context的获取

很多harmonyOS原生接口需要传入context作为参数。多数情况下可以直接调用harmonyOS全局方法`getContext()`获取。例如：

```ts
import settings from '@ohos.settings';
const context: Context =  getContext();
settings.getValue(context, settings.display.SCREEN_BRIGHTNESS_STATUS, (err, value) => {
  if (err) {
    console.error(`Failed to get the setting. ${err.message} `);
    return;
  }
  console.log(`SCREEN_BRIGHTNESS_STATUS: ${JSON.stringify(value)}`)
});
```
