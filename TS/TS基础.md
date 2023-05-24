## TypeScript的定位
- JavaScript的超集
- 编译期行为
- 不引入额外开销
- 不改变运行时行为
- 始终与 ESMAScript 语言标准一致 (stage 3语法)

## TS语法
### 基础类型
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

### 操作
#### & 和 | 操作符
> \ 类型之间的并集（`|`）会向上取顶部的类型。即`never | 'a' => 'a'`，`unknown | 'a' => 'unknown'` 

> 类型之间的交集（`&`）会向下取底部的类型。即`never & 'a' = never`，`unknown & 'a' => 'a'`
