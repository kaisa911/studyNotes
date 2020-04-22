# uni-app 开发环境搭建

都知道小程序越来越受欢迎，使用人数越来越多，但是原生小程序在开发过程中，总是感觉还停留在上个十年的 html+css+js 的过程中，不能使用框架，就很尴尬，最近发现可以使用 uni-app 这个类似 Taro 的多端合一的开发小程序的框架，特地来感受一下。

uni-app 开发微信小程序，使用 Vue 相关的语法，几乎可以无缝对接，现在就把大体的流程给记录一下。

因为习惯了 vsCode 开发，所以没有使用 uni-app 的的开发工具（嫌弃一下，mac 安装成功打不开）。所以使用的是 vue-cli 来安装和启动项目的。

## 安装 vue-cli

```bash
npm install -g @vue/cli
```

## 创建 uni-app 项目

```bash
vue create -p dcloudio/uni-preset-vue my-project
```

## 选择模版

![图图图](https://github.com/kaisa911/studyNotes/blob/master/src/uni-app%E9%A3%9F%E7%94%A8%E6%94%BB%E7%95%A5/images/img1.png?raw=true)

因为想体验 uni-app 的一些功能，所以就选用了默认模板（TypeScript）这样一个模板来开发，选择之后呢，继续等待…

![图图图](https://github.com/kaisa911/studyNotes/blob/master/src/uni-app%E9%A3%9F%E7%94%A8%E6%94%BB%E7%95%A5/images/img2.png?raw=true)

出现这样的情况呢，就算安装完成了～

## 进入项目

进入项目之后，我们会看到这样一个目录（当然还有 node_modules）
![图图图](https://github.com/kaisa911/studyNotes/blob/master/src/uni-app%E9%A3%9F%E7%94%A8%E6%94%BB%E7%95%A5/images/img3.png?raw=true)

这个目录就是 uni-app 生成的默认的 ts 模板。
在 sfc.d.ts 里声明了 Vue，让全局都可以使用到 Vue

```js
// sfc.d.ts
declare module "*.vue" {
  import Vue from 'vue'
  export default Vue
}
```

因为 uni-app 在 App.vue 和 pages 的.vue 文件里用的都是继承了 Vue 的类，这样在页面里还是使用的基本的 Vue 的基本 js 语法，与 TypeScript 没有太大的关系，所以这里做了一些改造，使用 `vue-property-decorator` 和 `vuex-class` 来处理 Vue 中使用 Typescript 的问题

## 改造 uni-app 的抛出的类

### 改造 App.vue（慎用，只能在小程序中使用）

```js
// 默认模版的App.vue
<script lang="ts">
import Vue from 'vue';
export default Vue.extend({
  mpType: 'app',
  onLaunch() {
    console.log('App Launch');
  },
  onShow() {
    console.log('App Show');
  },
  onHide() {
    console.log('App Hide');
  },
});
</script>

// 改造后
<script lang="ts">
import { Vue } from 'vue-property-decorator';

export default class App extends Vue implements Global.IVue {
  mpType: String = 'app';
  onLaunch() {
    console.log('App Launch');
  }
  onShow() {
    console.log('App Show');
  }
  onHide() {
    console.log('App Hide');
  }
}
</script>
```

创建一个 index.d.ts 用来写接口

```ts
declare namespace Global {
  interface IVue {
    mpType: String;
    onLaunch(): void;
    onShow(): void;
    onHide(): void;
  }
  interface IState {
    nickname: string;
  }
}
```

这样在全局中就可以不用引入接口了，直接写 Global.IVue 就可以使用了。

### 添加 store/index.ts

新建一个文件夹 store，文件夹里创建 index.ts

```ts
import Vue from 'vue';
import Vuex from 'vuex';

Vue.use(Vuex);
const store = new Vuex.Store({
  state: {
    nickname: '未设置',
  },
  mutations: {
    change: function (state: Global.IState, nickname: string): void {
      state.nickname = nickname;
    },
  },
});
export default store;
```

### 改造 main.ts

```js
// 默认模版
import Vue from 'vue';
import App from './App.vue';

Vue.config.productionTip = false;

new App().$mount();

// 改造后
import Vue from 'vue';
import App from './App.vue';
import store from './store/index';

Vue.config.productionTip = false;

const app = new Vue({
  store,
  ...App,
});
app.$mount();
```

### 页面改造

在改造页面的时候，因为使用了 Vuex，所以要添加一个新的库，来处理 Vuex 相关的问题

```bash
npm install vuex-class -S
```

安装这个库之后，需要在 tsconfig.json 里打开相关的属性

```json
"experimentalDecorators": true,
"strictFunctionTypes": true, // 使用vuex-class时打开
```

然后开始改造页面

```js
// 页面上的script
<script lang="ts">
import Vue from 'vue';
export default Vue.extend({
  data() {
    return {
      title: 'Hello',
    };
  },
  onLoad() {},
  methods: {},
});
</script>

// 改造之后
<script lang="ts">
import { Component, Vue } from 'vue-property-decorator';
import { State, Mutation } from 'vuex-class';

@Component({})
export default class Idnex extends Vue {
  private title: string = 'myTitle'; //响应式属性
  @State('nickname') private stateNickname!: string;
  @Mutation('change') private CHANGENICKNAME!: Function;

  onLoad() {
    let a: string = '123';
    this.CHANGENICKNAME('10');
  }
}
</script>
```

OK，通过这样的改造，就可以在 Vue 文件里很好的享用 TypeScript 了。

## 封装 request 和 api

把 uni-app 的请求方法封装成 async/await 的方式

```ts
// 封装request
import utils from './utils.js';

//请求接口函数
const request = (method: any, url: string, data: any, loading: boolean) => {
  return new Promise((resolve, reject) => {
    //显示加载动画
    if (loading) utils.showLoading();

    //发起请求
    uni.request({
      url,
      method,
      data,
      header: {
        // 数据被编码为名称/值对
        //"Content-Type": "application/x-www-form-urlencoded;charset=utf-8"
      },
      success: (res) => {
        resolve(res.data);
      },
      fail: (err) => {
        reject(err);
      },
      complete: () => {
        //结束加载动画
        if (loading) utils.hideLoading();
      },
    });
  });
};

export default {
  get: function (url: string, params: any, loading: boolean) {
    return request('GET', url, params, loading);
  },
  post: function (url: string, params: any, loading: boolean) {
    return request('POST', url, params, loading);
  },
};
```

```ts
// 封装api
const prefix: string = ‘';
const api = {};

export default api;
```

## 添加 MockJs

从 npm 上下载 mockjs

```bash
npm install mockjs -D
```

在 request 和 api 种做一下修改，添加 DEBUG 模式

```ts
// api
export const DEBUG: boolean = true;
const prefix: string = `${DEBUG ? '' : 'http://uni-app-eating-group/'}`;
const api = {};
export default api;
```

```ts
//request
import { DEBUG } from './api';
import mockFunc from '../mock/index';
…
if (DEBUG) {
  resolve(mockFunc(url));
  if (loading) utils.hideLoading();
  return;
}
…
```

在 src 下创建 mock 文件夹，里面创建 index.ts
![图图图](https://github.com/kaisa911/studyNotes/blob/master/src/uni-app%E9%A3%9F%E7%94%A8%E6%94%BB%E7%95%A5/images/img4.png?raw=true)

好的，做到这里，环境就算搭建的差不多啦。剩下的就是在搭建好的环境里开发小程序啦，下一部分，应该会是 uni-app 的关于小程序部分的流程啦～
