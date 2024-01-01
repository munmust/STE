- 接口抽象：类型行为的统一约束
- 泛型约束：泛型行为被trait限度在有限范围
- 抽象类型：运行时作为间接的抽象类型使用，动态分发给具体类型
- 标签trait：对类型的约束，直接作为标签使用
## 接口抽象
一组可以被共享的行为，只要实现了特征，你就能使用这组行为。让不同的类型实现同一种行为，也可以为类型添加新行为
对类型的抽象

- 接口可以定义方法
- 接口不能定义另一个接口
- 同一个接口可以被多个类型实现，但不能被同个类型实现多次
- 使用impl关键字为类型定义接口
- 使用trait定义接口


- rust当接口抽象的唯一方式
- 可以静态分发也可以动态分发

``` rust
struct Duck;
struct Pig;
trait Fly{
	fn fly(&self)->bool
}
impl Fly for Duck{
	fn fly(&self)->bool{
		true
	}
}
impl Fly for Pig{
	fn fly(&self)->bool{
		false
	}
}
fn fly_static<T:Fly>(s:T)->bool{
	s.fly()
}
fn fly_dyn(s:&dyn Fly)->bool{
	s.fly();
}
let duck=Duck;
let pig=Pig;
duck.fly();
pig.fly();
// 静态分发：编译器会生成特殊代码
fly_static::<Pig>(pig);
// 动态分发；运行时进行类型的查找
fly_dyn(&pig);
```
#### 关联类型
高内聚
关联类型增强了trait表示行为的的这种语意，表示了和某个trait相关的类型
``` rust
pub trait Add<RHS=Self>{
	type Output;
	fn add(self,rhs:RHS)->Self::Output
}

impl Add for $t{
	type Output=$t;
	fn add(self,other:$t)->$t{self + other}
}
impl Add<&str> for String{
	type Output = String;
	fn add(slef,other:&str)->String{
		self.push_str(other);
		self
	}
}
```
#### 一致性
> 孤儿规则：如果要实现某个trait，那么该trait和要实现该trait的那个类型至少要有一个存在于当前crate中定义

**如果你想要为类型** `A` **实现特征** `T`**，那么** `A` **或者** `T` **至少有一个是在当前作用域中定义的！**
``` rust
trait Add<RHS=Self>{
	type Output; // 关联类型必须指定具体的类型
	fn add(self,rhs:RHS)->Self::Output;
}
impl Add<u64> for u32{
	type Output = u64;
	fn add(self,other:u64)->Self::Output{
		(self as u64) + other
	}

}
```
#### 继承
子trait可以继承父trait
``` rust
trait A{
	fn setA(&self,other:i32){}
}
trait B{
	fn setB(&self,other:u32){}
}
trait C:A + B{
	fn setC(&self,other:String){}
}
impl <T:A+B> C for T{}
```
## 泛型约束
#### trait限定
鸭子类型
组合大于继承
trait可以看作一种类型，一种方法，或者一种行为的集合；
``` Rust
use std::ops::Add;
fn sum<T:Add<T,Output=T>>(a:T,b:T)->T{a+b}
fn gen<T:MyTrait + MyOtherTrait>(t:T)->T{} // 这个T类型必须实现MyTrait和myOtherTrait中定义的全部方法，才能使用该泛型函数
fn foo<T,K,R>(a:T,b:K,c:R) where T:A,K:B+C,R:D{} // where 关键字
```

## 抽象类型（存在类型）
#### trait对象
泛型中使用trait限定，可以将任意的类型范围根据类型的行为限定在更精确可控的范围内。也可以将共同拥有相同类型集合抽象为一个集合；

- trait中的Self类型参数不能被限定为Sized
- trait中所有的方法都必须上对象安全的
	- 方法受Self:Sized约束
	- 方法签名需要满足（没有额外的Self类型参数的非泛型成员方法）
		- 必须不包含任何泛型参数；当存在泛型时，trait对象在VTable中查找时将不确定该调用哪个方法
		- 第一个参数必须为Self类型或者可以解引用为Self的类型（需要有接收者，比如self，&self、&mut self，没有接收者的方法对trait对象来说没有意义）
		- Self不能出现在除第一个参数之外的地方，包括返回值。self出现在，意味着Self和self、&mut self的类型匹配。但对于trait对象来说，无法做到保证类型匹配
	- trait中不能包含关联常量

