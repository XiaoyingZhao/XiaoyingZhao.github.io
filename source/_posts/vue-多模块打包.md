---
title: vue-多模块打包
date: 2021-08-04 10:07:41
tags:
---
```js
/* eslint-disable no-useless-escape */
const path = require('path')
const glob = require('glob')
const projectname = process.argv[3]
console.log(projectname)
// 导入compression-webpack-plugin
const CompressionWebpackPlugin = require('compression-webpack-plugin')
// 定义压缩文件类型
const productionGzipExtensions = ['js', 'css']
process.env.VUE_APP_VERSION = require('./package.json').version
// const BundleAnalyzerPlugin = require('webpack-bundle-analyzer').BundleAnalyzerPlugin

// 配置pages多页面获取当前文件夹下的html和js
function getEntry (globPath) {
  let entries = {}
  let basename
  let tmp
  let pathname
  // 多项目打包
  if (process.env.NODE_ENV === 'production') {
    entries = {
      index: {
        // page的入口
        entry: `src/pages/${projectname}/${projectname}.js`,
        // 模板来源
        template: `src/pages/${projectname}/${projectname}.html`,
        // 在 dist/index.html 的输出
        filename: 'index.html',
        title: projectname
        // chunks: ['chunk-vendors', 'chunk-common', 'index']
      }
    }
    console.log(entries)
  } else {
    glob.sync(globPath).forEach(function (entry) {
      basename = path.basename(entry, path.extname(entry))
      tmp = entry.split('/').splice(-3)
      pathname = basename // 正确输出js和html的路径

      entries[pathname] = {
        entry: 'src/' + tmp[0] + '/' + tmp[1] + '/' + tmp[1] + '.js',
        template: 'src/' + tmp[0] + '/' + tmp[1] + '/' + tmp[2],
        filename: tmp[2]
      }
    })
  }

  return entries
}
let htmls = getEntry('./src/pages/**/*.html')

// 配置end
module.exports = {
  productionSourceMap: false,
  publicPath: '/',
  outputDir: 'dist/' + projectname,
  pages: htmls,
  // 统一配置打包插件
  configureWebpack: config => {
    const options = {
      externals: {
        'vue': 'Vue',
        'vue-router': 'VueRouter',
        'vuex': 'Vuex',
        'axios': 'axios'
      },
      resolve: {
        alias: {
          '@user': path.join(__dirname, './src/pages/user'),
          '@admin': path.join(__dirname, './src/pages/admin')
        }
      }
    }
    if (process.env.NODE_ENV === 'production') {
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
            deleteOriginalAssets: true // 是否删除源文件
          })
          // new BundleAnalyzerPlugin()
        ]
      }
    } else {
      return options
    }
  },
  chainWebpack: config => {
    config.module
      .rule('images')
      .use('url-loader')
      .loader('url-loader')
      .tap(options => {
        // 修改它的选项...
        options.limit = 10000
        return options
      })
    if (process.env.NODE_ENV === 'production') {
      const fonts = config.module.rule('fonts')
      fonts
        .use('url-loader')
        .loader('url-loader')
        .tap(options => {
          options = {
            limit: 10000,
            name: '[name].[ext]',
            //            publicPath: `http:${cdnPath}/fonts`,
            publicPath: '/fonts',
            outputPath: 'font/'
          }
          return options
        })
    }
  },
  pluginOptions: {
    'style-resources-loader': {
      preProcessor: 'less',
      patterns: [
        // 配置全局less变量自动引入，不用每次再手动引入
        path.join(__dirname, 'src/iview-theme/index.less')
      ]
    },
    optimization: {
      moduleIds: 'deterministic',
      runtimeChunk: 'single',
      splitChunks: {
        cacheGroups: {
          vendor: {
            test: /[\\/]node_modules[\\/]/,
            name: 'vendors',
            chunks: 'all',
            minChunkSize: 10000
          }
        }
      }
    }
  }
}

```