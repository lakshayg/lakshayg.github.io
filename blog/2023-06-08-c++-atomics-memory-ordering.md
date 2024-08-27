I have always used C++ atomics pretty without evey paying attention to the `memory_order` parameter that a number of member functions take.
Today I finally sat down to grasp it properly and this post summarizes what I found.

Before going into the different memory order, it makes sense to discuss the need for them.
Modern compilers and CPU architectures are sneaky and often re-order instructions in order to optimize program execution as long as they can guarantee that there are no visible side-effects.
This works nicely for single threaded programs but can introduce unexpected issues in multi-threaded programs because threads can read/write from/to the cache without syncronizing with the main memory.

The `memory_order` parameter gives the developer control over how memory is synchronized around atomic operations.

There are three main `memory_order` patterns:

**memory_order_relaxed** - No memory syncronization happens, only the atomicity of the operation is guaranteed. For example:

```cpp
// thread 1
data = 42;
done.store(true, memory_order_relaxed);

// thread 2
while (!done.load(memory_order_relaxed)) {}
std::cout << data << '\n';  // this may or may not print 42 since
                            // there are no guarantees on whether
                            // the write in thread 1 was synchronized
```

This kind of memory ordering is useful when we are using the atomic as a counter and not for synchronization.

**memory_order_release** - All writes before this atomic operation are released to main memory before the atomic

```cpp
data = 42;
done.store(true, memory_order_release); // this ensures that data is synchronized before `done` is written
```

**memory_order_acquire** - Acquires the value of this atomic and other released writes from the main memory

This allows threads to synchronize when the writing thread uses `memory_order_release`

```cpp
// thread 1
data = 42;
done.store(true, memory_order_release); // releases `data` to the main memory before `done`

// thread 2
if (done.load(memory_order_acquire)) {
  std::cout << data << '\n';  // guaranteed to read the values that were released
}
```

**memory_order_seq_cst** - This is easiest to understand by comparing with release/acquire. With release/acquire, we guarantee memory synchronization around a particular atomic. In the example above, we are synchronizing around `done`. For contrast, consider this:

```cpp
// thread 1
data = 42;
done1.store(true, memory_order_release); // releases `data` to main memory before `done1`

// thread 2
if (done2.load(memory_order_acquire)) { // will read all the data that was released before `done2`
  std::cout << data << '\n';            // may or may not print 42
}
```

However, if we were to use `memory_order_seq_cst`, all acquires synchronize with all releases.
