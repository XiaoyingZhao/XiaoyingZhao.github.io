---
title: vue多页面预渲染与静态资源cdn地址的处理
date: 2020-09-23 17:22:09
tags: js
categories: 
  - js
  - 学习日记
  - prerender-spa-plugin 
---
#### 目标
* 需要预渲染的页面实现预渲染
* 页面中静态资源地址使用cdn静态资源地址

#### 主要问题
publicPath在打包使用prerender-spa-plugin预渲染插件时publicPath必须为'','/'或者页面运行时服务端地址（127.0.0.1:port）

#### 尝试解决
prerender-spa-plugin 插件使用时可以使用postProcess函数对预渲染生成的页面进行进一步处理
```js
const cdnPath = '//cdn.com'
new PrerenderSpaPlugin({
    staticDir: path.join(__dirname, '/dist'),
    routes: ['/', '/aboutus'],
    postProcess (renderedRoute) {
      // 将地址替换
      renderedRoute.html = renderedRoute.html.replace(
        /(<script[^<>]*src=\")((?!http|https)[^<>\"]*)(\"[^<>]*>[^<>]*<\/script>)/ig,
        `$1${cdnPath}$2$3`
      ).replace(
        /(<link[^<>]*href=\")((?!http|https)[^<>\"]*)(\"[^<>]*>)/ig,
        `$1${cdnPath}$2$3`
      )
      return renderedRoute
    },
    renderer: new Renderer({
      injectProperty: '__PRERENDER_INJECTED__',
      inject: 'prerender'
    })
  })
```
如果是单页面这样就OK了，但我的项目是多页面应用，这几个多页面分别是index.html，admin.html，other.html，只有index.html中的`/` `about`这两个路由需要预渲染，我如果改了`vue.config.js`文件中的publicPath路径的话整个项目在打包时都换变成'/'路径，所以需要将`index.html `页面与其他页面分开打包。
```js
// package.json
 "scripts": {
    "build-index": "vue-cli-service build --mode index",
  }
```
#### 科普环境变量文件使用方式
```
为一个特定模式准备的环境文件 (例如 .env.production) 将会比一般的环境文件 (例如 .env) 拥有更高的优先级。


模式是 Vue CLI 项目中一个重要的概念。默认情况下，一个 Vue CLI 项目有三个模式：

development 模式用于 vue-cli-service serve
production 模式用于 vue-cli-service build 和 vue-cli-service test:e2e
test 模式用于 vue-cli-service test:unit

# 你可以替换你的项目根目录中的下列文件来指定环境变量：
.env                # 在所有的环境中被载入
.env.local          # 在所有的环境中被载入，但会被 git 忽略
.env.[mode]         # 只在指定的模式中被载入
.env.[mode].local   # 只在指定的模式中被载入，但会被 git 忽略

# 一个环境文件只包含环境变量的“键=值”对：
FOO=bar
VUE_APP_SECRET=secret

# 被载入的变量将会对 vue-cli-service 的所有命令、插件和依赖可用。
```

所以上面`--mode index`相当于指定模式为index，这里我们还需要一个index模式的环境变量文件`.env.index`
```js
// .env.index文件

// 指定其他模式都会默认是development模式，所以这里需要显式指定为production模式
NODE_ENV = 'production'
// 这里变量是自己定义的（这里用来区别打包时的页面）
VUE_APP_PAGE = 'index'
```

下面这些代码的作用：
1. 将index.html页面和其他页面单独输出打包
2. index.html页面打包时资源路径是'/'，执行预渲染之后再将链接替换成cdn地址
3. 其他页面打包时资源路径是'//cdn.com'

副作用：index页面无法使用路由懒加载（静态资源文件的引用路径无法修改）

```js
// package.json

// --no-clean表示打包时不删除dist文件
/**
1. vue-cli-service build 打包其他页面
2.  vue-cli-service build --mode index --no-clean 打包index页面不删除dist文件直接覆盖
*/
"scripts": {
    "build": "vue-cli-service build && vue-cli-service build --mode index --no-clean "
  },
```

