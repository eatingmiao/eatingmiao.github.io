title: JavaScript事件和事件模型
tags: [JavaScript]
date: 2018-02-26
---

## 事件

事件是可以被控件识别的操作。在前端领域，事件指与浏览器或文档交互的瞬间，如点击按钮，填写表格等。在现代浏览器中内置大量的事件处理器，如Window事件、表单事件、鼠标事件、媒介事件等。
事件流描述的是事件发生的顺序。

### 事件流

事件流有两种:  
- 事件冒泡（Event Capturing）: 是一种从下往上的传播方式。事件最开始由最具体的元素(文档中嵌套层次最深的节点, 也就是DOM最低层的子节点), 然后逐渐向上传播到最不具体的那个节点，也就是DOM中最高层的父节点。  
- 事件捕获（Event Bubbling）: 与事件冒泡相反。事件的处理将从DOM层次的根开始，而不是从触发事件的目标元素开始，事件被从目标元素的所有祖先元素依次往下传递。在这个过程中，事件会被从文档根到事件目标元素之间各个继承派生的元素所捕获，如果事件监听器在被注册时设置了useCapture属性为true，那么它们可以被分派给这期间的任何元素以对事件做出处理；否则，事件会被接着传递给派生元素路径上的下一元素，直至目标元素。

