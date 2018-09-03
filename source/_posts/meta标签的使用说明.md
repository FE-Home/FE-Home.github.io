title: meta 标签的使用说明
date: 2018/8/31
author: jiayankai
categories:
- html
---------------------------------
### meta 标签的使用说明

1. 什么是meta标签

    * meta是位于head中的一个标签，用于提供页面相关的一些原信息，比如针对搜索引擎的描述和关键词

    * meta标签在head内可以存在多个

2. mate标签中的属性

    * meta常用的属性有四个：charset、http-equiv、name、content

3. charset

    * 页面默认字符编码，常用 `<meta charset="utf-8">`

    * GB2312时，代表说明网站是采用的编码是简体中文；

    * BIG5时，代表说明网站是采用的编码是繁体中文；

    * iso-2022-jp时，代表说明网站是采用的编码是日文；

    * ks_c_5601时，代表说明网站是采用的编码是韩文；

    * ISO-8859-1时，代表说明网站是采用的编码是英文；

    * UTF-8时，代表世界通用的语言编码；

4. http-equiv

    * 相当于http的文件头信息，向浏览器回传一些有用的信息，属性格式为 `<meta http-equiv="参数" content="参数变量值">`

    * 可以设定页面的过期时间，一旦过期，必须到服务器上重新传输 `<meta http-equiv="expires" content="wed, 30 Sep 2018 23:59:59 GMT">`

    * 禁止缓存页面 `<meta http-equiv="pragma" content="no-cache">`

    * 自动刷新并跳转至新页面 `<meta http-equiv="refresh" content="2:url=https://baidu.com/">`

    * 优先使用的IE版本和Chrome版本 `<meta http-equiv="X-UA-Compatible" content="IE=edge, chrome=1">`

5. name

    * 和 http-equiv 类似，同样是和content配合，以类似键值对的形式出现

    * description：顾名思义，描述。对页面的简单说明。

    * keywords：关键词。对页面描述的一组关键字，是搜索引擎搜索的重要内容，但是由于对其的滥用，现在已经不再作为搜索引擎的重要标签

    * author：页面作者

    * viewport: 为移动设备添加视图设置

        * `meta name="viewport" content="initial-scale=1, maximum-scale=3, minimum-scale=1, user-scalable=no">`

        * `initial-scale` 页面的初始缩放值

        * `maximum-scale` 允许用户的最大缩放值

        * `minimum-scale` 允许用户的最小缩放值

        * `user-scalable` 是否允许用户进行缩放，`yes` 允许；`no` 不允许

        * 在IOS的输入框中进行输入操作时，系统会自动放大到当前输入框，要取消这种交互，可以通过配置 `maximum-scale=1，minimum-scale`，或者 `user-scalable=no`

    * 以下内容未测试使用，暂做留存
    ```javascript
    <!-- uc强制竖屏 -->
    <meta name="screen-orientation" content="portrait">
    <!-- QQ强制竖屏 -->
    <meta name="x5-orientation" content="portrait">
    <!-- UC强制全屏 -->
    <meta name="full-screen" content="yes">
    <!-- QQ强制全屏 -->
    <meta name="x5-fullscreen" content="true">
    ```










