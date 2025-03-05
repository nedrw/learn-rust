# 模式匹配
[toc]
## 模式匹配-基础

### 基本句法
```
Pattern :
      |? PatternNoTopAlt ( | PatternNoTopAlt )*

PatternNoTopAlt :
      PatternWithoutRange
   | RangePattern

PatternWithoutRange :
      LiteralPattern
   | IdentifierPattern
   | WildcardPattern
   | RestPattern
   | ReferencePattern
   | StructPattern
   | TupleStructPattern
   | TuplePattern
   | GroupedPattern
   | SlicePattern
   | PathPattern
   | MacroInvocation
```
参考文档:
- [rust reference](https://doc.rust-lang.org/reference/patterns.html)
- [rust圣经](https://course.rs/basic/match-pattern/intro.html)

### 匹配模式介绍
匹配模式分为两种: **范围模式**, **非范围模式**.

#### 范围模式
范围模式使用`..`和`..=`运算符来标识边界条件;在运算符左侧代表下界,右侧代表上界,`..`为右侧开区间,`..=`为右侧闭区间,一侧为空代表无限.

范围模式不止包括了整型(integer),浮点(float),还支持char,byte等类型.
```rust,edition2024
# #![allow(warnings)]
#fn main() {
    //integer
    let tar1 = 0;
    match tar1 {
        ..-1 => { println!("{}<-1", tar1); },
        -1..=2 => { println!("-1<={}<=2", tar1); },
        2..4 => { println!("2<={}<4", tar1); },
        4.. => { println!("4<={}", tar1); },
    }
    
    //float
    let tar2 = 0.0;
    match tar2 {
        ..-1.0 => { println!("{}<-1", tar2); },
        -1.0..=2.0 => { println!("-1<={}<=2", tar2); },
        2.0..4.0 => { println!("2<={}<4", tar2); },
        4.0.. => { println!("4<={}", tar2); },
        _ => { println!("nan"); },
    }
    
    //char,single Unicode character
    let tar3 = '卍';
    match tar3 {
        'A'..='f' => { println!("a<={}<=f", tar3); },
        'g'.. => { println!("g<={}", tar3); },
        _ => { println!("{}<a", tar3); }
    }
    //byte,just like char,but ASCII character
#}

```

#### 非范围模式
非范围模式包括:
- 字面量

    即0,1.1,'a',true等字面量
    ```rust,edition2024
    # #![allow(warnings)]
    #fn main() {
        let (x, y, z, u) = (1, 2.0, 'a', false); // 声明初始化中的类型匹配
        println!("{},{},{},{}",x,y,z,u);
    #}
    ```
- 标识符
    
    `ref? mut? IDENTIFIER (@ SubPattern) ?`
    
    即各种变量,允许添加**ref**,**mut**前缀以匹配引用和可变变量,可以后接`@`运算符匹配子模式
    ```rust,edition2024
    # #![allow(warnings)]
    #fn main() {
        let x = &2;
        match x {
            ref e @ 1 ..= 5 => println!("element {}", e),
            _ => println!("anything"),
        }
    #}
    ```
- 通配符

    即`_`,可以匹配任意条件
    ```rust,edition2024
    # #![allow(warnings)]
    #fn main() {
        let x = 1;
        match x {
            2 => { println!("match {}", x); },
            _ => { println!("match {}", x); },
        }
    #}
    ```
- rest模式

    用于 **切片**, **tuple**, **tuple struct**的匹配条件中,可以匹配零到多个变量.
    ```rust,edition2024,editable
    # #![allow(warnings)]
    #fn main() {
        let words = vec!["a", "b", "c", "d", "e"];
        let slice = &words[..];
        match slice {
            [] => println!("slice is empty"),
            [one] => println!("single element {}", one),
            // 以下三行可以轮流注释,运行,看看输出
            [head, tail @ ..] => println!("{} {:?}", head, tail),
            [start @ .., "e"] => println!("{:?}", start),
            [start, .., end] => println!("{} {}", start, end),
        }
    #}
    ```
- 引用
- 结构体
- 元组
- 切片

```rust,edition2024
# #![allow(warnings)]
#fn main() {
    
#}
```

### 使用方式
模式匹配的使用方式分为六种: **let**, **if let**, **while let**, **match**, **方法参数**, **for**.

同时你还可以使用`|`运算符来支持多个模式的匹配.

#### let
```rust,edition2024
# #![allow(warnings)]
#fn main() {
    // let PATTERN = VALUE;
    let x = 1; // yes, it's a match
    let (x, y) = (2, 3); // match
    let (x, 3|4) = (2, 3) else { panic!("let wrong match 1"); }; // match
    let (x, 4) = (2, 3) else { panic!("let wrong match 2"); }; // no match
#}
```

#### if let
**if let**通常用于option等枚举值的多个返回值的匹配,当匹配时会进入if代码段.
```rust,edition2024
# #![allow(warnings)]
fn option_some() -> Option<i32> { Some(1) }
fn option_none() -> Option<i32> { None }
fn main() {
    // if let PATTERN = VALUE {}
    if let Some(x) = option_some() { println!("match some {}", x); } // print 1
    if let Some(x) = option_none() { println!("match none {}", x); } // no print
}
```

#### while let
**while let**与**if let**类似,只要匹配成功就会不断执行while代码段.
```rust,edition2024
# #![allow(warnings)]
#fn main() {
    let list = vec![Some(1), Some(2), None, Some(3)];
    let mut i = 0;
    // while let PATTERN = VALUE {}
    while let Some(x) = list[i] {
        println!("{}", x);
        i += 1;
    } // 因为中间有个None,所以不会打印3
#}
```

#### match
**match**类似于switch语法,但是模式匹配使其功能更加强大.
```rust,edition2024
# #![allow(warnings)]
#fn main() {
    // match VALUE {
    //    PATTERN => {},
    //    _ => {},
    //}
    let x = 1;
    match (x, 3) {
        (2, 3) | (6, 3) => { println!("{}", x); },
        (3..=5, 3) => { println!("{}", x); }, // range pattern
        _ => { println!("{}", x); },
    }
#}
```

#### for
```rust,edition2024
# #![allow(warnings)]
#fn main() {
    let v = vec!['a', 'b', 'c'];
    // for PATTERN in Iterator
    for (index, value) in v.iter().enumerate() {
        println!("{} is at index {}", value, index);
    }
#}
```

#### 方法参数
```rust,edition2024
# #![allow(warnings)]
fn add((x, y):(i32, i32)) { println!("add {} & {}", x, y); }
fn main() {
    // function(PATTERN...)
    let a = (2, 3);
    add(a);
}
```
