This document walks you through the list of notable changes and new features in the major Netty release to give you an idea to port your application to the new version.

## Project structure changes

The package name of Netty has been changed from `org.jboss.netty` to `io.netty` since [we are not part of JBoss.org anymore](http://netty.io/news/2011/11/04/new-web-site.html).

The binary JAR has been split into multiple submodules so that a user can exclude unnecessary features from the class path.  The current structure looks like this following:

| Artifact ID            | Description                                       
|------------------------|---------------------------------------------------
| `netty-parent`         | Maven parent POM
| `netty-common`         | Utility classes and logging facade
| `netty-buffer`         | `ByteBuf` API that replaces `java.nio.ByteBuffer`
| `netty-transport`      | Channel API and core transports
| `netty-transport-rxtx` | [Rxtx](http://goo.gl/vTFBv) transport
| `netty-transport-sctp` | [SCTP](http://goo.gl/oXxaU) transport
| `netty-transport-udt`  | [UDT](http://udt.sourceforge.net/) transport
| `netty-handler`        | Useful `ChannelHandler` implementations
| `netty-codec`          | Codec framework that helps write an encoder and a decoder
| `netty-codec-http`     | Codecs related with HTTP, Web Sockets, SPDY, and RTSP
| `netty-codec-socks`    | Codecs related with SOCKS protocol
| `netty-all`            | All-in-one JAR that combines all artifacts above
| `netty-tarball`        | Tarball distribution
| `netty-example`        | Examples
| `netty-testsuite-*`    | A collection of integration tests
| `netty-microbench`     | Microbenchmarks

All artifacts (except for `netty-all.jar`) are now OSGi bundles and can be used in your favorite OSGi container.

## General API changes

* Most operations in Netty now support method chaining for brevity.
* Non-configuration getters have no `get-` prefix anymore. (e.g. `Channel.getRemoteAddress()` → `Channel.remoteAddress()`)
  * Boolean properties are still prefixed with `is-` to avoid confusion (e.g. 'empty' is both an article and a verb, so `empty()` can have two meanings.)
* For API changes between 4.0 CR4 and 4.0 CR5 see [Netty 4.0.0.CR5 released with new-new API](http://netty.io/news/2013/06/18/4-0-0-CR5.html)

## Buffer API changes

### `ChannelBuffer` → `ByteBuf`

Thanks to the structural changes mentioned above, the buffer API can be used as a separate package.  Even if you are not interested in adopting Netty as a network application framework, you can still use our buffer API.  Therefore, the type name `ChannelBuffer` does not make sense anymore, and has been renamed to `ByteBuf`.

The utility class `ChannelBuffers`, which creates a new buffer, has been split into two utility classes, `Unpooled` and `ByteBufUtil`.  As can be guessed from its name `Unpooled`, 4.0 introduced pooled `ByteBuf`s which can be allocated via `ByteBufAllocator` implementations.

### `ByteBuf` is not an interface but an abstract class

According to our internal performance test, converting `ByteBuf` from an interface to an abstract class improved the overall throughput around 5%.

### Most buffers are dynamic with maximum capacity

In 3.x, buffers were fixed or dynamic.  The capacity of a fixed buffer does not change once it is created while the capacity of a dynamic buffer changes whenever its `write*(...)` method requires more space.

Since 4.0, all buffers are dynamic.  However, they are better than the old dynamic buffers.  You can decrease or increase the capacity of a buffer more easily and more safely.  It's easy because there is a new method `ByteBuf.capacity(int newCapacity)`. It's safe because you can set the maximum capacity of a buffer so that it does not grow boundlessly.

```java
// No more dynamicBuffer() - use buffer().
ByteBuf buf = Unpooled.buffer();

// Increase the capacity of the buffer.
buf.capacity(1024);
...

// Decrease the capacity of the buffer (the last 512 bytes are deleted.)
buf.capacity(512);
```

The only exception is the buffer which wraps a single buffer or a single byte array, created by `wrappedBuffer()`.  You cannot increase its capacity because it invalidates the whole point of wrapping an existing buffer - saving memory copies.  If you want to change the capacity after you wrap a buffer, you should just create a new buffer with enough capacity and copy the buffer you wanted to wrap.

### New buffer type: `CompositeByteBuf`

A new buffer implementation named `CompositeByteBuf` defines various advanced operations for composite buffer implementations.  A user can save bulk memory copy operations using a composite buffer at the cost of relatively expensive random access.  To create a new composite buffer, use either `Unpooled.wrappedBuffer(...)` like before, `Unpooled.compositeBuffer(...)`, or `ByteBufAllocator.compositeBuffer()`.

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
### Pooled buffers

Netty 4 introduces a high-performance buffer pool which is a variant of [jemalloc](http://www.canonware.com/jemalloc/) that combines [buddy allocation](http://en.wikipedia.org/wiki/Buddy_memory_allocation) and [slab allocation](http://en.wikipedia.org/wiki/Slab_allocation). It gives the following benefits:

* Reduced GC pressure incurred by frequent allocation and deallocation of the buffers
* Reduced memory bandwidth consumption incurred when creating a new buffer which  inevitably has to be filled with zeroes
* Timely deallocation of direct buffers

To take advantage of this feature, unless a user wants to get an unpooled buffer, he or she should get a buffer from a [`ByteBufAllocator`](http://netty.io/4.0/api/index.html?io/netty/buffer/AbstractByteBufAllocator.html):

```java
Channel channel = ...;
ByteBufAllocator alloc = channel.alloc();
ByteBuf buf = alloc.buffer(512);
....
channel.write(buf);
 
ChannelHandlerContext ctx = ...
ByteBuf buf2 = ctx.alloc().buffer(512);
....
channel.write(buf2)
```

Once a `ByteBuf` is written to the remote peer it will be returned automatically to the pool it originated from.

The default `ByteBufAllocator` is `PooledByteBufAllocator`. If you do not wish to use buffer pooling or use your own allocator, use `Channel.config().setAllocator(...)` with an alternative allocator such as `UnpooledByteBufAllocator`.

NOTE: At the moment, the default allocator is `UnpooledByteBufAllocator`.  Once we ensure there's no memory leak in `PooledByteBufAllocator`, we will default back again to it.

#### `ByteBuf` is always reference counted

To control the life cycle of a `ByteBuf` in a more predictable way, Netty does not rely on the garbage collector anymore but employs an explicit reference counter.  Here's the basic rule:

* When a buffer is allocated, its initial reference count is 1.
* If the reference count of the buffer is decreased to 0, it is deallocated or returned to the pool it originated from.
* The following attempts trigger an `IllegalReferenceCountException`:
  * Accessing a buffer whose reference count is 0,
  * Decreasing the reference count to a negative value, or
  * Increasing the reference count beyond `Integer.MAX_VALUE`.
* Derived buffers (e.g. slices and duplicates) and swapped buffers (i.e. little endian buffers) share the reference count with the buffer it was derived from.  Note that the reference count does not change when a derived buffer is created.

When a `ByteBuf` is used in a `ChannelPipeline`, there are additional rules you need to keep in mind:

* Each inbound (a.k.a. upstream) handler in a pipeline has to release the received messages. Netty does not release them automatically for you.
  * Note that codec framework does release the messages automatically and a user has to increase the reference count if he or she wants to pass a message as-is to the next handler.
* When an outbound (a.k.a. downstream) message reaches at the beginning of the pipeline, Netty will release it after writing it out.

#### Automatic buffer leak detection

Although reference counting is very powerful, it is also error-prone.  To help a user find where he or she forgot to release the buffers, the leak detector logs the stack trace of the location where the leaked buffer was allocated automatically.

Because the leak detector relies on `PhantomReference` and obtaining a stack trace is a very expensive operation, it samples approximately 1% of allocations only.  Therefore, it's a good idea to run the application for a reasonably long time to find all possible leaks.

Once all leaks are found and fixed.  You can turn this feature off to remove its runtime overhead completely by specifying the `-Dio.netty.noResourceLeakDetection` JVM option.

## `io.netty.util.concurrent`

Along with the new standalone buffer API, 4.0 provides various constructs which are useful for writing asynchronous applications in general at the new package called `io.netty.util.concurrent`.  Some of those constructs are:

* `Future` and `Promise` - similar to `ChannelFuture`, but has no dependency to `Channel`
* `EventExecutor` and `EventExecutorGroup` - generic event loop API

They are used as the base of the channel API which will be explained later in this document.  For example, `ChannelFuture` extends `io.netty.util.concurrent.Future` and `EventLoopGroup` extends `EventExecutorGroup`.

![Event loop type hierarchy diagram](http://img.motd.kr/uml/gist/8a6c3c47800370ee898a "")

## Channel API changes

In 4.0, many classes under the `io.netty.channel` package have gone through a major overhaul, and thus simple text search-and-replace will not make your 3.x application work with 4.0.  This section will try to show the thought process behind such a big change, rather than being an exhaustive resource for all the changes.

### Revamped ChannelHandler interface
#### Upstream → Inbound, Downstream → Outbound

The terms 'upstream' and 'downstream' were pretty confusing to beginners.  4.0 uses 'inbound' and 'outbound' wherever possible.

#### New `ChannelHandler` type hierarchy

In 3.x, `ChannelHandler` was just a tag interface, and `ChannelUpstreamHandler`, `ChannelDownstreamHandler`, and `LifeCycleAwareChannelHandler` defined the actual handler methods.  In Netty 4, `ChannelHandler` merges `LifeCycleAwareChannelHandler` along with a couple more methods which are useful to both an inbound and an outbound handler:

```java
public interface ChannelHandler {
    void handlerAdded(ChannelHandlerContext ctx) throws Exception;
    void handlerRemoved(ChannelHandlerContext ctx) throws Exception;
    void exceptionCaught(ChannelHandlerContext ctx, Throwable cause) throws Exception;
}
```

The following diagram depicts the new type hierarchy:

![ChannelHandler type hierarchy diagram](http://img.motd.kr/uml/gist/188244c4b3d6b01c0156)

#### `ChannelHandler` with no event object

In 3.x, every I/O operation created a `ChannelEvent` object.  For each read / write, it additionally created a new `ChannelBuffer`.  It simplified the internals of Netty quite a lot because it delegates resource management and buffer pooling to the JVM.  However, it often was the root cause of GC pressure and uncertainty which are sometimes observed in a Netty-based application under high load.

4.0 removes event object creation almost completely by replacing the event objects with strongly typed method invocations.  3.x had catch-all event handler methods such as `handleUpstream()` and `handleDownstream()`, but this is not the case anymore.  Every event type has its own handler method now:

```java
// Before:
void handleUpstream(ChannelHandlerContext ctx, ChannelEvent e);
void handleDownstream(ChannelHandlerContext ctx, ChannelEvent e);
 
// After:
void channelRegistered(ChannelHandlerContext ctx);
void channelUnregistered(ChannelHandlerContext ctx);
void channelActive(ChannelHandlerContext ctx);
void channelInactive(ChannelHandlerContext ctx);
void channelRead(ChannelHandlerContext ctx, Object message);
 
void bind(ChannelHandlerContext ctx, SocketAddress localAddress, ChannelPromise promise);
void connect(
        ChannelHandlerContext ctx, SocketAddress remoteAddress,
        SocketAddress localAddress, ChannelPromise promise);
void disconnect(ChannelHandlerContext ctx, ChannelPromise promise);
void close(ChannelHandlerContext ctx, ChannelPromise promise);
void deregister(ChannelHandlerContext ctx, ChannelPromise promise);
void write(ChannelHandlerContext ctx, Object message, ChannelPromise promise);
void flush(ChannelHandlerContext ctx);
void read(ChannelHandlerContext ctx);
```

`ChannelHandlerContext` has also been changed to reflect the changes mentioned above:

```java
// Before:
ctx.sendUpstream(evt);
 
// After:
ctx.fireChannelRead(receivedMessage);
```

All these changes mean a user cannot extend the non-existing `ChannelEvent` interface anymore.  How then does a user define his or her own event type such as `IdleStateEvent`?  `ChannelInboundHandler` in 4.0 has a handler method called `userEventTriggered()` which is dedicated to this specific user case.

#### Simplified channel state model

When a new connected `Channel` is created in 3.x, at least three `ChannelStateEvent`s are triggered: `channelOpen`, `channelBound`, and `channelConnected`.  When a `Channel` is closed, at least 3 more: `channelDisconnected`, `channelUnbound`, and `channelClosed`.

![Netty 3 Channel state diagram](http://img.motd.kr/uml/gist/4335d63c530b6e1c5e2e)

However, it's of dubious value to trigger that many events.  It is more useful for a user to get notified when a `Channel` enters the state where it can perform reads and writes.

![Netty 4 Channel state diagram](http://img.motd.kr/uml/gist/6effb68ad2515ca0d618)

`channelOpen`, `channelBound`, and `channelConnected` have been merged to `channelActive`.  `channelDisconnected`, `channelUnbound`, and `channelClosed` have been merged to `channelInactive`.  Likewise, `Channel.isBound()` and `isConnected()` have been merged to `isActive()`.

Note that `channelRegistered` and `channelUnregistered` are not equivalent to `channelOpen` and `channelClosed`.  They are new states introduced to support dynamic registration, deregistration, and re-registration of a `Channel`, as illustrated below:

![Netty 4 Channel state diagram for re-registration](http://img.motd.kr/uml/gist/6382530f7890b9f16472)

##### `write()` does not flush automatically

4.0 introduced a new operation called `flush()` which explicitly flushes the outbound buffer of a `Channel`, and `write()` operation does not flush automatically.  You can think of this as a `java.io.BufferedOutputStream`, except that it works at message level.

Because of this change, you must be very careful not to forget to call `ctx.flush()` after writing something.  Alternatively, you could use a shortcut method `writeAndFlush()`.

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
public class MyHandler extends ChannelOutboundHandlerAdapter {
    ...
    public void write(ChannelHandlerContext ctx, Object msg, ChannelPromise p) {
        ...
        ctx.write(msg, p);
        
        // Schedule a write timeout.
        ctx.executor().schedule(new MyWriteTimeoutTask(p), 30, TimeUnit.SECONDS);
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

There's no more `releaseExternalResources()`.  You can close all open channels immediately and make all I/O threads stop themselves by calling `EventLoopGroup.shutdownGracefully()`.

### Type-safe `ChannelOption`

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
public class MyHandler extends ChannelInboundHandlerAdapter {
 
    private static final AttributeKey<MyState> STATE =
            new AttributeKey<MyState>("MyHandler.state");
 
    @Override
    public void channelRegistered(ChannelHandlerContext ctx) {
        ctx.attr(STATE).set(new MyState());
        ctx.fireChannelRegistered();
    }
 
    @Override
    public void channelRead(ChannelHandlerContext ctx, Object msg) {
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
    EventLoopGroup bossGroup = new NioEventLoopGroup();
    EventLoopGroup workerGroup = new NioEventLoopGroup();
    try {
        ServerBootstrap b = new ServerBootstrap();
        b.group(bossGroup, workerGroup)
         .channel(NioServerSocketChannel.class)
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
        bossGroup.shutdownGracefully();
        workerGroup.shutdownGracefully();
        
        // Wait until all threads are terminated.
        bossGroup.terminationFuture().sync();
        workerGroup.terminationFuture().sync();
    }
}
```

#### `ChannelPipelineFactory` → `ChannelInitializer`

As you noticed in the example above, there is no `ChannelPipelineFactory` anymore.  It has been replaced with `ChannelInitializer`, which gives more control over `Channel` and `ChannelPipeline` configuration.

Please note that you don't create a new `ChannelPipeline` by yourself.  After observing many use cases reported so far, the Netty project team concluded that it has no benefit for a user to create his or her own pipeline implementation or to extend the default implementation.  Therefore, `ChannelPipeline` is not created by a user anymore.  `ChannelPipeline` is automatically created by a `Channel`.

### `ChannelFuture` → `ChannelFuture` and `ChannelPromise`

`ChannelFuture` has been split into `ChannelFuture` and `ChannelPromise`. This not only makes the contract of consumer and producer of an asynchronous operation explicit, but also makes it more safe to use the returned `ChannelFuture` in a chain (like filtering), because the state of the `ChannelFuture` cannot be changed.

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

### No more `ExecutionHandler` - it's in the core.

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

## Codec framework changes

There were substantial internal changes in the codec framework because 4.0 requires a handler to create and manage its buffer (see Per-handler buffer section in this document.)  However, the changes from a user's perspective are not very big.

* Core codec classes are moved to the `io.netty.handler.codec` package.
* `FrameDecoder` has been renamed to `ByteToMessageDecoder`.
* `OneToOneEncoder` and `OneToOneDecoder` were replaced with `MessageToMessageEncoder` and `MessageToMessageDecoder`.
* The method signatures of `decode()`, `decodeLast()`, `encode()` were changed slightly to support generics and to remove redundant parameters.

### Codec embedder → `EmbeddedChannel`

Codec embedder has been replaced by `io.netty.channel.embedded.EmbeddedChannel` to allow a user to test any kind of pipeline including a codec.

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

* OIO SCTP transport
* UDT transport

## Case study: porting the Factorial example

This section shows rough steps to port the Factorial example from 3.x to 4.0.  The Factorial example has been ported to 4.0 already in the `io.netty.example.factorial` package.  Please browse the source code of the example to find every bits changed.

### Porting the server

1. Rewrite `FactorialServer.run()` method to use the new bootstrap API.
  1. No `ChannelFactory` anymore.  Instantiate `NioEventLoopGroup` (one for accepting incoming connections and the other for handling the accepted connections) by yourself.
1. Rename `FactorialServerPipelineFactory` to `FactorialServerInitializer`.
  1. Make it extends `ChannelInitializer<Channel>`.
  1. Instead of creating a new `ChannelPipeline`, get it via `Channel.pipeline()`.
1. Make `FactorialServerHandler` extends `ChannelInboundHandlerAdapter`.
  1. Replace `channelDisconnected()` with `channelInactive()`.
  1. handleUpstream() is not used anymore.
  1. Rename `messageReceived()` into `channelRead()` and adjust the method signature accordingly.
  1. Replace `ctx.write()` with `ctx.writeAndFlush()`.
1. Make `BigIntegerDecoder` extend `ByteToMessageDecoder<BigInteger>`.
1. Make `NumberEncoder` extend `MessageToByteEncoder<Number>`.
  1. `encode()` does not return a buffer anymore.  Fill the encoded data to the buffer provided by `ByteToMessageDecoder`.

### Porting the client

Mostly same with porting the server, but you need to pay attention when you write a potentially large stream.

1. Rewrite `FactorialClient.run()` method to use the new bootstrap API.
1. Rename `FactorialClientPipelineFactory` to `FactorialClientInitializer`.
1. Make `FactorialClientHandler` extends `ChannelInboundHandlerAdapter`