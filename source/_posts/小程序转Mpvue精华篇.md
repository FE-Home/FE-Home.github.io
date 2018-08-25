title: 小程序转Mpvue精华篇
date: 2018/8/17
categories:
- baolong
---

本篇旨在整理小程序转mpvue中可能遇到的高级坑
- 组件库选择
- mpVue生命周期迁移
- data迁移与使用
- 默认事件值获取
- 页面栈数据获取
- 页面不卸载问题
- module.exports 不可用
- 新建页面需要重新执行 npm run dev
- 模拟器正常，手机查看页面数据不显示

<!--more-->

### 组件库使用

- [mpvue框架](http://mpvue.com/mpvue/#_13)

- [请求处理 flyio](https://juejin.im/post/5aaf222b51882555791873ed)

- [样式库mp-weui](https://youngluo.github.io/mp-weui/)


### 生命周期迁移方式

#### 注意：
- created：页面的created函数会在项目加载的时候被一起调用，进入页面的时候不会再被调用，所以这个钩子不能使用，推荐用小程序的原生的`onLoad`钩子
- mounted：页面返回时`mounted`钩子不会被触发，页面没有被重新加载，所以不能替代 `onShow`

综上所述: 还是使用小程序的生命周期，mpvue提供的生命周期不足以支持小程序的需求。




### data迁移与使用
小程序 `js` 中的 `data{}` 属性可全部迁移到 `vue` 的 `data` 中，但需要注意的是小程序内数据的引用比 `mpvue` 多一层 `data`

```javascript
//小程序
data: {
  flag: false
}

引用： this.data.flag
```
转换为:

```javascript
//mpvue
data() {
 return {
   flag: false
 }
}

引用： this.flag
```

### 默认事件值获取

mpvue将小程序默认事件返回值封了一层对象，命名为 `mp`

```javascript
//小程序
bindchange="bindChange"

bindChange: function(e) {
   console.log(e.detail.value )
}
```
转换成:

```javascript
//mpvue
@change="bindChange"

bindChange: function(e) {
   console.log(e.mp.detail.value )
}
```

### 页面栈数据获取
mpvue将小程序页面栈数据结构加了两层，命名为 `$root` 并将`data`对象封进了数组
```javascript
//小程序
this.currentPages[1].data;
```
转换成:

```javascript
//mpvue
this.currentPages[1].data.$root[0];
```

### 页面返回不卸载问题
小程序中页面返回后该页面就会出栈，下次进入重新触发 onLoad，但是 mpvue 返回后页面没有销毁，数据会缓存，所以进入页面我们需要重置数据。

```javascript
 onLoad() {
   // 解决页面返回后，数据留存问题
   Object.assign(this, this.$options.data());
 },
```
### module.exports 不可用
`webpack` 可以使用 `require` 和 `export` ，但是不能混合使用 `import` 和 `module.exports`

解决办法


```javascript
module.exports = {}
```
转化为：

```javascript
export default {}
```
或者：更新根目录下的`.babelrc`文件配置


```json
{
    "presets": [
        ["env", {
            "modules": false,
            "targets": {
                "browsers": ["> 1%", "last 2 versions", "not ie <= 8"]
            }
        }],
        "stage-2"
    ],
    "plugins": [
        ["transform-runtime", {
            "polyfill": false,
            "regenerator": true
        }]
    ],
    "env": {
        "test": {
            "presets": ["env", "stage-2"],
            "plugins": ["istanbul"]
        }
    }
}
```





### 新建页面需要重新执行 npm run dev
新建页面需要重新执行 npm run dev 才会生效

### 模拟器正常，手机查看页面数据不显示
导致这种问题的原因可能是其他页面有执行错误，虽然模拟器可以正常使用，但是手机端的js就会停止执行，导致页面数据不渲染。解决其他页面的报错，或暂时删除报错页面的 `mian.js` 可解决移动端无法查看问题。


