# useToggle
两值切换
### 源码
``` typescript
export interface Actions<T> {
	setLeft: () => void;
	setRight: () => void;
	set: (value: T) => void;
	toggle: () => void;
}
// useToggle()
function useToggle<T = boolean>(): [boolean, Actions<T>];
// useToggle(x) x默认false
function useToggle<T>(defaultValue: T): [T, Actions<T>];
// useToggle(x,y)
function useToggle<T, U>(defaultValue: T, reverseValue: U): [T | U, Actions<T | U>];
function useToggle<D, R>(defaultValue: D = false as unknown as D, reverseValue?: R) {
	const [state, setState] = useState<D | R>(defaultValue);
	const actions = useMemo(() => {
		// 反值不存在的时候，取默认值的反值不然为 反值
		const reverseValueOrigin = (reverseValue === undefined ? !defaultValue : reverseValue) as D | R;
		// 判断当前的值是否为默认值，不是就取反值，反之亦然
		const toggle = () => setState((s) => (s === defaultValue ? reverseValueOrigin : defaultValue));
		// 直接设置
		const set = (value: D | R) => setState(value);
		// 设置为默认值
		const setLeft = () => setState(defaultValue);
		// 设置为反值
		const setRight = () => setState(reverseValueOrigin);
		return {
			toggle,
			set,
			setLeft,
			setRight,
		};
	}, []);
return [state, actions];
}
```

例子
``` typescript
const [state, { toggle, set, setLeft, setRight }] = useToggle('X', 'Y');
```

# useBoolean
布尔值的切换控制
``` typescript
export interface Actions {
	setTrue: () => void;
	setFalse: () => void;
	set: (value: boolean) => void;
	toggle: () => void;
}
export default function useBoolean(defaultValue = false): [boolean, Actions] {
	const [state, { toggle, set }] = useToggle(!!defaultValue);
	const actions: Actions = useMemo(() => {
		const setTrue = () => set(true);
		const setFalse = () => set(false);
		return {
			toggle,
			set: (v) => set(!!v),
			setTrue,
			setFalse,
		};
	}, []); 
	return [state, actions];
}
```