# Redux
## createStore
会先去判断reducer是否为纯函数；
如果有第三个参数，就是applyMiddleware的话会将store作为参数去执行第三个函数
创建一个当前监听的器的map对象
**getState**：返回当前的state
**subscribe**：订阅函数，往订阅器里添加订阅，返回取消订阅的方法
**Action**：一个对象，一定有type参数
**dispatch**：判断action是否为对象，判断action的type；通过执行reducer得到新的state；之后得到当前的监听列表进行依次执行，还会返回action对象
**replaceReducer**：替换老的reducer函数，然后执行dispatch方法，得到新的state，还会触发监听的触发
createStore的时候会先执行一次dispatch，保证reducer处理得到初始数据
## applyMiddleware
洋葱模型
接受一个createStore的传入，通过传入的reducer、初始state进行一个store的创建；从新定义一个dispatch；
每个中间件都传入getState和dispatch方法；
重写dispatch，利用compose，柯里化的从右向左的执行中间件，上个中间件的结果就是下个中间件的入参

``` javascript
applyMiddleware(thunkMiddleware)(createStore)(reducer, initialState)
```
##  bindActionCreator
主要作用就是，生成一个dispatch执行的方法，例如传递给子组件的时候，不希望子组件感知到store和disopatch

# React-Redux
## Provider
利用react的Context来实现一个Provider
创建一个subscribe，接受store以及上层的subscribe
创建一个context，包括监听，store等信息

在组件useLayoutEffect或者useEffect时，绑定监听的 onStateChange方法为通知监听；添加一个订阅队列；之后会判断当前的store和之前的store是否一致，是否需要触发通知；之后确认销毁时进行监听的清除
### subscribe
**trySubscribe**：订阅个数加一，如果不存在监听；判断是否上层的监听器，有的话往上层监听器上添加一个监听，如果没有就在当前的监听列表添加监听器
**addNestedSub**：添加一个监听，调用trySubscribe；返回取消函数
`Subscription` 的作用,首先通过 `trySubscribe` 发起订阅模式，如果存在这父级订阅者，就把自己更新函数`handleChangeWrapper`，传递给父级订阅者，然后父级由 `addNestedSub` 方法将此时的回调函数（更新函数）添加到当前的 `listeners` 中 。如果没有父级元素(`Provider`的情况)，则将此回调函数放在`store.subscribe`中，`handleChangeWrapper` 函数中`onStateChange`，就是 `Provider` 中 `Subscription` 的 `notifyNestedSubs` 方法，而 `notifyNestedSubs` 方法会通知`listens` 的 `notify` 方法来触发更新。子代`Subscription`会把更新自身`handleChangeWrapper`传递给`parentSub`，来统一通知`connect`组件更新

`state`更改 -> `store.subscribe` -> 触发 `provider` 的 `Subscription` 的 `handleChangeWrapper` 也就是 `notifyNestedSubs` -> 通知 `listeners.notify()` -> 通知每个被 `connect` 容器组件的更新 -> `callback` 执行 -> 触发子组件`Subscription` 的 handleChangeWrapper ->触发子 `onstatechange`
## connect

**mapStateToProps**：组件依赖`redux`的 `state`,映射到业务组件的 `props`中，`state`改变触发,业务组件`props`改变，触发业务组件更新视图
**mapDispatchToProps**：将 `redux` 中的`dispatch` 方法，映射到，业务组件的`props`中
**mergeProps**：自定义的合并规则
* stateProps , state 映射到 props 中的内容
* dispatchProps， dispatch 映射到 props 中的内容。
* ownProps 组件本身的 props

返回一个**wrapWithConnect** 
新的**ConnectFunction**中，会先判断得到props，store等，然后在store上创建一个subscription订阅，和通知函数，然后得到一个新的contextValue ,把父级的 subscription 换成自己的 subscription
；会缓存部分变量方便下次修改的时候进行对比；如果存在会新去触发一次通知；
- 合并child的props
- 创建一个 `subscription` ,并且和上一层`provider`的`subscription`建立起关联，`this.parentSub = contextValue.subscription`。然后分离出 `subscription` 和 `notifyNestedSubs`，把父级的 `subscription` 换成自己的 `subscription`，用于通过 `Provider` 传递新的 `context` 》〉》〉》〉》〉`connect`自己控制自己的更新，并且多个上下级 `connect`不收到影响。所以一方面通过`useMemo`来限制**业务组件不必要的更新**,另一方面来通过`forceComponentUpdateDispatch`来更新 `HOC` 函数，产生`actualChildProps`,`actualChildProps` 改变 ,`useMemo`执行，触发组件渲染
- checkForUpdate会作为onstateChange；如果store中的state发生改变，那么在组件订阅了state后，相关联的state改变会触发当前最爱你的onStateChange，合并得到新的props；subscrition.try Subscribe把订阅函数onStatechange绑定给父级subscription，进行层层订阅；
> 在provider中的useEffect中会取出store最新值和老值进行对比，触发是否需要进行发布； 发布后通知子subscribe的change方法，进行执行
![[Pasted image 20240519153729.png]]
![[Pasted image 20240519154833.png]]

#### 整个订阅流程
context得到最近的subscription，然后创建一个subscribe和父级的subscription建立联系；初始化之后会先执行一次checkForUpdate的方法，通过trySubscrible和this.parentSub.addNextedSub加入到父级的subscription的listener中；
#### 更新流程
store改变会触发根的store.subscribe，触发发布；然后会执行checkForUpdate方法。checkForUpdate会判断该组件组合后的props是否发生改变，改变就会更新组件；之后去notifyNextsubs通知subscript的listenser检查更新，层层checkForUpdate下去，完成更新流程

# redux-Thunk
普通的通过传递store的dispatch可以实现；但是封装成方法后 它不是一个action creator，返回也不是action creator；异步和同步易搞混
可以让我们和普通action一样去定义
thunk会传入dispatch和getState支持读取store的值

# redux-saga
利用genrator的方式依次执行
![[Pasted image 20240519201924.png]]

- 通过sagaMiddleware来将所有的generator初始化为iterator并执行；
- 每个saga在yield都会返回一个Effect给runEffect处理，runEffect会根据effect的类型调用不同的函数处理