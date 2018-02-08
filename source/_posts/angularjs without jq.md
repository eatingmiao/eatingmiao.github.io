title: AngularJs without JQ
tags: [AngularJs,JQuery]
date: 2015-12-04
---
#### 引言
>当建立了Angular的项目时，你不应该添加完整的jQuery库。jQlite已经包含在Angular中，所有jQuery必要的功能它都有。这是因为把jQuery添加到Angular项目中将很难让你完全掌握Angular的核心优势和数据绑定的力量。
	
刚开始踏入angular的大坑时，因为项目比较赶，大家都还没掌握好ng的要点就上阵了，于是前期项目引用了完整的JQ库。在码控制层的时候，脑海中第一时间浮现通常都是各种$，现在功能做得差不多了，就开始项目优化，尝试把所有JQ都用ng的方式代替。
举个栗子：
比如做下面这种侧边菜单栏时，
![side-bar](http://7xowup.com1.z0.glb.clouddn.com/angular%20without%20jq.gif)
如果通过JQ来实现，就是对DOM进行addClass和removeClass更替操作，如果完全不用JQ呢？
其实完全可以利用ng设计的ng-class和ng-if(ng-show/ng-hide)绑定数据来进行样式变化：
#### HTML:	
	<ion-side-menu side="left" width="180" expose-aside-when="large">
		<ion-content class="menu-list">
			<ul>
				<li ng-repeat="parent in menuList" ng-click="setPActive(parent.id)">
					<a class="item item-icon-left item-icon-right" ng-class="{0:'',1:'parent-active'}[parent.isPActive]" ng-click="setPActive(parent.id)">
						<i class="icon {{parent.icon}}"></i>{{parent.name}}
						<i class="icon" ng-class="{0:'ion-ios-arrow-right',1:'ion-ios-arrow-down'}[parent.isOpen]"></i>
					</a>
					<ul ng-if="!!parent.child" ng-class="{0:'',1:'child-open'}[parent.isOpen]" class="child-list">
						<li ng-repeat="child in parent.child" ng-class="{0:'',1:'child-active'}[child.isCActive]" ng-click="setCActive(parent.id,child.id)">{{child.name}}</li>
				    </ul>
				</li>
			</ul>
		</ion-content>
	</ion-side-menu>
#### JS:
	$scope.menuList=[
		{id:1,name:'热点',icon:'ion-pinpoint'},
		{id:2,name:'xbms',icon:'ion-android-settings',child:[
			{id:1,name:'电力监控'},
			{id:2,name:'电梯监控'},
			{id:3,name:'电力监控'},
			{id:4,name:'照明监控'},
			{id:5,name:'排水监控'},
			{id:6,name:'水电计费'},
			{id:7,name:'空调系统'}
		]},
		{id:3,name:'车位状况',icon:'ion-android-car'},
		{id:4,name:'巡更系统',icon:'ion-android-star-outline'},
	];
	angular.forEach($scope.menuList,function(e){
		e.isOpen=0;
		e.isPActive=0;
		if(!!e.child) e.isCActive=0;
	});
	$scope.setPActive = function($id) {
		angular.forEach($scope.menuList,function(e){
			if(e.id===$id){
				if(!!e.child) {
					e.isOpen=1;
					e.isCActive=1;
				}
				e.isPActive=1;
			}else{
				e.isOpen=0;
				e.isPActive=0;
			}
		});	
	};
	$scope.setCActive = function($pid,$cid) {
		angular.forEach($scope.menuList,function(e1){
			if(e1.id===$pid){
				angular.forEach(e1.child,function(e2){
					if(e2.id===$cid){
						e2.isCActive=1;
					}else{
						e2.isCActive=0;
					}
				});	
			}
		});	
	};

其实说到底思维是差不多的，只是既然决定在Angular的框架下进行工作，还另外引入一个JQ库的确是显得累赘和多余，尝试用ng的方式来实现一样的效果也未免不可。

相关资料：
* [Think in AngularJs](http://www.angularjs.cn/A09X)

