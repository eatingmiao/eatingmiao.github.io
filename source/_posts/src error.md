title: 图片的404处理
tags: [AngularJs,JQuery]
date: 2016-03-16
---
做网站时最常遇到的问题之一，先说说jq的做法。
在标签上绑定onerror事件：
#### HTML:
	<img src="exist.png" onerror="errorSrc(this) "/> 

再写个方法把加载失败的图片替换成404图片：
#### JQ:
	function errorSrc(img){ 
	    $(img).attr("src", "img/404.png");
	}
就大功告成了！但是要注意的是要把errorSrc放在html的head里提前加载，要不然网页载入时即使图片出错也可能还没读取到errorSrc处理函数。而且浏览器的控制窗其实还是会报get的错。

如果在angular的框架下就更方便了，angular自带一个ngSrc指令，我们可以利用这个指令，自己写个angular指令，如下：
#### ngJS:
	(function() {
	    'use strict';
	    //加载指令
	    angular.module('app')
	        .directive('errSrc', [
	            function () {
	                return {
	                    link: function(scope, element, attrs) {
	                          element.bind('error', function() {
	                            if (attrs.src != attrs.errSrc) {
	                                  attrs.$set('src', attrs.errSrc);
	                            }
	                          });
	                    }
	                }
	            }
	        ])
	}());
#### ngHTML:
	<img ng-src="" err-src="img/404.jpg"/>
于是，angular在每次编译网页的时候，就会把已经不存在的图片换成了指定的img/404.jpg，而且这么浏览器控制窗也没有任何错误警告（强迫症福音www）。