``` Rust 
pub trait Draw { fn draw(&self); }
pub struct Button {
    pub width: u32,
    pub height: u32,
    pub label: String,
}

impl Draw for Button {
    fn draw(&self) {
    }
}

struct SelectBox {
    width: u32,
    height: u32,
    options: Vec<String>,
}

impl Draw for SelectBox {
    fn draw(&self) {
    }
}
pub struct Screen { pub components: Vec<Draw>, }
```
**特征对象**指向实现了 `Draw` 特征的类型的实例，也就是指向了 `Button` 或者 `SelectBox` 的实例，这种映射关系是存储在一张表中，可以在运行时通过特征对象找到具体调用的类型方法
```rust 
trait Draw { fn draw(&self) -> String; } 
impl Draw for u8 { fn draw(&self) -> String { format!("u8: {}", *self) } } 
impl Draw for f64 { fn draw(&self) -> String { format!("f64: {}", *self) } } 
// 若 T 实现了 Draw 特征， 则调用该函数时传入的 Box<T> 可以被隐式转换成函数参数签名中的 Box<dyn Draw>
fn draw1(x: Box<dyn Draw>) {
// 由于实现了 Deref 特征，Box 智能指针会自动解引用为它所包裹的值，然后调用该值对应的类型上定义的 `draw` 方法 x.draw(); }
fn draw2(x: &dyn Draw) { x.draw(); }
fn main() { let x = 1.1f64; 
// do_something(&x);
let y = 8u8; 
// x 和 y 的类型 T 都实现了 `Draw` 特征，因为 Box<T> 可以在函数调用时隐式地被转换为特征对象 Box<dyn Draw> 
// 基于 x 的值创建一个 Box<f64> 类型的智能指针，指针指向的数据被放置在了堆上 
draw1(Box::new(x)); 
// 基于 y 的值创建一个 Box<u8> 类型的智能指针
draw1(Box::new(y));
draw2(&x);
draw2(&y);
}
```
- `draw1` 函数的参数是 `Box<dyn Draw>` 形式的特征对象，该特征对象是通过 `Box::new(x)` 的方式创建的
- `draw2` 函数的参数是 `&dyn Draw` 形式的特征对象，该特征对象是通过 `&x` 的方式创建的
- `dyn` 关键字只用在特征对象的类型声明上，在创建时无需使用 `dyn`

静态分发 `Box<T>` 和动态分发 `Box<dyn Trait>` 的区别
![[Pasted image 20231015161127.png]]
- **特征对象大小不固定**：这是因为，对于特征 `Draw`，类型 `Button` 可以实现特征 `Draw`，类型 `SelectBox` 也可以实现特征 `Draw`，因此特征没有固定大小
- **几乎总是使用特征对象的引用方式**，如 `&dyn Draw`、`Box<dyn Draw>`
    - 虽然特征对象没有固定大小，但它的引用类型的大小是固定的，它由两个指针组成（`ptr` 和 `vptr`），因此占用两个指针大小
    - 一个指针 `ptr` 指向实现了特征 `Draw` 的具体类型的实例，也就是当作特征 `Draw` 来用的类型的实例，比如类型 `Button` 的实例、类型 `SelectBox` 的实例
    - 另一个指针 `vptr` 指向一个虚表 `vtable`，`vtable` 中保存了类型 `Button` 或类型 `SelectBox` 的实例对于可以调用的实现于特征 `Draw` 的方法。当调用方法时，直接从 `vtable` 中找到方法并调用。之所以要使用一个 `vtable` 来保存各实例的方法，是因为实现了特征 `Draw` 的类型有多种，这些类型拥有的方法各不相同，当将这些类型的实例都当作特征 `Draw` 来使用时(此时，它们全都看作是特征 `Draw` 类型的实例)，有必要区分这些实例各自有哪些方法可调用

