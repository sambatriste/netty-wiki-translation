Since Netty version 4, the life cycle of certain objects are managed by their reference counts rather than the garbage collector.  [`ByteBuf`](http://netty.io/4.0/api/index.html?io/netty/buffer/ByteBuf.html) is the most notable type which takes advantage of reference counting to improve the allocation and deallocation performance.

## Basics of reference counting

The initial reference count of a new reference-counted object is 1:

```java
ByteBuf buf = ctx.alloc().directBuffer();
assert buf.refCnt() == 1;
```

When you release the reference-counted object, its reference count is decreased by 1.  If the reference count reaches at 0, the reference-counted object is deallocated or returned to the object pool it came from:

```java
assert buf.refCnt() == 1;
// release() returns true only if the reference count becomes 0.
boolean destroyed = buf.release();
assert destroyed;
assert buf.refCnt() == 0;
```

### Dangling reference

Attempting to access the reference-counted object whose reference count is 0 will trigger an `IllegalReferenceCountException`:

```java
assert buf.refCnt() == 0;
try {
  buf.writeLong(0xdeadbeef);
  throw new Error("should not reach here");
} catch (IllegalReferenceCountExeception e) {
  // Expected
}
```

### Increasing the reference count

A reference count can also be incremented via the `retain()` operation as long as it is not destroyed yet:

```java
ByteBuf buf = ctx.alloc().directBuffer();
assert buf.refCnt() == 1;

buf.retain();
assert buf.refCnt() == 2;

boolean destroyed = buf.release();
assert !destroyed;
assert buf.refCnt() == 1;
```

### Who destroys it?

The general rule of thumb is that the party who accesses a reference-counted object lastly is responsible for the destruction of the reference-counted object. More specifically:

* If a component is supposed to pass a reference-counted object to the other component, the copmonent usually does not need to destroy it but defer the decision to the other component.
* If a component consumes a reference-counted object and nothing else will access it anymore, the component should destroy it by its own.

Here is a simple example:

```java
public ByteBuf a(ByteBuf input) {
    input.writeByte(42);
    return input;
}

public ByteBuf b(ByteBuf input) {
    try {
        output = input.alloc().directBuffer(input.readableBytes() + 1);
        output.writeBytes(input);
        output.writeByte(42);
        return output;
    } finally {
        input.release();
    }
}

public void c(ByteBuf input) {
    System.out.println(input);
    input.release();
}

public void main() {
    ...
    ByteBuf buf = ...;
    // This will print buf to System.out and destroy it.
    c(b(a(buf)));
    assert buf.refCnt() == 0;
}
```

| Action                              | Who should release? | Who released?
|-------------------------------------|---------------------|----------------
| 1. `main()` creates `buf`           | `buf`→`main()`      |
| 2. `main()` calls `a()` with `buf`  | `buf`→`a()`         |
| 3. `a()` returns `buf` merely.      | `buf`→`main()`      |
| 4. `main()` calls `b()` with `buf`  | `buf`→`b()`         |
| 5. `b()` returns the copy of `buf`  | `buf`→`b()`, `copy`→`main()` | `b()` releases `buf`
| 6. `main()` calls `c()` with `copy` | `copy`→`c()`        |
| 7. `c()` swallows `copy`            | `copy`→`c()`        | `c()` releases `copy`

### Derived buffers

`ByteBuf.duplicate()`, `ByteBuf.slice()` and `ByteBuf.order(ByteOrder)` create a _derived_ buffer which shares the memory region of the parent buffer.  A derived buffer does not have its own reference count but shares the reference count of the parent buffer.

```java
ByteBuf parent = ctx.alloc().directBuffer();
ByteBuf derived = parent.duplicate();

// Creating a derived buffer does not increase the reference count.
assert parent.refCnt() == 1;
assert derived.refCnt() == 1;
```

Note that a parent buffer and its derived buffers share the same reference count, and the reference count does not increase when a derived buffer is created.  Therefore, if you are going to pass a derived buffer to other component of your application, you'll have to call `retain()` it first.

```java
ByteBuf parent = ctx.alloc().directBuffer(512);
parent.writeBytes(...);

try {
    while (parent.isReadable(16)) {
        ByteBuf derived = parent.readSlice(16);
        derived.retain();
        process(derived);
    }
} finally {
    parent.release();
}
...

public void process(ByteBuf buf) {
    ...
    buf.release();
}
```

### `ByteBufHolder` interface

Sometimes, a `ByteBuf` is contained by a buffer holder, such as [`DatagramPacket`](http://netty.io/4.0/api/index.html?io/netty/channel/socket/DatagramPacket.html), [`HttpContent`](http://netty.io/4.0/api/index.html?io/netty/handler/codec/http/HttpContent.html), and [`WebSocketframe`](http://netty.io/4.0/api/index.html?io/netty/handler/codec/http/websocketx/WebSocketFrame.html).  Those types extend a common interface called [`ByteBufHolder`](http://netty.io/4.0/api/index.html?io/netty/buffer/ByteBufHolder.html).

A buffer holder shares the reference count of the buffer it contains, just like a derived buffer.

## Reference-counted messages in `ChannelHandler`

### Inbound messages

When an event loop reads data into a `ByteBuf` and triggers a `channelRead()` event with it, it is the responsibility of the `ChannelHandler` in the corresponding pipeline to release the buffer.  Therefore, the handler that consumes the received data should call `release()` on the data in its `channelRead()` handler method:

```java
public void channelRead(ChannelHandlerContext ctx, Object msg) {
    ByteBuf buf = (ByteBuf) msg;
    try {
        ...
    } finally {
        buf.release();
    }
}
```

As explained in the 'Who destroys?' section of this document, if your handler passes the buffer (or any reference-counted object) to the next handler, you don't need to release it:

```java
public void channelRead(ChannelHandlerContext ctx, Object msg) {
    ByteBuf buf = (ByteBuf) msg;
    ...
    ctx.fireChannelRead(buf);
}
```

Note that `ByteBuf` isn't the only reference-counted type in Netty.  If you are dealing with the messages generated by decoders, it is very likely that the message is also reference-counted:

```java
// Assuming your handler is placed next to `HttpRequestDecoder`
public void channelRead(ChannelHandlerContext ctx, Object msg) {
    if (msg instanceof HttpRequest) {
        HttpRequest req = (HttpRequest) msg;
        ...
    }
    if (msg instanceof HttpContent) {
        HttpContent content = (HttpContent) msg;
        try {
            ...
        } finally {
            content.release();
        }
    }
}
```

If you are in doubt or you want to simplify releasing the messages, you can use `ReferenceCountUtil.release()`:

```java
public void channelRead(ChannelHandlerContext ctx, Object msg) {
    try {
        ...
    } finally {
        ReferenceCountUtil.release(msg);
    }
}
```

Alternatively, you could consider extending `SimpleChannelHandler` which calls `ReferenceCountUtil.release(msg)` for all messages you receive.

### Outbound messages

Unlike inbound messages, the messages are created by your application, and it is the responsibility of Netty to release it after writing it out to the wire.  However, the handlers that intercepts your write requests should make sure to release any intermediary objects properly. (e.g. encoders)

```java
// Simple-pass through
public void write(ChannelHandlerContext ctx, Object message, ChannelPromise promise) {
    System.err.println("Writing: " + message);
    ctx.write(message, promise);
}

// Transformation
public void write(ChannelHandlerContext ctx, Object message, ChannelPromise promise) {
    if (message instanceof HttpContent) {
        // Transform HttpContent to ByteBuf.
        HttpContent content = (HttpContent) message;
        try {
            ByteBuf transformed = ctx.alloc().buffer();
            ....
            ctx.write(transformed, promise);
        } finally {
            content.release();
        }
    } else {
        // Pass non-HttpContent through.
        ctx.write(message, promise);
    }
}
```

## Troubleshooting buffer leaks

The disadvantage of reference counting is that it is easy to leak the reference-counted objects.  Because JVM is not aware of the reference counting Netty implements, it will automatically GC them once they become unreachable even if their reference counts are not zero. An object once garbage collected cannot be resurrected, and thus cannot be returned to the pool it came from or will produce memory leak.


Fortunately, despite its difficulty of finding leaks, Netty will by default sample about 1% of buffer allocations to check if there is a leak in your application.  In case of leak, you will find the following log message:

> LEAK: `ByteBuf.release()` was not called before it's garbage-collected. Enable advanced leak reporting to find out where the leak occurred. To enable advanced leak reporting, specify the JVM option '`-Dio.netty.leakDetectionLevel=advanced`' or call `ResourceLeakDetector.setLevel()`

Relaunch your application with the JVM option mentioned above, then you'll see the recent locations of your application where the leaked buffer was accessed.  The following output shows a leak from our unit test (`XmlFrameDecoderTest.testDecodeWithXml()`):

```
Running io.netty.handler.codec.xml.XmlFrameDecoderTest
15:03:36.886 [main] ERROR io.netty.util.ResourceLeakDetector - LEAK: ByteBuf.release() was not called before it's garbage-collected.
Recent access records: 1
#1:
	io.netty.buffer.AdvancedLeakAwareByteBuf.toString(AdvancedLeakAwareByteBuf.java:697)
	io.netty.handler.codec.xml.XmlFrameDecoderTest.testDecodeWithXml(XmlFrameDecoderTest.java:157)
	io.netty.handler.codec.xml.XmlFrameDecoderTest.testDecodeWithTwoMessages(XmlFrameDecoderTest.java:133)
	...

Created at:
	io.netty.util.ResourceLeakDetector.open(ResourceLeakDetector.java:169)
	io.netty.buffer.AbstractByteBufAllocator.toLeakAwareBuffer(AbstractByteBufAllocator.java:42)
	io.netty.buffer.UnpooledByteBufAllocator.newDirectBuffer(UnpooledByteBufAllocator.java:55)
	io.netty.buffer.AbstractByteBufAllocator.directBuffer(AbstractByteBufAllocator.java:155)
	io.netty.buffer.UnpooledUnsafeDirectByteBuf.copy(UnpooledUnsafeDirectByteBuf.java:465)
	io.netty.buffer.WrappedByteBuf.copy(WrappedByteBuf.java:697)
	io.netty.buffer.AdvancedLeakAwareByteBuf.copy(AdvancedLeakAwareByteBuf.java:656)
	io.netty.handler.codec.xml.XmlFrameDecoder.extractFrame(XmlFrameDecoder.java:198)
	io.netty.handler.codec.xml.XmlFrameDecoder.decode(XmlFrameDecoder.java:174)
	io.netty.handler.codec.ByteToMessageDecoder.callDecode(ByteToMessageDecoder.java:227)
	io.netty.handler.codec.ByteToMessageDecoder.channelRead(ByteToMessageDecoder.java:140)
	io.netty.channel.ChannelHandlerInvokerUtil.invokeChannelReadNow(ChannelHandlerInvokerUtil.java:74)
	io.netty.channel.embedded.EmbeddedEventLoop.invokeChannelRead(EmbeddedEventLoop.java:142)
	io.netty.channel.DefaultChannelHandlerContext.fireChannelRead(DefaultChannelHandlerContext.java:317)
	io.netty.channel.DefaultChannelPipeline.fireChannelRead(DefaultChannelPipeline.java:846)
	io.netty.channel.embedded.EmbeddedChannel.writeInbound(EmbeddedChannel.java:176)
	io.netty.handler.codec.xml.XmlFrameDecoderTest.testDecodeWithXml(XmlFrameDecoderTest.java:147)
	io.netty.handler.codec.xml.XmlFrameDecoderTest.testDecodeWithTwoMessages(XmlFrameDecoderTest.java:133)
	...
```

### Leak detection levels

There are currently 4 levels of leak detection:

* `DISABLED` - disables leak detection completely. Not recommended.
* `SIMPLE` - tells if there is a leak or not for 1% of buffers. Default.
* `ADVANCED` - tells where the leaked buffer was accessed for 1% of buffers.
* `PARANOID` - Same with `ADVANCED` except that it's for every single buffer.  Useful for automated testing phase. You could fail the build if the build output contains '`LEAK: `'.

You can specify the leak detection level as a JVM option `-Dio.netty.leakDetectionLevel`

```
java -Dio.netty.leakDetectionLevel=advanced ...
```

### Best practice for leak-less application

* Run your unit tests and integration tests at `PARANOID` leak detection level, as well as at `SIMPLE` level.
* Canary your application before rolling out to the entire cluster at `SIMPLE` level for a reasonably long time to see if there's a leak.
* If there is a leak, canary again at `ADVANCED` level to get some hints about where the leak is coming from.
* Do not deploy an application with a leak to the entire cluster.

### Fixing leaks in unit tests

It is very easy to forget to release a buffer or a message in a unit test.  It will generate a leak warning, but it does not necessarily mean that your application has a leak.  Instead of wrapping your unit tests with `try-finally` blocks to release all buffers, you can use `ReferenceCountUtil.releaseLater()` utility method:

```java
import static io.netty.util.ReferenceCountUtil.*;

@Test
public void testSomething() throws Exception {
    // ReferenceCountUtil.releaseLater() will keep the reference of buf,
    // and then release it when the test thread is terminated.
    ByteBuf buf = releaseLater(Unpooled.directBuffer(512));
    ...
}
```