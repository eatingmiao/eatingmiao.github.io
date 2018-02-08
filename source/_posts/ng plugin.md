title: Ionic HybridApp 定位与wifi插件
tags: [AngularJs,Ionic,Cordova]
date: 2016-07-31
---
ngCordova虽然提供了大部分HybridApp开发常用的插件，不过有些还是挺鸡肋的，比如定位插件$cordovaGeolocation，调试半天不是返回空就是连接超时，转而用HTML5的Geolocation调试才发现，原来用的是google的服务……毕竟在大天朝开发自带翻墙能力的app不太现实，于是转而求助github上的资源。

最后我选用的定位插件是：
* [Phonegap Baidu Location Plugin](https://github.com/DoubleSpout/phonegap_baidu_sdk_location)

wifi插件则是：
* [WifiWizard](https://github.com/makeroo/WifiWizard)

定位插件暂时只提供了安卓的接口，调用的是百度的服务，所以不必担心在天朝的使用问题，wifi插件则是兼容了ios和安卓，我暂时不考虑ios开发，先做好安卓的功能性开发。
两者都挂在了window下，在github下载后执行ionic plugin install，可以直接调用：

    //获取用户此时所在经纬度
	$scope.getCurrentPosition = function () {
		document.addEventListener("deviceready", function () {
	        var noop = function() {};
	        window.locationService.getCurrentPosition(function(pos) {
	            $scope.nowPoint = {
	            	lng: pos.coords.longitude,
	            	lat: pos.coords.latitude
	            };
	            window.locationService.stop(noop,noop);
	        }, function(e){
	            toastr.error(e.message);
	            window.locationService.stop(noop,noop);
	        });
	    }, false);
	};

	//获取wifi列表
	$scope.getWifiList = function() {
		document.addEventListener("deviceready", function() {
			window.WifiWizard.listNetworks(function(list) {
				$scope.wifiList = list;
			}, false);
		}, false);
	}
