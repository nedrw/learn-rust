# 如何理解Send和Sync的含义

> [This Send/Sync Secret Separates Professional From Amateur Rust Developers](https://blog.cuongle.dev/p/this-sendsync-secret-separates-professional-and-amateur)

## Send和Sync的简单介绍

Send和Sync是Rust中的两个trait

Send trait表示一个类型可以在线程间安全地传递。这意味着，如果一个类型实现了Send trait，那么它可以在任何线程中创建和使用，而不会出现数据竞争或数据不一致的问题。例如，String、Vec和Box等类型都实现了Send trait。

Sync trait表示一个类型可以在线程间安全地共享。这意味着，如果一个类型实现了Sync trait，那么它可以在多个线程中同时读取和修改，而不会出现数据竞争或数据不一致的问题。例如，&str、&[T]和&Box<T>等类型都实现了Sync trait。

## 为什么需要send和sync

因为Rust的并发模型是基于所有权和借用规则的，而Send和Sync trait则是保证这些规则在并发场景下仍然成立的关键。

## send与sync的语义如何做到并发安全

send的语义是转移所有权至另一个线程，天然能避免数据竞争

sync的语义是在多个线程中共享引用，需要一些线程间的协作避免数据竞争

## String的send，sync示例

``` rust
use std::thread;

fn main() {
    let data = String::from("Hello, world!");
    // data的所有权会被转移到新的thread中
    let handle = thread::spawn(move || {
        println!("{}", data);
    });
    handle.join().unwrap();
}
```

``` rust
use std::sync::Arc;
use std::thread;

fn main() {
    let data = Arc::new(String::from("Hello, world!"));
    let mut handles = Vec::new();
    for _ in 0..10 {
        let data = data.clone();
        // 多个线程共享data引用，需要协作避免竞争
        // 此处均为readonly，因此没有数据竞争
        let handle = thread::spawn(move || {
            println!("{}", data);
        });
        handles.push(handle);
    }
    for handle in handles {
        handle.join().unwrap();
    }
}
```

## 何时需要send与sync

一句正确的废话是，当需要使用对应的语义的时候去实现对应的trait
但什么时候需要对应的语义的理解边界其实依然模糊，不如反过来回答，即什么时候是!send与!sync的

### 对象何时是!send

* 当对象没有关联任何竞争数据时是!send的

反过来理解即：
* 转移所有权后完全独占（即无竞争数据）是send的
* 转移所有权后共享数据多线程安全时是send的

尽管在转移所有权之后，理论上当前对象与其他的线程再无瓜葛，但是如果对象本身关联了某些共享的数据则会出现数据安全问题

以Rc<T>为例，其内部维护了一个引用计数，当存在多个引用时，单个指针的所有权被转移，此时引用计数需要修改，但是因为缺乏原子语义，多个线程修改会产生数据竞争，因此需要是!send的
使用Arc<T>的原子语义可以避免数据竞争，因此是send的

#### 示例

```rust
use std::rc::Rc;

fn main() {
    let rc1 = Rc::new(42);
    let rc2 = rc1.clone(); // Both point to the same data

    // If we could send rc1 to another thread (we can't!)...
    thread::spawn(move || {
        drop(rc1); // Decrements reference count
    });

    // ...while rc2 exists on the original thread
    drop(rc2); // Also decrements the same reference count
}
```

``` rust
use std::sync::Arc;

fn main() {
    let arc1 = Arc::new(42);
    let arc2 = arc1.clone();

    // This works because Arc makes the shared state thread-safe
    thread::spawn(move || {
        drop(arc1); // Atomically decrements counter
    });

    drop(arc2); // Also atomically decrements counter
}
```

### 对象何时是!sync

* 当通过共享引用修改数据时多线程不安全，对象是!sync的

反过来理解即：
* 当共享引用不修改数据时是sync的
* 当共享引用修改数据多线程安全是sync的

#### 示例

Cell<T>对象即是常见的!sync的对象，因为其内部可变性，导致多个线程共享引用时，可以不经数据同步直接修改数据
> 数据同步操作指原子语义、锁之类的操作，即多线程安全

``` rust
fn main() {
    // This WILL NOT compile
    let cell = Cell::new(0);
    thread::scope(|scope| {
        // error[E0277]: `Cell<i32>` cannot be shared between threads safely
        scope.spawn(|| cell.set(1)); // Thread 1 writing
        scope.spawn(|| cell.set(2)); // Thread 2 writing simultaneously
    });
}
```

``` rust
fn main() {
    use std::sync::atomic::{AtomicU64, Ordering};
    use std::sync::Mutex;

    // AtomicU64: Uses hardware-level atomic operations
    let atomic = AtomicU64::new(0);
    thread::scope(|scope| {
        scope.spawn(|| atomic.store(1, Ordering::Relaxed)); // Safe
        scope.spawn(|| atomic.store(2, Ordering::Relaxed)); // Safe
    });

    // Mutex: Uses locking to serialize access
    let mutex = Mutex::new(0);
    thread::scope(|scope| {
        scope.spawn(|| *mutex.lock().unwrap() = 1); // Safe
        scope.spawn(|| *mutex.lock().unwrap() = 2); // Safe
    });
}
```

## 实践决策树

``` mermaid
---
title: send决策树
---
flowchart LR
    A[是否有共享数据] --no --> B[send];
    A --yes--> C[数据是否多线程安全];
    C --yes--> D[send];
    C --no --> E[!send];
```

``` mermaid
---
title: sync决策树
---
flowchart LR
    A[是否修改共享数据] --no --> B[sync];
    A --yes--> C[是否多线程安全];
    C --yes--> D[sync];
    C --no --> E[!sync];
```
