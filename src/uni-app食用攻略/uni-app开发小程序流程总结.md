# uni-app 开发小程序流程总结

author:@泡泡鱼

## 创建

1.通过 HBuilderX 可视化界面创建 （可参考 uniapp 的文档）  
2.通过 vue-cli 命令创建，如下：

```bash
npm install -g @vue/cli
vue create -p dcloudio/uni-preset-vue my-project
// 选择默认模板
```

## 目录结构

┌─components uni-app 组件目录
│ └─comp-a.vue 可复用的 a 组件
├─hybrid 存放本地网页的目录，详见
├─platforms 存放各平台专用页面的目录，详见
├─pages 业务页面文件存放的目录
│ ├─index
│ │ └─index.vue index 页面
│ └─list
│ └─list.vue list 页面
├─static 存放应用引用静态资源（如图片、视频等）的目录，注意：静态资源只能存放于此
├─wxcomponents 存放小程序组件的目录，详见
├─main.js Vue 初始化入口文件
├─App.vue 应用配置，用来配置 App 全局样式以及监听 应用生命周期
├─manifest.json 配置应用名称、appid、logo、版本等打包信息，详见
└─pages.json 配置页面路由、导航条、选项卡等页面类信息，详见

## 运行环境判断

### 运行环境

uni-app 可通过 process.env.NODE_ENV 判断当前环境是开发环境还是生产环境。一般用于连接测试服务器或生产服务器的动态切换。
例如：

```js
if (process.env.NODE_ENV === 'development') {
  console.log('开发环境');
} else {
  console.log('生产环境');
}
```

### 运行平台判断

有 2 种场景，一种是在编译期判断，一种是在运行期判断。

- 编译期判断 编译期判断，即条件编译，不同平台在编译出包后已经是不同的代码。详见：条件编译

```js
// #ifdef H5
alert('只有h5平台才有alert方法');
// #endif
```

如上代码只会编译到 H5 的发行包里，其他平台的包不会包含如上代码。

- 运行期判断 运行期判断是指代码已经打入包中，仍然需要在运行期判断平台，此时可使用 `uni.getSystemInfoSync().platform` 判断客户 端环境是 `Android`、`iOS` 还是小程序开发工具（在百度小程序开发工具、微信小程序开发工具、支付宝小程序开发工具中使 用 `uni.getSystemInfoSync().platform` 返回值均为 `devtools`）。

## 条件编译

条件编译是用特殊的注释作为标记，在编译时根据这些特殊的注释，将注释里面的代码编译到不同平台
写法：以 `#ifdef` 或 `#ifndef` 加 `%PLATFORM%` 开头，以 `#endif` 结尾。

- #ifdef：if defined 仅在某平台存在
- #ifndef：if not defined 除了某平台均存在
- %PLATFORM%：平台名称
  详见：https://uniapp.dcloud.io/platform

## 文件引入

js 文件引入 js 文件或 script 标签内（包括 renderjs 等）引入 js 文件时，可以使用相对路径和绝对路径，形式如下

```js
// 绝对路径，@指向项目根目录，在 cli 项目中@指向 src 目录
import add from '@/common/add.js';
// 相对路径
import add from '../../common/add.js';
```

## 应用生命周期

1. onLaunch ----当 uni-app 初始化完成时触发（全局只触发一次）
2. onShow-------当 uni-app 启动，或从后台进入前台显示
3. onHide------- 当 uni-app 从前台进入后台
4. onError-------当 uni-app 报错时触发
5. onUniNViewMessage-----对 nvue 页面发送的数据进行监听，应用生命周期仅可在 App.vue 中监听，在其它页面监听无效。
6. onlaunch 里进行页面跳转，如遇白屏报错，请参考 https://ask.dcloud.net.cn/article/35942

## 页面生命周期

onLoad/onShow/onReady/onHide/onUnload/onResize/onPullDownRefresh/ 等等

## 尺寸单位

uni-app 支持的通用 css 单位包括 px、rpx

px 即屏幕像素
rpx 即响应式 px，一种根据屏幕宽度自适应的动态单位。以 750 宽的屏幕为基准，750rpx 恰好为屏幕宽度。屏幕变宽，rpx 实际显示效果会等比放大。
计算公式 750 _ 元素在设计稿中的宽度 / 设计稿基准宽度
**举例说明：**
若设计稿宽度为 750px，元素 A 在设计稿上的宽度为 100px，那么元素 A 在 uni-app 里面的宽度应该设为：750 _ 100 / 750，结果为：100rpx。
若设计稿宽度为 640px，元素 A 在设计稿上的宽度为 100px，那么元素 A 在 uni-app 里面的宽度应该设为：750 \* 100 / 640，结果为：117rpx。

## getCurrentPages()/getApp()

不要在 `App.onLaunch` 的时候调用 `getCurrentPages()`，此时 `page`还没有生成。

```ts
let routes = getCurrentPages(); // 获取当前打开过的页面路由数组
let curRoute = routes[routes.length - 1].route; // 获取当前页面路由，也就是最后一个打开的页面路由
```

`getApp()` 函数用于获取当前应用实例，一般用于获取 `globalData` 。

```js
const app = getApp() console.log(app.globalData)
```

## 数据存储缓存

- `uni.setStorage` 将数据存储在本地缓存中指定的 `key` 中，会覆盖掉原来该 `key`对应的内容，这是一个异步接口。

```js
uni.setStorage({
  key: 'storage_key',
  data: 'hello',
  success: function () {
    console.log('success');
  },
  fail: function () {},
  complete: function () {
    console.log('接口调用结束的回调函数（调用成功、失败都会执行）');
  },
});
```

- `uni.setStorageSync('storage_key', 'hello');`将 `data` 存储在本地缓存中指定的 `key` 中，会覆盖掉原来该 `key` 对应的内容，同步接口。
- `uni.getStorage` 从本地缓存中异步获取指定 `key` 对应的内容。

```js
uni.getStorage({
  key: 'storage_key',
  success: function (res) {
    console.log(res.data);
  },
});
```

- `uni.getStorageSync`
- `uni.removeStorage` 从本地缓存中异步移除指定 `key`。
- `uni.clearStorage()`清理本地数据缓存。

## Router

uniapp 路由配置：设置`page.json` 中的`pages`字段如下，`pages` 中的第一个页面为默认显示页面

```json
{
  "pages": [
    {
      "path": "pages/index/index",
      "style": {
        "navigationBarTitleText": "FITLIFE"
      }
    },
    {
      "path": "pages/payOrder/index",
      "style": {
        "navigationBarTitleText": "订单只付"
      }
    }
  ]
}
```

页面路由跳转，路由传参
（1）navigateTo，redirectTo 只能打开非 tabBar 页面
navigateTo 保留当前页面，跳转到应用内的某个页面，使用 uni.navigateBack 可以返回到原页面。

（2）switchTab 只能打开 TabBar 页面，跳转到 tabBar 页面，并关闭其他所有非 tabBar 页面。

（3）reLaunch 可以打开任意界面，关闭所有页面，打开到应用内的某个页面。

（4）页面底部的 tabBar 由页面决定，即只要是定义为 tabBar 的页面，底部都有 tabBar

（5）不能在 App.vue 里面进行页面跳转

```js
uni.navigateTo({
  url: '/pages/assignedCourse/index?type=' + item.type,
  success: (res) => {},
  fail: () => {},
  complete: () => {},
});
```

获取路由中的参数，在 `onLoad()`获取

```js
onLoad: function (option) {
  //option 为 object 类型，会序列化上个页面传递的参数option.路由中携带的参数名
}
```
