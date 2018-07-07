---
title: webpack caching
date: 2017-07-24 10:36:31
tags: [webpack]
categories: [打包]
---

> 参考 https://webpack.js.org/guides/caching/

使用webpack打包好久了，但是对具体的配置的含义，有的还是不明白，其中一个就是webpack caching。
webpack caching主要目的是，使浏览器缓存公共文件，不必每次都从服务器请求新的bundle文件。
下面是基本的配置。
```js
// webpack.config.prod.js
const path = require('path');
const webpack = require('webpack');
const HtmlWebpackPlugin = require('html-webpack-plugin')
module.exports = {
  context: path.resolve(__dirname, ".."),//我把配置文件放在了config文件夹下
  entry: {
    app: ['./src/index.js'],
    vendor: [
      'react',
      'react-dom',
      'redux',
      'react-router'
    ]
  },
  output: {
    path: path.resolve(__dirname, '../dist'),
    filename: "[name].[chunkhash:8].js",
    chunkFilename: "[name].[chunkhash:8].js",
    publicPath: '/'
  },
  module: {
    rules: [
      {
        test: /\.js$/,
        exclude: /node_modules/,
        loader: 'babel-loader',
        options: {
          presets: [['env', { modules: false }], 'react'],
          plugins: ['transform-runtime'],
          cacheDirectory: true
        }
      },
      {
        test: /\.css$/,
        use: ['style-loader', 'css-loader']
      }
    ]
  },
  plugins: [
    new webpack.optimize.CommonsChunkPlugin({
      name: ["vendor","manifest"],
      filename: '[name].[chunkhash:8].js',
    }),
    new HtmlWebpackPlugin({
      filename: 'index.html',
      template: 'index.html'
    }),
    new webpack.NamedModulesPlugin(),
  ]
}
```
接下来对上面配置做个具体分析。
对于有多个入口的打包，如果不提取公共文件，打包后的文件是不相相互影响的。但是提取公共文件后，公共文件包含公共manifest，这样打包后的文件就是相互影响的了。
输出文件文件名配置有[hash]和[chunkhash]，他们的区别是如下
* [hash]这种方式，所有的文件hash后缀,都会相同，因此每次打包只要有一个文件改变所有的文件名都会改变。但是只有文件内容改变的文件，打包后的文件内容才会改变。
* [chunkhash]这种方式会为每个文件单独起一个文件名，文件hash根据每一个文件的内容的hash值确定。因此文件内容不改变，文件名不变。如果不提取公共文件，没有改变的bundle，依旧可以被浏览器缓存。

如果将公共代码提取出来，那么公共文件vendor中包含manifest (along with bootstrapping/runtime code) ，在每次应用代码文件修改的时候，文件的hash都会改变，在公共文件的bootstrapping中含有文件的哈希值，因此每次文件改变，公共文件内容也会改变，这就使得公共文件的意义不大了。因此需要把manifest单独提取出来。（ps:可以自己看一下manifest文件的内容，可以比对两次打包manifest内容的变化）。

这里没有使用webpack官网的配置方式，因为没有必要，manifest文件不太大，每次文件改变都对manifest文件重新发出请求的开销还是可以接受的。
上面的配置文件中使用了，`HtmlWebpackPlugin`插件，每次打包会在index.html中自动插入打包后的文件链接。就算多次打包文件名发生改变，也不用手动更改引用。