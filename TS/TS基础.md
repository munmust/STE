## TypeScript的定位
- JavaScript的超集
- 编译期行为
- 不引入额外开销
- 不改变运行时行为
- 始终与 ESMAScript 语言标准一致 (stage 3语法)

## TS语法
### 类型

![typescript类型关系](/assest/img/typescript_type.png "typescript 类型关系")

> 类型之间的并集（`|`）会向上取顶部的类型。即`never | 'a' => 'a'`，`unknown | 'a' => 'unknown'` 

> 类型之间的交集（`&`）会向下取底部的类型。即`never & 'a' = never`，`unknown & 'a' => 'a'`

- 布尔：`boolean`
- 数字：`number`
- 字符串：`string`
- 数组：`number[]` / `Array<number>`
- 元组：`[number, string]`
- 枚举：`enum Color{ RED, GREEN, BLUE }`
- `any`
- `void`
- `null`、`undefined`
- `never`
- `object`

#### never
*是其他任意类型的子类型的类型被称为底部类型*
在 TypeScript 中，never 类型便为空类型和底部类型。never 类型的变量无法被赋值，与其他类型求交集为自身，求并集不参与运算
- 一个从来不会有返回值的函数

### 操作
#### & 和 | 操作符
`|`表示满足任意一个契约即可 
`&`表示必须同时满足多个契约

``` javascript
interface IA {
	a: string;
	b: number;
};
type TB = { 
	b: number; 
	c: number[]; 
};
type TC = IA | TB; // TC类型的变量的键只需包含ab或bc即可，当然也可以abc都有
type TD = IA & TB; // TD类型的变量的键必需包含abc
```
