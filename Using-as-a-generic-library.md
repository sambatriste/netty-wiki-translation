Netty is a framework for building network applications, but also provides foundation classes handy for other uses, even programs that do not perform socket I/O.

## Buffer API

`io.netty.buffer` provides a generic buffer type called `ByteBuf`. It is like `java.nio.ByteBuffer`, but faster, more user-friendly, and extensible.

### User-friendliness

Have you ever forgotten to call `java.nio.ByteBuffer.flip()` and wondered why the buffer does not contain anything?  It never happens in `ByteBuf` because it has two indexes, one for reads and one for writes:

```java
ByteBuf buf = ...;
buf.writeUnsignedInt(42);
assertThat(buf.readUnsignedInt(), is(42));
```

It has a richer set of access methods to more easily access a buffer's contents. For example, it has accessor methods for signed and unsigned integers, searches, and strings.

### Extensibility

You can't subclass `java.nio.ByteBuffer`, but you can with `ByteBuf`. An abstract skeletal implementation is also provided for your convenience. You can thus write your own buffer implementation, such as file-backed ones, composite ones, and even a hybrid.

### Performance

When a new `java.nio.ByteBuffer` is allocated, its content is filled with zeroes.  This "zeroing" consumes CPU cycles and memory bandwidth.  Normally, the buffer is then immediately filled from some data source, so the zeroing did no good.

To be reclaimed, `java.nio.ByteBuffer` relies on the JVM garbage collector.  It works OK for heap buffers, but not direct buffers.  By design, direct buffers are expected to live a long time. Thus, allocating many short-lived direct NIO buffers often causes an `OutOfMemoryError`.  Also, deallocating a direct buffer explicitly using the (hidden, proprietary) API isn't very fast.

A `ByteBuf`'s life cycle is bound to its reference count. When its count goes to zero, its underlying memory region (`byte[]` or direct buffer) is explicitly dereferenced, deallocated, or returned to the pool.

Netty also provides a solid buffer pool implementation that does not waste CPU cycles or memory bandwidth with zeroing its buffer:

```java
ByteBufAllocator alloc = PooledByteBufAllocator.DEFAULT;
ByteBuf buf = alloc.directBuffer(1024);
...
buf.release(); // The direct buffer is returned to the pool.
```

However, reference counting isn't a holy grail.  If the JVM garbage-collects the pooled buffer before its underlying memory region is returned to the pool, the leaks will eventually exhaust the pool.

To help you troubleshoot a leak, Netty provides a leak-detection mechanism which is flexible enough to let you trade off between your application's performance and the detail of the leak report.  For more information, please refer to [[Reference-counted-objects]].

## Listenable futures and event loops

Performing a task asynchronously - scheduling a task and getting notified on completion - is common, and should be easy. When `java.util.concurrent.Future` appeared first, our excitement didn't last long.  We had to _block_ to get notified on completion.  In asynchronous programming, you specify _what to do_ on completion rather than wait for the outcome.

`io.netty.concurrent.Future` is a subtype of JDK `Future`.  It lets you add a listener which will be invoked by an event loop when the future is fulfilled.

`io.netty.util.concurrent.EventExecutor` is a single-threaded event loop that extends `java.util.concurrent.ScheduledExecutorService`. You can build your own event loop or use it as a feature-rich task executor.  Usually, you create multiple `EventExecutor's to take advantage of parallelism:

```java
EventExecutorGroup group = new DefaultEventExecutorGroup(4); // 4 threads
Future<?> f = group.submit(new Runnable() { ... });
f.addListener(new FutureListener<?> {
  public void operationComplete(Future<?> f) {
    ..
  }
});
...
```

### The global event loop

Sometimes you want a unique executor that is always available and that doesn't need life cycle management. `GlobalEventExecutor` is a single-threaded singleton `EventExecutor` which starts its thread lazily and stops when there have been no pending tasks for a while. 

```java
GlobalEventExecutor.INSTANCE.execute(new Runnable() { ... });
```

Internally, Netty uses it to notify the termination of other `EventExecutor`s.

## Platform-dependent operations

_Note that this feature is for internal use only. We are considering moving it out of the internal package if there is sufficient demand._

`io.netty.util.internal.PlatformDependent` provides platform-dependent and potentially unsafe operations.  You can think of it as a thin layer over `sun.misc.Unsafe` and other platform-dependent proprietary APIs.

## Other utilities

To build a high-performing network application framework, we introduced utilities. You might find some useful.

### Thread-local object pool

If your thread is long-running and you allocate many short-living objects of same type, you can use a thread-local object pool, called `Recycler`. `Recycler` reduces the amount of the garbage you produce, reducing memory bandwidth and load on the garbage collector.

```java
public class MyObject {

  private static final Recycler<MyObject> RECYCLER = new Recycler<MyObject>() {
    protected MyObject newObject(Recycler.Handle<MyObject> handle) {
      return new MyObject(handle);
    }
  }

