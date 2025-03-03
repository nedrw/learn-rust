# 模式匹配
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
匹配模式分为两种: **范围模式**, **非范围模式**.其中非范围模式包含了多种子模式.

#### 非范围模式

#### 范围模式

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
