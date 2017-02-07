#Vue核心之数据劫持

##前瞻
当前前端界空前繁荣，各种框架横空出世，包括各类mvvm框架横行霸道，比如Anglar,Regular,Vue,React等等，它们最大的优点就是可以实现数据绑定，再也不需要手动进行DOM操作了，它们实现的原理也基本上是脏检查或数据劫持。那么本文就以Vue框架出发，探索其中数据劫持的奥秘（本文所选取的相关代码源自于Vue v2.0.3版本的源码）。
##什么是数据劫持
首先我们应该搞清楚什么是数据劫持，说白了就是通过Object.defineProperty()来劫持对象属性的setter和getter操作，在数据变动时做你想要做的事情，举个栗子：

	
	var data = {
		name:'lhl'
	}

	Object.keys(data).forEach(function(key){
		Object.defineProperty(data,key,{
			enumerable:true,
			configurable:true,
			get:function(){
				console.log('get');
			},
			set:function(){
				console.log('监听到数据发生了变化');
			}
		})
	})；
	data.name //控制台会打印出 “get”
	data.name = 'hxx' //控制台会打印出 "监听到数据发生了变化"
	
上面的这个栗子可以看出，我们完全可以控制对象属性的设置和读取。在Vue中，作者在很多地方都非常巧妙的运用了defineProperty这个方法，具体用在哪里并且它又解决了哪些问题，下面做详细的介绍：

###监听对象属性的变化
这个应该是Vue非常重要的一块，其主要思想是observer每个对象的属性，添加到订阅器dep中，当数据发生变化的时候发出notice通知。
相关源代码如下：（作者采用的是ES6+flow写的，代码在src/core/observer/index.js模块里面）
	
	export function defineReactive (
	  obj: Object,
	  key: string,
	  val: any,
	  customSetter?: Function
	) {
	  const dep = new Dep()//创建订阅对象
	
	  const property = Object.getOwnPropertyDescriptor(obj, key)
	  if (property && property.configurable === false) {
	    return
	  }
	
	  // cater for pre-defined getter/setters
	  const getter = property && property.get
	  const setter = property && property.set
	
	  let childOb = observe(val)//创建一个观察者对象
	  Object.defineProperty(obj, key, {
	    enumerable: true,
	    configurable: true,
	    get: function reactiveGetter () {
	      const value = getter ? getter.call(obj) : val
	      //这里也是作者一个巧妙设计，在创建watcher实例的时候，通过调用对象的get方法往订阅器			dep上添加这个创建的watcher实例
	      if (Dep.target) {
	        dep.depend()
	        if (childOb) {
	          childOb.dep.depend()
	        }
	        if (Array.isArray(value)) {
	          dependArray(value)
	        }
	      }
	      return value
	    },
	    set: function reactiveSetter (newVal) {
	      const value = getter ? getter.call(obj) : val
	      if (newVal === value) {
	        return
	      }
	      if (process.env.NODE_ENV !== 'production' && customSetter) {
	        customSetter()
	      }
	      if (setter) {
	        setter.call(obj, newVal)
	      } else {
	        val = newVal
	      }
	      childOb = observe(newVal)//继续监听新的属性值
	      dep.notify()//这个是真正劫持的目的，要对订阅者发通知了
	    }
	  })
	}
	
以上是监听对象属性的变化，那么下面再看看如何监听数组的变化：
###监听数组的变化
	const arrayProto = Array.prototype
	export const arrayMethods = Object.create(arrayProto)
	
	;[
	  'push',
	  'pop',
	  'shift',
	  'unshift',
	  'splice',
	  'sort',
	  'reverse'
	]
	.forEach(function (method) {
	  // cache original method
	  const original = arrayProto[method]
	  def(arrayMethods, method, function mutator () {
	    // avoid leaking arguments:
	    // http://jsperf.com/closure-with-arguments
	    let i = arguments.length
	    const args = new Array(i)
	    while (i--) {
	      args[i] = arguments[i]
	    }
	    const result = original.apply(this, args)
	    const ob = this.__ob__
	    let inserted
	    switch (method) {
	      case 'push':
	        inserted = args
	        break
	      case 'unshift':
	        inserted = args
	        break
	      case 'splice':
	        inserted = args.slice(2)
	        break
	    }
	    if (inserted) ob.observeArray(inserted)
	    // notify change
	    ob.dep.notify()
	    return result
	  })
	})
	
	...
	/**
	 * Define a property.
	 */
	function def (obj, key, val, enumerable) {
	  Object.defineProperty(obj, key, {
	    value: val,
	    enumerable: !!enumerable,
	    writable: true,
	    configurable: true
	  });
	}
	
通过上面的代码可以看出Vue是通过修改了数组的几个操作的原型来实现的。

###实现对象属性代理
正常情况下我们是这样实例化一个Vue对象:
	
	var VM = new Vue({
		data:{
			name:'lhl'
		},
		el:'#id'
	})
	
按理说我们操作数据的时候应该是VM.data.name = ‘hxx’才对，但是作者觉得这样不够简洁，所以又通过代理的方式实现了VM.name = ‘hxx’的可能。
相关代码如下：

	function proxy (vm, key) {
	  if (!isReserved(key)) {
	    Object.defineProperty(vm, key, {
	      configurable: true,
	      enumerable: true,
	      get: function proxyGetter () {
	        return vm._data[key]
	      },
	      set: function proxySetter (val) {
	        vm._data[key] = val;
	      }
	    });
	  }
	}

表面上看起来我们是在操作VM.name，实际上还是通过Object.defineProperty()中的get和set方法劫持实现的。
##总结
Vue框架很好的利用了Object.defineProperty()这个方法来实现了数据的监听和修改，同时也达到了很好的模块间解耦，在日常开发用好这个方法说不定会达到令人意想不到的结果。

本篇文章是我对Vue的浅薄之悟，如有理解不足之处，还请大家批评指正，Thank you ～
