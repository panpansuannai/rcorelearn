## 记录有意思的用法

### 使用泛型为所有类型绑一个trait
```rust
use std::fmt::Display;

pub trait Greet {
    fn greet(self);
}
// 为所有实现Display trait的类型实现Greet 
impl<T: Display> Greet for T {
    fn greet(self) {
        println!("Hello, I'm {}", self);
    }
}

pub fn traits() {
    4.greet();
    (3 as u8).greet();
    "hidjfai".greet();
}
```

### 静态类型
```rust
// Add trait
// RHS = Self 表示若不加以标明，默认使RHS为Self类型
trait Add<RHS = Self> {
  type Output;
  fn add(self, rhs : RHS) -> Self::Output ;
}
```

### 自动派生的trait
- 比较 trait: Eq, PartialEq, Ord, PartialOrd
- Clone, 用来从 &T 创建副本 T。
- Copy，使类型具有 “复制语义”（copy semantics）而非 “移动语义”（move semantics）。
- Hash，从 &T 计算哈希值（hash）。
- Default, 创建数据类型的一个空实例。
- Debug，使用 {:?} formatter 来格式化一个值。
上述trait可以通过derive来自动派生

### trait 对象
[keyword dyn](https://doc.rust-lang.org/std/keyword.dyn.html)
dyn is a prefix of a trait object’s type.

The dyn keyword is used to highlight that calls to methods on the associated Trait are dynamically dispatched. To use the trait this way, it must be ‘object safe’.

Unlike generic parameters or impl Trait, the compiler does not know the concrete type that is being passed. That is, the type has been erased. As such, a dyn Trait reference contains two pointers. One pointer goes to the data (e.g., an instance of a struct). Another pointer goes to a map of method call names to function pointers (known as a virtual method table or vtable).

At run-time, when a method needs to be called on the dyn Trait, the vtable is consulted to get the function pointer and then that function pointer is called.

```rust
// 一般使用Box<dyn trait>传递一个泛型对象
// 或者&dyn trait 、&mut dyn trait等
trait Eat {
  fn eat(&self);
}
struct Me {}
struct You {}
impl Eat for Me {
  fn eat(&self){
    println!("Elegent!");
  }
}
impl Eat for You {
  fn eat(&self) {
    println!("Rude");
  }
}

fn getOnce(rude : bool) -> Box<dyn Eat> {
  if rude {
    Box::new(You{})
  }else {
    Box::new(Me{})
  }
}

fn main() {
  let person = getOnce(true);
  person.eat();
}
```
#### safe trait object
值得注意的是，当我们使用函数返回确定类型时可以使用impl trait替代，但是对于上面
返回多个trait对象的不可以使用,并且使用像Box<dyn Eat>这样的类型中，trait必须是object safe的
一个trait满足trait safe有如下规则
A trait can’t have methods that do any of the following:
- Take self by value as the receiver without a Sized.
- Include any associated functions (which don’t take self at all).
- Reference the Self type outside of the receiver position, except to access associated types for Self’s impl of the current trait or any supertraits.
- Include any generic parameters.

个人理解
- 当包含关联函数时，该关联函数不出现在虚函数表中，无法通过trait object调用
- 当函数返回Self时，无法判断Self的类型，不方便编译器进行类型分析
- 使用泛型时一个函数或结构对应多个实例的函数或类，与上一条相似
```rust
pub struct TraitObject {
    pub data: *mut (),
    pub vtable: *mut (),
}
```
如上是std::raw::TraitObject的内容，当函数接收到一个trait object时，编译器只有
该trait object的对象指针和虚函数表，而trait的定义中也不包含对象的类型信息，
如果trait 函数返回Self，编译器无法得知Self的实际类型,不能进行后续的分析. trait
中应该不能出现关联函数，因为关联函数不需要虚函数表.

#### impl trait
```rust 
fn getOnce(m : Box<Me>) -> impl Eat{
  *m
}
```

### 重叠trait消除
不同的trait中可能有重叠的方法，使用限定符调用特定trait的
函数
```rust
trait A {
  fn get(&self);
}
trait B {
  fn get(&self);
}
struct T {}
impl A for T {...}
impl B for T {...}
...
<T as A>::get();
<T as B>::get();
```
