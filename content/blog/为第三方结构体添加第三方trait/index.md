+++
title = "Rust中为外部类型实现外部 trait"

date = 2024-07-11

[taxonomies]
tags = ["Rust"]
categories = ["Rust"]
+++

由于孤儿规则 (*orphan rule*) 的限制，在Rust中无法直接为外部类型实现外部trait。但是我们可以通过构造一个外部类型的wrapper来间接实现这个目的。

一个比较常见的使用情形是，外部类型并没有实现`Display` trait，而我们想为其实现。这里，我们以标准库中的`String`为例进行介绍。

```Rust
extern crate std;

use std::fmt;
use std::ops::{Deref, DerefMut};
use std::string::String;

// 这个wrapper本质上是一个类似于`Box`但是只针对`String`
// 的智能指针
pub struct StringWrapper(String);

impl From<&str> for StringWrapper {
    // 针对不同类型的不包含self输入的方法需要分别自行重新实现
    pub fn from(source: &str) -> Self {
        StringWrapper(String::from(source))
    }
}

impl fmt::Display for StringWrapper {
    // 实现`Display` trait
    pub fn fmt(&self, f: &mut fmt::Formatter<'_>) -> fmt::Result {
        write!(f, "I am a String: {}", self.0)
    }
}

// 分别利用`Deref`和`DerefMut`两个trait重载`StringWrapper`的
// 解引用操作。使得针对`&StringWrapper`调用的方法实际上指向了
// `&StringWrapper.0`也就是内部`String`。
impl Deref for StringWrapper {
    type Target = String;

    fn deref(&self) -> &Self::Target {
        &self.0
    }
}

impl DerefMut for StringWrapper {
    fn deref_mut(&mut self) -> &mut Self::Target {
        &mut self.0
    }
}

fn main() {
    let a = StringWrapper::from("123");
    println!("{}", a);
}
```
