title: 变量提升 & this的指向
tags: [JavaScript]
date: 2018-02-08
---
## 变量提升  
### 函数及变量的声明都将被提升到执行代码的最顶部  

函数声明：

	hello();
	
	function hello() {
	  console.log("Hello");
	}

变量声明：

	x = 5;
	console.log(x);
	var x;

### 变量初始化和函数表达式则不会被提升

变量初始化：

	var x = 5;
	console.log(x+y); // NaN
	var y = 7;
	
函数表达式：

	add(); // TypeError: add is not a function
	
	var ab = function add() {}; // 表达式
	new function add() {}; // new表达式
	(function add() {}); // 分组操作符只能包含表达式
	
* 块中的声明大部分都不会被现代浏览器提升，包括块中的隐式声明


## this的指向
> this与当前执行环境，而非声明环境有关。  

this的指向需要根据函数上下文来判断，一般指向函数执行时的当前对象，但也可以通过其它方式来改变它的指向。

### 作为单独的函数被调用

	var name = "global";
	function echoName() {
	  var name = "func"
	  console.log(this.name); // global
	}
	echoName();
	
可以理解为add()是window的一个属性，于是this指向全局变量。

### 作为构造函数被调用

	var name = "global";
	function echoName() {
	  this.name = "func";
	}
	
	var fn = new echoName();
	console.log(fn.name); // func
	
构造函数中的this一般指向构造出来的实例化对象，但也有另一种情况：

	// 构造函数1
	function echo1() {
	  this.name = "name1";
	  return { name: 1 }; // 返回一个对象
	}
	var echo1 = new echo1();
	console.log(echo1.name); // 1
	
	// 构造函数2
	function echo2() {
	  this.name = "name2";
	  return 2; // 返回一个非对象
	}
	var echo2 = new echo2();
	console.log(echo2.name); // name2
        
当构造函数的返回值是一个对象时，this指向返回对象，  
如果返回值不是对象时，this指向构造函数的实例对象。

### 作为对象的方法被调用

	var Obj = {
	  name: "obj",
	  say: function() {
	    console.log(this.name);
	  }
	}
	
	Obj.say(); // obj
	
say()作为对象Obj的方法被Obj调用，this指向的便是Obj对象。  
但如果使用表达式调用say方法的话：

	var Obj = {
	  name: "obj",
	  say: function() {
	  	 console.log(this);
	    console.log(this.name);
	  }
	}
	
	var echoName = Obj.say;
	echoName(); // 打印出window和空

echoName是一个全局变量，把Obj.say先赋值给echoName再调用的话，this会指向window。

### 使用call或者apply方法
call()和apply()方法从Function对象的原型继承而来，使用call()和apply()方法会改变this的指向，其函数内部的this可绑定到 call()和apply()方法指定的第一个对象上。

	var Obj = {
	  name: "obj",
	  say: function(name) {
	  	 this.name = name;
	  	 console.log(this);
	    console.log(this.name);
	  }
	}
	var Obj2 = {
  	  name: "obj2"
  	};
  	var echoName = Obj.say;
  	echoName.call(Obj2,2); // {name: 2} 2
  	console.log(Obj.name); // obj

### 使用bind方法
bind()和call、apply不同的是，bind是新创建一个函数，然后把当前上下文this绑定到bind()的参数对象中并返回该函数。bind后函数不会执行，只是返回一个改变了this指向的函数副本，而call和apply是直接执行函数。

	var Obj = {
	  name: "obj",
	  say: function() {
	    function _say() {
	      console.log(this.name);    
	    }
	    return _say.bind(Obj);
	  }()
	}
	Obj.say(); // 空
	
需要注意的是，函数表达式的声明不会被提升，所以在_say方法调用的时候，Obj还没有被定义，this指向的是全局变量window。  
声明Obj后再调用_say方法则能打印出obj，此时this指向Obj:

	var Obj = {name: "obj"};
	Obj.say = function() {
	  function _say() {
	    console.log(this.name);    
	  }
	  return _say.bind(Obj);
	}()
	Obj.say(); // obj

### 使用定时器setTimeout和setInterval
	var Obj = {
	  name: "time",
	  say: function() {
	    setTimeout(function() {
	    	console.log(this.name);
	    }, 0);
	  }
	}
	Obj.say(); // 空

> 超时调用的代码都是在全局作用域中执行的，因此函数中this的值在非严格模式下指向window对象，在严格模式下是undefined。 ——《JavaScript高级程序设计》

setTimeout和setInterval方法是挂在window对象下的。此时的this指向应该为全局变量window。

### ES6箭头函数
箭头函数有几个使用注意点:
* 函数体内的this对象，就是定义时所在的对象，而不是使用时所在的对象。

* 不可以当作构造函数，也就是说，不可以使用new命令，否则会抛出一个错误。

* 不可以使用arguments对象，该对象在函数体内不存在。如果要用，可以用Rest参数代替。

* 不可以使用yield命令，因此箭头函数不能用作Generator函数。

上面四点中，第一点尤其值得注意。this对象的指向是可变的，但是在箭头函数中，它是固定的。

定时器的示例中，若是把定时器的function改成箭头函数：

	var Obj = {
	  name: "time",
	  say: function() {
	    setTimeout(() => {
	    	console.log(this.name);
	    }, 0);
	  }
	}
	Obj.say(); // time
	
此时定时器的this指向Obj对象。
	
### 严格模式
* 在全局作用域中，this指向window对象。
* 在全局作用域中，作为单独的函数被调用时，this指向undefined。
* 在全局作用域中，作为对象的方法被调用时，this指向调用函数的对象实例。
* 在全局作用域中，作为构造函数被调用时，this指向构造函数创建的对象实例。

另外，严格模式创设了第三种作用域：eval作用域。  
正常模式下，eval语句的作用域，取决于它处于全局作用域，还是处于函数作用域。  
严格模式下，eval语句本身就是一个作用域，所生成的变量只能用于eval内部。

	"use strict";
	var x = 2;
	console.info(eval("var x = 5; x")); // 5
	console.info(x); // 2