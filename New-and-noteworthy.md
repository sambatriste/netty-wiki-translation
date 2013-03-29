This document walks you through the list of notable changes and new features in the major Netty release to give you an idea to port your application to the new version.

## Project structure changes

The package name of Netty has been changed from org.jboss.netty to io.netty since [we are not part of JBoss.org anymore](http://netty.io/news/2011/11/04/new-web-site.html).

The binary JAR has been split into multiple submodules so that a user can exclude unnecessary features from the class path.  The current structure looks like this following:

| Module | Description    |
|--------|----------------|
| [netty](https://github.com/netty/netty)  | project parent |
| [common](https://github.com/netty/netty/tree/master/common) | utility and logging |
| [buffer](https://github.com/netty/netty/tree/master/buffer) | buffer API |
| [transport](https://github.com/netty/netty/tree/master/transport) | channel API and its core implementations |
| [transport-rtrx](https://github.com/netty/netty/tree/master/transport-rxtx) | RTRX transport implementation |
| [transport-sctp](https://github.com/netty/netty/tree/master/transport-sctp) | SCTP transport implementation |
| [transport-udt](https://github.com/netty/netty/tree/master/transport-udt)  | UDT transport implementation |
| [handler](https://github.com/netty/netty/tree/master/handler) | channel handlers |
| [codec](https://github.com/netty/netty/tree/master/codec) | codec framework |
| [codec-http](https://github.com/netty/netty/tree/master/codec-http) | HTTP, Web Sockets, SPDY, and RTSP codec |
| [codec-socks](https://github.com/netty/netty/tree/master/codec-socks) | Socks codec |
| [example](https://github.com/netty/netty/tree/master/example) | examples |
| [all](https://github.com/netty/netty/tree/master/all) | generates an all-in-one JAR |
| [tarball](https://github.com/netty/netty/tree/master/tarball) | generates a tarball distribution |

All Netty jars (except for the netty-all jar) are now OSGi bundles and can be used in your favorite OSGi-Container.

## General API changes

* Most operations in Netty now support method chaining for brevity.
* Non-configuration getters have no `get/is` prefix anymore (e.g. `Channel.getRemoteAddress()` → `Channel.remoteAddress()`)

## Buffer API changes

### ChannelBuffer → ByteBuf

Thanks to the structural changes mentioned above, the buffer API can be used as a separate package.  Therefore, the type name `ChannelBuffer` does not make sense anymore, and has been renamed to `ByteBuf`.

The utility class `ChannelBuffers`, which creates a new buffer, has been split into two utility classes, `Unpooled` and `BufUtil`.  As can be guessed from its name, 4.0 introduced pooled `ByteBuf`s which can be allocated via the `ByteBufAllocator` implementations.

### Most buffers are dynamic with configurable maximum capacity

In 3.x, buffers were fixed or dynamic.  The capacity of a fixed buffer does not change once it is created while the capacity of a dynamic buffer changes whenever `write*(...)` method requires more space.

Since 4.0, all buffers are dynamic.  However, they are better than the old dynamic.  You can decrease or increase the capacity of a buffer more easily and more safely.  It's easy because there is a new method `ByteBuf.capacity(int newCapacity)`. It's safe because you can set the maximum capacity of a buffer so that it does not grow boundlessly.

```java
// No more dynamicBuffer() - use buffer().
ByteBuf buf = ByteBuf.buffer();

// Increase the capacity of the buffer.
buf.capacity(1024);
...

// Decrease the capacity of the buffer (the last 512 bytes are deleted.)
buf.capacity(512);
```

The only exception is the buffer which wraps a single buffer or a single byte array, created by `wrappedBuffer()`.  You cannot increase its capacity because it invalidates the whole point of wrapping an existing buffer - saving memory copies.  If you want to change the capacity after you wrap a buffer, you should just create a new buffer with enough capacity and copy the buffer you wanted to wrap.

### New interface: CompositeByteBuf

A new interface named `CompositeByteBuf` defines various advanced operations for composite buffer implementations.  A user can save bulk memory copy operations using a composite buffer at the cost of relatively expensive random access.  To create a new composite buffer, use either `Unpooled.wrappedBuffer(...)` like before or `Unpooled.compositeBuffer(...)`.

### Predictable NIO buffer conversion

The contract of `ChannelBuffer.toByteBuffer()` and its variants were not deterministic enough in 3.x.  It was impossible for a user to know if they would return a view buffer with shared data or a copied buffer with separate data.  4.0 replaces `toByteBuffer()` with `ByteBuf.nioBufferCount()`, `nioBuffer()`, and `nioBuffers()`.  If `nioBufferCount()` returns `0`, a user can always get a copied buffer by calling `copy().nioBuffer()`.

### Little endian support changes

Little endian support has been changed significantly.  Previously, a user was supposed to specify a `LittleEndianHeapChannelBufferFactory` or wrap an existing buffer with the desired byte order to get a little endian buffer.  4.0 adds a new method: `ByteBuf.order(ByteOrder)`.  It returns a view of the callee with the desired byte order:

```java
import io.netty.buffer.ByteBuf;
import io.netty.buffer.Unpooled;
import java.nio.ByteOrder;
 
ByteBuf buf = Unpooled.buffer(4);
buf.setInt(0, 1);
// Prints '00000001'
System.out.format("%08x%n", buf.getInt(0)); 
 
ByteBuf leBuf = buf.order(ByteOrder.LITTLE_ENDIAN);
// Prints '01000000'
System.out.format("%08x%n", leBuf.getInt(0));
 
assert buf != leBuf;
assert buf == buf.order(ByteOrder.BIG_ENDIAN);
```
### Pooled ByteBuf

As mentioned before, Netty 4 introduced pooled `ByteBuf` instances. This can be useful for many reasons, including but not limited to:

* Limit GC pressure because of heavy allocation/deallocation when using unpooled `ByteBuf`s
* Better handling of direct (native) `ByteBuf`
* A `ByteBuf` can be obtained via a `ByteBufAllocator`.

```java
public interface ByteBufAllocator {
 
    ByteBuf buffer();
    ByteBuf buffer(int initialCapacity);
    ByteBuf buffer(int initialCapacity, int maxCapacity);
    ByteBuf heapBuffer();
    ByteBuf heapBuffer(int initialCapacity);
    ByteBuf heapBuffer(int initialCapacity, int maxCapacity);
    ByteBuf directBuffer();
    ByteBuf directBuffer(int initialCapacity);
    ByteBuf directBuffer(int initialCapacity, int maxCapacity);
    ByteBuf ioBuffer();
 
    CompositeByteBuf compositeBuffer();
    CompositeByteBuf compositeBuffer(int maxNumComponents);
    CompositeByteBuf compositeHeapBuffer();
    CompositeByteBuf compositeHeapBuffer(int maxNumComponents);
    CompositeByteBuf compositeDirectBuffer();
    CompositeByteBuf compositeDirectBuffer(int maxNumComponents);
}
```

To get the current `ByteBufAllocator` from a handler, use the `ChannelHandlerContext.alloc()` or `Channel.alloc()` method:

```java
Channel channel = ...;
ByteBuf buf = channel.alloc().buffer(512);
....
channel.write(buf);
 
ChannelHandlerContext ctx = ...
ByteBuf buf2 = ctx.alloc().buffer(512);
....
channel.write(buf2)
```

Once a `ByteBuf` is written to the remote peer it will automatically get released to the pool again.

The default `ByteBufAllocator` is `PooledByteBufAllocator`. If you do not wish to use buffer pooling or use your own allocator, you can use `Channel.config().setAllocator(..)` with an alternative allocator such as `UnpooledByteBufAllocator`.

## Channel API changes

In 4.0, many classes under the `io.netty.channel` package have gone through a major overhaul, and thus simple text search-and-replace will not make your 3.x application work with 4.0.  This section will try to show the thought process behind such a big change, rather than being an exhaustive resource for all the changes.

### Revamped ChannelHandler interface
#### Upstream → Inbound, Downstream → Outbound

The terms 'upstream' and 'downstream' were pretty confusing to beginners.  4.0 uses 'inbound' and 'outbound' wherever possible.

#### New ChannelHandler hierarchy

In 3.x, `ChannelHandler` was just a tag interface, and `ChannelUpstreamHandler`, `ChannelDownstreamHandler`, and `LifeCycleAwareChannelHandler` defined the actual handler methods.  In Netty 4, `ChannelHandler` merges `LifeCycleAwareChannelHandler` along with a couple more methods which are going to be useful to any handler implementations:

```java
public interface ChannelHandler {
 
    void beforeAdd(ChannelHandlerContext ctx) throws Exception;
    void afterAdd(ChannelHandlerContext ctx) throws Exception;
    void beforeRemove(ChannelHandlerContext ctx) throws Exception;
    void afterRemove(ChannelHandlerContext ctx) throws Exception;
 
    void exceptionCaught(ChannelHandlerContext ctx, Throwable cause) throws Exception;
    void userEventTriggered(ChannelHandlerContext ctx, Object evt) throws Exception;
    ...
}
```

The following diagram depicts the new type hierarchy:

fixme

#### ChannelHandler with no event object

In 3.x, every I/O operation created a `ChannelEvent` object.  For each read / write, it additionally created a new `ChannelBuffer`.  It simplified the internals of Netty quite a lot because it delegates resource management and buffer pooling to the JVM.  However, it often was the root cause of GC pressure and uncertainty which are sometimes observed in a Netty-based application under high load.

4.0 removes event object creation almost completely by replacing the event objects with strongly typed method invocations.  3.x had catch-all event handler methods such as `handleUpstream()` and `handleDownstream()`, but this is not the case anymore.  Every event type has its own handler method now:

```java
// Before:
void handleUpstream(ChannelHandlerContext ctx, ChannelEvent e) throws Exception;
void handleDownstream(ChannelHandlerContext ctx, ChannelEvent e) throws Exception;
 
// After:
void channelRegistered(ChannelHandlerContext ctx) throws Exception;
void channelUnregistered(ChannelHandlerContext ctx) throws Exception;
void channelActive(ChannelHandlerContext ctx) throws Exception;
void channelInactive(ChannelHandlerContext ctx) throws Exception;
void inboundBufferUpdated(ChannelHandlerContext ctx) throws Exception;
 
void bind(ChannelHandlerContext ctx, SocketAddress localAddress, ChannelFuture future) throws Exception;
void connect(
        ChannelHandlerContext ctx, SocketAddress remoteAddress,
        SocketAddress localAddress, ChannelFuture future) throws Exception;
void disconnect(ChannelHandlerContext ctx, ChannelFuture future) throws Exception;
void close(ChannelHandlerContext ctx, ChannelFuture future) throws Exception;
void deregister(ChannelHandlerContext ctx, ChannelFuture future) throws Exception;
void flush(ChannelHandlerContext ctx, ChannelFuture future) throws Exception;
void read(ChannelHandlerContext ctx);
void sendFile(ChannelHandlerContext ctx, FileRegion region, ChannelPromise promise) throws Exception;
```

`ChannelHandlerContext` has also been changed to reflect the changes mentioned above:

```java
// Before:
ctx.sendUpstream(evt);
 
// After:
ctx.fireInboundBufferUpdated();
```

All these changes mean a user cannot extend the non-existing `ChannelEvent` interface anymore.  How then does the user define his or her own event type such as `IdleStateEvent`?  `ChannelHandler` in 4.0 has a handler method called `userEventTriggered()` which is dedicated to this specific user case.

#### Simplified channel state model

When a new connected `Channel` is created in 3.x, at least three `ChannelStateEvent`s are triggered: `channelOpen`, `channelBound`, and `channelConnected`.  When a `Channel` is closed, at least 3 more: `channelDisconnected`, `channelUnbound`, and `channelClosed`.

fixme

However, it's of dubious value to trigger that many events.  It is more useful for a user to get notified when a `Channel` enters the state where it can perform reads and writes.

fixme

`channelOpen`, `channelBound`, and `channelConnected` have been merged to `channelActive`.  `channelDisconnected`, `channelUnbound`, and `channelClosed` have been merged to `channelInactive`.  Likewise, `Channel.isBound()` and `isConnected()` have been merged to `isActive()`.

Note that `channelRegistered` and `channelUnregistered` are not equivalent to `channelOpen` and `channelClosed`.  They are new states introduced to support dynamic registration, deregistration, and re-registration of a `Channel`, as illustrated below:

fixme

#### Per-handler buffer

Unlike 3.x, which creates a new heap buffer on every read operation to trigger a upstream `MessageEvent`, 4.0 does not create a new buffer every time.  It reads data from a socket directly into the inbound buffer created by the user's `ChannelInboundByteHandler` and `ChannelInboundMessageHandler` implementation.

Because the inbound buffer created by the handler is reused until the associated channel is closed, the GC overhead and the memory bandwidth consumption stays minimal.  Also, a user has complete control over when the received data is discarded, and the codec implementation becomes much simpler and efficient.

Similarly, it is not Netty that creates an outbound buffer: a user's `ChannelOutboundByteHandler` and `ChannelOutboundMessageHandler` do.

##### Not necessarily one event per message

4.0 does not have `messageReceived` or `writeRequested` handler method anymore.  They are replaced with `inboundBufferUpdated` and `flush`.  A user enqueues one or more messages to an inbound (or outbound) buffer and triggers an `inboundBufferUpdated` (or `flush`) event.

```java
public void inboundBufferUpdated(ChannelHandlerContext ctx) {
    Queue<MyMessage> in = ctx.inboundMessageBuffer();
    Queue<MyNewMessage> out = ctx.nextInboundMessageBuffer();
    for (;;) {
        MyMessage m = in.poll();
        if (m == null) {
            break;
        }
        MyNewMessage decoded = decode(m);
        out.add(decoded);
    }
    ctx.fireInboundBufferUpdated();
}

public void flush(ChannelHandlerContext ctx, ChannelFuture future) {
    Queue<MyNewMessage> in = ctx.outboundMessageBuffer();
    Queue<MyMessage> out = ctx.nextOutboundMessageBuffer();
    for (;;) {
        MyNewMessage m = in.poll();
        if (m == null) {
            break;
        }
        MyMessage encoded = encode(m);
        out.add(encoded);
    }
    ctx.flush(future);
}
```

Alternatively, a user can trigger such event for every single inbound (or outbound) message to emulate the old behavior although it might be less efficient than the new way.

##### Message handler vs. Byte handler

In 3.x, a `MessageEvent` holds an arbitrary object.  It can be either a `ChannelBuffer` or a user-defined object and they are treated the same:

```java
@Override
public void messageReceived(ChannelHandlerContext ctx, MessageEvent evt) {
    Object msg = evt.getMessage();
    if (msg instanceof ChannelBuffer) {
        ChannelBuffer buf = (ChannelBuffer) msg;
        ...
    } else {
        MyMessage myMsg = (MyMessage) msg;
        ...
    }
}
```

In 4.0, they are treated differently since a handler does not handle an individual message but handles multiple messages:

```java
public void inboundBufferUpdated(ChannelHandlerContext ctx) {
    if (ctx.hasInboundByteBuffer()) {
        ByteBuf buf = ctx.inboundByteBuffer();
        ...
    } else {
        Queue<MyMessage> buf = ctx.inboundMessageBuffer();
        for (;;) {
            MyMessage msg = buf.poll();
            if (msg == null) {
                break;
            }
            ...
        }
    }
}
```

You might find it interesting that a `ServerChannel`'s handler is an inbound handler whose inbound buffer is `Queue<Channel>`.

##### Handler adapters

Most users will find creating and managing its life cycle tedious, and therefore a user is supposed to extend the predefined adapter classes to make their life easier:

* `ChannelHandlerAdapter`
* `ChannelStateHandlerAdapter`
* `ChannelOperationHandlerAdapter`
* `ChannelInboundMessageHandlerAdapter`
* `ChannelInboundByteHandlerAdapter`
* `ChannelOutboundMessageHandlerAdapter`
* `ChannelOutboundByteHandlerAdapter`

### Sensible and less error-prone inbound traffic suspension

3.x had an unintuitive inbound traffic suspension mechanism provided by `Channel.setReadable(boolean)`.  It introduced complicated interactions between ChannelHandlers and the handlers were easy to interfere with each other if implemented incorrectly.

In 4.0, a new outbound operation called `read()` has been added.  If you turn off the default auto-read flag with `Channel.config().setAutoRead(false)`, Netty will not read anything until you explicitly invoke the `read()` operation.  Once the `read()` operation you issue is complete and the channel again stops reading, an inbound event called `channelReadSuspended()` will be triggered so that you can re-issue another `read()` operation.  You can also intercept a `read()` operation to perform more advanced traffic control.

#### Suspension of accepting incoming connections

There was no way for a user to tell Netty 3.x to stop accepting incoming connections except for blocking the I/O thread or closing the server socket.  4.0 respects the `read()` operation when the auto-read flag is not set, just like an ordinary channel.

### Half-closed sockets

TCP and SCTP allow a user to shut down the outbound traffic of a socket without closing it completely.  Such a socket is called 'a half-closed socket', and a user can make a half-closed socket by calling `SocketChannel.shutdownOutput() method`. If a remote peer shuts down the outbound traffic, `SocketChannel.read(..)` will return `-1`, which was seemingly indistinguishable from a closed connection.

3.x did not have `shutdownOutput()` operation.  Also, it always closed the connection when `SocketChannel.read(..)` returns `-1`.

To support a half-closed socket, 4.0 adds `SocketChannel.shutdownOutput()` method, and a user can set the '`ALLOW_HALF_CLOSURE`' `ChannelOption` to prevent Netty from closing the connection automatically even if `SocketChannel.read(..)` returns `-1`.

### Flexible I/O thread allocation

In 3.x, a `Channel` is created by a `ChannelFactory` and the newly created `Channel` is automatically registered to a hidden I/O thread.  4.0 replaces `ChannelFactory` with a new interface called `EventLoopGroup` which consists of one or more `EventLoop`s.  Also, a new `Channel` is not registered to the `EventLoopGroup` automatically but a user has to call `EventLoopGroup.register()` explicitly.  

Thanks to this change (i.e. separation of `ChannelFactory` and I/O threads), a user can register different `Channel` implementations to the same `EventLoopGroup`, or same `Channel` implementations to different `EventLoopGroup`s.  For example,  you can run a NIO server socket, NIO client sockets, NIO UDP sockets, and in-VM local channels in the same I/O thread.  It should be very useful when writing a proxy server which requires minimal latency.

### Ability to create a `Channel` from an existing JDK socket

3.x provided no way to create a new Channel from an existing JDK socket such as `java.nio.channels.SocketChannel`.  You can with 4.0.

### Deregistration and re-registration of a Channel from·to an I/O thread

Once a new `Channel` is created in 3.x, it is completely tied to a single I/O thread until its underlying socket is closed.  In 4.0, a user can deregister a `Channel` from its I/O thread to gain the full control of its underlying JDK socket.  For example, you can take advantage of high-level non-blocking I/O Netty provides to deal with complex protocols, and then later deregister the `Channel` and switch to blocking mode to transfer a file at possible maximum throughput.  Of course, it is possible to register the deregistered `Channel` back again.

```java
java.nio.channels.FileChannel myFile = ...;
java.nio.channels.SocketChannel mySocket = java.nio.channels.SocketChannel.open();
 
// Perform some blocking operation here.
...
 
// Netty takes over.
SocketChannel ch = new NioSocketChannel(mySocket);
EventLoopGroup group = ...;
group.register(ch);
...
 
// Deregister from Netty.
ch.deregister().sync();
 
// Perform some blocking operation here.
mySocket.configureBlocking(false);
myFile.transferFrom(mySocket, ...);
 
// Register back again to another event loop group.
EventLoopGroup anotherGroup = ...;
anotherGroup.register(ch);
```

### Scheduling an arbitrary task to be run by an I/O thread

When a `Channel` is registered to an `EventLoopGroup`, the `Channel` is actually registered to one of the `EventLoop`s which is managed by the `EventLoopGroup`.  `EventLoop` implements `java.util.concurrent.ScheduledExecutorService`.  It means a user can execute or schedule an arbitrary `Runnable` or `Callable` in an I/O thread where the user's channel belongs to.  Along with the new well-defined thread model, which will be explained later, it became extremely easier to write a thread-safe handler.

```java
public class MyHandler extends ChannelOutboundMessageHandlerAdapter {
    ...
    public void flush(ChannelHandlerContext ctx, ChannelFuture f) {
        ...
        ctx.flush(f);
 
        // Schedule a write timeout.
        ctx.executor().schedule(new MyWriteTimeoutTask(), 30, TimeUnit.SECONDS);
        ...
    }
}
 
public class Main {
    public static void main(String[] args) throws Exception {
        // Run an arbitrary task from an I/O thread.
        Channel ch = ...;
        ch.executor().execute(new Runnable() { ... });
    }
}
```

### Simplified shutdown

There's no more `releaseExternalResources()`.  You can close all open channels immediately and make all I/O threads stop themselves by calling `EventLoopGroup.shutdown()`, just like you shut down your thread pool with `java.util.concurrent.ExecutorService.shutdown()`.

### Type-safe ChannelOptions

There are two ways to configure the socket parameters of a `Channel` in Netty.  One is to call the setters of a `ChannelConfig` explicitly, such as `SocketChannelConfig.setTcpNoDelay(true)`.  It is the most type-safe way.  The other is to call `ChannelConfig.setOption()` method.  Sometimes you have to determine what socket options to configure in runtime, and this method is ideal for such cases.  However, it is error-prone in 3.x because a user has to specify the option as a pair of a string and an object.  If a user calls with the wrong option name or value, he or she will encounter a `ClassCastException` or the specified option might even be ignored silently.

4.0 introduces a new type called `ChannelOption`, which provides type-safe access to socket options.

```java
ChannelConfig cfg = ...;
 
// Before:
cfg.setOption("tcpNoDelay", true);
cfg.setOption("tcpNoDelay", 0);  // Runtime ClassCastException
cfg.setOption("tcpNoDelays", true); // Typo in the option name - ignored silently
 
// After:
cfg.setOption(ChannelOption.TCP_NODELAY, true);
cfg.setOption(ChannelOption.TCP_NODELAY, 0); // Compile error
```

### AttributeMap

In response to user demand, you can attach any object to `Channel` and `ChannelHandlerContext`.  A new interface called `AttributeMap`, which `Channel` and `ChannelHandlerContext` extend, has been added.  Instead, `ChannelLocal` and `Channel.attachment` are removed.  The attributes are garbage-collected when their associated `Channel` is garbage-collected.

```java
public class MyHandler extends ChannelInboundMessageHandlerAdapter<MyMessage> {
 
    private static final AttributeKey<MyState> STATE =
            new AttributeKey<MyState>("MyHandler.state");
 
    @Override
    public void channelRegistered(ChannelHandlerContext ctx) {
        ctx.attr(STATE).set(new MyState());
        ctx.fireChannelRegistered();
    }
 
    @Override
    public void messageReceived(ChannelHandlerContext ctx, MyMessage msg) {
        MyState state = ctx.attr(STATE).get();
    }
    ...
}
```

### New bootstrap API

The bootstrap API has been rewritten from scratch although its purpose stays same; it performs the typical steps required to make a server or a client up and running, often found in boilerplate code.

The new bootstrap also employs a fluent interface.

```java
public static void main(String[] args) throws Exception {
    // Configure the server.
    ServerBootstrap b = new ServerBootstrap();
    try {
        b.group(new NioEventLoopGroup(), new NioEventLoopGroup())
         .channel(new NioServerSocketChannel())
         .option(ChannelOption.SO_BACKLOG, 100)
         .localAddress(8080)
         .childOption(ChannelOption.TCP_NODELAY, true)
         .childHandler(new ChannelInitializer<SocketChannel>() {
             @Override
             public void initChannel(SocketChannel ch) throws Exception {
                 ch.pipeline().addLast(handler1, handler2, ...);
             }
         });
 
        // Start the server.
        ChannelFuture f = b.bind().sync();
 
        // Wait until the server socket is closed.
        f.channel().closeFuture().sync();
    } finally {
        // Shut down all event loops to terminate all threads.
        b.shutdown();
    }
}
```

#### ChannelPipelineFactory → ChannelInitializer

As you noticed in the example above, there is no `ChannelPipelineFactory` anymore.  It has been replaced with `ChannelInitializer`, which gives more control over `Channel` and `ChannelPipeline` configuration.

Please note that you don't create a new `ChannelPipeline` by yourself.  After observing many use cases reported so far, the Netty project team concluded that it has no benefit for a user to create his or her own pipeline implementation or to extend the default implementation.  Therefore, `ChannelPipeline` is not created by a user anymore.  `ChannelPipeline` is automatically created by a `Channel`.

### ChannelFuture split to ChannelFuture and ChannelPromise

`ChannelFuture` has been split into `ChannelFuture` and `ChannelPromise`. This not only makes the contract of consumer and producer of an asynchronous operation explicit, but also make it more safe to use the returned `ChannelFuture` in a chain (like filtering), because the state of the `ChannelFuture` cannot be changed.

Due to this change, some methods now accept `ChannelPromise` rather than `ChannelFuture` to modify its state.

## Well-defined thread model

There is no well-defined thread model in 3.x although there was an attempt to fix its inconsistency in 3.5.  4.0 defines a strict thread model that helps a user write a ChannelHandler without worrying too much about thread safety.

* Netty will never call a `ChannelHandler`'s methods concurrently, unless the `ChannelHandler` is annotated with `@Shareable`. This is regardless of the type of handler methods - inbound, outbound, or life cycle event handler methods.
  * A user does not need to synchronize either inbound or outbound event handler methods anymore.
  * 4.0 disallows adding a `ChannelHandler` more than once unless it's annotated with `@Sharable`.
* There is always [happens-before](http://docs.oracle.com/javase/7/docs/api/java/util/concurrent/package-summary.html#MemoryVisibility) relationship between each `ChannelHandler` method invocations made by Netty.
  * A user does not need to define a `volatile` field to keep the state of a handler.
* A user can specify an `EventExecutor` when he or she adds a handler to a `ChannelPipeline`.
  * If specified, the handler methods of the `ChannelHandler` are always invoked by the specified `EventExecutor`.
  * If unspecified, the handler methods are always invoked by the `EventLoop` that its associated `Channel` is registered to.
* `EventExecutor` and `EventLoop` assigned to a handler or a channel are always single-threaded.
  * The handler methods will always be invoked by the same thread.
  * If multithreaded `EventExecutor` or `EventLoop` is specified, one of the threads will be chosen first and then the chosen thread will be used until deregistration.
  * If two handlers in the same pipeline are assigned with different `EventExecutor`s, they are invoked simultaneously.  A user has to pay attention to thread safety if more than one handler access shared data even if the shared data is accessed only by the handlers in the same pipeline.
* The `ChannelFutureListeners` added to `ChannelFuture` are always invoked by the `EventLoop` thread assigned to the future's associated `Channel`.

### No more ExecutionHandler - it's in the core.

You can specify an `EventExecutor` when you add a `ChannelHandler` to a `ChannelPipeline` to tell the pipeline to always invoke the handler methods of the added `ChannelHandler` via the specified `EventExecutor`.

```java
Channel ch = ...;
ChannelPipeline p = ch.pipeline();
EventExecutor e1 = new DefaultEventExecutor(16);
EventExecutor e2 = new DefaultEventExecutor(8);
 
p.addLast(new MyProtocolCodec());
p.addLast(e1, new MyDatabaseAccessingHandler());
p.addLast(e2, new MyHardDiskAccessingHandler());
```

`EventExecutor` is a supertype of `EventLoop`, and it also extends `ScheduledExecutorService`.

fixme

## Codec framework changes

There were substantial internal changes in the codec framework because 4.0 requires a handler to create and manage its buffer (see Per-handler buffer section in this document.)  However, the changes from a user's perspective are not very big.

* Core codec classes are moved to the `io.netty.handler.codec` package.
* `FrameDecoder` has been renamed to `ByteToMessageDecoder`.
* `OneToOneEncoder` and `OneToOneDecoder` were replaced with `MessageToMessageEncoder` and `MessageToMessageDecoder`.
* The method signatures of `decode()`, `decodeLast()`, `encode()` were changed slightly to support generics and to remove redundant parameters.

### Codec embedder → EmbeddedChannel

Codec embedder has been replaced by `io.netty.channel.embedded.EmbeddedByteChannel` and `EmbeddedMessageChannel`. `EmbeddedChannel` allows a user unit-test any kind of pipeline including a codec.

### HTTP codec

HTTP decoders now always generates multiple message objects per a single HTTP message:

```
1       * HttpRequest / HttpResponse
0 - n   * HttpContent
1       * LastHttpContent
```

For more detail, please refer to the updated `HttpSnoopServer` example.  If you wish not to deal with multiple messages for a single HTTP message, you can put an `HttpObjectAggregator` in the pipeline.  `HttpObjectAggregator` will transform multiple messages into a single `FullHttpRequest` or `FullHttpResponse`.

### Changes in transport implementations

The following transports were newly added:

* AIO socket transport which uses the NIO.2 `AsynchronousSocketChannel`
* OIO SCTP transport
* UDT transport

## Case study: porting the Factorial example

This section shows rough steps to port the Factorial example from 3.x to 4.0.  The Factorial example has been ported to 4.0 already in the `io.netty.example.factorial` package.  Please browse the source code of the example to find every bits changed.

### Porting the server

1. Rewrite `FactorialServer.run()` method to use the new bootstrap API.
  1. No `ChannelFactory` anymore.  Instantiate `NioEventLoop` (one for accepting incoming connections and the other for handling the accepted connections) by yourself.
1. Rename `FactorialServerPipelineFactory` to `FactorialServerInitializer`.
  1. Make it extends `ChannelInitializer<Channel>`.
  1. Instead of creating a new `ChannelPipeline`, get it via `Channel.pipeline()`.
1. Make `FactorialServerHandler` extends `ChannelInboundMessageHandlerAdapter<BigInteger>`.
  1. Replace channelDisconnected() with channelInactive().
  1. handleUpstream() is not used anymore.
1. Make `BigIntegerDecoder` extend `ByteToMessageDecoder<BigInteger>`.
1. Make `NumberEncoder` extend `MessageToByteEncoder<Number>`.
  1. `encode()` does not return a buffer anymore.  Fill the encoded data to the buffer provided by `ByteToMessageDecoder`.

### Porting the client

Mostly same with porting the server, but you need to pay attention when you write a potentially large stream.

1. Rewrite `FactorialClient.run()` method to use the new bootstrap API.
1. Rename `FactorialClientPipelineFactory` to `FactorialClientInitializer`.
1. Make `FactorialClientHandler` extends `ChannelInboundMessageHandlerAdapter<BigInteger>`
  1. At this point, you find there is no `Channel.isWritable()` nor `channelInterestChanged()` in 4.0.  Instead, you maintain the number of pending writes by yourself.  The new `sendNumbers()` could look like the following:

```
    private void sendNumbers() {
        // Do not send more than 4096 numbers.
        boolean finished = false;
        MessageBuf<Object> out = ctx.nextOutboundMessageBuffer();
        while (out.size() < 4096) {
            if (i <= count) {
                out.add(Integer.valueOf(i));
                i ++;
            } else {
                finished = true;
                break;
            }
        }

        ChannelFuture f = ctx.flush();
        if (!finished) {
            f.addListener(numberSender);
        }
    }

    private final ChannelFutureListener numberSender = new ChannelFutureListener() {
        @Override
        public void operationComplete(ChannelFuture future) throws Exception {
            if (future.isSuccess()) {
                sendNumbers();
            }
        }
    };
```