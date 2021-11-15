# Why Async?

We all love how Rust empowers us to write fast, safe software.
But how does asynchronous programming fit into this vision?

Asynchronous programming, or async for short, is a _concurrent programming model_
supported by an increasing number of programming languages.
It lets you run a large number of concurrent
tasks on a small number of OS threads, while preserving much of the
look and feel of ordinary synchronous programming, through the
`async/await` syntax.

เราทุกคนชอบที่ Rust ให้อำนาจเราเขียนซอฟต์แวร์ที่รวดเร็วและปลอดภัย แต่การเขียนโปรแกรมแบบอะซิงโครนัส จะเหมาะสมกับข้อดีนี้อย่างไร

การเขียนโปรแกรมแบบอะซิงโครนัส (อะซิง)เป็นรูปแบบการเขียนโปรแกรมที่ทำงานพร้อมกัน ซึ่งปัจจุบันมีภาษาโปรแกรมรองรับแนวคิดนี้เพิ่มขึ้นจำนวนมาก มันช่วยให้โปรแกรมสามารถทำงานหลายๆอย่างพร้อมกันได้บนเธรด OS จำนวนน้อย ซึ่งทำได้ด้วยการเขียนโปรแกรมซิงโครนัสทั่วไปผ่าน tag [`async/await`]

## Async vs other concurrency models

Concurrent programming is less mature and "standardized" than
regular, sequential programming. As a result, we express concurrency
differently depending on which concurrent programming model
the language is supporting.
A brief overview of the most popular concurrency models can help
you understand how asynchronous programming fits within the broader
field of concurrent programming:

## async เทียบกับแบบจำลองการทำงานพร้อมกันอื่นๆ 
การเขียนโปรแกรมพร้อมกันนั้นมีความสมบูรณ์น้อยกว่าและ "ได้มาตรฐาน" กว่าการเขียนโปรแกรมแบบต่อเนื่องแบบปกติ เป็นผลให้เราแสดงการทำงานพร้อมกันแตกต่างกันไป ขึ้นอยู่กับรูปแบบการเขียนโปรแกรมที่เกิดขึ้นพร้อมกันที่ภาษาสนับสนุน 

- **OS threads** don't require any changes to the programming model,
  which makes it very easy to express concurrency. However, synchronizing
  between threads can be difficult, and the performance overhead is large.
  Thread pools can mitigate some of these costs, but not enough to support
  massive IO-bound workloads.
  
จำนวนเธรดบน cpu ไม่มีผลต่อโมเดลการเขียนโปรแกรมแบบ async อย่างไรก็ตาม การซิงโครไนซ์ระหว่างเธรดอาจทำได้ยาก และมีปัญหาคอขวดด้านประสิทธิภาพในบางกรณี ซึ่ง เธรดพูล จะเข้ามาช่วยได้บางส่วน แต่ก็ยังไม่เพียงพอที่จะรองรับปริมาณงานจำนวนมากที่ที่ผูกกับ IO

- **Event-driven programming**, in conjunction with _callbacks_, can be very
  performant, but tends to result in a verbose, "non-linear" control flow.
  Data flow and error propagation is often hard to follow.
  
- **Event-driven programming** _callbacks_ มีประสิทธิภาพมาก แต่ก็เป็นเรื่องยากที่จะติดตามการทำงานของโปรแกรมแบบ async ซึ่งจำทำให้เกิดปัญหาอื่นตามมา

- **Coroutines**, like threads, don't require changes to the programming model,
  which makes them easy to use. Like async, they can also support a large
  number of tasks. However, they abstract away low-level details that
  are important for systems programming and custom runtime implementors.

- **Coroutines**, คล้ายกับ **OS threads** อย่างไรก็ตาม รายละเอียดของการทำงานในระดับต่ำเป็นส่วนสำคัญ

- **The actor model** divides all concurrent computation into units called
  actors, which communicate through fallible message passing, much like
  in distributed systems. The actor model can be efficiently implemented, but it leaves
  many practical issues unanswered, such as flow control and retry logic.
