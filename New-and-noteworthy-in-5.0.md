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

### Even more flexible thread model

In Netty 4.x each [`EventLoop`] is tightly coupled with a fixed thread that executes all I/O events of its registered [`Channels`] and any tasks submitted to it.

Starting with version 5.0 an `EventLoop` does no longer use threads directly but instead makes use of an [`Executor`] abstraction. That is, it takes an `Executor` object as a parameter in its constructor and instead of polling for I/O
events in an endless loop each iteration is now a task that is submitted to this `Executor`.

If not specified else, the `Executor` used by default is a [`ForkJoinPool`]. A `ForkJoinPool` has the nice property of using thread-local queues. That is, a task submitted to a `ForkJoinPool` from `Thread A` is very likely to be executed by `Thread A again`. This should provide `EventLoops` with a high level of thread affinity.

Furthermore, developers may also provide their own `Executor` (aka thread pool) and take over the scheduling of the `EventLoops`. One scenario where this can prove useful is when Netty is used as a part of a large scale software system. Let's say this system already uses a thread pool with a high level of parallelism to optimally perform all its tasks. Netty 4.x would simply spawn its own threads and completely ignore the fact that it's part of a larger system. Starting with Netty 5.0, developers can run Netty and the rest of the system in the same thread pool and potentially improve performance by applying better scheduling strategies and through less scheduling overhead (due to fewer threads). For a detailed discussion of this change please have a look at GitHub issue [2250].

It shall be mentioned, that this change does not in any way affect the way [`ChannelHandlers`] are developed. From a developer's point of view, the only thing that changes is that it's no longer guaranteed that a `ChannelHandler` will always be executed by the same thread. It is, however, guaranteed that it will never be executed by two or more threads at the same time. Furthermore, Netty will also take care of any memory visibility issues that might occur. So there's no need to worry about thread-safety and `volatile` variables within a `ChannelHandler`.

An implication of this change is that [`NioEventLoop`], [`NioEventLoopGroup`], [`EpollEventLoop`] and
[`EpollEventLoopGroup`] do no longer take [`ThreadFactory`] objects as constructor arguments. The constructors of these classes have been updated to take [`Executor`] and [`ExecutorFactory`] objects instead.

### A better `Channel.deregister(...)`

Originally introduced in Netty 4.0, the behaviour of `Channel.deregister(...)` was updated in version 5.0 to be in line
with Netty's thread model.

It's now ensured that all tasks submitted to an [`EventLoop`] from within a [`ChannelHandler`] will have been executed by that
[`EventLoop`] before the [`Channel`] is actually deregistered. However, `Channel.deregister(...)` remains a non blocking operation, so one has to wait for the returned [`ChannelFuture`] to succeed before it's safe to register the `Channel` with another `EventLoop`.

After having called `Channel.deregister(...)` any attempt to submit a new task ([`Runnable`] or [`Callable`]) to the
`EventLoop` from within a `ChannelHandler` will trigger a [`RejectedExecutionException`]. Once the `Channel` is registered with another `EventLoop` again, everything will work as usual.

A task submitted from within `ChannelHandler` via any of the `EventLoop.schedule*(...)` methods will stop execution
after the `Channel` was deregistered from its `EventLoop`. The tasks are automatically moved to the new 
`EventLoop` and will resume execution after the `Channel` is registered again. This limitation only affects tasks
that are scheduled to execute while the `Channel` is in a deregistered state. Tasks who's delay/period is long enough
will not be affected.

Even though it's not recommended, it is possible to overcome the above restrictions. Netty 5.0 introduces a new method 
`EventLoop.unwrap()` that returns the original `EventLoop` that does not perform any sanity checks. More precisely, when submitting tasks to or scheduling tasks with an "unwrapped" `EventLoop`, it cannot be assured that these tasks will not be executed concurrently and scheduled tasks will also not be moved between `EventLoops` automatically.

[`ChannelHandler`]: http://netty.io/5.0/api/io/netty/channel/ChannelHandler.html
[`ChannelHandlerAdapter`]: http://netty.io/5.0/api/io/netty/channel/ChannelHandlerAdapter.html
[`ChannelHandlerAppender`]: http://netty.io/5.0/api/io/netty/channel/ChannelHandlerAppender.html
[`SimpleChannelInboundHandler`]: http://netty.io/5.0/api/io/netty/channel/SimpleChannelInboundHandler.html
[`EventLoop`]: http://netty.io/5.0/api/io/netty/channel/EventLoop.html
[`Executor`]: http://docs.oracle.com/javase/8/docs/api/java/util/concurrent/Executor.html
[`Channels`]: http://netty.io/5.0/api/io/netty/channel/Channel.html
[`ForkJoinPool`]: http://docs.oracle.com/javase/8/docs/api/java/util/concurrent/ForkJoinPool.html
[`ChannelHandlers`]: http://netty.io/5.0/api/io/netty/channel/ChannelHandler.html
[`NioEventLoop`]: http://netty.io/5.0/api/io/netty/channel/nio/NioEventLoop.html
[`NioEventLoopGroup`]: http://netty.io/5.0/api/io/netty/channel/nio/NioEventLoopGroup.html
[`EpollEventLoop`]: http://netty.io/5.0/api/io/netty/channel/epoll/EpollEventLoop.html
[`EpollEventLoopGroup`]: http://netty.io/5.0/api/io/netty/channel/epoll/EpollEventLoopGroup.html
[`ThreadFactory`]: http://docs.oracle.com/javase/8/docs/api/java/util/concurrent/ThreadFactory.html
[`ExecutorFactory`]: http://netty.io/5.0/api/io/netty/concurrent/ExecutorFactory.html
[`Channel`]: http://netty.io/5.0/api/io/netty/channel/Channel.html
[`Runnable`]: http://docs.oracle.com/javase/8/docs/api/java/lang/Runnable.html
[`Callable`]: http://docs.oracle.com/javase/8/docs/api/java/util/concurrent/Callable.html
[`RejectedExecutionException`]: http://docs.oracle.com/javase/8/docs/api/java/util/concurrent/RejectedExecutionException.html
[2250]: https://github.com/netty/netty/issues/2250
[`ChannelFuture`]: http://netty.io/5.0/api/io/netty/channel/ChannelFuture.html