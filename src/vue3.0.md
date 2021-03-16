## Vue 3.0 + Vite 之初体验

- 代码仓库: 
- 线上地址: 


###涉及技术点
>vite版本、vue3、ts、集成路由、集成vuex、集成axios、配置Vant3、移动端适配、请求代理

#
###创建项目 vite + ts + vue3 只需要一行命令
```
npm init @vitejs/app my-vue-app --template vue-ts
```

#
###配置路由
- 安装
```
npm install vue-router@4 --save
```
- 在src下新建router目录，新建```index.ts```文件
```js
import { createRouter, createWebHashHistory, RouteRecordRaw } from "vue-router";
const routes: Array<RouteRecordRaw> = [
  {
    path: "/",
    name: "Home",
    meta: {
      title: "首页",
      keepAlive: true
    },
    component: () => import("../views/Home/index.vue"),
  },
  {
    path: "/login",
    name: "Login",
    meta: {
      title: "登录",
      keepAlive: true
    },
    component: () => import("../views/Login/index.vue"),
  },
];
const router = createRouter({
  history: createWebHashHistory(),
  routes
});
export default router;
```

- 在```main.ts```挂载路由
```js
import { createApp } from 'vue'
import App from './App.vue'
import router from "./router";
createApp(App)
.use(router)
.mount('#app')
```
#
###配置数据中心vuex(4.x)
- 安装
```
npm i vuex@next --save
```

- 配置
> 在src下创建store目录，并在store下创建```index.ts```
```js
import { createStore } from "vuex";
export default createStore({
  state: {
    listData:{1:10},
    num:10
  },
  mutations: {
    setData(state,value){
        state.listData=value
    },
    addNum(state){
      state.num=state.num+10
    }
  },
  actions: {
    setData(context,value){
      context.commit('setData',value)
    },
  },
  modules: {}
});
```

- 挂载
> 在```main.ts```挂载数据中心
```js
import { createApp } from 'vue'
import App from './App.vue'
import router from "./router";
import store from "./store";
createApp(App)
.use(router)
.use(store)
.mount('#app')
```

#
###Vant3
- 安装
> vite版本不需要配置组件的按需加载，因为Vant 3.0 内部所有模块都是基于 ESM 编写的，天然具备按需引入的能力，但是样式必须全部引入137.2k
```
npm i vant@next -S
```

- 在main.ts全局引入样式
```js
import { createApp } from 'vue'
import App from './App.vue'
import router from "./router";
import store from "./store";
import 'vant/lib/index.css'; // 全局引入样式
createApp(App)
.use(router)
.use(store)
.use(ant)
.mount('#app')
```

- 移动端适配
> 安装```postcss-pxtorem```
```
npm install postcss-pxtorem -D
```
> 在根目录下创建```postcss.config.js```
```js
module.exports = {
  "plugins": {
    "postcss-pxtorem": {
      rootValue: 37.5, 
      // Vant 官方根字体大小是 37.5
      propList: ['*'],
      selectorBlackList: ['.norem'] 
      // 过滤掉.norem-开头的class，不进行rem转换
    }
  }
}
```
> 在根目录src中新建util目录下新建```rem.ts```等比适配文件
```js
// rem等比适配配置文件
// 基准大小
const baseSize = 37.5 
// 注意此值要与 postcss.config.js 文件中的 rootValue保持一致
// 设置 rem 函数
function setRem () {
  // 当前页面宽度相对于 375宽的缩放比例，可根据自己需要修改,一般设计稿都是宽750(图方便可以拿到设计图后改过来)。
  const scale = document.documentElement.clientWidth / 375
  // 设置页面根节点字体大小（“Math.min(scale, 2)” 指最高放大比例为2，可根据实际业务需求调整）
  document.documentElement.style.fontSize = baseSize * Math.min(scale, 2) + 'px'
}
// 初始化
setRem()
// 改变窗口大小时重新设置 rem
window.onresize = function () {
  console.log("我执行了")
  setRem()
}
```
> 在mian.ts引入
```js
import "./utils/rem"
```

#
###Axios
- 安装
```
npm i -s axios
```
- 配置axios
> 在src创建utils文件夹,并在utils下创建request.ts
```js
import axios from "axios";
const service = axios.create({
  baseURL,
  timeout: 5000 // request timeout
});
// 发起请求之前的拦截器
service.interceptors.request.use(
  config => {
    // 如果有token 就携带tokon
    const token = window.localStorage.getItem("accessToken");
    if (token) {
      config.headers.common.Authorization = token;
    }
    return config;
  },
  error => Promise.reject(error)
);
// 响应拦截器
service.interceptors.response.use(
  response => {
    const res = response.data;

    if (response.status !== 200) {
      return Promise.reject(new Error(res.message || "Error"));
    } else {
      return res;
    }
  },
  error => {
    return Promise.reject(error);
  }
);
export default service;
```
- 使用
```js
import request from "../utils/request";
request({url: "/profile ",method: "get"})
.then((res)=>{
  console.log(res)
})
```
- 配置请求代理 ```vite.config.ts```
```js
import { defineConfig } from 'vite'
import vue from '@vitejs/plugin-vue'
import path from 'path';

// https://vitejs.dev/config/
export default defineConfig({
  plugins: [vue()],
  base:"./",//打包路径
  resolve: {
    alias:{
      '@': path.resolve(__dirname, './src')//设置别名
    }
  },
  server: {
    port:4000,//启动端口
    open: true,
    proxy: {
      // 选项写法
      '/api': 'http://123.56.85.24:5000'//代理网址
    },
    cors:true
  }

})

```

#
###vite对```<script setup>```和```<style vars>```的支持格外友好
- 正常写法
```html
<script lang="ts">
import { defineComponent } from "vue";
import { useRouter } from "vue-router";
export default defineComponent({
  setup() {
    const router = useRouter();
    const goLogin = () => {
      router.push("/");
    };
    return { goLogin };
  },
});
</script>
```
- ```<script setup>```写法
```html
<script lang="ts" setup="props">
import { useRouter } from "vue-router";
const router = useRouter();
const goLogin = () => {
  router.push("/");
};
</script>
```
> 是不是简洁了很多

- ```<style vars>```如何用？
```html
<script lang="ts" setup="props">
const state = reactive({
  color: "#ccc",
});
</script>
```
```html
<style >
.text {
  color: v-bind("state.color");
}
</style>
```
> 就这么简单！