```js
// vue.config.js

// publicPath配置
publicPath: (process.env.VUE_APP_PAGE === 'index' || process.env.NODE_ENV === 'development') ? '/' : '//cdn.com'
```
```js
// vue.config.js

// 配置pages多页面获取当前文件夹下的html和js
function getEntry (globPath) {
  let entries = {}
  let basename
  let tmp
  let pathname
  let result

  glob.sync(globPath).forEach(function (entry) {
    basename = path.basename(entry, path.extname(entry))
    tmp = entry.split('/').splice(-3)
    pathname = basename // 正确输出js和html的路径

    // 打包环境
    if (process.env.NODE_ENV === 'production') {
      if (basename === 'index') {
        result = {
          index: {
            entry: 'src/' + tmp[0] + '/' + tmp[1] + '/' + tmp[1] + '.js',
            template: 'src/' + tmp[0] + '/' + tmp[1] + '/' + tmp[2],
            filename: tmp[2]
          }
        }
        return true
      }
      entries[pathname] = {
        entry: 'src/' + tmp[0] + '/' + tmp[1] + '/' + tmp[1] + '.js',
        template: 'src/' + tmp[0] + '/' + tmp[1] + '/' + tmp[2],
        filename: tmp[2]
      }
    } else {
      // 运行环境
      entries[pathname] = {
        entry: 'src/' + tmp[0] + '/' + tmp[1] + '/' + tmp[1] + '.js',
        template: 'src/' + tmp[0] + '/' + tmp[1] + '/' + tmp[2],
        filename: tmp[2]
      }
    }
  })
  // 打包时index模式
  if (process.env.VUE_APP_PAGE === 'index') {
    return result
  }

  return entries
}
let htmls = getEntry('./src/pages/**/*.html')

```

```js
// vue.config.js

// 统一配置打包插件
  configureWebpack: config => {
    // index.html打包
    if (process.env.NODE_ENV === 'production' && process.env.VUE_APP_PAGE === 'index') {
      return {
        ...options,
        plugins: [
          new CompressionWebpackPlugin({
            filename: '[path].gz[query]',
            algorithm: 'gzip',
            test: new RegExp('\\.(' + productionGzipExtensions.join('|') + ')$'), // 匹配文件名
            threshold: 10240, // 对10K以上的数据进行压缩
            minRatio: 0.8,
            // 删除源文件, 配合webpack-bundle-analyzer时不能删除源文件，且需要配置静态服务器
            deleteOriginalAssets: false // 是否删除源文件
          }),
          new PrerenderSpaPlugin({
            staticDir: path.join(__dirname, '/dist'),
            routes: ['/', '/aboutus'],
            postProcess (renderedRoute) {
              // add CDN
              renderedRoute.html = renderedRoute.html.replace(
                /(<script[^<>]*src=\")((?!http|https)[^<>\"]*)(\"[^<>]*>[^<>]*<\/script>)/ig,
                `$1${cdnPath}$2$3`
              ).replace(
                /(<link[^<>]*href=\")((?!http|https)[^<>\"]*)(\"[^<>]*>)/ig,
                `$1${cdnPath}$2$3`
              )
              return renderedRoute
            },
            renderer: new Renderer({
              injectProperty: '__PRERENDER_INJECTED__',
              inject: 'prerender'
            })
          })
        ]
      }
    } else if (process.env.NODE_ENV === 'production') {
      // 其他页面打包
      return {
        ...options,
        plugins: [
          new CompressionWebpackPlugin({
            filename: '[path].gz[query]',
            algorithm: 'gzip',
            test: new RegExp('\\.(' + productionGzipExtensions.join('|') + ')$'), // 匹配文件名
            threshold: 10240, // 对10K以上的数据进行压缩
            minRatio: 0.8,
            // 删除源文件, 配合webpack-bundle-analyzer时不能删除源文件，且需要配置静态服务器
            deleteOriginalAssets: false // 是否删除源文件
          })
        ]
      }
    } else {
      // 运行环境
      return options
    }
  },
```