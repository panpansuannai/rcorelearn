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
