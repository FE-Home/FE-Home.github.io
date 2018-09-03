title: mpvue获取用户权限问题
date: 2018/8/17
categories:
- 微信小程序
---

本篇旨在整理小程序转mpVue遇到的一些基础事项

在用mpvue重构的时候遇到点击按钮从A页面跳到B页面时没有触发按钮的绑定事件，
但代码是正常的没有任何逻辑或语法错误；在webstorm里也并没有报错；但在开发
者估计里面却报了一下这样一个错误：Do not have XX handler in current page:
 RR. Please make sure that XX handler has been defined in RR, or RR has been 
 added into app.json；

明明已经在page（）里面注册好了，为什么还是 not have XX handler in current page，
仔细检查，路径添加了，事件也添加了，依然错误，无法找到错误原因为此我把按钮的点击
事件换成<navigatorurl="../mycase/main"></navigator>这样就不会有任何问题；但在实际函数里，我们
需要根据获取到权限来判断是否需要跳转；这样不符合我们的需求；

到网上查的时候几乎都提到了是跟Page()函数注册页面有关，Page()函数注册页面顺序会导致
后续页面注册中断，也就是app.json里面的路由填写顺序,一般二级界面就写在一级界面的下面,
千万别写在末尾，写在末尾就容易出现以上的问题；

但这个却不是最终问题所在，我调整了页面注册顺序但还是报错，因此我考虑可能是获取权限的
原因；鲍龙给我一个获取用户权限open-type="getUserInfo"的页面看看是否用；但用了无效，但
看了觉得乎就是获取用户权限open-type="getUserInfo"这里出了问题；然后沿着这方向查，查到
一些解决方法
<!--more-->
这是因为微信官方为了优化用户体验，不希望在用户不希望的情况下弹出授权框！为此会遇到一些
问题：button，open-type，监听事件方法 bindgetuserinfo="onGetUserInfo" 这个方法写到 method 里绑定不上
```html
 <button open-type="getUserInfo" @getuserinfo="bindGetUserInfo" @click="getUserInfo1">获取权限</button>
 ```
 ```javascript
 onload(){
    // 这个时候 不行，可能与生命周期有关系
    // this.getSetting()
  },
  mounted(){
    // 一进来看看用户是否授权过
    this.getSetting()
  },
  methods: {
    getSetting(){
      wx.getSetting({
        success: function(res){
          if (res.authSetting['scope.userInfo']) {
            wx.getUserInfo({
              success: function(res) {
                console.log(res.userInfo)
                //用户已经授权过
                console.log('用户已经授权过')
              }
            })
          }else{
            console.log('用户还未授权过')
          }
        }
      })

    },
    getUserInfo1(){
      console.log('click事件首先触发')
      // 判断小程序的API，回调，参数，组件等是否在当前版本可用。  为false 提醒用户升级微信版本
      // console.log(wx.canIUse('button.open-type.getUserInfo'))
      if(wx.canIUse('button.open-type.getUserInfo')){
        // 用户版本可用
      }else{
        console.log('请升级微信版本')
      }
    },
    bindGetUserInfo(e) {
      // console.log(e.mp.detail.rawData)
      if (e.mp.detail.rawData){
        //用户按了允许授权按钮
        console.log('用户按了允许授权按钮')
      } else {
        //用户按了拒绝按钮
        console.log('用户按了拒绝按钮')
      }
    }
```
2018-08-17   黄镇
