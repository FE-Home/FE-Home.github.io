title: 绑定后端返回的popover标签
date: 2018/8/31
categories:
- javascript
---

背景：在做hackathon项目时，遇到一个需求：是一段话中，部分专业名词需要点击弹出popover框来解释

gs-ui用法：
```html
 <gs-popover
    ref="popover2"
    supernatant
    placement="bottom"
    width="200"
    trigger="click"
  >
    浮层浮层浮层浮层浮层浮层浮层浮层浮层浮层浮层浮层浮层浮层浮层浮层浮层浮层浮层浮层
  </gs-popover>
  <gs-button type='primary' v-popover:popover2>click 激活</gs-button>
```

于是我让后端返回类似结构，每个名词引用的popover的ref值必须唯一。如下：
```html
    <ul>
        <li class="relPos" v-for="(question, index) in answerData.taskQaOptions">
          <div class="margin-left24" v-html="question.relTitle" />
        </li>
    </ul>
```
此时不仅仅无法弹出，还会报错popover的引用未定义

排查问题后发现，问题有两点：
1. popover不要以上述方式和v-for循环一起使用，否则会出问题
2. popover在页面上的实现。类似于你实现一个按钮的点击事件，仅仅把dom结构绑定上去是不可以的，需要重编译。

解决方式：
问题1:在for循环中popover组件的用法如下，此时不会再报popover引用找不到的问题
```html
    <gs-popover
      placement="top-start"
      width="200"
      trigger="click">
        财务指标是指什么呢哈哈哈哈
      <a slot="reference">财务指标</a>
    </gs-popover
```
问题2:问题1解决后，popover还是无法弹出的，需要我们编译一下,先写一个编译组件，如下

```js
import Vue from 'vue'

export default Vue.component('componentCompiler', {
  props: {
    template: {
      type: String,
      default:'<div></div>'
    }
  },
  data() {
    return {
      content: ''
    }
  },
  created() {
    var tpl =this.template
    this.$options.template = `<div>${tpl}</div>`
  }
})
```

然后我们按如下方式引用,popover就可以正常弹出了

ps:relTitle是后端返回的，此结构包含popover用法，须按照问题1的解决方式返回
```html
    <li class="relPos" v-for="(question, index) in answerData.taskQaOptions">
      <div class="margin-left24">
        <component-compile
          :template="question.relTitle"
        />
      </div>
    </li>
```

