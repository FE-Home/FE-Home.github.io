title: 如何在地图的自定义弹窗使用Vue
date: 2018/8/18
categories:
- zzd
author: test
---
<blockquote class="blockquote-center">blah blah blah</blockquote>
## 背景
最近接触一个以地图为基础的项目，满心欢喜。
![alt](http://www.happyzzd.top/static/upload/20180815/upload_e3ebe5214ed3f2f99161aa4baab8804c.png)

但是发现被套路了。

因为项目是vue 项目所以需要对 [天地图](http://www.tianditu.gov.cn/)进行一次封装，（我也第一次听说天地图地图） 好了废话不多说。
<!--more-->
## 问题
由于地图自定弹窗只支持 string，没有办法使用vue 如

```
    var sContent =
                "<div style='margin:0px;'>" +
                "<div style='margin:10px 10px; '>" +
                "<img style='float:left;margin:0px 10px' src='http://lbs.tianditu.gov.cn/images/logo.png' width='101' height='49' title='天安门'/>" +
                "<div style='margin:10px 0px 10px 120px;'>电话 : (010)88187700 <br>地址：北京市顺义区机场东路国门商务区地理信息产业园2号楼天地图大厦" +
                "</div>" +
                "<input  id='keyWord' value='机场' type='text' style='border: 1px solid #cccccc;width: 180px;height: 20px;line-height: 20px;padding-left: 10px;display: block;float: left;'>" +
                "<input style='margin-left:195px;  width: 80px;height: 24px; text-align: center; background: #5596de;color: #FFF;border: none;display: block;cursor: pointer;' type='button' value='周边搜索'  onClick=\"localsearch.searchNearby(document.getElementById('keyWord').value,center,radius)\">" +
                "</div>" +
                "</div>";
```


在我看来十分不优雅，与vue 感觉完全不配

## 解决方法
使用Vue 的  compile方法 结合 ES6的模板字符串
#### compile
参数：
   `{string} template`

Vue.compile 返回了{render:Function,staticRenderFns:Array}，render 可直接应用于 Vue 的配置项 render。而render和staticRenderFns到底是是什么？（我们下回再说。）

用法：
```
在 render 函数中编译模板字符串。只在独立构建时有效

var res = Vue.compile('<div><span>{{ msg }}</span></div>')

new Vue({
  data: {
    msg: 'hello'
  },
  render: res.render,  /
  staticRenderFns: res.staticRenderFns
})
```



```
let html = Vue.compile(`
      <div style="position: relative; width:200px; text-align:center;">
      <gs-button type="text" @click="closeWindow" style="float: right;">x</gs-button>
      <div>姓名：{{person.XM}}</div>
      <div><img style="width:100px" :src="imgUrl"/></div>
      <div>出现时间:</div>
       <div v-for="(item1,index) in person.Location" :key="index">{{item1.OccDateTime}}</div>
      <gs-button type="text-primary" @click="openline()">查看路线</gs-button>
      <gs-button type="text" @click="closeline()">取消查看</gs-button>
      </div>`);
 let vNode = new Vue({
        data: {
          person: item,
          imgUrl: bioassay
        },
        methods: {
          closeWindow() {
            me.map.removeOverLay(me.customerWinInfo);
          },
          openline() {
            me.createLine(item);
          },
          closeline() {
            me.map.removeOverLay(me.line);
          }
        },
        render: html.render
      });
      let dom = document.createElement("div");
      let el = vNode.$mount(dom).$el; //  挂接到div中

      let config = {
        offset: new TPixel(0, 0),
        position: item.marker.getLngLat()
      };
      this.customerWinInfo = new TLabel(config);

      this.customerWinInfo.setTitle("");

      this.customerWinInfo.setLabel(el);

      this.customerWinInfo.getObject().style.zIndex = 10000;

      this.map.addOverLay(this.customerWinInfo);
      let obj = this.customerWinInfo.getObject();
      let width = parseInt(obj.offsetWidth);
      let height = parseInt(obj.offsetHeight);

      let pixel = new TPixel(width / -2, height / -2 - 40);

      this.customerWinInfo.setOffset(pixel);
```

# 总结

优雅了好多。开心~
