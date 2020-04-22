# uni-app 开发小程序流程

上一篇讲了怎么搭建一个 Vue+TS 的 uni-app 开发环境，这一篇就来总结一下如何开发小程序

大概分为几个步骤，比如配置小程序相关信息，配置页面信息，页面开发等

## 配置小程序相关信息

在 manifest.json 里添加小程序的 appid

```json
"mp-weixin": { /* 小程序特有相关 */
    "usingComponents":true,
    "appid": “1234567890",
    "setting" : {
       "urlCheck" : true
    }
},
```

在 pages.json 里配置小程序的颜色，样式，tab，分包，预加载等信息

```json
{
  "pages": [
    // 每一页的处理信息
    {
      "path": "pages/index/index",
      "style": {
        "navigationBarTitleText": "index"
      }
    }
  ],
  // 整体的样式等
  "globalStyle": {
    "navigationBarTextStyle": "black",
    "navigationBarTitleText": "uni-app",
    "navigationBarBackgroundColor": "#F8F8F8",
    "backgroundColor": "#F8F8F8"
  },

  // tab页处理
  "tabBar": {
    "color": "#7A7E83",
    "selectedColor": "#3cc51f",
    "borderStyle": "black",
    "backgroundColor": "#fff",
    "list": [
      {
        "pagePath": "pages/index/index",
        "iconPath": "static/image/icon_component_HL.png",
        "selectedIconPath": "static/image/icon_component.png",
        "text": "首页"
      },
      {
        "pagePath": "pages/curriculum/index",
        "iconPath": "static/image/icon_API.png",
        "selectedIconPath": "static/image/icon_API_HL.png",
        "text": "课程"
      },
      {
        "pagePath": "pages/userCenter/index/index",
        "iconPath": "static/image/user.png",
        "selectedIconPath": "static/image/user_HL.png",
        "text": "我的"
      }
    ]
  }
}
```

## 配置页面相关信息

原生小程序都是在每个页面的.json 里配置的，在 uni-app 里是在 pages.json 的 `pages` 属性里配置的
每一个 `pages` 的元素都是一个对象，包括一个 `path` 一个 `style` 属性，相关的配置，都在 `style` 里比如 `navigationBarTitleText`，`disableScroll` 等

## 页面开发

### 创建页面

uni-app 把小程序和 Vue 的生命周期封装进了都封装好了，可以直接使用，直接创建一个.vue 文件

```html
<template>
  <view class="my-order"></view>
</template>

<script lang="ts">
  import { Component, Vue } from 'vue-property-decorator';

  @Component({
    name: 'MyOrder',
  })
  export default class MyOrder extends Vue {
    onLoad() {}
  }
</script>

<style lang="scss" scoped>
  @import '../user.scss';
</style>
```

### 页面跳转

页面的跳转，就会有几种方式：

```ts
// 直接跳转
uni.navigateTo({
  url: '/pages/otherPage/index?name=haha&id=2333',
});

// 重定向
uni.redirectTo({ url: 'test?id=1' });

// 跳转tab页
uni.switchTab({ url: '/pages/index/index' });

// 返回上一页
uni.navigateBack();
```

### 数据传递

大量的数据传递，可以放在 Vuex 里进行数据传输，少量的信息，可以在 url 上携带信息，通过 onLoad 方法中来获取相应的数据

```ts
// 上一个页面跳转方法
private switchToOtherPage ():void {
  uni.navigateTo({
    url: '/pages/otherPage/index?name=haha&id=2333'
  });
}

onLoad(params: any) {
  const { id, name } = params;
}
```

### 数据请求

数据请求一般放在 onShow 方法里

```ts
async onShow() {
  await this.getUserInfo();
}

// 获取用户信息
private async getUserInfo(): Promise<void> {
  const { code, msg, data } = await request.post(api.getUserInfo, {});
  if (code !== 200) {
    uni.showModal({
      title: '提示',
      content: msg
    });
    return;
  }
  this.$set(this, 'userInfo', data.userInfo);
}
```

### 自定义标题栏

```js
// 首先在pages里的style里把改成custom
style:{
    "navigationStyle": “custom"
}

// 然后页面的标题栏就没了，可以自定义

// 在页面里可以写个标题
<view class="nav-bar">测试标题</view>
// 就可以了
```

### 分享

```ts
onShareAppMessage(res: any): any {
  return {
    title: '页面分享',
    path: '/pages/userCenter/index',
    success: function(res: any) {
      console.log(res);
    }
  };
}
```

### 支付

```ts
uni.requestPayment({
  provider: 'wxpay’,
  timeStamp: String(Date.now()),
  nonceStr: 'A1B2C3D4E5’,
  package: 'prepay_id=wx20180101abcdefg’,
  signType: 'MD5’,
  paySign: '’,
  success: function (res) {
    console.log('success:' + JSON.stringify(res));
  },
  fail: function (err) {
    console.log('fail:' + JSON.stringify(err));
  }
});
```

### 授权

```ts
uni.authorize({
  scope: 'scope.userLocation',
  success() {
    uni.getLocation();
  },
});
```

### 登录

```ts
uni.login({
  provider: 'weixin’,
  success: function (loginRes) {
    console.log(loginRes.authResult);
    // 获取用户信息
    uni.getUserInfo({
      provider: 'weixin’,
      success: function (infoRes) {
        console.log('用户昵称为：' + infoRes.userInfo.nickName);
      }
    });
  }
});
```

具体登录信息等问题，可以根据需求来开发。