- **The actor model** จะแบ่งการทำงานออกเป็นส่วยย่อย เรียกว่า actors มันมีประสิทธิภาพในกรณีที่มีการทำงานแยกส่วนกันอย่างชัดเจน ไม่เป็นงานที่ทับซ้อนแล้วใช้ทรัพยากรณ์ตัวเดียวกัน


In summary, asynchronous programming allows highly performant implementations
that are suitable for low-level languages like Rust, while providing
most of the ergonomic benefits of threads and coroutines.

โดยสรุป การเขียนโปรแกรมแบบอะซิงโครนัสช่วยให้โปรแกรมมีประสิทธิภาพสูงขึ้นมาก 😂😂😂


## Async in Rust vs other languages

Although asynchronous programming is supported in many languages, some
details vary across implementations. Rust's implementation of async
differs from most languages in a few ways:

การใช้งาน async ของ Rust ต่างจากภาษาส่วนใหญ่สองสามอย่าง:

- **Futures are inert** in Rust and make progress only when polled. Dropping a
  future stops it from making further progress. 😑
    
- **Async is zero-cost** in Rust, which means that you only pay for what you use.
  Specifically, you can use async without heap allocations and dynamic dispatch,
  which is great for performance!
  This also lets you use async in constrained environments, such as embedded systems.
- **Async is zero-cost** ไม่สำรองทรัพยากรณ์ไว้ล่วงหน้า ใช้เท่าไหร่ เอาเท่านั้น ไม่เปลือง ประหยัด  
  
- **No built-in runtime** is provided by Rust. Instead, runtimes are provided by
  community maintained crates.
  
  **No built-in runtime** จัดทำโดยชุมชน
  
- **Both single- and multithreaded** runtimes are available in Rust, which have
  different strengths and weaknesses.
- **Both single- and multithreaded** การทำงาน 1เทรด หรือหลายเทรด มีจุดเด่นต่างกัน

## Async vs threads in Rust

The primary alternative to async in Rust is using OS threads, either
directly through [`std::thread`](https://doc.rust-lang.org/std/thread/)
or indirectly through a thread pool.
Migrating from threads to async or vice versa
typically requires major refactoring work, both in terms of implementation and
(if you are building a library) any exposed public interfaces. As such,
picking the model that suits your needs early can save a lot of development time.

**OS threads** are suitable for a small number of tasks, since threads come with
CPU and memory overhead. Spawning and switching between threads
is quite expensive as even idle threads consume system resources.
A thread pool library can help mitigate some of these costs, but not all.
However, threads let you reuse existing synchronous code without significant
code changes—no particular programming model is required.
In some operating systems, you can also change the priority of a thread,
which is useful for drivers and other latency sensitive applications.

**Async** provides significantly reduced CPU and memory
overhead, especially for workloads with a
large amount of IO-bound tasks, such as servers and databases.
All else equal, you can have orders of magnitude more tasks than OS threads,
because an async runtime uses a small amount of (expensive) threads to handle
a large amount of (cheap) tasks.
However, async Rust results in larger binary blobs due to the state
machines generated from async functions and since each executable
bundles an async runtime.

On a last note, asynchronous programming is not _better_ than threads,
but different.
If you don't need async for performance reasons, threads can often be
the simpler alternative.

### Example: Concurrent downloading

In this example our goal is to download two web pages concurrently.
In a typical threaded application we need to spawn threads
to achieve concurrency:

```rust,ignore
{{#include ../../examples/01_02_why_async/src/lib.rs:get_two_sites}}
```

However, downloading a web page is a small task; creating a thread
for such a small amount of work is quite wasteful. For a larger application, it
can easily become a bottleneck. In async Rust, we can run these tasks
concurrently without extra threads:

```rust,ignore
{{#include ../../examples/01_02_why_async/src/lib.rs:get_two_sites_async}}
```

Here, no extra threads are created. Additionally, all function calls are statically
dispatched, and there are no heap allocations!
However, we need to write the code to be asynchronous in the first place,
which this book will help you achieve.

## Custom concurrency models in Rust

On a last note, Rust doesn't force you to choose between threads and async.
You can use both models within the same application, which can be
useful when you have mixed threaded and async dependencies.
In fact, you can even use a different concurrency model altogether,
such as event-driven programming, as long as you find a library that
implements it.
