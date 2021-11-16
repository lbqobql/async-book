# Under the Hood: Executing `Future`s and Tasks

การทำงานเบื้องหลัง `Future` และการจัดการงาน

In this section, we'll cover the underlying structure of how `Future`s and
asynchronous tasks are scheduled. If you're only interested in learning
how to write higher-level code that uses existing `Future` types and aren't
interested in the details of how `Future` types work, you can skip ahead to
the `async`/`await` chapter. However, several of the topics discussed in this
chapter are useful for understanding how `async`/`await` code works,
understanding the runtime and performance properties of `async`/`await` code,
and building new asynchronous primitives. If you decide to skip this section
now, you may want to bookmark it to revisit in the future.

บทนี้ครอบคลุมพื้นฐานของ ไลบรารี่ `Future` และการจัดการงานแบบ async 
สามารถข้ามบทนี้ได้ ถ้าไม่สนใจการทำงานพื้นฐานของ ไลบรารี่ `Future`
อย่างไรก็ตาม อ่านเถอะ จะได้ไม่ต้องกลับมาอ่านทีหลัง


Now, with that out of the way, let's talk about the `Future` trait.
