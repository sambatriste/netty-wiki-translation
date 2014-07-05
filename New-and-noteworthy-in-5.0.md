This document walks you through the list of notable changes and new features in the major Netty release (since 4.1) to give you an idea to port your application to the new version.

Unlike the changes between 3.x and 4.0, 5.0 did not change a lot although it made quite a bit of breakthrough in its design simplicity.  We tried to make the transition from 4.x to 5.0 as smooth as possible, but please let us know if you encounter any issues during migration.

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

[`ChannelHandler`]: http://netty.io/5.0/api/io/netty/channel/ChannelHandler.html
[`ChannelHandlerAdapter`]: http://netty.io/5.0/api/io/netty/channel/ChannelHandlerAdapter.html
[`ChannelHandlerAppender`]: http://netty.io/5.0/api/io/netty/channel/ChannelHandlerAppender.html

[`SimpleChannelInboundHandler`]: http://netty.io/5.0/api/io/netty/channel/SimpleChannelInboundHandler.html
