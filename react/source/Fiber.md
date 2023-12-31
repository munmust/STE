### 作用
- **工作单元 任务分解** ：Fiber最重要的功能就是作为工作单元，保存原生节点或者组件节点对应信息（包括优先级），这些节点通过指针的形似形成Fiber树
- **增量渲染**：通过jsx对象和current Fiber的对比，生成最小的差异补丁，应用到真实节点上
- **根据优先级暂停、继续、排列优先级**：Fiber节点上保存了优先级，能通过不同节点优先级的对比，达到任务的暂停、继续、排列优先级等能力，也为上层实现批量更新、Suspense提供了基础
- **保存状态：**因为Fiber能保存状态和更新的信息，所以就能实现函数组件的状态更新，也就是hooks
- ReactElement：react元素，createElement返回值
- React Component：开发者定义组件
- FiberNode：Fiber的节点
``` javascript
// React Component
const App = () =>{
	return <div>app</div>
}

// ReactElement
const app= <App />

// app对应FiberNode 
React.createRoot(rootNode).render(app)

```
## Fiber 节点
``` Typescript
function FiberNode(
	this: $FlowFixMe,
	//组件的类型，在后期协调阶段`beginWork`、`completeWork`的流程里根据不同的类型组件去做不同的`fiber`节点的处理
	tag: WorkTag,
	pendingProps: mixed,
	key: null | string,
	mode: TypeOfMode
) {
	// Instance
	// 组件类型 FUN CLASS
	this.tag = tag;
	// 组件props上的key
	this.key = key;
	// ReactElement.type 组件的dom类型， 比如`div, p`
	this.elementType = null;
	//func或者class
	this.type = null;
	// 在浏览器环境对应dom节点
	this.stateNode = null;
	// Fiber
	// 指向父节点
	this.return = null;
	// 指向子节点
	this.child = null;
	// 指向兄弟节点
	this.sibling = null;
	// 第几个
	this.index = 0;
	// ref
	this.ref = null;
	this.refCleanup = null;
	// 新的props 一般在渲染开始时，作为新的Props出现
	this.pendingProps = pendingProps;
	// 上一次渲染完成的props 会在整个渲染流程结尾部分被更新，存储FiberNode的props。
	this.memoizedProps = null;
	// 组件产生的update信息队列
	this.updateQueue = null;
	// 上一次渲染完成的state
	this.memoizedState = null;
	// 依赖项
	this.dependencies = null;
	this.mode = mode;
	// Effects
	// 需要执行的副作用，比如增删改查
	this.flags = NoFlags; 
	this.subtreeFlags = NoFlags; // 当前Fiber的子节点是否存在的flag（flag是需要变动）
	this.deletions = null; // 待删除的子节点，render阶段diff算法如果检测到Fiber的子节点应该被删除就会保存到这里
	// 优先级相关
	this.lanes = NoLanes; // 表示当前节点是否需要更新
	this.childLanes = NoLanes; // 表示当前节点的子节点是否需要更新
	//current和workInProgress的指针，树中节点相互关联的属性
	//可以用于判断是否需要更新还是创建，有值表示更新，反之则需要创建
	this.alternate = null;
	if (enableProfilerTimer) {
		this.actualDuration = Number.NaN;
		this.actualStartTime = Number.NaN;
		this.selfBaseDuration = Number.NaN;
		this.treeBaseDuration = Number.NaN;
		// It's okay to replace the initial doubles with smis after initialization.
		// This won't trigger the performance cliff mentioned above,
		// and it simplifies other profiler code (including DevTools).
		this.actualDuration = 0;
		this.actualStartTime = -1;
		this.selfBaseDuration = 0;
		this.treeBaseDuration = 0;
	}
}
```

### 副作用（flags）
我们通过某些手段去修改`props`、`state`等数据，数据修改完毕之后，但是同时引起了`dom不必要的变化`，那么这个变化就是`副作用`，当然这个副作用是`必然存在的`
副作用不仅仅只是一个，所以`React`中在`render`阶段中采用的是`深度遍历`的策略去找出当前 fiber 树中所有的副作用，并维护一个副作用链表`EffectList`，与链表相关的字段还有`firstEffect`、`nextEffect` 和 `lastEffect`
![[fiberEffect.webp]]

