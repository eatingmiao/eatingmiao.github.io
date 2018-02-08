title: 一个大坑
tags: [AngularJs,Ionic]
date: 2016-02-12
---
开始使用ng和ionic做项目也是赶鸭子上架，想不到真的踏进了一个大坑orz

刚开始的布局很乱，后来改了页面布局整理了一下view和content，然后发现在ion-content的$scope内，有些数据在改变布局前流动正常，但是改变后就不能双向流动了。

百思不得其解就查了一下，原来ionContent基于controller又创建了一个新的child scope，于是在ion-content对变量属性赋值时，新的值就自动绑定到了这个child scope上，怪不得找不到值啊！ 

于是只好把这种数据包在一个对象里了，通过原型链来访问到controller scope上的数据对象。
于是……要调用的变量名变得更长了orz

相关资料：
* [ngModel 值不更新/显示](http://www.cnblogs.com/whitewolf/p/ngmodel-zhi-bu-geng-xin-slash-xian-shi.html)

