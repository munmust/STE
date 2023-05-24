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
- 表示那些在正常执行流程中永远不会返回的值的类型。它在类型系统中的作用是帮助开发者进行全面性的类型检查，确保代码的覆盖性和错误处理的完整性
- never 类型便为空类型和底部类型；never 类型的变量无法被赋值，与其他类型求交集为自身，求并集不参与运算
##### 应用
######  联合类型中的过滤
``` typescript
type Exclude<T, U> = T extends U ? never : T;
// 相当于: type A = 'a'
type A = Exclude<'x' | 'a', 'x' | 'y' | 'z'>
T | never // 结果为T
T & never // 结果为never

interface SomeProps {
    a: string
    b: number
    c: (e: MouseEvent) => void
    d: (e: TouchEvent) => void
}
// 如何得到 'c' | 'd' ？ 
type GetKeyByValueType<T, Condition> = {
    [K in keyof T]: T[K] extends Condition ? K : never
} [keyof T];

type FunctionPropNames = GetKeyByValueType<SomeProps, Function>; // 'c'|'d'

```
######  防御性编程
``` ty
```

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
