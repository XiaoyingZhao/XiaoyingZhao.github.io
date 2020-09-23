---
title: 学习日记vue配置-持续更新
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