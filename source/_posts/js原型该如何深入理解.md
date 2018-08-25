title: js原型该如何深入理解
date: 2018/8/18

---


大家刚开始学习js的时候，会遇到一些问题，比如什么是原型，继承的方式，还有闭包又是什么，今天小编就为大家讲解一下我对原型的理解，如果觉得我讲的有问题可以提出来，大家一起探讨一下

1. js原型
2. js原型练习题
3. 原型总结
4. 原型链继承总结


<!--more-->
## <span id="github_ssh">(1) js原型</span>

*先来讲一下我对原型的理解吧，首先原型是什么，原型是一个对象？还是一个实例呢，其实原型是一个对象，相当于一个构造函数的实例，下面举个例子

function Person(name,age,number){
	this.name=name;
	this.age=age;
	this.number=number;
}//构造函数
//实例
var person1=new Person(likaizhu,24,1);
var person2=new Person(zhangshaoqi,22,2);

if(person1.constructor==Person
)//true

if(person2.constructor==Person
)//true
title: hexo 多人博客使用指南
date: 2018/4/4
categories:
- hexo
---
本篇指南意在指导本团队开发者使用由hexo搭建的多人博客

面向阅读对象：专业开发者

1. [github与ssh](#github_ssh)
2. [基本环境搭建](#base)
3. [如何写博客](#write)
4. [如何上传文章或博客系统修改](#upload)
5. [对博客系统进行修改(未完待续)](#modifiy)


<!--more-->
## <span id="github_ssh">(1)github与ssh</span>

* 先将ssh添加到github上避免每次上传文章时都要输入账号密码


## <span id="base">(2)基本环境搭建</span>

* git clone git@github.com:FE-Home/FE-Home.github.io.git localFileName (将源代码下载本地)

* git checkout -b hexo origin/hexo (将分支切换到hexo分支)

* cnpm i (下载js依赖包)



* hexo generate (生成静态文件)

* hexo server (运行本地服务，如果在http://localhost:4000/可以看到相应的网页，则说明基本环境搭建成功,如果不成功有可能是端口造成的可以换个端口使用如： hexo server -p 4090)

![base](hexo多人博客使用指南/base.png)
 需要注意的是是前两个步骤只需要在第一次写博客的时候需要执行的，而后面的步骤在每次写博客都要去执行

## <span id="write">(3)如何写博客</span>

* 写博客主要在source的_posts下面利用markdown写相应的文章，具体语法可以参考[markdown语法](https://www.jianshu.com/p/b03a8d7b1719)

* hexo写博客可以参照 [hexo文档](https://hexo.io/zh-cn/docs/),这里面主要有三个命令[hexo generate](https://hexo.io/zh-cn/docs/generating.html)生成命令，[hexo server](https://hexo.io/zh-cn/docs/server.html)服务命令，[hexo deploy](https://hexo.io/zh-cn/docs/deployment.html)部署命令


## <span id="upload">(4)如何上传文章或博客系统修改</span>

* 记得每次上传前使用git pull(先将更新从远端拉取)

* hexo deploy (将静态文件提交到master)

* git add .   git commit -m '修改' git push origin hexo(将源文件同步到hexo分支:这里有点冗余最终可能会做成脚本命令)

* 最终在https://FE-Home.github.io/ 看到你的文章

//
Person.prototype.constructor==Person
person1.constructor==Person
person1.__proto__==Person.prototype
Person.__proto__==Function.prototype
Object.__proto__==Function.prototype
Person.prototype.__proto__==Object.prototype
Object.prototype.__proto__==null


*理解一下
函数上有prototype属性
原型对象相当于构造函数的一个实例  Person.prototype相当于person1
构造函数上有prototype constructor需要在prototype上获取
实例上是可以获取到construcor属性的


*其中的一种继承方式---原型链继承
function Person(name){
	this.name=name;
};
Person.prototype={
	sayName:function(){
		return this.name
	}
};
var person1=new Person('likaizhu');
person1.sayName()

//
person1.__proto__===Person.prototype
Person.prototype.__proto__===Function.prototype



## <span id="base">(2)js原型练习题</span>

1.person1.__proto__ 是什么？
2.Person.__proto__ 是什么？
3.Person.prototype.__proto__ 是什么？
4.Object.__proto__ 是什么？
5.Object.prototype__proto__ 是什么？
答案：
第一题：
因为 person1.__proto__ === person1 的构造函数.prototype
因为 person1的构造函数 === Person
所以 person1.__proto__ === Person.prototype

第二题：
因为 Person.__proto__ === Person的构造函数.prototype
因为 Person的构造函数 === Function
所以 Person.__proto__ === Function.prototype

第三题：
Person.prototype 是一个普通对象，我们无需关注它有哪些属性，只要记住它是一个普通对象。
因为一个普通对象的构造函数 === Object
所以 Person.prototype.__proto__ === Object.prototype

第四题，参照第二题，因为 Person 和 Object 一样都是构造函数

第五题：
Object.prototype 对象也有proto属性，但它比较特殊，为 null 。因为 null 处于原型链的顶端，这个只能记住。
Object.prototype.__proto__ === null






## <span id="base">(3)原型总结</span>

构造函数.prototype.constructor===构造函数
实例.constructor===构造函数
构造函数.prototype===普通对象
构造函数.prototype.constructor==实例.constructor
实例.__proto__===构造函数.prototype



构造器.__proto__==Function.prototype
构造器.prototype.__proto__==Object.prototype
构造器.constructor==Function

## <span id="base">(4)原型链继承总结</span>

function Person(){};
Person.prototype={
	name:’likaizhu’
};
var person1=new Person();
person1.__proto__通过__proto__去继承Person.prototype上的方法和属性
person1这个实例继承了Person这个构造函数原型上的所有方法和属性
当穿件一个新的对象的时候，想去调用方法，找不到去构造函数.prototype(原型)上去查找，然后再去构造函数.prototype.__proto__上去查找。



function Person(name){
	this.name=name;
};
Person.prototype={
	sayName:function(){
		return name
	}
};//重写原型,Person是object,object.prototype是{}空对象
var p=new Person('likaizhu');
p.__proto__===p.constructor.prototype
false


function Person(name){
	this.name=name;
};
Person.prototype.getName=function(){
	return name
}//修改原型,原型是function
var p=new Person('likaizhu');
p.__proto__===p.constructor.prototype

true