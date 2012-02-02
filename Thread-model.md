To put simply, for a channel:

1. Regardless of its transport and type, its all upstream (i.e. inbound) events must be fired from the thread that performs I/O for the channel (i.e. I/O thread).
1. All downstream (i.e. outbound) events can be triggered from any thread including the I/O thread and non-I/O threads.  However, any upstream events triggered as a side effect of the downstream event must be fired from the I/O thread. (e.g. If `Channel.close()` triggers `channelDisconnected`, `channelUnbound`, and `channelClosed`, they must be fired by the I/O thread.

Current problems (UGLY - causes a race condition in an upstream handler, BAD - does not cause a race condition but violates the expected thread model):

* UGLY: The upstream events triggered as a side effect of a downstream event is triggered by the caller thread, 
* UGLY: The local transport always uses a caller thread to trigger an event.
* BAD: `channelOpen` is triggered by the thread that called `ChannelFactory.newChannel()`, which is not an I/O thread.
* BAD: Client-side channels are run by two I/O threads.  One that makes a connection attempts and the other that does actual I/O.

Action items:

* Merge client-side boss, server-side boss, and NioWorker into a universal I/O thread that can perform all I/O operations.  By doing this:
  * We solve the client-side channel problem because the thread which attempted a connection attempt can continue to perform reads and writes.
  * We solve the problem where Netty creates as many threads as the number of open server ports.
  * We can share a pool of NioWorkers more easily and will potentially have more flexibility in channel-worker mapping.
  * We also need to investigate if we can make an abstract I/O thread class so that all transports (socket, datagram, SCTP, ..) can extend it.  We currently have too much duplication between socket, datagram, and SCTP.
* If the caller thread is not the I/O thread, Netty triggers an upstream event later in the I/O thread.  Along with this change, allow a user to trigger one's own upstream event later in the I/O thread by adding the `sendUpstreamLater()` method to `ChannelPipeline` and `ChannelHandlerContext`.
  * However, we cannot use `sendUpstreamLater()` only if the current thread is not the I/O thread because `OMATPE` or `MATPE` will interfere with it, so we will have to let user decide. (i.e. to call `sendUpstream()` or `sendUpstreamLater()`)
* `ChannelFactory.newChannel()` must not trigger an event immediately.  `newChannel()` must wait until the I/O thread notifies the channel has been registered to the I/O thread, before returning the new channel to the caller.  
* Rewrite the local transport.

Questions:

* Can we make all these changes in v3 and keep things still backward-compatible?  Wouldn't it be easier to get this done in v4?  Fully asynchronous user application which does all I/O in an handler making heavy use of `ChannelFuture` shouldn't be affected by the current flawed thread model, which means a user can somehow work around this issue, so it might be better move on to v4 instead of making the same changes on two branches.