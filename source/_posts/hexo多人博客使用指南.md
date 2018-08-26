title: 移动端开发布局和样式的小技巧
date: 2018/8/26

---

## 苹果手机一些默认样式 ##

做移动端的时候写的时候很开心，但是在调试苹果手机的时候总是会出现一些问题之前我就遇到过email，telephone这些格式会在苹果手机中有默认的样式，下划线等，为了美观，我们可以添加一行简单的代码便可以实现

```
<meta name="format-detection" content="telephone=no">
<meta name="format-detection" content="email=no">
```
哈哈！！这下可以让你的视图更加的美观了

---

关于苹果手机像是按钮之类的最好使用div去写，因为苹果手机一些低版本会有一些小的问题，比如按钮有圆角，这种情况在苹果手机里面是没办法实现的。

苹果ios8中存在一个问题 display flex 布局是部分失效的，后来查看了一下发现safari是使用webkit内核的,改成下面这样就好了

```
display: flex;
display: -webkit-flex;
justify-content: center;
-webkit-justify-content:center;
```

---

iphone6 和 iphone6 plus，之前曾经遇到过一个这样的问题，页面跳转选择的时候，iphone6 plus的按钮出现挪窜的情况，针对不同手机出现的问题，我们去浏览器上去调试没有办法解决，我给大家推荐一个好用的东西
假如你有一台mac那么你就可以使用这个工具了

第一步：打开苹果手机打开设置 > safari浏览器 > 高级 > web检查器
第二步：打开mac的偏好设置 > 高级 > 在菜单栏中显示'开发'菜单
第三步：连接手机和电脑选择信任，然后在手机上打开你需要调试的页面
第四步：在控制台上面就可以捕捉到问题了

## 全屏活动落地页面之调试##

做活动落地页面的时候，大家肯定遇到过图片失真的问题，或者图片过大导致加载缓慢的问题，这里有一个小技巧

1.关于图片的问题，ps上有一个很好用插件cutterman，可以使用你灵活的小手指把他下载下来然后在window下找到Extensions选择cutterman切图神器，可以随意的切下你想要图片还可以选择图片的质量和1倍 2倍 3倍图，省去了切图的烦恼，图片质量可以调控不用担心加载过慢了

2.关于落地页显示问题，这个方法在落地页或者一些官网的banner上都可以使用一般ui给的图你可以计算好比例然后看代码

```
width:100%;
height:0;
padding-bottom:30%;
background:url('图片地址') no-repeat;
background-size:100% 100%;
```

这个办法可以让图片依然保持比例不至于失真




