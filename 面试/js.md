#### 闭包是什么？
有权访问另一个函数作用域的变量的函数；
内部函数和存在周围作用域的引用
一个普通的函数function，如果它可以访问外层作用域的自由变量，那么这个函数就是一个闭包；从广义上的角度来说，JavaScript中的函数都是闭包；从狭义的角度来说，JavaScript中一个函数，如果访问了外层作用域的变量，那么它是一个闭包

原因：**在JavaScript中，作用域对象是在堆中被创建的（至少表现出来的行为是这样的），所以在函数返回后它们也还是能够被访问到而不被销毁**作用域，使用变量时，如果当前作用域没有会向上父作用域进行查找；这时函数存在引用导致作用域没有销毁，

表现形式
`回调`：读取了
`返回匿名函数`：保存的仅仅是window和当前作用域
`IIFE`：保存了`全局作用域window`和`当前函数的作用域`

作用
`私有化内部变量`：例如和创建个funcation A 里面的定义一个name传出get和set方法，这时就存在闭包了，只能通过穿出的方法来控制A的的name
`延长变量生命周期`
`作为工厂模式`
`装饰器`：

问题：
`内存泄露`

#### 垃圾回收
对程序产生的不用的内存或者是之前用过了，以后不会再用的内存空间进行回收
当对象已经不存在引用时将会进行回收

js会定期的找出不在用到的内存进行清除
##### 标记清除法
分为标记和清除两个阶段
`标记`：运行的时候回给内存中所有的变量标记为 垃圾 ；然后会从各个根对象遍历，将非垃圾变量标记为 使用；
`清除`：将标记为垃圾的进行清除

问题：释放了之后，会导致内存不是连续的是碎片的，导致当需要一个大内存时存在问题；
##### 标记整理
释放之后会将使用中的内存向内存的一段进行移动，最后清楚内存边界
##### 引用计数法
当声明一个引用类型并赋值给变量时，这个值的引用次数初始为1，每有一个引用了它就会加一，变量被其他值覆盖之后就减一，当为0时说明没有引用可以进行回收；循环引用导致内存泄漏

##### 分代式内存回收
分为新生代老生代的内存
新、小、存活时间短的对象作为新生代，快速清理；提高回收效率
###### 新生代
新声代的大小比较小，
**分为使用区和空闲区**
`新加入的变量都会放到使用区里面,当使用区快写满时进行一次垃圾回收`：对使用区的活动对象进行标记，将使用区的标记对象复制到空闲区，空闲区中对需要清楚的对象进行回收；然后使用区和空闲区进行互换，空闲区变成使用区，使用区变成空闲区；以此往复；当一个对象在多次的垃圾回收中一一直存在，会复制到老生代中；如果复制一个对象到空闲区时，空闲区空间占用超过了 25%，那么这个对象会被直接晋升到老生代空间中；
`老生代`：标记整理
###### 增量标记
将一次GC分成很多的小步，每执行一部分就让给应用执行一会；交替多次完成一轮GC标记
*三色标记法*
三个颜色：当没有被检查引用是是白色的，当进行检查时会被标记为灰色，当发现被引用标记为黑色；当暂停之后，判断是否有灰色的标记又的话就继续从灰色部分开始标记；如果没有灰色了就直接开始清理；
当有黑色的标记对象引用了白色的对象，会将白色变为灰色
*惰性清理*
不会一次性全部清理，分多次清理


#### 事件循环原理
js引擎等待任务、执行任务、进入休眠状态切换的无限循环
同步任务、异步任务、宏任务、微任务；
主线程执行完执行栈中的所有任务后会去查看任务队列中是否还有任务
每来一个事件就会将这个任务进入执行栈中，如果为同步任务会直接进行执行，如果是异步任务会将其放到任务队列中，当主线程完成了没有同步任务后；只要异步任务有了运行结果，就在 **任务队列** 之中放置一个事件， 一旦 **执行栈** 中的所有同步任务执行完毕，系统就会读取 **任务队列**，看看里面有哪些待执行事件。那些对应的异步任务，于是结束等待状态，进入执行栈，开始执行
异步任务又分为宏任务和微任务；
同步任务->宏任务->微任务->宏任务

#### 什么是原型
每一个对象都与另一个对象相关联，那么关联的对象就是原型
函数或者对象都有一个prototype的属性指向一个对象，这个对象就是原型，原型中有个contructor函数指向这个对对象；当使用这个函数或者对象new 一个新实例的时候，新实例会有个proto的属性指向构造它的对象的原型proto->prototype；原型也是对象也有自己的原型对象，这样一层层的就构成了原形链

#### 如何实现继承
原型继承 ：直接替换prototype为新的函数；原型修改导致都修改了
构造器继承：使用call去执行，this指向自己；不能基层原型和方法

#### 执行上下文

#### 防抖节流
防抖： 一段时间内只执行一次，多次忽略
节流：一段时间后执行一次，每次有新的就从新计算
``` javascript
 const throttled(fn,delay){
	 let timer;
	 const sTime=Date.now();
	 return function(){
		 let curTime = Date.now();
		 let remaining = delay - (curTime - sTime);
		 const that=this;
		 args=arguments;
		 clearTimeout(timer)
		 if(remaining<=0){
			 dn.apply(that,args);
			 sTime=Date.now();
		 }else
		 {
		 timer=setTimeout(fn,delay)
		 }
	 }
 }
 function debounce(fn,delay,immdiate){
	 let timer=null;
	 return funciton(){
		 const that=this;
		 let args=arguments;
		 if(timer)clearTimeout(timer);
		 if(immdiate){
		  let isNow=!timer;
		  if(isNow){
			  fn.apply(that,args)
		  }
		  timer=setTimeout(function(){
			  timer=null
		  },delay)
		 }else{
			 timer=setTimeout(function(){
				 fn.apply(that,args)
			 },delay)
		 }
	 }
 }
```
#### instanceof
判断某个构造函数的prototype是否出现在某个实例对象上
``` javascript
function instanceof (a,b){
	if(typeof left !== 'object' || left === null) return false;
	let proto=Object.getPrototypeof(a);
	while(true){
		if(proto===null)return fasle
		if(a=== b.prototype) return true;
		proto=Object.getPrototypeOf(proto);
	}
}
```

