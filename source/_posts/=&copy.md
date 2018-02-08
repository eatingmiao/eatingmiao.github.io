title: “=”赋值和angular.copy()赋值的区别
tags: [AngularJs,Ionic]
date: 2016-09-10
---
刚开始在做以下这个案例时，
![choose-member](http://7xowup.com1.z0.glb.clouddn.com/=&copy.gif)
弹出插件的控制器传送的数据变量直接用了“=”来赋值，导致即使点击了插件的“取消”按钮，另一个控制器的数据也直接更新了，在此不需要即时更新，用户完成选择的时候才需要更新数据，这样的话用“=”就会形成一个即时更新的通道，所以应该用angular的copy来赋值才对。

还是以此场景为案例，详细来说说这两个赋值的区别：
内存里有一段地址储存了{name: "During"}这个数据，页面控制器的成员数据$scope.member指向了它，插件的控制器中的选择数据$scope.select也指向了它，当这两个控制器的数据通过angular的factory取出数据并使$scope.member = $scope.select的时候，事实上是让二者同时指向了该数据，因此一个变了另外一个也会跟着变。
如果用$scope.member = angular.copy($scope.select)，那就是先做了一份select数据的拷贝，也就是内存中多了另外一份数据，值相同但地址不同，然后让$scope.member指向了这份拷贝，所以二者指向了不同的内存地址，就不会相互影响了。