![](https://ss3.bdstatic.com/70cFv8Sh_Q1YnxGkpoWK1HF6hhy/it/u=3071504079,3576946828&fm=27&gp=0.jpg)

### 事件对象
当一个事件被触发时，会创建一个事件对象(Event Object), 这个对象里面包含了与该事件相关的属性或者方法。该对象会作为第一个参数传递给监听函数。

DOM事件模型中的事件对象常用属性:  

- type：获取事件类型
- target：获取事件目标
- stopPropagation()：阻止事件冒泡
- preventDefault()：阻止事件默认行为

IE事件模型中的事件对象常用属性:

- type：获取事件类型
- srcElement：获取事件目标
- cancelBubble：阻止事件冒泡
- returnValue阻止事件默认行为


## 事件模型
### DOM0级事件模型
DOM0级事件模型又称为原始事件模型，是早期的事件模型，所有的浏览器都是支持的，即直接在DOM对象上注册事件：

HTML中直接绑定：

	<input id="btn" type="button" onclick="fun()">
	
JS中绑定DOM：

	var btn = document.getElementById('btn');
	btn.onclick = function(event) {
       alert(event.target);
    };
    
解除绑定事件：

	btn.onclick = null;

在该模型中，事件不会传播，没有事件流的概念。一个DOM对象只能注册一个同类型的函数，注册多个同类型的函数则会发生覆盖。

### IE事件模型
IE8及之前的版本有特定的IE事件模型，该模型有两个阶段:  
1. 目标阶段(target phase)：事件到达目标元素, 触发目标元素的监听函数。    
2. 冒泡阶段(bubbling phase)：事件从目标元素冒泡到document, 依次检查经过的节点是否绑定了事件监听函数，如果有则执行。

事件绑定：

	attachEvent(eventType, handler)

事件解除：

	detachEvent(eventType, handler)

### DOM2级事件模型
该模型是由W3C标准制定的，除了IE8及之前的版本不支持外，现代浏览器都支持。该模型中有三个阶段：  
1. 捕获阶段(capturing phase)：事件从document一直向下传播到目标元素, 依次检查经过的节点是否绑定了事件监听函数，如果有则执行。  
2. 目标阶段(target phase)：事件到达目标元素, 触发目标元素的监听函数。  
3. 冒泡阶段(bubbling phase)：事件从目标元素冒泡到document, 依次检查经过的节点是否绑定了事件监听函数，如果有则执行。

事件绑定：

	addEventListener(eventType, handler, useCapture)

事件解除：

	removeEventListener(eventType, handler, useCapture)

参数 useCapture: boolean 用于指定是否在捕获阶段进行处理，一般设置为false与IE浏览器保持一致。

如果将useCapture设为true，指定在捕获阶段进行处理的话：

HTML: 

	<div id="outer">
	    <div id="inner">inner</div>
	</div>
JS:

	var inner = document.getElementById('inner');
    var outer = document.getElementById('outer');
    inner.addEventListener('click', function(){
        alert('inner');
    }, true);
    outer.addEventListener('click', function(){
        alert('outer');
    }, true);
 
由于inner是嵌套在outer中的，所以首先捕获到outer事件，其次才捕获inner事件。那么结果就是outer首先执行，其次是inner执行。如果在冒泡阶段处理的话，则是先执行inner后执行outer。

一个dom对象可以注册多个相同类型的事件，不会发生事件的覆盖，会依次的执行各个事件函数。如果inner注册两个处理阶段的事件，一个是在捕获阶段，另一个是在冒泡阶段：

	var inner = document.getElementById('inner');
    var outer = document.getElementById('outer');
    inner.addEventListener('click', function(){
        alert('capture');
    }, true);
    inner.addEventListener('click', function(){
        alert('bubble');
    }, true);

首先弹出的是capture，其次是bubble，该运行结果与注册的顺序有关，先注册则先执行。

## 事件代理和委托
事件在冒泡过程中会上传到父节点，因此可以把子节点的监听函数定义在父节点上，由父节点的监听函数统一处理多个子元素的事件，这种方式称为事件代理或事件委托（Event delegation）。

在父节点注册事件监听函数:  
HTML:  

	<div id="outer">
	    <input type="button" value="1" id="btn1">
	    <input type="button" value="2" id="btn2">
	    <input type="button" value="3" id="btn3">
	</div>
	
JS:

	var outer = document.getElementById('outer');
	outer.addEventListener('click', function(event){
	    alert(event.target.id)
	});

## 观察者模式和发布/订阅模式
### 观察者模式
观察者模式：定义对象间的一种一对多的依赖关系，当一个对象的状态发生改变时，所有依赖于它的对象都将得到通知。
JS的事件模型就是一种观察者模式的体现，当对应的事件被触发时，监听该事件的所有监听函数都会被调用。

### 发布/订阅模式
发布/订阅模式（Publish/Subscribe）属于广义上的观察者模式，是最常用的一种观察者模式的实现，并且从解耦和重用角度来看，更优于典型的观察者模式。

### 两者区别
在观察者模式中，观察者需要直接订阅目标事件；在目标发出内容改变的事件后，直接接收事件并作出响应：

	╭-------------╮  Fire Event  ╭--------------╮
	│             │─────────────>│              │
	│   Subject   │              │   Observer   │
	│             │<─────────────│              │
	╰-------------╯  Subscribe   ╰--------------╯
	
在发布订阅模式中，发布者和订阅者之间多了一个发布通道；一方面从发布者接收事件，另一方面向订阅者发布事件；订阅者需要从事件通道订阅事件，以此避免发布者和订阅者之间产生依赖关系：

	╭-------------╮                 ╭---------------╮   Fire Event   ╭--------------╮
	│             │  Publish Event  │               │───────────────>│              │
	│  Publisher  │────────────────>│ Event Channel │                │  Subscriber  │
	│             │                 │               │<───────────────│              │
	╰-------------╯                 ╰---------------╯    Subscribe   ╰--------------╯


## 自定义事件
自定义事件充分发挥了JavaScript的“事件驱动模型”，也是所谓的观察者模式，可以把复杂的逻辑解耦。

在元素绑定自定义事件：

	btn.addEventListener("formSubmit", function(event){
	    console.info("Custom data is: ", event.detail);
	}, false);

这与绑定内置的表单事件相比并没有什么差异，但自定义事件可以通过js创建和触发事件：

	// 创建
	let myEvent = new CustomEvent("formSubmit", {
	    detail: {
	        value: "special"
	    }
	});

	// 触发
	btn.dispatchEvent(myEvent);

[CustomEvent](https://developer.mozilla.org/zh-CN/docs/Web/API/CustomEvent)还可以通过配置bubbles和cancelable属性来触发特制的事件。




   
   

