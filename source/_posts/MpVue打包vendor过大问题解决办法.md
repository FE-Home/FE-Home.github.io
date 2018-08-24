title: MpVue打包vendor过大问题解决办法
date: 2018/8/24
categories:
- baolong
---
## MpVue打包vendor过大问题解决办法

### 问题描述： 
mpvue 打包的小程序会自动将重复引用打包到 `static` -> `js` -> `vendor`, 当引用的第三方库过多时，`vendor` 就会超出 500K ，小程序开发者工具有 500k 限制，大于 500k 的包不进行转码和压缩，此时就可能导致预览失败，文件体积超过 2M 限制

<!--more-->
#### 解决办法：
1、减少不必要第三方库的使用，或采用精简版的库，满足功能即可
2、拆分 `vendor` ，使每个 `js` 文件体积小于 500K 开发者工具进行一次压缩就可能低于 2M 了

本篇文章主要描述如何采用第二种方法

### 拆分 `vendor` 具体步骤
#### 1.对 `vendor` 进行拆包
- 找到 `build` -> `webpack.dev.conf.js`, mpvue 的 `npm run dev` 使用的就是这份打包配置
- 找到 `CommonsChunkPlugin`, 这是 `webpack` 提供的拆包工具 [详解CommonsChunkPlugin](https://segmentfault.com/a/1190000012828879)
- 修改如下配置

```javascript
//原配置
new webpack.optimize.CommonsChunkPlugin({
      name: 'vendor',
      minChunks: function (module, count) {
        // console.log(module.resource)
        // any required modules inside node_modules are extracted to vendor
        return (
          module.resource &&
          /\.js$/.test(module.resource) &&
          module.resource.indexOf('node_modules') >= 0
        )|| count > 1
      }
    }),
new webpack.optimize.CommonsChunkPlugin({
      name: 'manifest',
      chunks: ['vendor']
  	}),
```
修改为： 


```javascript
new webpack.optimize.CommonsChunkPlugin({
      name: 'vendor',
      minChunks: function (module, count) {
        // console.log(module.resource)
        // any required modules inside node_modules are extracted to vendor
        return (
          module.resource &&
          /\.js$/.test(module.resource) &&
          module.resource.indexOf('node_modules') >= 0
        )|| count > 1
      }
    }),

    //新增打包文件
    new webpack.optimize.CommonsChunkPlugin({
      name: 'webim', //新打包文件名
      chunks: ['vendor'], //拆分模块名
      minChunks: function (module, count) {
        console.log(module.resource + count)
        // 以下是拆分规则，返回true 则拆分，以下规则是将 libs 下的文件单独打包
        return (
          module.resource &&
          /\.js$/.test(module.resource) &&
          module.resource.indexOf('libs') >= 0 && 
          module.resource.indexOf('node_modules') === -1
        )
      }
    }),
	
//修改入口
new webpack.optimize.CommonsChunkPlugin({
      name: 'manifest',
      chunks: ['webim']
    }),

```
需要注意的是拆分顺序不能乱， `vendor` 拆出来的模块才会到 `webim`, 如果 `vendor` 对所有模块的返回值都是 false，则 `webim` 也为空,经过摸索，我的理解是 `webim` 是对 `vendor` 进行了再拆分，所以需要拆分的模块一定要先经过 `vendor` 的筛选。

经过以上配置， 你的 `dist` -> `static` -> `js` 下应该多了 `webim.js`,说明已经拆分成功

#### 2. 修改 mpvue-loader 模板
打包后的文件 `dist` -> `pages` 里的页面文件， `.js`中默认引入以下模块

```javascript
require('../../../static/js/manifest')
require('../../../static/js/vendor')
require('../../../static/js/pages/caseAdvice/main')
```
并没有刚才拆分出来的 `webim` 的引用，此时程序虽然打包成功，但是缺少模块无法运行

需要修改 `node_modules` 中 `mpvue-loader` -> `lib` -> `mp-compiler` -> `templates.js`


```javascript
//源码

require('${prefix}static/js/manifest')
require('${prefix}static/js/vendor')
require('${prefix}static/js/${name}')
```
修改为： 


```javascript
//修改后
require('${prefix}static/js/manifest')
require('${prefix}static/js/webim')
require('${prefix}static/js/vendor')
require('${prefix}static/js/${name}')
```
注意：模板的引用顺序不能乱需要按照 `webpack.dev.conf.js` 中的配置从后往前引入 。

#### 3.修改了 `node_modules` 中的包，会导致团队 npm install 出来的结果不一致，所以需要把 `mpvue-loader` 单独拿出来迁移到项目里。为了 webpack 任然能使用 mpvue-loader 需要在 `build` -> `webpack.base.conf.js` 中添加 loader 查询路径配置

```javascript
  resolveLoader: {
    modules: [
      'node_modules', //默认路径
      path.resolve("./src/lib") //添加的配置路径
    ]
  },
  module: {
    rules: [...此内容不变，故省略]
  }
```
使用 `resolveLoader.modules` 配置，webpack 将会从这些目录中搜索这些 loaders。不添加配置默认会在 `node_modules` 中查找，添加配置之后，会按照配置逐级查询， 我的 `mpvue-loader` 就放在 `src` 下的 `lib` 中,详情可参考 [webpack3.0之loader配置及编写](https://www.cnblogs.com/dupd/p/8418874.html)

#### 4. 如果执行了第三步，请将 `node_modules` 中的 `mpvue-loader` 删除，并删除 `package.json` 中的 `mpvue-loader`，并添加 `mpvue-loader` 的运行依赖到当前项目 `package.json` 的开发依赖中，以下提供我的 `mpvue-loader` 的运行依赖

```javascript
    "babelon": "^1.0.5",
    "consolidate": "^0.14.0",
    "hash-sum": "^1.0.2",
    "js-beautify": "^1.6.14",
    "loader-utils": "^1.1.0",
    "lru-cache": "^4.1.1",
    "postcss": "^6.0.6",
    "postcss-load-config": "^1.1.0",
    "postcss-selector-parser": "^2.0.0",
    "resolve": "^1.3.3",
    "source-map": "^0.5.6",
    "vue-hot-reload-api": "^2.1.0",
    "vue-loader": "^13.0.4",
    "vue-style-loader": "^3.0.0",
    "vue-template-es2015-compiler": "^1.5.3"

```

---

至此 `vendor` 的拆解工作完成，`vendor` 体积小于500K, 小程序开发者工具顺利完成打包工作。此方法治标不治本，最好的方法还是删库，删需求，小程序提倡简洁，如果功能太多，拆成两个小程序就好，肯德基的小程序就是这么做的。


