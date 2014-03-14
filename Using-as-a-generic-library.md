[Netty](https://netty.io/) does not only provide a framework dedicated to building a network application, but also provides a set of foundation classes that have evolved over time to work fine under various uses cases.  This page will give you a short tour of such utility classes you might find very useful when you write any type of applications, even the ones that do not perform any socket I/O.

## Buffer API

`io.netty.buffer` provides a generic buffer type called `ByteBuf`.  It aims to replace `java.nio.ByteBuffer` with user-friendliness, extensibility, and performance in mind.

### User-friendliness

Have you ever forgotten to call `java.nio.ByteBuffer.flip()` and wondered why the buffer does not contain anything?  It never happens in `ByteBuf` because it has two indexes - one for reads and the other for writes:

```java
ByteBuf buf = ...;
buf.writeInt(42);
assertThat(buf.readInt(), is(42));
```

It also has a richer set of access methods that will help you access the content of a buffer more conveniently.  For example, it has accessor methods for unsigned primitives, searches, and string conversions.

### Extensibility

It's impossible to subclass `java.nio.ByteBuffer`, but you can with `ByteBuf`. An abstract skeletal implementation is also provided for your convenience.  This means, you can implement your own buffer implementation, such as file-backed ones, composite ones, and even a hybrid.

### Performance

When a new `java.nio.ByteBuffer` is allocated, its content always must be filled with zeroes.  This 'zeroing' process consumes both CPU cycles and memory bandwidth.  Safety is a good thing, but what practically happens is that a buffer is almost always filled from other data sources as soon as it is allocated.

`java.nio.ByteBuffer` relies on the garbage collector of the JVM for reclamation of itself.  It works OK for heap buffers, but it never does for direct buffers.  By design, direct buffers are expected to live long, and therfore, trying to allocate many short-living direct NIO buffers often leads to an `OutOfMemoryError`.  Also, deallocating a direct buffer explicitly using the hidden proprietary API isn't very fast either.

The life cycle of `ByteBuf` is bound to its reference count.  When the reference count of a `ByteBuf` becomes zero, it's underlying memory region (byte[] or a direct buffer) is explicitly dereferenced, deallocated, or returned to the pool.

Netty also provides a solid buffer pool implementation that does not waste CPU cycles or memory bandwidth for zeroing the buffer:

```java
ByteBufAllocator alloc = PooledByteBufAllocator.DEFAULT;
ByteBuf buf = alloc.directBuffer(1024);
...
buf.release(); // The direct buffer is returned to the pool.
```

However, reference counting isn't a holy grail.  If the pooled buffer has been garbage-collected by JVM before its underlying memory region is returned to the pool, the pool will be exhausted eventually due to the leaks.

To help you troubleshoot a leak, Netty provides leak detection mechanism, which is flexible enough to let you trade off between the performance of your application and the detail of the leak report.  For more information, please refer to [[Reference-counted-objects]].

## Listenable futures and event loops

Performing a task asynchronously - scheduling a task and getting notified on completion - is becoming more and more important.

When `java.util.concurrent.Future` appeared first, our excitement didn't last long.  It was because we had to _block_ to get notified on completion.  In asynchronous programming, you must be able to specify _what has to be done_ on completion rather than waiting for the outcome.

`io.netty.concurrent.Future` is a subtype of JDK `Future`.  It allows you to add a listener to it so your listener is invoked by an event loop when the future is fulfilled.

`io.netty.util.concurrent.EventExecutor` is a single-threaded event loop that extends `java.util.concurrent.ScheduledExecutorService`.  You can build your own event loop with it or use it as a feature-rich task executor.  Usually, you create multiple `EventExecutor's to take advantage of parallelism:

```java
EventExecutorGroup group = new DefaultEventExecutorGroup(4); // 4 threads
Future<?> f = group.submit(new Runnable() { ... });
f.addListener(new FutureListener<?> {
  public void oeprationComplete(Future<?> f) {
    ..
  }
});
...
```

### The global event loop

Sometimes, it is very handy to have a unique executor that is always available and that doesn't need life cycle management.  `GlobalEventExecutor` is a single-threaded singleton `EventExecutor` which starts its thread lazily and stops when there's no pending task for a while. 

```java
GlobalEventExecutor.INSTANCE.execute(new Runnable() { ... });
```

Internally, Netty uses it to notify the termination of other `EventExecutor`s.

## Platform-dependent operations

_Note that this feature is for internal use only at the moment.  We are considering to move it out of the internal package if there is good demand for it._

`io.netty.util.internal.PlatformDependent` provides various platform-dependent and potentially unsafe operations.  You can think of it as a thin layer over `sun.misc.Unsafe` and other platform dependent proprietary APIs.

## Other utilities

To build a highly performing network application framework, we introduced various utilities to feed ourselves, and we found some of them might also be useful for you.

### Thread-local object pool

If your thread is long running and you allocate a lot of short-living objects of same type, you can take advantage of thread-local object pool, called `Recycler`.  `Recycler` will reduce the amount of the garbage you produce, reducing the memory bandwidth and the load on the garbage collector.

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

`enum` is a great language construct for representing a static set of constants, but you cannot extend it to define additional constants.  Extensible enums are useful when you have to add more constants in runtime or when you need to allow third parties to define additional constants.  For example, Use `io.netty.util.ConstantPool` instead:

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

Netty uses `ConstantPool` to define `ChannelOption`s so that non-core transports define transport-specific options in a type-safe way.

### Attribute map

Sometimes you have to implement the ability to attach an arbitrary property to an object, but you want to make sure accessing such a property is fast enough and as type-safe as possible.  Use `io.netty.util.AttributeMap` interface:

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

Hashed wheel timer is similar to `java.util.Timer`, but it differs in that it is capable of handling a large number of scheduled tasks and their cancellation efficiently.

### Even more miscellaneous utilities

The following classes are also useful but you will find their alternatives from other libraries such as Guava:

* `io.netty.util.CharsetUtil` provides commonly-used `java.nio.charset.Charset`s.
* `io.netty.util.NetUtil` provides commonly-used network-related constants such as the `InetAddress` for IPv4 localhost.
* `io.netty.util.DefaultThreadFactory` is a generic `ThreadFactory` implementation that lets you configures your executor threads easily.

