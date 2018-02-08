title: 无限树节点
tags: [AngularJs]
date: 2016-12-17
---
之前有在已知树节点的情况下做过两级树菜单，如果是未知树节点的情况下，有可能是无限节点的延伸，这时可以用ng的inclue功能来实现，html结构大概如下：
#### HTML:	
	<ul class="list-unstyled">
		<li ng-repeat="organ in $ctrl.organsList" ng-click="$ctrl.selectOrgan(organ,'parent',$event)">
			{{organ.organName}}
			<ul ng-include="'organ-children.html'"></ul>
		</li>
	</ul>
	<script type="text/ng-template" id="organ-children.html">
	    <li ng-repeat="organ in organ.children" ng-click="$ctrl.selectOrgan(organ,'child',$event)">
	    	{{organ.organName}}
	    	<ul ng-if="organ.hadChildren=='C02'" ng-include="'organ-children.html'"></ul>
	    </li>
	</script>
在子页面children.html的列表li也包着子页面本身，于是就像迭代自己一样可以形成未知的多级菜单。
如果一开始不是一次性获得整棵树，点击父节点才能获取子节点的话，还得注意使用阻止冒泡的方法来阻止父节点的事件。
ng的$event事件里就带有阻止冒泡方法stopPropagation()，点击节点的方法如下：
#### JS:
	$ctrl.selectOrgan = function(item,name,$e) {
		$ctrl.selected = item;
		if (!!name) {
			if (name == 'child') {
				// 阻止冒泡
				$e.stopPropagation();
			}
			if (item.hadChildren == 'C02' && !item.hasOwnProperty('children')) {
				$ctrl.getData(item);
			}
		}
	};