  public static MyObject newInstance(int a, String b) {
    MyObject obj = RECYCLER.get();
    obj.myFieldA = a;
    obj.myFieldB = b;
    return obj;
  }
    
  private final Recycler.Handle<MyObject> handle;
  private int myFieldA;
  private String myFieldB;

  private MyObject(Handle<MyObject> handle) {
    this.handle = handle;
  }
  
  public boolean recycle() {
    myFieldA = 0;
    myFieldB = null;
    return handle.recycle(this);
  }
}

MyObject obj = MyObject.newInstance(42, "foo");
...
obj.recycle();
```

### User-extensible enum

`enum` is great for a static set of constants, but you cannot extend one.  When you need to add more constants in runtime or allow third parties to define additional constants, use the extensible `io.netty.util.ConstantPool` instead:

```java
public final class Foo extends AbstractConstant<Foo> {
  Foo(int id, String name) {
    super(id, name);
  }
}

public final class MyConstants {

  private static final ConstantPool<Foo> pool = new ConstantPool<Foo>() {
    @Override
    protected Foo newConstant(int id, String name) {
      return new Foo(id, name);
    }
  };

  public static Foo valueOf(String name) {
    return pool.valueOf(name);
  }

  public static final Foo A = valueOf("A");
  public static final Foo B = valueOf("B");
}

private final class YourConstants {
  public static final Foo C = MyConstants.valueOf("C");
  public static final Foo D = MyConstants.valueOf("D");
}
```

Netty uses `ConstantPool` to define `ChannelOption`s so that non-core transports can define transport-specific options in a type-safe way.

### Attribute map

Use `io.netty.util.AttributeMap` interface for a fast, type-safe, thread-safe collection of key-value pairs:

```java
public class Foo extends DefaultAttributeMap {
  ...
}

public static final AttributeKey<String> ATTR_A = AttributeKey.valueOf("A");
public static final AttributeKey<Integer> ATTR_B = AttributeKey.valueOf("B");

Foo o = ...;
o.attr(ATTR_A).set("foo");
o.attr(ATTR_B).set(42);
```

As you might have noticed already, `AttributeKey` is a `Constant`.

### Hashed wheel timer

Hashed wheel timer is a scalable alternative to `java.util.Timer` and `java.util.concurrent.ScheduledThreadPoolExecutor`.  It can handle many scheduled tasks and their cancellations efficiently, as shown in the following table:

|                                                      | Schedule a new task | Cancel a task
|------------------------------------------------------|---------------------|-------------------------------------------
| `HashedWheelTimer`                                   | O(1)                | O(1)
| `java.util.Timer` and `ScheduledThreadPoolExecutor`  | O(logN)             | O(logN) where N = number of pending tasks

Internally, it uses a hash table whose key is a task's timing to yield constant time for most timer operations. (`java.util.Timer` uses a binary heap.)

For more information about hashed wheel timer, please see [these slides ("Hashed and Hierarchical Timing Wheels," Varghese and Lauck)](http://www.cse.wustl.edu/~cdgill/courses/cs6874/TimingWheels.ppt) and [this paper ("Hashed and Hierarchical Timing Wheels: Data Structures 
for the Efficient Implementation of a Timer Facility," Varghese and Lauck)](http://www.cs.columbia.edu/~nahum/w6998/papers/sosp87-timing-wheels.pdf).

### Even more miscellaneous utilities

The following classes are useful, but you will find fine alternatives in other libraries such as Guava:

* `io.netty.util.CharsetUtil` provides commonly-used `java.nio.charset.Charset`s.
* `io.netty.util.NetUtil` provides commonly-used network-related constants such as the `InetAddress` for IPv4 localhost.
* `io.netty.util.DefaultThreadFactory` is a generic `ThreadFactory` implementation that lets you configures your executor threads easily.

## Comparison with Guava and JDK8

Because Netty tries to minimize its set of dependencies, some of its utility classes are similar to those in other popular libraries, such as [Guava](http://code.google.com/p/guava-libraries/).

Such libraries provide various utility classes and alternative data types to make JDK API less painful to work with, and they usually do quite well at that.

Netty focuses on providing the constructs required for:

* Asynchronous programming
* Low-level operations (a.k.a "mechanical sympathy") such as:
  * Off-heap access
  * Access to the proprietary intrinsic operations
  * Platform-dependent behaviors

Java sometimes advances by adopting ideas that subsume constructs provided by Netty.  For example, JDK 8 adds [`CompletableFuture`](http://download.java.net/jdk8/docs/api/java/util/concurrent/CompletableFuture.html) which somewhat overlaps `io.netty.util.concurrent.Future`. In such a case, Netty's constructs provide a good migration path to you; We will diligently update the API with future migration in mind.