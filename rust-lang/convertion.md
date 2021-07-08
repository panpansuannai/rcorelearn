## 基本类型之间通过as关键字来转换
 
## 自定义类型可以通过trait转换
```rust
struct A {
  // ...
}

struct B { // ...
}

impl From<B> for A {
  fn from(b : B) -> A {
    // ...
  }
}

fn main () {
  let b = B { ... };
  let a = A::from(b):
  let b = B { ... };
  let c : A = b.into();
}
```
对于可能失败的转换使用TryFrom/TryInto
```rust
impl TryFrom for A {
  type Error = ... ;
  fn from(b: B) -> Result<Self, Self::Error> {
    // ...
  }
}
```
