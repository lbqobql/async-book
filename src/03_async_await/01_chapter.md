# `async`/`.await`

In [the first chapter], we took a brief look at `async`/`.await`.
This chapter will discuss `async`/`.await` in
greater detail, explaining how it works and how `async` code differs from
traditional Rust programs.

ในบทแรกเรากล่าวถึงภาพรวมของ async/.await ในบทนี้จะมาลงรายละเอียดมากขึ้น อธิบายวิธีการทำงาน async ที่แตกต่างกับ sync

`async`/`.await` are special pieces of Rust syntax that make it possible to
yield control of the current thread rather than blocking, allowing other
code to make progress while waiting on an operation to complete.

async/.await เป็นส่วนสำคัญ syntag ในการควบคุมการทำงานของเทรด จะอนุญาตให้โปรแกรมทำงานแบบขนาน ไม่ถูกบล็อก


There are two main ways to use `async`: `async fn` and `async` blocks.
Each returns a value that implements the `Future` trait:

 มีสองวิธีในการใช้คือ `async`: `async fn` และ `async` แต่ละอันจะรีเทรินค่าเป็น `Future`

```rust,edition2018,ignore
{{#include ../../examples/03_01_async_await/src/lib.rs:async_fn_and_block_examples}}
```

As we saw in the first chapter, `async` bodies and other futures are lazy:
they do nothing until they are run. The most common way to run a `Future`
is to `.await` it. When `.await` is called on a `Future`, it will attempt
to run it to completion. If the `Future` is blocked, it will yield control
of the current thread. When more progress can be made, the `Future` will be picked
up by the executor and will resume running, allowing the `.await` to resolve.

async จะไม่ทำงานอัตโนมัติ มันต้องถูกกระตุ้นด้วย `Future` ซึ่งมีคำสั่งคือ `.await` เมื่อ await ถูกเรียกมันจะพยามทำงานให้สำเร็จ ถ้า Future ถูกบล็อค เทรดปัจจุบันจะเข้ามาควบคุมมัน และเมื่อมีความเปลี่ยนแปลง Future จะถูกเรียกโดยระบบแล้วกลับมาทำงานอีกครั้ง 
    
    😵😵😵😵
## `async` Lifetimes

Unlike traditional functions, `async fn`s which take references or other
non-`'static` arguments return a `Future` which is bounded by the lifetime of
the arguments:

`async fn` มีอายุเท่ากับอาร์กิวเมนต์ที่เรียกใช้งาน

```rust,edition2018,ignore
{{#include ../../examples/03_01_async_await/src/lib.rs:lifetimes_expanded}}
```

This means that the future returned from an `async fn` must be `.await`ed
while its non-`'static` arguments are still valid. In the common
case of `.await`ing the future immediately after calling the function
(as in `foo(&x).await`) this is not an issue. However, if storing the future
or sending it over to another task or thread, this may be an issue.

One common workaround for turning an `async fn` with references-as-arguments
into a `'static` future is to bundle the arguments with the call to the
`async fn` inside an `async` block:

```rust,edition2018,ignore
{{#include ../../examples/03_01_async_await/src/lib.rs:static_future_with_borrow}}
```

By moving the argument into the `async` block, we extend its lifetime to match
that of the `Future` returned from the call to `good`.

## `async move`

`async` blocks and closures allow the `move` keyword, much like normal
closures. An `async move` block will take ownership of the variables it
references, allowing it to outlive the current scope, but giving up the ability
to share those variables with other code:

```rust,edition2018,ignore
{{#include ../../examples/03_01_async_await/src/lib.rs:async_move_examples}}
```

## `.await`ing on a Multithreaded Executor

Note that, when using a multithreaded `Future` executor, a `Future` may move
between threads, so any variables used in `async` bodies must be able to travel
between threads, as any `.await` can potentially result in a switch to a new
thread.

This means that it is not safe to use `Rc`, `&RefCell` or any other types
that don't implement the `Send` trait, including references to types that don't
implement the `Sync` trait.

(Caveat: it is possible to use these types as long as they aren't in scope
during a call to `.await`.)

Similarly, it isn't a good idea to hold a traditional non-futures-aware lock
across an `.await`, as it can cause the threadpool to lock up: one task could
take out a lock, `.await` and yield to the executor, allowing another task to
attempt to take the lock and cause a deadlock. To avoid this, use the `Mutex`
in `futures::lock` rather than the one from `std::sync`.

[the first chapter]: ../01_getting_started/04_async_await_primer.md
