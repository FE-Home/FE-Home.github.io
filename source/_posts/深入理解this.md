---
title: 深入理解 this
date: 2018-08-26 21:47
categories:
  - JavaScript
tags:
  - this
---

# 深入理解 this

这篇文章主要参考[《你不知道的 JavaScript》](https://www.amazon.cn/dp/B00W34DZ8K)的`第2章—this全面解析`

首先说结论：

> this 实际上是在函数被调用时发生的绑定，它指向什么完全取决于函数在哪里被调用。

本文使用了大量代码来说明，请耐心阅读。

<!--more-->

## 两种误解

### 认为 this 指向"自己"

```javascript
function foo(num) {
  console.log("foo: " + num)
  this.count++
}

foo.count = 0

for (let i = 0; i < 10; i++) {
  if (i > 5) {
    foo(i)
  }
}
// foo: 6
// foo: 7
// foo: 8
// foo: 9

console.log(foo.count)
// 0
console.log(window.count)
// NaN
```

上面代码中的 this 并没有按照预期指向“自己”（foo）。结果是在全局作用域下创建了 count 变量，但是由于没有赋值，count 为 undefined，而`undefined++`返回的是 NaN，`NaN++`同理，所以最后输出的是 NaN。

### 认为 this 指向“作用域”

```javascript
function foo() {
  var a = 2
  bar()
}
function bar() {
  console.log(this.a)
}
foo()
// undefined
```

this 并没有按照预期指向作用域也就是 bar=>foo=>window 这一作用域，而是输出了 window.a，也就是 undefined。

---

## 绑定规则

this 的四条绑定规则分别是：默认绑定、隐式绑定、显式绑定和 new 绑定

### 默认绑定

默认绑定是无法应用其他三条规则时的默认规则

```javascript
function foo() {
  console.log(this.a)
}
var a = 2

/* 2
 * foo在调用时使用的是没有任何修饰的函数引用，因此采用默认绑定。
 */
foo()
```

在函数嵌套调用的时候也会默认绑定到全局

```javascript
function foo() {
  var a = 2
  console.log(this.a)
  bar()
}
function bar() {
  var a = 3
  console.log(this.a)
  baz()
}
function baz() {
  var a = 4
  console.log(this.a)
}
var a = 1
foo()
```

### 隐式绑定

正常的隐式绑定的效果是这样的：

```javascript
function foo() {
  console.log(this.a)
}
var obj1 = {
  a: 2,
  foo: foo
}
obj1.foo()
// 2
```

隐式丢失

```javascript
function foo() {
  console.log(this.a)
}
var obj = {
  a: 2,
  foo: foo
}
var a = "window"
var bar = obj.foo
bar() // "window"
//实际上是直接调用的foo()
```

简单来说就是，函数的传递是靠引用的，也就是“指针”，所以当`bar = obj.foo`的时候，bar 已经和 obj 没有关系了，直接指向原来的 foo 函数。  
这样的情况也会发生在函数作为参数传递到回调函数中的情况，使用时要注意。

### 显式绑定

隐式绑定有一个缺陷，就是我们必须在一个对象内部包含一个指向函数的属性，显示绑定可以强制在某个对象上调用函数。在 function 类型的原型上都有 call()和 apply()方法。

```javascript
function foo() {
  console.log(this.a)
}
var obj = {
  a: 2
}
foo.call(obj)
// 2
```

硬绑定

```javascript
function foo() {
  console.log(this.a)
}
var obj = {
  a: 2
}
var bar = function() {
  foo.call(obj)
}
bar()
// 2
bar.call(window)
// 2
```

硬绑定用于解决显式绑定下的绑定丢失问题。ES5 中提供了 Function.prototype.bind 方法用于硬绑定，返回一个硬绑定后的新函数。

### new 绑定

new 操作符在许多其他面向对象语言中都有，使用方法是`myClass = new MyClass()`，从而调用类的构造函数。但是在 JavaScript 中并不存在什么“构造函数”，只有对于函数的“构造调用”。  
使用 new 操作符调用函数时会执行以下操作：

1. 创建新的对象，相当于创建字面量{}
2. 构造函数指向 new 的函数 this.constructor = foo 该对象的原型链接到 Foo.prototype
3. 新对象绑定到函数调用的 this
4. 传入的参数赋给新对象

```javascript
function foo(a){
  this.a = a
}
var bar - new foo(2)
console.log(bar.a)
// 2
```

---

## 优先级

这四种规则的优先级，默认绑定是最低的。

显式绑定 > 隐式绑定

```javascript
function foo() {
  console.log(this.a)
}
var obj1 = {
  a: 2,
  foo: foo
}
var obj2 = {
  a: 3,
  foo: foo
}

obj1.foo() // 2
obj2.foo() // 3

obj1.foo.call(obj2) // 3
obj2.foo.call(obj1) // 2
```

显然，通过显示绑定调用已经隐式绑定的函数，最终的效果是显式的效果。

new > 显式绑定

```javascript
function foo(something) {
  this.a = something
}
var obj1 = {}
var bar = foo.bind(obj1)
bar(2)
console.log(obj1.a) // 2

var baz = new bar(3)
console.log(obj1.a) // 2
console.log(baz.a) // 3
```

new 操作符会检测硬绑定，并用新创建的 this 替换。

应用：预置参数

```javascript
function foo(p1, p2) {
  this.val = p1 + p2
}
var bar = foo.bind(null, "p1")
var baz = new bar("p2")
baz.val //p1p2
```

---

## 总结

1. 使用了 new ? 绑定到新创建的对象
2. 通过 call,apply 调用 ? 绑定到指定的对象
3. 通过对象隐式调用 ? 绑定到该对象
4. 使用默认绑定，严格模式下绑定到 undefined

## 箭头函数

箭头函数并不会应用以上四条规则，而是直接继承外层函数的 this，和平时用的`var me/self/that = this`一样的效果。（真香警告

---

## 参考文献

1. [《你不知道的 JavaScript》](https://www.amazon.cn/dp/B00W34DZ8K)
2. [《JavaScript 专家编程》](https://www.amazon.cn/dp/B01251LYTW)

<small>朱耀华\_20180826</small>
