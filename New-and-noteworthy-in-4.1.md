This document walks you through the list of notable changes and new features between Netty 4.1 and 4.0.

## TL;DR

Although we did our best to keep the backward compatibility from 4.0, 4.1 contains multiple additions which might not be fully backward-compatible with 4.0. Please make sure to recompile your application against the new version.

When you recompile your application, you might find some deprecation warnings. Please make sure to fix them all by using the suggested alternative, so that you have less trouble when upgrading to the next version.

## Core changes

### Android support

Given that:

* mobile devices are becoming more and more powerful,
* the most known issues with NIO and SSLEngine in ADK were fixed since Ice Cream Sandwich, and
* users obviously want to reuse their codecs and handlers in their mobile applications,

we decided to support Android (4.0 or above) officially.

However, we do not have an automated testsuite for Android yet.  If you find any issues with Android, please feel free to [file an issue](https://github.com/netty/netty/issues).  Please also consider contributing to the project to make Android tests a part of the build process.

### `ChannelHandlerContext.attr(..)` == `Channel.attr(..)`

Both [`Channel`] and [`ChannelHandlerContext`] implement the interface [`AttributeMap`] to enable a user to attach one or more user-defined attributes to them.  What sometimes made a user confused was that a [`Channel`] and a [`ChannelHandlerContext`] had its own storage for the user-defined attributes.  For example, even if you put an attribute 'KEY_X' via `Channel.attr(KEY_X).set(valueX)`, you will never find it via `ChannelHandlerContext.attr(KEY_X).get()` and vice versa.  This behavior is not only confusing but also is waste of memory.

To address this issue, we decided to keep only one map per [`Channel`] internally.  [`AttributeMap`] always uses [`AttributeKey`] as its key. [`AttributeKey`] ensures uniqueness between each key, and thus there's no point of having more than one attribute map per [`Channel`].  As long as a user defines its own [`AttributeKey`] as a private static final field of his or her [`ChannelHandler`], there will be no risk of duplicate keys.

### `Channel.hasAttr(...)`

It is now possible to check if an attribute exists or not efficiently.

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

### `ByteBufAllocator.calculateNewCapacity()`

The logic that calculates the new capacity of the expanded `ByteBuf` has been moved from `AbstractByteBuf` to `ByteBufAllocator` because `ByteBufAllocator` knows better about the capacity calculation of the buffers it manages.

## New codecs and handlers

* Binary memcache protocol codec
* Compression codecs
  * BZip2
  * FastLZ
  * LZ4
  * LZF
* DNS protocol codec
* HAProxy protocol codec
* MQTT protocol codec
* SPDY/3.1 support
* STOMP codec
* SOCKSx codec that supports version 4, 4a, and 5; see `socksx` package.
* [`XmlFrameDecoder`] that enables streaming of XML documents.
* [`JsonObjectDecoder`] that enables streaming of JSON objects.
* IP filtering handlers

## Other codec changes

### `AsciiString`

[`AsciiString`] is a new `CharSequence` implementation which contains only 1-byte characters.  You will find this class useful when you deal with a US-ASCII or ISO-8859-1 string.

For instance, the HTTP codec and STOMP codec in Netty use `AsciiString` to represent the header names.  Because `AsciiString` does not have any conversion cost when encoding it into a `ByteBuf`, it guarantees better performance than using `String`.

### `TextHeaders`

[`TextHeaders`] provides a generic data structure for HTTP header-like string [multimaps](http://en.wikipedia.org/wiki/Multimap).  `HttpHeaders` also has been rewritten using `TextHeaders`.

### `MessageAggregator`

[`MessageAggregator`] provides generic functionality that aggregates multiple small messages into a larger one, just like `HttpObjectAggregator` does.  `HttpObjectAggregator` also has been rewritten using `MessageAggregator`.

### Better oversized message handling with `HttpObjectAggregator`

In 4.0, there was no way to reject an oversized HTTP message before a client sends the content even if 100-continue header was present.

This release adds an overridable method called `handleOversizedMessage` so that a user can perform his or her preferred task.  By default, it responds with '413 Request Entity Too Large' response and closed the connection.

### `ChunkedInput` and `ChunkedWriteHandler`

[`ChunkedInput`] has two new methods; `progress()` and `length()` which return the progress of its transfer and the total length of the stream respectively.  [`ChunkedWriteHandler`] uses this information to notify [`ChannelProgressiveFutureListener`].

### `SnappyFramedEncoder` and `SnappyFramedDecoder`

These two classes have been renamed to `SnappyFrameEncoder` and `SnappyFrameDecoder`. The old classes were marked as deprecated and they are actually the subclasses of the new ones.

[`AttributeKey`]: http://netty.io/4.1/api/io/netty/util/AttributeKey.html
[`AttributeMap`]: http://netty.io/4.1/api/io/netty/util/AttributeMap.html
[`EventExecutor`]: http://netty.io/4.1/api/io/netty/util/concurrent/EventExecutor.html

[`Channel`]: http://netty.io/4.1/api/io/netty/channel/Channel.html
[`ChannelHandler`]: http://netty.io/4.1/api/io/netty/channel/ChannelHandler.html
[`ChannelHandlerAdapter`]: http://netty.io/4.1/api/io/netty/channel/ChannelHandlerAdapter.html
[`ChannelHandlerContext`]: http://netty.io/4.1/api/io/netty/channel/ChannelHandlerContext.html
[`ChannelHandlerInvoker`]: http://netty.io/4.1/api/io/netty/channel/ChannelHandlerInvoker.html
[`ChannelPipeline`]: http://netty.io/4.1/api/io/netty/channel/ChannelPipeline.html

[`SimpleChannelInboundHandler`]: http://netty.io/4.1/api/io/netty/channel/SimpleChannelInboundHandler.html

[`EmbeddedChannel`]: http://netty.io/4.1/api/io/netty/channel/embedded/EmbeddedChannel.html

[`JsonObjectDecoder`]: http://netty.io/4.1/api/io/netty/handler/codec/json/JsonObjectDecoder.html
[`XmlFrameDecoder`]: http://netty.io/4.1/api/io/netty/handler/codec/xml/XmlFrameDecoder.html

[`AsciiString`]: http://netty.io/4.1/api/io/netty/handler/codec/AsciiString.html
[`TextHeaders`]: http://netty.io/4.1/api/io/netty/handler/codec/TextHeaders.html
[`MessageAggregator`]: http://netty.io/4.1/api/io/netty/handler/codec/MessageAggregator.html

[`ChunkedInput`]: http://netty.io/4.1/api/io/netty/handler/stream/ChunkedInput.html
[`ChunkedWriteHandler`]: http://netty.io/4.1/api/io/netty/handler/stream/ChunkedWriteHandler.html
[`ChannelProgressiveFutureListener`]: http://netty.io/4.1/api/io/netty/channel/ChannelProgressiveFutureListener.html
