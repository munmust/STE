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
	// 新的props
	this.pendingProps = pendingProps;
	// 上一次渲染完成的props
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
	this.lanes = NoLanes;
	this.childLanes = NoLanes;
	//current和workInProgress的指针
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




