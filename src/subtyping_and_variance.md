# 子类型化与型变
[toc]

子类型化和型变的作用是为了实现对生命周期的灵活使用并防止滥用。

参考文档:
- [rust reference](https://rustwiki.org/en/reference/subtyping.html)
- [rust死灵书](https://nomicon.purewhite.io/subtyping.html)

## 子类型化
一个类型**A**可以被当做另一种类型**B**使用时，即**A**是**B**的子类型。
例如，类型**Cat**可以被当成是类型**Animal**，**Cat**即是**Animal**的子类型，但反过来则不成立，因为**Animal**有可能是**Dog**，**Dog**不能被当成**Cat**。

## 型变
型变分为三种：逆变，协变，不变。它们描述的是两个不同类型集合之间的子类型映射关系。

假设有Animal、Dog两个类型，属于第一个集合。Dog是Animal的子类型。
第二个集合中包含：F\<Animal>、F\<Dog>两个类型，与第一个集合元素一一对应。

请问Dog是Animal的子类型，此时F\<Dog>与F\<Animal>的子类型关系是如何的呢？
此时可以分成三种情况：
* F\<Dog>是F\<Animal>的子类型：**F\<T>** 在T上是协变的
* F\<Animal>是F\<Dog>的子类型：**F\<T>** 在T上是逆变的
* 其他情况（F\<Animal>与F\<Dog>没有关系）：**F\<T>** 在T上是不变的

举例：
有个函数 **fn(T)->()**，T是U的子类型，此时函数 **fn(U)->()** 是 函数 **fn(T)->()** 的子类型，这时fn对于T是逆变的。

## 生命周期
生命周期也能子类型化和型变。
**’static** 是所有生命周期的子类型，即 **’static** 可以被认为是任意生命周期类型。
生命周期可以通过引用传递，因此对于一个 **&‘static \<T>** 类型，它是 **&’func \<T>** 的子类型。

## 型变总结
| Type | 'a | T |
|---|---|---|
&'a T|协变|	协变
&'a mut T|协变|	不变
*const T|	|协变
*mut T|	|不变
[T] and [T; n]|	|协变
fn() -> T|	|协变
fn(T) -> ()|	|逆变
std::cell::UnsafeCell\<T>|	|不变
std::marker::PhantomData\<T>|	|协变
dyn Trait<T> + 'a	|协变|不变
