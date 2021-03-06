# Box

`Box<T>` :智能指针,指向堆上的数据并具有已知大小,数据在堆上，指针在栈上。没有性能开销，也没有很多额外的功能.它实现了`Deref` trait，它允许将`Box<T>`值视为引用。当`Box<T> `值超出范围时，由于`Drop` trait实现，该`box`指向的堆数据也会被清除。

* 如果您的类型的大小在编译时无法知道，并且您希望在需要精确大小的上下文中使用该类型的值
* 当您拥有大量数据并且想要转移所有权但确保在执行此操作时不会复制数据
* 当你想拥有一个值而你只关心它是一个实现特定trait而不是特定类型的类型

将在`Box与递归类型`部分中演示第一种情况，在第二种情况下，转移大量数据的所有权可能需要很长时间，因为数据在堆栈中被复制。为了提高这种情况下的性能，我们可以在一个盒子中将大量数据存储在堆上。然后，只有少量指针数据被复制到堆栈上，而它引用的数据则保留在堆上的一个位置。第三种情况称为`trait`对象

`Box <T>`有一些特殊功能，Rust目前不允许用户定义的类型。

* `Box <T>`的解引用运算符产生一个可以移动的位置。 这意味着`*`操作和析构`box <T>`都内置于语言中。
* 方法可以将`Box <Self>`作为接收者。
* 可以在与T相同的crate中实现`Box <T>`的trait，孤儿规则可以防止其他泛型类型。

```rust
fn main() {
    let x = 5;
    let y = &x;

    assert_eq!(5, x);
    assert_eq!(5, *y);
}
// 相等于
fn main() {
    let x = 5;
    let y = Box::new(x);

    assert_eq!(5, x);
    assert_eq!(5, *y);
}
```

通过实现`Deref` Trait来处理类似引用的类型

## Box与递归类型

在编译时，Rust需要知道类型占用了多少空间。在编译时无法知道其大小的一种类型是递归类型，其中值可以作为其自身的一部分具有相同类型的另一个值。因为这种值的嵌套理论上可以无限延续，所以Rust不知道递归类型的值需要多少空间。但是，`box`具有已知大小，因此通过在递归类型定义中插入`box`，您可以使用递归类型。

## 使用`Box<T>`以获取与已知大小递归类型

```rust
enum List {
    Cons(i32, Box<List>),
    Nil,
}

use List::{Cons, Nil};

fn main() {
    let list = Cons(1,
        Box::new(Cons(2,
            Box::new(Cons(3,
                Box::new(Nil))))));
}
```

Box只提供间接和堆分配; 它们没有任何其他特殊功能，比如我们将看到的其他智能指针类型。 它们也没有任何这些特殊功能所带来的性能开销，因此它们在诸如conslist之类的情况下非常有用，其中间接是我们需要的唯一功能。