# The `Future` Trait

The `Future` trait is at the center of asynchronous programming in Rust.
A `Future` is an asynchronous computation that can produce a value
(although that value may be empty, e.g. `()`). A *simplified* version of
the future trait might look something like this:

`Future` เป็นศูนย์กลางของ `async`

```rust
{{#include ../../examples/02_02_future_trait/src/lib.rs:simple_future}}
```

Futures can be advanced by calling the `poll` function, which will drive the
future as far towards completion as possible. If the future completes, it
returns `Poll::Ready(result)`. If the future is not able to complete yet, it
returns `Poll::Pending` and arranges for the `wake()` function to be called
when the `Future` is ready to make more progress. When `wake()` is called, the
executor driving the `Future` will call `poll` again so that the `Future` can
make more progress. 
Futures จะถูกเรียกใช้โดย `poll` ถ้าฟังชันนั้น ๆ ทำงานงานเสร็จแล้ว จะส่งคืน `Poll::Ready(result)` แต่ถ้าไม่เสร็จจะได้  `Poll::Pending` และเมื่อ ฟังชันนั้น ๆ ทำงานงานเสร็จแล้วก็จะเรียก  `wake()` ให้มาเรียกการทำงานของ `poll`อีกที โดยมี Futures เป็น driver 😑


Without `wake()`, the executor would have no way of knowing when a particular
future could make progress, and would have to be constantly polling every
future. With `wake()`, the executor knows exactly which futures are ready to
be `poll`ed.

wake() เป็นตัวที่คอยติดตามความคืบหน้าของงานที่ทำอยู่ ถ้าไม่มีมันก็จะไม่รู้ถึงความคืบหน้า


For example, consider the case where we want to read from a socket that may
or may not have data available already. If there is data, we can read it
in and return `Poll::Ready(data)`, but if no data is ready, our future is
blocked and can no longer make progress. When no data is available, we
must register `wake` to be called when data becomes ready on the socket,
which will tell the executor that our future is ready to make progress.
A simple `SocketRead` future might look something like this:

เช่น อ่านข้อมูลจาก `socket ` ถ้ามีข้อมูลก็ได้รับ `Poll::Ready(data)` ถ้าไม่มีก็รอให้  `wake` มาเรียกก่อนค่อยอ่านข้อมูลจาก `socket ` อีกที
```rust,ignore
{{#include ../../examples/02_02_future_trait/src/lib.rs:socket_read}}
```

This model of `Future`s allows for composing together multiple asynchronous
operations without needing intermediate allocations. Running multiple futures
at once or chaining futures together can be implemented via allocation-free
state machines, like this:

โมเดลการทำงานนี้จะช่วยเชื่อมโยง asynchronous fn หลายๆตัว กับ io นั้นๆ ไม่ต้องมีเทรดหรือตัวจัดการที่คอยตรวจสอบผลการทำงานตลอดเวลา

```rust,ignore
{{#include ../../examples/02_02_future_trait/src/lib.rs:join}}
```

This shows how multiple futures can be run simultaneously without needing
separate allocations, allowing for more efficient asynchronous programs.
Similarly, multiple sequential futures can be run one after another, like this:

จะเห็นว่า ไม่ต้องมีเทรดหรือตัวจัดการที่คอยตรวจสอบผลการทำงานตลอดเวลา

```rust,ignore
{{#include ../../examples/02_02_future_trait/src/lib.rs:and_then}}
```

These examples show how the `Future` trait can be used to express asynchronous
control flow without requiring multiple allocated objects and deeply nested
callbacks. With the basic control-flow out of the way, let's talk about the
real `Future` trait and how it is different.


```rust,ignore
{{#include ../../examples/02_02_future_trait/src/lib.rs:real_future}}
```

The first change you'll notice is that our `self` type is no longer `&mut Self`,
but has changed to `Pin<&mut Self>`. We'll talk more about pinning in [a later
section][pinning], but for now know that it allows us to create futures that
are immovable. Immovable objects can store pointers between their fields,
e.g. `struct MyFut { a: i32, ptr_to_a: *const i32 }`. Pinning is necessary
to enable async/await.

Secondly, `wake: fn()` has changed to `&mut Context<'_>`. In `SimpleFuture`,
we used a call to a function pointer (`fn()`) to tell the future executor that
the future in question should be polled. However, since `fn()` is just a
function pointer, it can't store any data about *which* `Future` called `wake`.

In a real-world scenario, a complex application like a web server may have
thousands of different connections whose wakeups should all be
managed separately. The `Context` type solves this by providing access to
a value of type `Waker`, which can be used to wake up a specific task.

[pinning]: ../04_pinning/01_chapter.md
