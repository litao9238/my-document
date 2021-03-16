## Vue 单页SPA项目性能优化

- ##### [Gzip压缩](#gzip)
- ##### [CDN加速](#cdn)
- ##### [预加载](#prefetch)

#### <span id = "gzip">Gzip压缩</span>

>**gizp压缩是一种http请求优化方式, 通过减少文件体积来提高加载速度. html, js, css文件甚至json数据都可以用它压缩, 可以减小60%以上的体积.**

1. 安装 compression-webpack-plugin 插件
```
yarn add compression-webpack-plugin -D
```

2. 打开项目根目录的 vue.config.js
```js
// 是否为生产环境
const isProduction = process.env.NODE_ENV !== 'development';
// gzip压缩
const CompressionWebpackPlugin = require('compression-webpack-plugin')

module.exports = {
  configureWebpack: (config) => {
    // 生产环境相关配置
    if (isProduction) {
      config.plugins.push(
        // gzip压缩
        new CompressionWebpackPlugin({
          filename: '[path].gz[query]',
          algorithm: 'gzip',
          test: new RegExp('\\.(' + productionGzipExtensions.join('|') + ')$'),
          threshold: 10240, // 只有大小大于该值的资源会被处理 10240
          minRatio: 0.8, // 只有压缩率小于这个值的资源才会被处理
          deleteOriginalAssets: false // 删除原文件
        })
      )
    }
  }
}
```

3. 完成 打包后的js文件名则会显示为`xxx.js.gz`, 注意需要对服务器进行配置，根据Request Headers的Accept-Encoding标签进行鉴别, 如果支持gzip就返回.gz文件.


#### <span id="cdn">CDN加速</span>

>**我们工作中搭建Vue项目时, 一般都使用官方的vue cli进行搭建, 在打包的时候默认会把引入库的js, css打包进`vendor.js`,导致打包出来过大, 解决方案是把一些外部的库文件使用CND资源有利于网页的加载速度.**
>
>**下面以引入 `vue`, `vuex`, `vue-router` 为例**

1. 打开项目根目录的 vue.config.js 配置CDN列表
```js
// 是否为生产环境
const isProduction = process.env.NODE_ENV !== 'development'
// 本地环境是否需要使用CDN
const devNeedCdn = false
  // CDN链接
const cdn = {
  externals: {
    vue: 'Vue',
    vuex: 'Vuex',
    'vue-router': 'VueRouter'
  },
  // cdn的css链接
  css: [], 
  // cdn的js链接
  js: [
    'https://cdn.staticfile.org/vue/2.6.10/vue.min.js',
    'https://cdn.staticfile.org/vuex/3.0.1/vuex.min.js',
    'https://cdn.staticfile.org/vue-router/3.0.3/vue-router.min.js'
  ]
}
```

2. 配置环境变量及打包时忽略对应资源
```js
module.exports = {
  chainWebpack: config => {
    config
      .plugin('html')
      .tap(args => {
        // 生产环境或本地需要cdn时, 注入cdn环境变量
        if (isProduction || devNeedCdn) args[0].cdn = cdn
        return args
      })
  },
  configureWebpack: (config) => {
    // 用cdn方式引入, 则构建时要忽略相关资源
    if (isProduction || devNeedCdn) config.externals = cdn.externals
  }
}
```

3. 打开 public/index.html 引用CDN文件
```html
<!-- 使用CDN的CSS文件 --S-- -->
<% for (var i in htmlWebpackPlugin.options.cdn && htmlWebpackPlugin.options.cdn.css) { %>
<link href="<%= htmlWebpackPlugin.options.cdn.css[i] %>" rel="stylesheet"/>
<% } %>
<!-- 使用CDN的CSS文件 --E-- -->
```
```html
<div id="app"></div>
<!-- built files will be auto injected -->
<!-- 使用CDN的JS文件 --S-- -->
<% for (var i in htmlWebpackPlugin.options.cdn && htmlWebpackPlugin.options.cdn.js) { %>
<script src="<%= htmlWebpackPlugin.options.cdn.js[i] %>"></script>
<% } %>
<!-- 使用CDN的JS文件 --E-- -->
```

4. 打开 src/router/index.js
```js
import Vue from 'vue'
import Router, {RouteConfig} from 'vue-router'
// 把原来的Vue.use(Router)修改一下
if (!window.VueRouter) Vue.use(Router)
```

5. 完成, 当发布到线上环境时, 查看network下的`vue`, `vuex`, `vue-router` 会显示为CDN资源


#### <span id="prefetch">预加载</span>

>**利用浏览器空闲时间下载或预取指定资源, 缓存起来, 方便后续用户获取时能直接从缓存里拿到, 节省了加载时间**

1. 示例
```html
<!-- 预加载某个完整网页 -->
<link rel="prefetch" href="http://www.xxx.com/" />

<!-- 预加载某个资源 -->
<link rel="prefetch" href="http://www.xxx.com/js/a.js" as="script" />

```

2. 在Vue脚手架项目中可以配合一些其它设置 `vue.config.js`
```js
module.exports = {
  chainWebpack: config => {
    // 移除 prefetch 插件, 并手动操作, 
    // 具体参阅(https://cli.vuejs.org/zh/guide/html-and-static-assets.html#preload)
    config.plugins.delete('prefetch')
  }
}
```