`fristEffect`指向第一个有副作用的`fiber`节点，`lastEffect`指向最后一个具有副作用的`fiber`节点，中间都是用`nextEffect`链接，这样组成了一个单向链表。render 阶段里面这一段处理就完了，在后面的`commit`阶段里面，`React`会根据`EffectList`里面`fiber`节点的副作用，会对应的处理相应的`DOM`，然后生成无副作用的`虚拟节点`，进行`真实dom`的创建
## Fiber链表树
`fiber`链表树里面有四个字段`return`、`child`、`sibling`、`index`  
- return:指向父节点，没有父节点则为 null。
- child:指向下一个子节点，没有下一个子节点则为 null。
- sibling:指向兄弟节点，没有下一个兄弟节点则为 null。
- index:父 fiber 下面的子 fiber 下标通过这些字段那么我们可以形成一个闭环链表

``` Typescript
function App() { 
	return (
		<div className="App">
			<header className="App-header">
				<img src='' className="App-logo" alt="logo" />
				<CountButton/>
			</header> 
		</div> 
	);
}
```
对于生成的Fiber树结果
![[fiberTree.jpg]]
### 双缓存
**内存中构建并直接替换**

屏幕上显示内容对应的`Fiber树`称为`current Fiber树`，正在内存中构建的`Fiber树`称为`workInProgress Fiber树`
`current Fiber树`中的`Fiber节点`被称为`current fiber`，`workInProgress Fiber树`中的`Fiber节点`被称为`workInProgress fiber`，他们通过`alternate`属性连接
`React`应用的根节点通过使`current`指针在不同`Fiber树`的`rootFiber`间切换来完成`current Fiber`树指向的切换
即当`workInProgress Fiber树`构建完成交给`Renderer`渲染在页面上后，应用根节点的`current`指针指向`workInProgress Fiber树`，此时`workInProgress Fiber树`就变为`current Fiber树`。
每次状态更新都会产生新的`workInProgress Fiber树`，通过`current`与`workInProgress`的替换，完成`DOM`更新

#### mount
整个应用的首次渲染，首次进入页面时
某个组件的首次渲染（隐藏变为可见）
1 创建fiberRootNode（只有应用首次渲染执行）
2 创建tag为3的FiberNode，代表HostRootFiber（某个组件的首次渲染无改操作）
3 从HostRootFiber开始，以DFS的顺序生成FiberNode
4 在遍历过程中，为fiberNode标记代表不同副作用的flags，以便后续在render中使用
1，ReactDom.createRoot 指向Current Fiber Tree的根节点，当前只有一个HostRootFiber（rootFiber）（首屏渲染时仅有一个根节点的空页面）
``` html
<body>
	<div id="root"></div>
</body
```
![[Pasted image 20231209135641.png]]
2，mount时，每个React元素根据 DFS的顺序，生成workInProcess FiberNode，并构成workInProgress Tree；workInProgress Tree会复用Current Fiber Tree的同级节点，并且由alternate连接；当workInProgress tree生成完毕后，workInProgress tree 会被传递给renderer，根据过程中标记的“副作用tag”执行对应的操作

![[Pasted image 20231209135741.png]]
3，当renderer工作完成之后，workInProgress Tree对应当UI已经渲染到了宿主环境，这时FiberRootNode。current指向workInProgress；此时workInProgress变成current Tree；完成了一次双缓存树的切换
![[Pasted image 20231209140830.png]]
> FiberRootNode 管理全应用全局：current fiber Tree和wip fiber tree的切换、应用任务过期时间、应用调度信息；全局上下文, 保存 fiber 构建过程中所依赖的全局状态；其大部分实例变量用来存储`fiber 构造循环`过程的各种状态.react 应用内部, 可以根据这些实例变量的值, 控制执行逻辑

> HostRootNode 为应在宿主环境挂载的根节点（rootElement）， react 应用中的第一个 Fiber 对象, 是 Fiber 树的根节点, 节点的类型是`HostRoot`

#### update
update时会生成一棵新的workInprocess树，最终完成后将fiberRootNode的current 指向workInprocess
![[Pasted image 20231209141106.png]]![[Pasted image 20231209141127.png]]



