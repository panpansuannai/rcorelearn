## 闭包由三个trait实现
```rust
pub trait FnOnce<Args> {
    type Output;
    extern "rust-call" fn call_once(self, args: Args) -> Self::Output;
}

pub trait FnMut<Args>: FnOnce<Args> {
    extern "rust-call" fn call_mut(
        &mut self, 
        args: Args
    ) -> Self::Output;
}

pub trait Fn<Args>: FnMut<Args> {
    extern "rust-call" fn call(&self, args: Args) -> Self::Output;
}
```
闭包由其是否捕获变量、对捕获变量的操作最终实现为上述三个trait
- 不捕获变量 -> FnOnce
- 捕获变量，且修改 -> FnMut
- 捕获变量，未修改 -> Fn
