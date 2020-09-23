---
title: 学习日记vuecli-webpack配置-持续更新
date: 2020-07-10 10:14:09
tags:
---
```js
// 修改字体路径
chainWebpack: config => {
  const fonts = config.module.rule('fonts')
  fonts
    .use('url-loader')
    .loader('url-loader')
    .tap(options => {
      options = {
        limit: 10000,
        name: '[name].[ext]',
        publicPath: 'https://xxx',
        outputPath: 'font/'
      }
      return options
    })
}
```
```js
// Vue单页面预渲染
// vue.config.js
1. 第一步，publicPath的值必须是''或'/'
publicPath: '/'

2. 第二步，引用插件
const PrerenderSpaPlugin = require('prerender-spa-plugin')
const Renderer = PrerenderSpaPlugin.PuppeteerRenderer

new PrerenderSpaPlugin({
  staticDir: path.join(__dirname, '/dist'),
  routes: ['/', '/aboutus'],
  renderer: new Renderer({
    injectProperty: '__PRERENDER_INJECTED__',
    inject: 'prerender'
  })
})

// main.js
3. 第三步，main.js中触发
new Vue({
  router,
  store,
  i18n, // 多语言i18n
  render: h => h(App),
  /* 这句非常重要，否则预渲染将不会启动 */
  mounted () {
    document.dispatchEvent(new Event('render-event'))
  }
}).$mount('#app')

4. 如果publicPath是cdn地址，需要做单独处理，下一篇博客我将专门记录prerender-spa-plugin与静态资源cdn地址的处理
```