Get started with the user guide and the API reference. Also, [join the community](http://netty.io/community.html) to get more information.

* v 4.0
  * [[User Guide|User guide for 4.x]]
  * [API Reference](http://netty.io/4.0/api/) (Javadoc)
  * [Source Code](http://netty.io/4.0/xref/) (Xref)
  * [[New and Noteworthy]]
* v 3.6
  * [[User Guide|User guide for 3.x]]
  * [API Reference](http://netty.io/3.6/api/) (Javadoc)
  * [Source Code](http://netty.io/3.6/xref/) (Xref)

### 3rd-party Articles

We often find there are people in the world who are much more talented in technical writing than we, and they kindly wrote [[great articles|Related Articles]] for the community.

## Examples (v 4.0)

### UDT

UDT is [UDP-based Data Transfer Protocol](http://en.wikipedia.org/wiki/UDP-based_Data_Transfer_Protocol)

* [Byte Streams](https://github.com/netty/netty/tree/master/example/src/main/java/io/netty/example/udt/echo/bytes) - use UDT in TCP-like bytes streaming mode
* [Message Flows](https://github.com/netty/netty/tree/master/example/src/main/java/io/netty/example/udt/echo/message) - use UDT in UDP-like guaranteed message delivery mode
* [Rendezvous Connect](https://github.com/netty/netty/tree/master/example/src/main/java/io/netty/example/udt/echo/rendevous) - use UDT in symmetric peer-to-peer rendezvous connect mode

## Examples (v 3.6)

There are several examples to help your experience with Netty. It is recommended to start from the first one and to finish to the last one.

### Fundamental

* [Echo](https://github.com/netty/netty/tree/3/src/main/java/org/jboss/netty/example/echo) - the very basic client and server
* [Discard](https://github.com/netty/netty/tree/3/src/main/java/org/jboss/netty/example/discard) - prevent OutOfMemoryError which is caused by writing too fast
* [UpTime](https://github.com/netty/netty/tree/3/src/main/java/org/jboss/netty/example/uptime) - implement reliable client-side reconnection

### Text protocols

* [Telnet](https://github.com/netty/netty/tree/3/src/main/java/org/jboss/netty/example/telnet) - a classic line-based network application
* [Quote](https://github.com/netty/netty/tree/3/src/main/java/org/jboss/netty/example/qotm) of the Moment - broadcast UDP/IP client and server
* [SecureChat](https://github.com/netty/netty/tree/3/src/main/java/org/jboss/netty/example/securechat) - an SSL-based chat server derived from the Telnet example

### Binary protocols

* [ObjectEcho](https://github.com/netty/netty/tree/3/src/main/java/org/jboss/netty/example/objectecho) - exchange serializable Java objects
* [Factorial](https://github.com/netty/netty/tree/3/src/main/java/org/jboss/netty/example/objectecho) - write a stateful client / server based on custom binary protocol
* [LocalTime](https://github.com/netty/netty/tree/3/src/main/java/org/jboss/netty/example/localtime) - rapid protocol prototyping with Google Protocol Buffers integration

### HTTP

* [HTTP (Snoop)](https://github.com/netty/netty/tree/3/src/main/java/org/jboss/netty/example/http/snoop) - build your own extremely light-weight HTTP client and server
* [HTTP (File Server)](https://github.com/netty/netty/tree/3/src/main/java/org/jboss/netty/example/http/file) - asynchronous large file streaming in HTTP
* Web Socket [Client](https://github.com/netty/netty/tree/3/src/main/java/org/jboss/netty/example/http/websocketx/client) & [Server](https://github.com/netty/netty/tree/3/src/main/java/org/jboss/netty/example/http/websocketx/server) - add two-way full-duplex communication channels to HTTP using Web Sockets protocol

### Advanced

* [Proxy Server](https://github.com/netty/netty/tree/3/src/main/java/org/jboss/netty/example/proxy) - write a highly efficient NIO proxy server
* [Port Unification](https://github.com/netty/netty/tree/3/src/main/java/org/jboss/netty/example/portunification) - run heterogenous services listening on a single port
