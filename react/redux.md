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
