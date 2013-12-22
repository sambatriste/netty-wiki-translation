This document walks you through the list of notable changes and new features in the major Netty release to give you an idea to port your application to the new version.

Unlike the changes between 3.x and 4.x, 5.x did not change a lot although it made quite a bit of breakthrough in its design simplicity.  We tried to make the transition from 4.x to 5.x as smooth as possible, but please let us know if you encounter any issues during migration.

## Core changes

### Simplified handler type hierarchy

`ChannelInboundHandler` and `ChannelOutboundHandler` have been merged into [`ChannelHandler`].  [`ChannelHandler`] now has both inbound and outbound handler methods.

`ChannelInboundHandlerAdapter`, `ChannelOutboundHandlerAdapter`, and `ChannelDuplexHandlerAdapter` have been deprecated and replaced by [`ChannelHandlerAdapter`].

Because it is now impossible to tell if a handler is an inbound handler or an outbound handler, `CombinedChannelDuplexHandler` has been replaced by [`ChannelHandlerAppender`].

For more information about this change, please refer to [the pull request #1999](https://github.com/netty/netty/pull/1999).

### `channelRead0()` → `messageReceived()`

I know.  It was [a silly mistake](https://github.com/netty/netty/issues/1590).  If you are using [`SimpleChannelInboundHandler`], you have to rename `channelRead0()` to `messageReceived()`.

### Removal of deregistration

`Channel.deregister()` has been removed.  It was neither working nor used by users.  Instead, we are going to allow the re-registration of a [`Channel`] to a different event loop in the future.

### Easier and more precise buffer leak tracking

Previously, it was not easy to find where the buffer leak occurred, and the leak warning was not very helpful.  We now have an advanced leak reporting mechanism which can be enabled at the cost of increased overhead.

See [[Reference counted objects]] for more information.  This feature also has been backported to 4.0.14.Final due to its importance.

### `PooledByteBufAllocator` as the default allocator

In 4.x, `UnpooledByteBufAllocator` was the default allocator in spite of its limitation.  Now that `PooledByteBufAllocator` has been in the wild for a while and we have advanced buffer leak tracking mechanism, it is time to make it a new default.

### Globally unique channel ID

Every [`Channel`] now has a globally unique ID which is generated from:

* MAC address (EUI-48 or EUI-64), preferrably a globally unique one,
* the current process ID,
* `System#currentTimeMillis()`
* `System#nanoTime()`
* a random 32-bit integer, and
* a sequentially incremented 32-bit integer.

The ID of a [`Channel`] can be obtained using the `Channel.id()` method.

### Even more flexible thread model

A new interface called [`ChannelHandlerInvoker`] has been added to give a user much more control over which thread should call an event handler method.  Instead of specifying an [`EventExecutor`] when adding a [`ChannelHandler`] to a [`ChannelPipeline`], specify a custom [`ChannelHandlerInvoker`] implementation to take advantage of this feature.

For more information about this change, please refer to [the commit 132af3a](https://github.com/netty/netty/commit/132af3a485015ff912bd567a47881814d2ce1828).

### `EmbeddedChannel` usability

`readInbound()` and `readOutbound()` in [`EmbeddedChannel`] return an ad-hoc type parameter so you do not need to downcast their return values.  This will reduce the verbosity of your unit test code quite a bit.

```java
EmbeddedChannel ch = ...;

// BEFORE:
FullHttpRequest req = (FullHttpRequest) ch.readInbound();

// AFTER:
FullHttpRequest req = ch.readInbound();
```

### Ability to use `Executor` instead of `ThreadFactory`

Some applications require a user to run his or her task in a given `Executor`.  4.x required a user to specify `ThreadFactory` when creating an event loop, but not anymore.

For more information about this change, please refer to [the pull request #1762](https://github.com/netty/netty/pull/1762).

### Class loader friendliness

Some types such as `AttributeKey` were unfriendly to the applications that run in a container environment, but not anymore.

## Codecs and handlers

* [`XmlFrameDecoder`] that enables streaming of XML documents.
* Binary memcache protocol codec
* SPDY/3.1 support (also backported to 4.x)
* Refactored HTTP multipart codec

[`EventExecutor`]: http://netty.io/5.0/api/io/netty/util/concurrent/EventExecutor.html

[`Channel`]: http://netty.io/5.0/api/io/netty/channel/Channel.html
[`ChannelHandler`]: http://netty.io/5.0/api/io/netty/channel/ChannelHandler.html
[`ChannelHandlerAdapter`]: http://netty.io/5.0/api/io/netty/channel/ChannelHandlerAdapter.html
[`ChannelHandlerAppender`]: http://netty.io/5.0/api/io/netty/channel/ChannelHandlerAppender.html
[`ChannelHandlerInvoker`]: http://netty.io/5.0/api/io/netty/channel/ChannelHandlerInvoker.html
[`ChannelPipeline`]: http://netty.io/5.0/api/io/netty/channel/ChannelPipeline.html

[`SimpleChannelInboundHandler`]: http://netty.io/5.0/api/io/netty/channel/SimpleChannelInboundHandler.html

[`EmbeddedChannel`]: http://netty.io/5.0/api/io/netty/channel/embedded/EmbeddedChannel.html

[`LoggingHandler`]: http://netty.io/5.0/api/io/netty/handler/logging/LoggingHandler.html
[`XmlFrameDecoder`]: http://netty.io/5.0/api/io/netty/handler/codec/xml/XmlFrameDecoder.html