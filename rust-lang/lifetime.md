## 生命周期
```rust
fn main() {
  let mut a = &10;
  { 
    let b = &1;
    a = func(a, b);
  }
  println!("{}", a);
}
```
在上面这种情况下，rust不知道a的引用在使用println宏时是不是有效引用,如果func返回a那么就是有效的
如果func返回b那么就是不合法的，但是仅从这无法看出func返回引用的信息，使用生命周期标注协助
编译器
```rust
fn func<'a, 'b>(lhs : &'a i32, _rhs : &'b i32) -> &'a i32 {
  lhs
}
```
这样编译器分析func调用时能得知func能返回一个合法的引用，编译通过
