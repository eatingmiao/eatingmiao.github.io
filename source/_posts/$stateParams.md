title: $stateParams
tags: [AngularJs]
date: 2016-01-17
---
关于ng路由的各种使用情况，要是认真写起来就一匹布那么长了。虽然项目容积一大起来感觉路由就容易乱，但如果能灵活运用路由的各种机制还是蛮方便的。
比如说，跳转页面时，两个页面的controller之间传输数据，如果只是传个参数id什么的，就没必要调用另一个函数来进行传输啦，用route自带的机制来传id也是蛮方便的。
只要在页面的data-ui-sref跳转state中加上参数，然后在$stateProvider声明state的url时加上参数名，那么在controller中就可以调用$stateParams来获取需要的参数了。
#### HTML:	
	<button data-ui-sref="order.comment({orderId:detail.orderId,goodsId:detail.goodsId})">订单</button>
#### JS:
	.state('order.comment', {
		js:
		url: '/comment?orderId&goodsId',
		cache:false,
		views: {
			'content@': {
				templateUrl: 'module/mine/views/my-order/order-comment.html',
				controller: 'orderCommentController'
			}
		}
	})

后来在stackoverflow上面看到一个问题也是值得注意的：

What is the difference between: $routeParams and $stateParams？

有一个回答这么说道：
Both are from different router modules. You can use anyone in your application.
If you use ngRoute module, then you should use [$routeParams](https://docs.angularjs.org/api/ngRoute) . This is provided by Angular team. It has only one ng-view. you can not do nested views functionality.
If you use ui-router module, then you should use [$stateParams](ttps://github.com/angular-ui/ui-router). This is from contributed module. It has number of additional functionality compare than ngRoute . It supports nested view concepts. you can specify multiple ui-view

See more: http://www.amasik.com/angularjs-ngroute-vs-ui-router/

