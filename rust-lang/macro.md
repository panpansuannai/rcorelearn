## 杂项
- 在宏中可以使用()[]{}, 例如vec![1,2,3],vec!(1,2,3),vec!{1,2,3}是
  等价的,在声明宏的宏体内同理
``` rust
macro_rules! m {
  (how are you) => ( println!("fine"); );
  [how old are you] => { println!("18"); };
}
```

## macro\_rules
### 模式匹配
macro\_rules 内使用模式匹配,输入与规则匹配后将规则右边的结构在宏调用处展开为AST
```rust
macro_rules! four{
  () => { 1 + 3 };
}

let a = 1 + four!();
```

### 元变量
- block 块：比如用大括号包围起来的语句和/或表达式
- expr 表达式 (expression)
- ident 标识符 (identifier)：包括关键字 (keywords)
- item 条目：比如函数、结构体、模块、impl 块
- lifetime 生命周期注解：比如 'foo、'static
- literal 字面值：比如 "Hello World!"、3.14、'🦀'
- meta 元信息：指 #[...] 和 #![...] 属性内部的元信息条目
- pat 模式 (pattern)
- path 路径：比如 foo、::std::mem::replace、transmute::<_, int>
- stmt 语句 (statement)
- tt：单棵标记树 (single token tree)
- ty 类型
- vis 可视标识符：可能为空的可视标识符，比如 pub、pub(in crate)

```rust
macro_rules! multi {
  ($a:expr, $b:expr) => { $a * $b };
}
```

### 反复捕获
一般模式为$(...) sep rep,其中sep为分隔符,可以省略,rep为以下三种情况
- * 0次到多次
- + 1次到多次
- ? 0次或1次
在宏展开的地方同样使用上述模式
```rust
macro_rules! vec {
  ($($e:expr),*) => {
    let v = Vec::new();
    $(
      v.push($e);
    )*
    v
  };
}
```
如果规则内包含多个重复,调用时每个重复的次数需保持一致

### 回调
宏展开不能将一个宏展开的内容传递给另一个宏
```
macro1!(macro2()) // macro2不会展开
```
使用回调宏可以实现
```
macro_rules! callback {
    ($name:ident( $($args:tt),* ) ) => {
        $name!( $($args),* )
    };
}

macro_rules! m {
    ($($n:tt),*) => {println!("hello {} {}",$($n),*)};
}

fn main(){
    m1!(m2("first", "second"));
}
```
结果为
```
hello first second
```

### 内用规则
```
// 使用as_expr作为名称声明了一个内部宏,
// 该宏对外部不可见
macro_rules! foo {
    (@as_expr $e:expr) => {$e};

    ($($tts:tt)*) => {
        foo!(@as_expr $($tts)*)
    };
}

fn main(){
  assert_eq!(foo!(40+2), 42);
}
```