typeof返回基本类型，instanceof返回布尔值
typeof对于引用类型判断不行
instanceof可以判断引用类型，但是不能判断基本类型

#### new
1，将构造函数通过原型和对象连接起来
2，将构造函数的this绑定到对象上
3，判断构造函数的返回值，为对象将会返回结果，否则返回对象
``` javascript
function news(Func,...args){
	const obj={};
	obj._prop_=Func.prototype;
	let result=Func.call(obj.args);
	return result instanceof Object ? result:obj;
}
```

#### 函数式编程
提高代码的无状态性和不变性，无副作用的函数 固定的输入，固定的输出
不会有超出作用域的变化，例如不会去修改全局的变量或者参数

#### 虚拟dom是什么? 原理? 优缺点?
react中虚拟DOM其实就是一个个Fiber节点，每个Fiber节点的stateNode又对应页面上真实的DOM节点；一个个的Fiber就构成一棵Fiber树，这个Fiber树就是页面根节点开始的整个页面的内容构成的；利用虚拟DOM在通过react的虚拟DOM的Diff算法以及双缓存机制，可以避免页面中频繁出现DOM的操作，提高响应
#### react 在虚拟dom的diff上，做了哪些改进使得速度很快?
diff主要是对同级的节点进行diff；只会进行同级diff不会跨级，只会直接删除然后建新的；
diff的判断分成两个部分第一个部分会先进行key的判断，如果key不同会认为是修改过的，之后对type、最后才是对对应的props、state、context等进行判断；这时候如果都相同会认为是没有改变的，可以进行服用，就会clone一个出现使用，不然不然都话会进入第二个部分，里面会便利不可复用的节点以及它后面的节点，会将老的fiber几点生成一个map，然后通过key开判断是有有相同的，有就说明是可服用；这样避免了移动的情况下生成新的消耗性能；
#### react 里的key的作用是什么? 为什么不能用Index？用了会怎样? 如果不加key会怎样?

发生改变时，会对新老的vDom进行diff判断；首先就会先使用key进行判断，如果key不相同会直接删除然后新建一个；如果相同判断其他是否相同来判断是否需要替换
如果没有key会直接进行创建了，不会走判断逻辑
例如删除，会导致key发生改变，会产生很多没有必要的更新



#### react中的状态管理

##### redux
redux主要就是传出一个store对象，包含subscribe、getState、applywiddle

subscribe就是一个订阅监听的方法

dispatch 会对传入的type进行reducer的调用，以此得到新的state；并且会去执行监听的方法

applywiddle使用的中间件，是createStore的第三个参数； applywiddle会创建一个store，然后去覆盖其中的dispatch方法，新的dispatch会从右到左去一次执行中间件数组中的中间件，上一个的返回值是下一个的参数

combineReducer 利用Map来保存各个reducer的方法

##### react-redux

 Prodiver 
 通过**react** **的** **context** **传递** **subscription** **和** **redux** **中的****store** **,并且建立了一个最顶部根** **Subscription 
 react-redux 利用的是react-context去实现的，将store放到value中进行透传，并且创建根的subscribe；
 

**以链表的形式收集对应的** **listeners** **(每一个****Subscription****) 的****handleChangeWrapper****函数，通过batch来进行一次批处理**

state更改 -> store.subscribe -> 触发 provider 的 Subscription 的 handleChangeWrapper 也就是 notifyNestedSubs -> 通知 listeners.notify() -> 通知每个被 connect 容器组件的更新 -> callback 执行 -> 触发子组件Subscription 的 handleChangeWrapper ->触发子 onstatechange

**1** **react-redux** **中的** **provider** **作用 ，通过** **react** **的** **context** **传递** **subscription** **和** **redux** **中的****store** **,并且建立了一个最顶部根** **Subscription** **。**  
  

**2** **Subscription** **的作用：起到发布订阅作用，一方面订阅** **connect** **包裹组件的更新函数，一方面通过** **store.subscribe** **统一派发更新。**  
  

**3** **Subscription** **如果存在这父级的情况，会把自身的更新函数，传递给父级** **Subscription** **来统一订阅。**

**connect的subscribereact 会先执行一次checkForUpdate，防止渲染后，store已经改变了；检查是否派发了当前组件的更新**

**react-redux** **通过** **subscription** **进行层层订阅**

![](https://cdn.nlark.com/yuque/0/2024/png/238125/1715243551741-fefb3763-c5a1-47e6-baaf-2b3301ce2f35.png)

checkForUpdates 通过调用 childPropsSelector来形成新的props,然后判断之前的 prop 和当前新的 prop 是否相等。如果相等，证明没有发生变化,无须更新当前组件，那么通过调用notifyNestedSubs来通知子代容器组件，检查是否需要更新。如果不相等证明订阅的store.state发生变化，那么立即执行触发组件的更新

![](https://cdn.nlark.com/yuque/0/2024/png/238125/1715244095864-0b7a73f7-a97a-400e-942d-a96a0cf7e17b.png)

  
  

被connect包裹，并且有第一个参数，通过context获取最近的subscribe，然后创建一个新的subscribe，并和父级的subscribe相关联，父的subscribe中也会添加一个订阅，



箭头函数和