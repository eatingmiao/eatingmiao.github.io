title: RxJs订阅问题
tags: [AngularJs,Ionic,RxJs]
date: 2018-02-02
---

> RxJS是一种针对异步数据流编程工具，或者叫响应式扩展编程。  

RxJS的目标就是异步编程，Angular2引入RxJS为了就是让异步变得简单和可控。

### 取消订阅
RxJS的核心就是基于观察者模式开发的，关于观察者模式有个关键点要注意，订阅和取消订阅一定要成对出现，不取消容易造成重复订阅的bug，严重的话可能内存泄露。

以下是一个引入RxJs Subject的广播服务：

	export class Broadcaster {
	  private _eventBus: Subject<BroadcastEvent>;
	
	  constructor() {
	    this._eventBus = new Subject<BroadcastEvent>();
	  }
	
	  broadcast(key: any, data?: any) {
	    this._eventBus.next({key, data});
	  }
	
	  on<T>(key: any): Observable<T> {
	    return this._eventBus.asObservable()
	      .filter(event => event.key === key)
	      .map(event => <T>event.data);
	  }
	}

其中Subject既是一个Observable，也是一个Observer(被观察者和观察者):
  
- 更改数据时触发observer的next()进行传递；  
- filter()根据key值返回一个新的Observable，调用next()传递event给下一个observer；  
- map()根据event返回一个新的Observable，并调用next()传递event.data给下一个observer;  
- 最后将结果传递给页面的subscribe订阅块（订阅块用完要取消掉）。

这个服务有一个不够完善的地方就是没有订阅块的取消，直接交给了页面组件处理，如果不了解观察者模式的人恐怕就会忘记取消订阅而出现重复的问题。

angular@v2~4的EventEmitter也是参考RxJS的产物，若用EventEmitter再写一个类似广播的服务，可以带一个尾巴用于取消订阅，把订阅和取消订阅封装在一起成对出现：


	public on(callback: any, didLeave: any): any {
	    let subscriber = this.emitter.subscribe(callback);
	    didLeave.subscribe(() => {
	      this.unsubscribe(subscriber);
	    });
	    return subscriber;
	}

didLeave是ionView的页面离开Observable，并且这应该是一个SafeSubscriber，RxJs里SafeSubscriber在onCompleted和onError时会调用unsubscribe()。

关于angular1.x我也曾经遇到一个bug，就是监听路由stateChange时有重复累加的问题，跟没有取消订阅是一样的道理。

### 管理订阅

除了取消订阅这个操作，我们也可以使用takeUntil来管理subscription。

比如在页面组件离开时取消订阅的方式可以改成：

	public on(callback: any, didLeave: any): any {
	  return this.emitter
	    .takeUntil(didLeave)
	    .subscribe(callback);
	}

其他操作符：
  
- take(n): 在停止observable前发出N个值。  
- takeWhile(predicate): 通过断言来测试发出的值，如果一旦函数返回false，则完成observable。  
- first(): 发出首个值和完成通知。  
- first(predicate): 根据断言检查每个值，如果函数返回true，则发出值和完成通知。

当有多个订阅时，可以把订阅组合成单个subscription，通过创建一个父订阅把其他subscriptions作为子merge进来，这样的话只取消父订阅就可以了。

### 总结:
- 应该尽可能的使用takeUntil、takeWhile及其他操作符来管理RxJS subscriptions，而不是频繁地取消订阅。  
- 如果在一个组件中发现2个或以上的subscriptions，应该用组合形式来进行订阅和取消订阅的处理。  

对于跨时间的复杂异步任务，回调和Promise捉襟见肘，(俗称回调地狱)，RxJS可以比较优雅地组合和控制。

相关文档：

- [RxJS DOC](http://reactivex.io/rxjs/class/es6/Subject.js~Subject.html)
- [Specification for Observable](https://github.com/jhusain/observable-spec)