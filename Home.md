<div id="wiki-notoc" class="bookpromo well">
<ul class="nav nav-list">
<li class="nav-header">Need a Book?</li>
<li><a href="http://www.manning.com/maurer/"><img src="https://github.com/netty/netty/wiki/images/netty-in-action.gif" alt="Book: Netty in Action"></a></li>
<li><a href="http://www.manning.com/maurer/">Use code <strong>13netco</strong> for a 37% discount!</a></li>
</ul>
</div>

Get started with the user guide and the API reference. Also, [join the community](http://netty.io/community.html) to get more information.

* Version 4.0
  * [[User Guide|User guide for 4.x]]
  * [API Reference](http://netty.io/4.0/api/) (Javadoc)
  * [Source Code](http://netty.io/4.0/xref/) (Xref)
  * [[New and Noteworthy]]
* Version 3.6
  * [[User Guide|User guide for 3.x]]
  * [API Reference](http://netty.io/3.6/api/) (Javadoc)
  * [Source Code](http://netty.io/3.6/xref/) (Xref)

### 3rd-party Articles

We often find there are people in the world who are much more talented in technical writing than we, and they kindly wrote [[great articles|Related Articles]] for the community.

## Examples

There are several examples to help your experience with Netty. It is recommended to start from the first one and to finish to the last one.  You also might want to browse the complete list of examples ([3.6](https://github.com/netty/netty/tree/3/src/main/java/org/jboss/netty/example) and [4.0](https://github.com/netty/netty/tree/master/example/src/main/java/io/netty/example).)

<div class="tabbable">
<ul class="nav nav-tabs">
<li class="active"><a href="#wiki-example-3">3.6</a></li>
<li><a href="#wiki-example-4">4.0</a></li>
</ul>
<div class="tab-content">
<div class="tab-pane active" id="wiki-example-3">
<h3>Examples for version 3.6</h3>
<h4>Fundamental</h4>
<ul>
<li>
<a href="https://github.com/netty/netty/tree/3/src/main/java/org/jboss/netty/example/echo">Echo</a> - the very basic client and server</li>
<li>
<a href="https://github.com/netty/netty/tree/3/src/main/java/org/jboss/netty/example/discard">Discard</a> - prevent <tt>OutOfMemoryError</tt> which is caused by writing too fast</li>
<li>
<a href="https://github.com/netty/netty/tree/3/src/main/java/org/jboss/netty/example/uptime">UpTime</a> - implement reliable client-side reconnection</li>
</ul>
<h4>Text protocols</h4>
<ul>
<li>
<a href="https://github.com/netty/netty/tree/3/src/main/java/org/jboss/netty/example/telnet">Telnet</a> - a classic line-based network application</li>
<li>
<a href="https://github.com/netty/netty/tree/3/src/main/java/org/jboss/netty/example/qotm">Quote</a> of the Moment - broadcast UDP/IP client and server</li>
<li>
<a href="https://github.com/netty/netty/tree/3/src/main/java/org/jboss/netty/example/securechat">SecureChat</a> - an SSL-based chat server derived from the Telnet example</li>
</ul>
<h4>Binary protocols</h4>
<ul>
<li>
<a href="https://github.com/netty/netty/tree/3/src/main/java/org/jboss/netty/example/objectecho">ObjectEcho</a> - exchange serializable Java objects</li>
<li>
<a href="https://github.com/netty/netty/tree/3/src/main/java/org/jboss/netty/example/factorial">Factorial</a> - write a stateful client / server based on custom binary protocol</li>
<li>
<a href="https://github.com/netty/netty/tree/3/src/main/java/org/jboss/netty/example/localtime">LocalTime</a> - rapid protocol prototyping with Google Protocol Buffers integration</li>
</ul>
<h4>HTTP</h4>
<ul>
<li>
<a href="https://github.com/netty/netty/tree/3/src/main/java/org/jboss/netty/example/http/snoop">HTTP (Snoop)</a> - build your own extremely light-weight HTTP client and server</li>
<li>
<a href="https://github.com/netty/netty/tree/3/src/main/java/org/jboss/netty/example/http/file">HTTP (File Server)</a> - asynchronous large file streaming in HTTP</li>
<li>Web Socket <a href="https://github.com/netty/netty/tree/3/src/main/java/org/jboss/netty/example/http/websocketx/client">Client</a> &amp; <a href="https://github.com/netty/netty/tree/3/src/main/java/org/jboss/netty/example/http/websocketx/server">Server</a> - add two-way full-duplex communication channels to HTTP using Web Sockets protocol</li>
</ul>
<h4>Advanced</h4>
<ul>
<li>
<a href="https://github.com/netty/netty/tree/3/src/main/java/org/jboss/netty/example/proxy">Proxy Server</a> - write a highly efficient NIO proxy server</li>
<li>
<a href="https://github.com/netty/netty/tree/3/src/main/java/org/jboss/netty/example/portunification">Port Unification</a> - run heterogeneous services listening on a single port</li>
</ul>
</div>
<div class="tab-pane" id="wiki-example-4">
<h3>Examples for version 4.0</h3>
<h4>Fundamental</h4>
<ul>
<li>
<a href="https://github.com/netty/netty/tree/master/example/src/main/java/io/netty/example/echo">Echo</a> - the very basic client and server</li>
<li>
<a href="https://github.com/netty/netty/tree/master/example/src/main/java/io/netty/example/discard">Discard</a> - prevent <tt>OutOfMemoryError</tt> which is caused by writing too fast</li>
<li>
<a href="https://github.com/netty/netty/tree/master/example/src/main/java/io/netty/example/uptime">UpTime</a> - implement reliable client-side reconnection</li>
</ul>
<h4>Text protocols</h4>
<ul>
<li>
<a href="https://github.com/netty/netty/tree/master/example/src/main/java/io/netty/example/telnet">Telnet</a> - a classic line-based network application</li>
<li>
<a href="https://github.com/netty/netty/tree/master/example/src/main/java/io/netty/example/qotm">Quote</a> of the Moment - broadcast UDP/IP client and server</li>
<li>
<a href="https://github.com/netty/netty/tree/master/example/src/main/java/io/netty/example/securechat">SecureChat</a> - an SSL-based chat server derived from the Telnet example</li>
</ul>
<h4>Binary protocols</h4>
<ul>
<li>
<a href="https://github.com/netty/netty/tree/master/example/src/main/java/io/netty/example/objectecho">ObjectEcho</a> - exchange serializable Java objects</li>
<li>
<a href="https://github.com/netty/netty/tree/master/example/src/main/java/io/netty/example/factorial">Factorial</a> - write a stateful client / server based on custom binary protocol</li>
<li>
<a href="https://github.com/netty/netty/tree/master/example/src/main/java/io/netty/example/worldclock">WorldClock</a> - rapid protocol prototyping with Google Protocol Buffers integration</li>
</ul>
<h4>HTTP</h4>
<ul>
<li>
<a href="https://github.com/netty/netty/tree/master/example/src/main/java/io/netty/example/http/snoop">HTTP (Snoop)</a> - build your own extremely light-weight HTTP client and server</li>
<li>
<a href="https://github.com/netty/netty/tree/master/example/src/main/java/io/netty/example/http/file">HTTP (File Server)</a> - asynchronous large file streaming in HTTP</li>
<li>Web Socket <a href="https://github.com/netty/netty/tree/master/example/src/main/java/io/netty/example/http/websocketx/client">Client</a> &amp; <a href="https://github.com/netty/netty/tree/master/example/src/main/java/io/netty/example/http/websocketx/server">Server</a> - add two-way full-duplex communication channels to HTTP using Web Sockets protocol</li>
</ul>
<h4>Advanced</h4>
<ul>
<li>
<a href="https://github.com/netty/netty/tree/master/example/src/main/java/io/netty/example/proxy">Proxy Server</a> - write a highly efficient NIO proxy server</li>
<li>
<a href="https://github.com/netty/netty/tree/master/example/src/main/java/io/netty/example/portunification">Port Unification</a> - run heterogeneous services listening on a single port</li>
</ul>
<h4><abbr title="UDP-based Data Transfer Protocol" class="initialism">UDT</abbr></h4>
<ul>
<li>
<a href="https://github.com/netty/netty/tree/master/example/src/main/java/io/netty/example/udt/echo/bytes">Byte Streams</a> - use <a href="http://en.wikipedia.org/wiki/UDP-based_Data_Transfer_Protocol">UDT</a> in TCP-like bytes streaming mode</li>
<li>
<a href="https://github.com/netty/netty/tree/master/example/src/main/java/io/netty/example/udt/echo/message">Message Flows</a> - use <a href="http://en.wikipedia.org/wiki/UDP-based_Data_Transfer_Protocol">UDT</a> in UDP-like guaranteed message delivery mode</li>
<li>
<a href="https://github.com/netty/netty/tree/master/example/src/main/java/io/netty/example/udt/echo/rendevous">Rendezvous message flow Connect</a> - use <a href="http://en.wikipedia.org/wiki/UDP-based_Data_Transfer_Protocol">UDT</a> in symmetric peer-to-peer message flow rendezvous connect mode</li>
<li>
<a href="https://github.com/netty/netty/tree/master/example/src/main/java/io/netty/example/udt/echo/rendezvousBytes">Rendezvous byte stream Connect</a> - use <a href="http://en.wikipedia.org/wiki/UDP-based_Data_Transfer_Protocol">UDT</a> in symmetric peer-to-peer byte stream rendezvous connect mode</li>
</ul>
</div>
</div>
</div>