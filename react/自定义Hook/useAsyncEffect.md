
useEffect支持异步函数
> useEffect 不支持异步 async 函数, 理由是 effect 函数应该返回一个销毁函数，如果 useEffect 第一个参数传入 async 函数，返回值则变成了 Promise，会导致react在调用销毁函数的时候报错

它返回的函數的執行時機如下：Ï
- 首次渲染不會進行清理，會在下一次渲染，清除上一次的副作用。
- 卸載階段也會執行清除操作。
不管是哪個，我們都不希望這個返回值是異步的，這樣我們無法預知代碼的執行情況，很容易出現難以定位的 Bug。所以 React 就直接限制了不能 useEffect 回調函數中不能支持 async...await...
### 方法：
#### 创建一个一步函数然后调用
``` typescript
useEffect(() => {
  const asyncFun = async () => {
    setPass(await mockCheck());
  };
  asyncFun();
}, []);
```
#### 使用 IIFE

``` typescript
useEffect(() => {
  (async () => {
    setPass(await mockCheck());
  })();
}, []);
```
#### 自定义hook（ahooks）
###  API
``` typescript
function useAsyncEffect(
  effect: () => AsyncGenerator | Promise,
  deps: DependencyList
);
```
### 源码
``` typescript
function isAsyncGenerator(
	val: AsyncGenerator<void, void, void> | Promise<void>,
): val is AsyncGenerator<void, void, void> {
	return isFunction(val[Symbol.asyncIterator]);
}
function useAsyncEffect(
	// 异步可迭代的对象
	effect: () => AsyncGenerator<void, void, void> | Promise<void>,
	deps?: DependencyList,
) {
	useEffect(() => {
		const e = effect();
		let cancelled = false;
		async function execute() {
			// 是迭代的函数
			if (isAsyncGenerator(e)) {
				while (true) {
					// 通过next不断执行得到
					const result = await e.next();
					// 迭代完成后或者取消时
					if (result.done || cancelled) {
						break;
					}
				}
			} else {
				// 非迭代的正常执行
				await e;
			}
		}
		execute();
		return () => {
			// 组件被销毁，修改变量 当前effect被销毁
			cancelled = true;
		};
	}, deps);
}
```
### 例子

``` typescript
useAsyncEffect(async () => {
    setPass(await mockCheck());
  }, []);
useAsyncEffect(async function* () {
	setPass(undefined);
    const result = await mockCheck(value);
    // 中断
    yield;
	setPass(result);
},[value]);
```