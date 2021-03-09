## webpack 自定义插件

示例: 给项目添加一个打包时间的变量, 用于线上环境查看是否为最新版本

1. 在根目录新建一个`WebpackPluginVersion.js`
```js
const fs = require('fs');
const path = require('path');
// 获取路径
const htmlPath = path.resolve(__dirname, './public/index.html');

class WebpackPluginVersion {
  apply(compiler) {
    compiler.hooks.done.tap('done', function() {
      // 以本文的形式读取文件
      const htmlText = fs.readFileSync(htmlPath, 'utf-8')
      // 写入一段script
      const scriptHtml = `
        <script>
          window.buildTime = '${dateFormat("YYYY-mm-dd HH:MM:SS", new Date())}'
        </script>
      `;
      // 使用正则替换原来的htmlText
      const newHtmlText = htmlText.replace('</body>', scriptHtml + '</body>')
      // 写入
      fs.writeFileSync(path.resolve(__dirname, 'dist/index.html'), newHtmlText);
    })
  }
}
module.exports = WebpackPluginVersion;

const dateFormat = (fmt, date) => {
  let ret;
  const opt = {
    "Y+": date.getFullYear().toString(),        // 年
    "m+": (date.getMonth() + 1).toString(),     // 月
    "d+": date.getDate().toString(),            // 日
    "H+": date.getHours().toString(),           // 时
    "M+": date.getMinutes().toString(),         // 分
    "S+": date.getSeconds().toString()          // 秒
    // 有其他格式化字符需求可以继续添加，必须转化成字符串
  }
  for (let k in opt) {
    ret = new RegExp("(" + k + ")").exec(fmt);
    if (ret) {
      fmt = fmt.replace(ret[1], (ret[1].length === 1) ? (opt[k]) : (opt[k].padStart(ret[1].length, "0")))
    }
  }
  return fmt;
}
```

2. 打开`vue.config.js`添加插件
```js
// 是否为生产环境
const isProduction = process.env.NODE_ENV !== 'development';
// 版本号
const webpackPluginVersion = require('./WebpackPluginVersion');

module.exports = {
  devServer: {
    port: 8077,
    disableHostCheck: true,
  },
  configureWebpack: (config) => {
    // 生产环境相关配置
    if (isProduction) {
      config.plugins.push(
        // 版本号
        new webpackPluginVersion(),
      )
    }
  },
}
```

3. 当执行`yarn build`后, dist目录下的`index.html` 会注入上面所添加的内容