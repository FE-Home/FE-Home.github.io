title: 移动端 app 部署流程
date: 2018/8/17
author: denghaobo
categories:

- hexo

---

## 部署步骤

### Android

- 进入 android 文件夹，并打包命令：cd android && ./gradlew assembleRelease(gradlew 是 android 的工程化工具, [常用的命令](https://blog.csdn.net/zhihui_520/article/details/53783347))
- 打包后的 apk 文件在\android\app\build\outputs\apk\文件夹下
- android 打包后的 apk 可以直接安装到手机上，比较简单

### IOS（必须用 mac）

- 首先需要有个开发者账号，开发者账号分三种（个人、公司、企业），公司内部现有两个账号：戴飞的企业版账号、勇坚的公司版账号
- 企业版账号： 不用发布到 appStore，属于为特定人群定制的 app，不需要在市场中推广，发布周期比较短，比较适合企业内部 app、b2b 业务等
- 公司版账号： 需要发布到 appStore，需要经过一周甚至更长时间的审核，周期比较长，比较适合需要在市场中推广，带有公司的品牌特征的 app

###### 本 app 选择了企业版账号，以下基于这个基础展开：

- 用开发者账号登陆[开发者管理员后台](https://developer.apple.com/account/)
- 进入 Certificates, IDs & Profiles，注册这三样东西，这是部署一个 ipa 包必备的三要素， [这里面详细解说了这三要素的原理/生成步骤](https://www.cnblogs.com/A--G/p/4627590.html)，比较长，耐心点看，很有帮助
- 证书：如果你认真看完了上面的内容后，你会发现戴飞的账号中发布证书已经被注册满了，也就是说你不能有自己的发布证书了，那么就要找发布证书的相关负责人（戴飞和我都注册过）去拿发布证书导出的.p12 文件，并且装到自己的 mac 本上；
- appID 和 profile: 没什么可说的，都比较容易创建
- **以上三要素都分为开发和发布两种，权限不一样，酌情选择**

* 接下来就是通过 XCode 将上面的三要素配置到自己的项目中，主要是配置 display name、bundle identifier、signing，[具体的配置方式](https://www.jianshu.com/p/c8e86f62687a)中的 xCode 配置章节
* 配置完成之后就可以打档案文件了 点击 Product 菜单中的 Archive 选项（当然在打包前应该先 build 一下看看是否有错误，这个应该不用多说），**要注意的是 destination 必须选择真机设备或者 Generic ios Deveice，否则 Archive 选项是灰色不能点击的**
* 如果以上步骤顺利的话，这时候打出来的包是 Archive 文件，还不是我们安装到手机上的 ipa 文件，要通过 Archive 文件生成 ipa 以及一些配置文件
* 进入 Window 菜单中的 Organizer，就会看到刚才生成的 Archives，页面的右侧点击 export，接下来依次操作选择 enterprise（开发版选择 development） -> 勾选上 include manifest for over-the-air installation（可以通过 safari 来下载 app）
  -> 配置 app 和图片的存放地址 -> 选择 automatically manage signing 然后就开始打包了
* 接下来打出来的包如何通过 safari 来安装？主要是通过 itms-services 协议，[步骤](https://www.cnblogs.com/xiaoc1314/p/5952555.html)

- 最后，如果你已经看懂了，并且整体部署一遍，你会发现，[过程其实就这些](https://www.jianshu.com/p/e7b302486186)
