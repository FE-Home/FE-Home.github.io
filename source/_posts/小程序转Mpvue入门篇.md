title: 小程序转Mpvue入门篇
date: 2018/8/17
categories:
- baolong
---

本篇旨在整理小程序转mpVue遇到的一些基础事项

改造前：
```html
<input class="weui-cell__ft {{class}}" placeholder='--' value='{{item.userName}}' bindtap="gotoEdits" data-item="{{item}}" data-isAgent="false" disabled="{{disabledInput}}"></input>
```
改造后：
```html
<input class="weui-cell__ft" :class="class" placeholder='--' :model.lazy='item.userName' @click="gotoEdits(item, false)" :disabled="disabledInput"/>
```

<!--more-->
## 改造说明  

### 不可使用 data-xxx 属性
data-xxx 写法是小程序提供的标签值绑定，有利于标签方法获取到需要的值，但是vue中支持对方法直接传值，同时不支持 data-xxx写法，所以需要将 data-xxx 中的值直接写成以下方式

```javascript
bindtap="gotoEdits" data-item="{{item}}" data-isAgent="false"
```
转换成：

```javascript
@click="gotoEdits(item, false)"
```
### 正常样式和动态样式需要拆分
小程序可以使用 xxx 绑定样式，vue则需要使用 `:class=""` 绑定样式

```javascript
class="weui-cell__ft {{class}}"
```
转换成：

```javascript
class="weui-cell__ft" :class="class"
```
### value值的两种改造方式

```javascript
:value = "item.userName"
```
或者：（推荐使用第二种双向绑定，这样就不需要再去手动赋值）

```javascript
:model = "item.userName"
```


### 其他属性值

大部分其他属性值遇到动态绑定的需要在属性前添加 `:`， 并去除双 {} 符号

- #### if属性

```javascript
wx:if="item"
```
转换成：

```javascript
v-if="item"
```
- #### for 遍历

```javascript
wx:for="Arr"
```
转换成：

```javascript
v-for="(item , index) in Arr" :key="index"
```
### 事件对照表


```javascript
// 事件映射表，左侧为 WEB 事件，右侧为 小程序 对应事件
{
    click: 'tap',
    touchstart: 'touchstart',
    touchmove: 'touchmove',
    touchcancel: 'touchcancel',
    touchend: 'touchend',
    tap: 'tap',
    longtap: 'longtap',
    input: 'input',
    change: 'change',
    submit: 'submit',
    blur: 'blur',
    focus: 'focus',
    reset: 'reset',
    confirm: 'confirm',
    columnchange: 'columnchange',
    linechange: 'linechange',
    error: 'error',
    scrolltoupper: 'scrolltoupper',
    scrolltolower: 'scrolltolower',
    scroll: 'scroll'
}
```
在 `input` 和 `textarea` 中 `change` 事件会被转为 `blur` 事件。



