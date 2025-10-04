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

    ` (&|&&) mut? PatternWithoutRange`
    与标识符模式类似,不过匹配的是变量引用,即借用匹配对象
    ```rust,edition2024
    # #![allow(warnings)]
    #fn main() {
        let int_reference = &3;
        // 下面的两个匹配是等价的
        let a = match *int_reference { 0 => "zero", _ => "some" };
        let b = match int_reference { &0 => "zero", _ => "some" };
    #}
    ```
- 结构体
    
    可以匹配**struct**,**tuple struct**,**enum**,**union**等结构.多用于解构结构,提取参数.同时内部结构支持子模式匹配.
    ```rust,edition2024,editable
    # #![allow(warnings)]
    #struct AStruct { x:i32, y:i32 }
    #struct ATupleStruct (i32, i32, i32);
    #enum AEnum { E1, E2, E3(i32,i32) }
    #fn main() {
        let a_s = AStruct { x:1, y:2 };
        match a_s {
            AStruct { x: 10, y: 20 } => println!("10, 20"),//匹配子模式
            AStruct { y: 10, x: 20 } => println!("20, 10"),//顺序无所谓
            AStruct { x: 10, .. } => println!("x is 10"),
            AStruct { x, y } => println!("x: {}, y: {}", x, y),
        }
        
        let a_ts = ATupleStruct(1,2,3);
        let ATupleStruct(x,y,z) = a_ts;
        println!("{},{},{}", x, y, z);
        
        let a_e = AEnum::E3(1,2);
        match a_e {
            AEnum::E1 => println!("E1"),
            AEnum::E2 => println!("E2"),
            AEnum::E3(1, ..) => println!("1,.."),
            AEnum::E3(..) => println!("E3"),
        }
    #}
    ```
- 元组

    **tuple**模式与**tuple struct**模式类似.支持内部子模式匹配.
    ```rust,edition2024,editable
    # #![allow(warnings)]
    #fn main() {
        let pair = (10, 1.0, "ten");
        let (a, b, c) = pair;
        match pair {
            (1, x, y) => println!("{},{}",x,y),
            (8, ..) => println!("(8,xx,xx)"),
            (x, 1.0, ..) => println!("(xx,1.0,xx)"),
            _ => println!("_"),
        }
    #}
    ```
- 切片

    ```rust,edition2024,editable
    # #![allow(warnings)]
    #fn main() {
        let arr = [3, 2, 1];
        match arr {
            [1, _, _] => println!("1,xx,xx"),
            [a, b, c] => println!("{},{},{}",a,b,c),
        }
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
    if let Some(x) = option_some() println!("match some {}", x), // print 1
    if let Some(x) = option_none() println!("match none {}", x), // no print
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
        (2, 3) | (6, 3) => println!("{}", x),
        (3..=5, 3) => println!("{}", x), // range pattern
        _ => println!("{}", x),
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
