# patten 在rust中用来匹配值的结构

## 用到patten的语法
- match分支
```
match VALUE {
  PATTEN => EXPRESSION ,
  PATTEN => EXPRESSION
}
```

- if let条件表达式
```
if let PATTEN = EXPRESSION {
    
} else if ... {

}
```

- while let, 与if let类似
- for 循环
```
let v = vec!['a', 'b', 'c'];

for (index, value) in v.iter().enumerate() {
    println!("{} is at index {}", value, index);
}
```

- let 语句
```
let (x, y, z) = (1, 2, 3);
```

- 函数参数
```
fn func(&(x, y) : &(i32, i32)) {

}
```

## 模式重要语法

- 使用| 匹配多个模式
```
let x = 10;
let y = match x {
  1 | 2 => 1,
  3 => 2,
  _ => 3,
};
```

- 使用..=匹配范围
```
let a =  10;
match a {
  1..10 => println!("[1, 10)"),
  10..=11 => println!("[10, 11]"),
  _ => println!("other"),
}

let c = 'a';
match c {
  'a'..='z' => println!("['a', 'z']"),
  _ => println!("other"),
}
```

### 解构
#### 解构结构体
```
struct Point {
    x: i32,
    y: i32,
}

fn main() {
    let p = Point { x: 0, y: 7 };

    let Point { x: a, y: b } = p;
    let ((feet, inches), Point {x, y}) = ((3, 10), Point { x: 3, y: -10 });
    assert_eq!(0, a);
    assert_eq!(7, b);
}

```

#### 解构枚举
```
enum Message {
    Quit,
    Move { x: i32, y: i32 },
    Write(String),
    ChangeColor(i32, i32, i32),
}

fn main() {
    let msg = Message::ChangeColor(0, 160, 255);

    match msg {
        Message::Quit => {
            println!("The Quit variant has no data to destructure.")
        }
        Message::Move { x, y } => {
            println!(
                "Move in the x direction {} and in the y direction {}",
                x,
                y
            );
        }
        Message::Write(text) => println!("Text message: {}", text),
        Message::ChangeColor(r, g, b) => {
            println!(
                "Change the color to red {}, green {}, and blue {}",
                r,
                g,
                b
            )
        }
    }
}
```
### 忽略值
- 使用_ 忽略值
```
let x = Some(10);
let y = match x {
  Some(val) => val,
  _ => 0,
};

match x {
  Some(_) => println!("See me"),
  None => println!("I'm None"),
}

fn func(_ : i32, y : i32) {

}
```
- 使用.. 忽略值
```
fn main() {
    let numbers = (2, 4, 8, 16, 32);

    match numbers {
        (first, .., last) => {
            println!("Some numbers: {}, {}", first, last);
        },
    }
}
```
### 匹配守卫
```
let num = Some(4);

match num {
    Some(x) if x < 5 => println!("less than five: {}", x),
    Some(x) => println!("{}", x),
    None => (),
}
```
### @绑定
```
enum Message {
    Hello { id: i32 },
}

let msg = Message::Hello { id: 5 };

match msg {
    Message::Hello { id: id_variable @ 3..=7 } => {
        println!("Found an id in range: {}", id_variable)
    },
    Message::Hello { id: 10..=12 } => {
        println!("Found an id in another range")
    },
    Message::Hello { id } => {
        println!("Found some other id: {}", id)
    },
}
```

## 使用Enum实现一个链表
```rust
enum List {
    Cons(i32, Box<List>),
    Nil,
}
impl List {
    fn new() -> List {
        List::Nil
    }

    /// Append an element to the tail of the List
    fn append(&mut self, val : i32) {
        let mut cur : &mut Self = self; 
        while let List::Cons(_, c) = cur {
            cur = c;
        }
        match cur {
            List::Cons(_, v ) => {
                *v = Box::new(List::Cons(val, Box::new(List::Nil) ));
            },
            _ => {
                *cur = List::Cons(val, Box::new(List::Nil) );
            }
        }
    }

    /// Get the len of the List
    fn len(&self) -> u32 {
        let mut len = 0;
        let mut head = self;
        while let List::Cons(_, n) = head{
            len += 1;
            if let List::Nil = **n {
                break;
            }
            head = n;
        }
        len
    }
}
```
