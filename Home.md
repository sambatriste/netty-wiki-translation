Get started with the user guide and the API reference. Also, [join the community](http://netty.io/community.html) to get more information.

* 4.0
  * [[User Guide|User guide for 4.x]]
  * [API Reference](http://netty.io/3.6/api/) (Javadoc)
  * [Source Code](http://netty.io/3.6/xref/) (Xref)
  * [[New and Noteworthy]]
* 3.6
  * [[User Guide|User guide for 3.x]]
  * [API Reference](http://netty.io/3.6/api/) (Javadoc)
  * [Source Code](http://netty.io/3.6/xref/) (Xref)

### 3rd-party Articles

We often find there are people in the world who are much more talented in technical writing than we, and they kindly wrote [[great articles|Related Articles]] for the community.

## Examples

There are several examples to help your experience with Netty. It is recommended to start from the first one and to finish to the last one.

### Fundamental

* [Echo](http://netty.io/3.6/xref/org/jboss/netty/example/echo/package-summary.html) - the very basic client and server
* [Discard](http://netty.io/3.6/xref/org/jboss/netty/example/discard/package-summary.html) - prevent OutOfMemoryError which is caused by writing too fast
* [UpTime](http://netty.io/3.6/xref/org/jboss/netty/example/uptime/package-summary.html) - implement reliable client-side reconnection

### Text protocols

* [Telnet](http://netty.io/3.6/xref/org/jboss/netty/example/telnet/package-summary.html) - a classic line-based network application
* [Quote](http://netty.io/3.6/xref/org/jboss/netty/example/qotm/package-summary.html) of the Moment - broadcast UDP/IP client and server
* [SecureChat](http://netty.io/3.6/xref/org/jboss/netty/example/securechat/package-summary.html) - an SSL-based chat server derived from the Telnet example

### Binary protocols

* [ObjectEcho](http://netty.io/3.6/xref/org/jboss/netty/example/objectecho/package-summary.html) - exchange serializable Java objects
* [Factorial](http://netty.io/3.6/xref/org/jboss/netty/example/objectecho/package-summary.html) - write a stateful client / server based on custom binary protocol
* [LocalTime](http://netty.io/3.6/xref/org/jboss/netty/example/localtime/package-summary.html) - rapid protocol prototyping with Google Protocol Buffers integration

### HTTP

* [HTTP (Snoop)](http://netty.io/3.6/xref/org/jboss/netty/example/http/snoop/package-summary.html) - build your own extremely light-weight HTTP client and server
* [HTTP (File Server)](http://netty.io/3.6/xref/org/jboss/netty/example/http/file/package-summary.html) - asynchronous large file streaming in HTTP
* Web Socket [Client](http://netty.io/3.6/xref/org/jboss/netty/example/http/websocketx/client/package-summary.html) & [Server](http://netty.io/3.6/xref/org/jboss/netty/example/http/websocketx/server/package-summary.html) - add two-way full-duplex communication channels to HTTP using Web Sockets protocol

### Advanced

* [Proxy Server](http://netty.io/3.6/xref/org/jboss/netty/example/proxy/package-summary.html) - write a highly efficient NIO proxy server
* [Port Unification](http://netty.io/3.6/xref/org/jboss/netty/example/portunification/package-summary.html) - run heterogenous services listening on a single port
