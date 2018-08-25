title: 微信 JS-SDK 分享网页链接到朋友圈
date: 2018/8/25
categories:
- javascript
----------
1. 首先要有一个公众号
    * [微信公众平台](https://mp.weixin.qq.com/)

2. 获取 AppID 和 AppSecret
    * 登陆[微信公众平台](https://mp.weixin.qq.com/)

    * 开发 -> 基本配置，查看 AppID 和 AppSecret

3. 配置安全域名
    * 设置 -> 公众号设置 -> 功能设置，在 JS接口安全域名 设置安全域名

4. 码砖
    * index.html 中引入 JS 接口文件 http://res.wx.qq.com/open/js/jweixin-1.2.0.js

    * 获取access_token
        * https://api.weixin.qq.com/cgi-bin/token?grant_type=client_credential&appid=AppID&secret=AppSecret
        * 返回结果
        ```json
        {
        	access_token: AccessToken,
            expires_in: 7200
        }
        ```
        * AccessToken 有效时长为7200秒，即2小时

    * 根据 AccessToken 获取ticket
        * https://api.weixin.qq.com/cgi-bin/ticket/getticket?access_token=AccessToken&type=jsapi
        * 返回结果
        ```json
        {
            errcode: 0,
            errmsg: "ok",
            ticket: ticket,
            expires_in: 7200
        }
        ```

    * 获取签名signature
        * 定义时间戳 ` timestamp = +(new Date) `
        * 定义随机字符串 `noncestr = Math.random() * 10000`
        * 当前网页地址 `url = window.location.href.split('#')[0]`
        * 有效的jsapi_ticket `jsapi_ticket = ticket`
        * 获取签名signature
        ```json
          var shaStr = 'jsapi_ticket=ticket&noncestr=noncestr&timestamp=timestamp&url=url'
          var signature = sha1(shaStr)
        ```

    * 注入权限验证配置
    ```javascript
        wx.config({
          debug: true,
          appId: AppID,
          timestamp: timestamp,
          nonceStr: nonceStr,
          signature: signature,
          jsApiList: [ // 需要使用的接口列表
            ‘onMenuShareTimeline’， // 分享到朋友圈
            ‘onMenuShareAppMessage’， // 分享给朋友
            ‘onMenuShareQQ’ // 分享到QQ
          ]
        })
    ```
        * `debug: true` 调试进行debug，配置成功会自动提示 `errmsg: {config:ok}`

    * 通过ready接口处理成功验证
    ```javascript
    	wx.ready(function () {
        	wx.onMenuShareTimeline({
            	title: Title,
                link: url,
                imgUrl: imgUrl,
                success: function () {}
            })

            wx.onMenuShareAppMessage({
            	title: Title,
                link: url,
                desc: desc,
                imgUrl: imgUrl,
                success: function () {}
            })

            wx.onMenuShareQQ({
            	title: Title,
                link: url,
                desc: desc,
                imgUrl: imgUrl,
                success: function () {}
            })
        })
    ```

5. 参考
    * [微信JS-SDK说明文档](https://mp.weixin.qq.com/wiki?t=resource/res_main&id=mp1421141115)










