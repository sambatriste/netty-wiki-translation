<!-- -*- coding: utf-8-unix -*- -->

<!--
## Preface
-->

## 序言

<!--
### The Problem
-->

### 問題

<!-- Nowadays we use general purpose applications or libraries to communicate with each other. For example, we often use an HTTP client library to retrieve information from a web server and to invoke a remote procedure call via web services. -->
<!-- However, a general purpose protocol or its implementation sometimes does not scale very well. It is like we don't use a general purpose HTTP server to exchange huge files, e-mail messages, and near-realtime messages such as financial information and multiplayer game data. What's required is a highly optimized protocol implementation which is dedicated to a special purpose. For example, you might want to implement an HTTP server which is optimized for AJAX-based chat application, media streaming, or large file transfer. You could even want to design and implement a whole new protocol which is precisely tailored to your need. -->
<!-- Another inevitable case is when you have to deal with a legacy proprietary protocol to ensure the interoperability with an old system. What matters in this case is how quickly we can implement that protocol while not sacrificing the stability and performance of the resulting application. -->

今日、我々はお互いにコミュニケーションを取るのに、汎用的なアプリケーションやライブラリを使用します。例えば、HTTPクライアントを使用してWebサーバから情報を取得したり、Webサービスを使ってリモートプロシージャ呼び出しを行うといったことをよくやります。

しかし、汎用的なプロトコルやその実装というのは、時としてうまくスケールしないことがあります。これは、巨大なファイルや電子メール、金融情報や対戦型ゲームのようなリアルタイム性の高いメッセージをやりとりするのに、本来多目的であるHTTPサーバは使わない、というのに似ています。特定の用途のために高度に最適化されたプロトコルを実装するためには、何が必要でしょうか？例えば、Ajaxベースのチャットアプリケーションやメディアストリーミング、巨大なファイル転送に最適化されたHTTPサーバを実装したい、ということがあるかもしれません。さらには、要件に応じて、全く新しいプロトコルを設計、実装したいと考えることもあるかもしれません。

もう一つ、避けがたいケースとして、古いシステムとの相互運用性を保証するために、プロプライエタリなレガシープロトコルを扱わなければならないケースがあります。このケースで重要なのは、アプリケーションの安定性や性能を犠牲にせずに、いかに素早くそのプロトコルを実装できるかです。

<!--## The Solution-->
## 解決法

<!-- _[The Netty project](http://netty.io/)_ is an effort to provide an asynchronous event-driven network application framework and tooling for the rapid development of maintainable high-performance · high-scalability protocol servers and clients. -->


_[Nettyプロジェクト](http://netty.io)_ は、イベント駆動型の非同期ネットワークアプリケーションフレームワークと、保守性が高く高性能でスケーラビリティの高いサーバやクライアントを素早く開発するためのツールを提供する活動をしています。

<!-- In other words, Netty is an NIO client server framework which enables quick and easy development of network applications such as protocol servers and clients. It greatly simplifies and streamlines network programming such as TCP and UDP socket server development. -->

言い換えると、Nettyは、素早く簡単にネットワークアプリケーションを開発できるようにするためのNIOクライアント・サーバフレームワークということです。Nettyを使うことによって、TCPやUDPのサーバ開発といったネットワークプログラミングが、非常に簡潔で効率的になります。


<!-- 'Quick and easy' does not mean that a resulting application will suffer from a maintainability or a performance issue. Netty has been designed carefully with the experiences earned from the implementation of a lot of protocols such as FTP, SMTP, HTTP, and various binary and text-based legacy protocols. As a result, Netty has succeeded to find a way to achieve ease of development, performance, stability, and flexibility without a compromise. -->

「素早く簡単に」と言っても、できあがったアプリケーションが保守性や性能問題に悩まされる、ということではありません。Nettyは、FTP,SMTP,HTTPや多様なバイナリベース、テキストベースのレガシープロトコルといった多くのプロトコル実装から得られた経験を元に、注意深くデザインされています。その結果として、Nettyは、開発の容易さ、性能、安定性、柔軟性を、妥協することなく達成する方法を見つけたのです。

<!-- Some users might already have found other network application framework that claims to have the same advantage, and you might want to ask what makes Netty so different from them. The answer is the philosophy it is built on. Netty is designed to give you the most comfortable experience both in terms of the API and the implementation from the day one. It is not something tangible but you will realize that this philosophy will make your life much easier as you read this guide and play with Netty. -->


ユーザのなかには、これと同じ長所をもつと主張している他のネットワークアプリケーションフレームワークを見つけた方もいるかもしれません。また、Nettyがそれらとどうちがうのか尋ねたいと思っているかもしれません。その答えは、Nettyが依拠している哲学にあります。Nettyは、APIと実装の両方において、最初の１日目から最も快適であるようにデザインされています。これは、はっきりとしたものではありませんが、このガイドを読んでNettyで遊ぶうちに、この哲学によって生活がより快適になるとわかるようになるでしょう。

<!-- ## Getting Started -->
## はじめよう

<!-- This chapter tours around the core constructs of Netty with simple examples to let you get started quickly. You will be able to write a client and a server on top of Netty right away when you are at the end of this chapter. -->

この章では、素早くはじめられるよう、簡単な例を使ってNettyコア部分を概観します。この章の終わりには、Netty上でクライアントやサーバが書けるようになっているでしょう。

<!-- If you prefer top-down approach in learning something, you might want to start from Chapter 2, Architectural Overview and get back here. -->

トップダウンアプローチがお好みでしたら、第２章のアーキテクチャ概要から始めて、その後ここに戻ってくるとよいでしょう。

<!-- ### Before Getting Started -->

### はじめる前に

<!-- The minimum requirements to run the examples which are introduced in this chapter are only two; the latest version of Netty and JDK 1.6 or above. The latest version of Netty is available in [the project download page](http://netty.io/downloads.html). To download the right version of JDK, please refer to your preferred JDK vendor's web site. -->

この章で示されるサンプルを動作させるための最小要件は、２つだけです。最新バージョンのNettyとJDKの1.6以上です。Netty最新バージョンは [ダウンロードページ](http://netty.io/downloads.html)で入手できます。正しいバージョンのJDKをダウンロードするには、お好みのJDKベンダーのWebサイトを参照してください。


<!-- As you read, you might have more questions about the classes introduced in this chapter. Please refer to the API reference whenever you want to know more about them. All class names in this document are linked to the online API reference for your convenience. Also, please don't hesitate to [contact the Netty project community](http://netty.io/community.html) and let us know if there's any incorrect information, errors in grammar and typo, and if you have a good idea to improve the documentation. -->


読んでいるうちに、この章で紹介されるクラスに関して疑問がわくかもしれません。クラスの詳細を知りたくなった場合はAPIリファレンスを参照してください。このドキュメントに出てくるクラス名は全てオンラインAPIリファレンスにリンクしています。また、気軽に[Nettyコミュニティ](http://netty.io/community.html)に尋ねてください。また、誤った情報、文法誤り、誤字や、よい改善案があれば、我々に知らせてください。

### Writing a Discard Server

<!-- The most simplistic protocol in the world is not 'Hello, World!' but [`DISCARD`](http://tools.ietf.org/html/rfc863). It's a protocol which discards any received data without any response. -->

世界一単純化されたプロトコルは「Hello World!」ではなく、[`DISCARD`](http://tools.ietf.org/html/rfc863)です。これは、受信したデータを、何のレスポンスも返さずに捨ててしまうプロトコルです。

<!-- To implement the `DISCARD` protocol, the only thing you need to do is to ignore all received data. Let us start straight from the handler implementation, which handles I/O events generated by Netty. -->

`DISCARD`プロトコルを実装するのに必要なのは、受信したデータを全て無視することだけです。ハンドラを実装するところから始めましょう。このハンドラは、Nettyが生成したI/Oイベントを処理します。

```java
package io.netty.example.discard;

import io.netty.buffer.ByteBuf;

import io.netty.channel.ChannelHandlerContext;
import io.netty.channel.ChannelInboundHandlerAdapter;

/**
 * Handles a server-side channel.
 */
public class DiscardServerHandler extends ChannelInboundHandlerAdapter { // (1)

    @Override
    public void channelRead(ChannelHandlerContext ctx, Object msg) { // (2)
        // Discard the received data silently.
        ((ByteBuf) msg).release(); // (3)
    }

    @Override
    public void exceptionCaught(ChannelHandlerContext ctx, Throwable cause) { // (4)
        // Close the connection when an exception is raised.
        cause.printStackTrace();
        ctx.close();
    }
}
```

<!--
1. `DiscardServerHandler` extends [`ChannelInboundHandlerAdapter`], which is an implementation of [`ChannelInboundHandler`]. [`ChannelInboundHandler`] provides various event handler methods that you can override. For now, it is just enough to extend [`ChannelInboundHandlerAdapter`] rather than to implement the handler interface by yourself.
-->
<!--
1. We override the `channelRead()` event handler method here. This method is called with the received message, whenever new data is received from a client.  In this example, the type of the received message is [`ByteBuf`].
-->
<!--
1. To implement the `DISCARD` protocol, the handler has to ignore the received message.  [`ByteBuf`] is a reference-counted object which has to be released explicitly via the `release()` method.  Please keep in mind that it is the handler's responsibility to release any reference-counted object passed to the handler.  Usually, `channelRead()` handler method is implemented like the following:
-->
<!--
1. The `exceptionCaught()` event handler method is called with a Throwable when an exception was raised by Netty due to an I/O error or by a handler implementation due to the exception thrown while processing events. In most cases, the caught exception should be logged and its associated channel should be closed here, although the implementation of this method can be different depending on what you want to do to deal with an exceptional situation. For example, you might want to send a response message with an error code before closing the connection.
-->

1. `DiscardServerHandler`は[`ChannelInboundHandlerAdapter`]を継承します。ChannelInboundHandlerAdapterは[`ChannelInboundHandler`]の実装クラスです。[`ChannelInboundHander`]は、オーバーライドすることができる様々なイベントハンドラメソッドを提供します。今のところは、自分でハンドラインタフェースを実装しなくても、[`ChannelInboundHandlerAdapter`]を継承するので十分です。
2. ここでは、`channlRead()`イベントハンドラメソッドをオーバします。このメソッドは、クライアントから新しいデータを受信した時は(whenever)、受信したメッセージを引数にして呼ばれます。この例では、受信したメッセージの型は[`ByteBuf`]です。
3. `DISCARD`プロトコルを実装するためには、ハンドラは受信したメッセージを無視する必要があります。[`ByteBuf`]は参照数が管理された(reference-counted)オブジェクトで、典型的には、`release()`メソッドで解放されなければなりません。ハンドラに渡された、参照数が管理された(referece-counted)オブジェクトを解放するのはハンドラの責務であることを覚えておいてください。
```java
  @Override
  public void channelRead(ChannelHandlerContext ctx, Object msg) {
      try {
          // Do something with msg
      } finally {
          ReferenceCountUtil.release(msg);
      }
  }
```
4. Nettyで入出力エラーが発生した場合、または、ハンドラ実装クラスでイベント処理中に例外が発生した場合、イベントハンドラメソッド`exceptionCaught()` は、例外が発生した場合にThrowableを引数にして起動されます。例外的な状況をどのように扱いたいかによって、このメソッドの実装内容は異なってくるのですが、通常は、キャッチされた例外はログ出力されるべきであり、関連付けられているチャネルはここでクローズされる必要があります。例えば、コネクションをクローズする前に、エラーコードを付与した応答メッセージを送信したいということがあるかもしれません。


<!--
So far so good. We have implemented the first half of the `DISCARD` server. What's left now is to write the `main()` method which starts the server with the `DiscardServerHandler`.
-->

ここまでは順調です。`DISARD`サーバの最初の半分を実装しました。今、残されているのは、`DiscardServerHandler`を使ったサーバを起動する`main()`メソッドを書くことです。

```java
package io.netty.example.discard;
    
import io.netty.bootstrap.ServerBootstrap;

import io.netty.channel.ChannelFuture;
import io.netty.channel.ChannelInitializer;
import io.netty.channel.ChannelOption;
import io.netty.channel.EventLoopGroup;
import io.netty.channel.nio.NioEventLoopGroup;
import io.netty.channel.socket.SocketChannel;
import io.netty.channel.socket.nio.NioServerSocketChannel;
    
/**
 * Discards any incoming data.
 */
public class DiscardServer {
    
    private int port;
    
    public DiscardServer(int port) {
        this.port = port;
    }
    
    public void run() throws Exception {
        EventLoopGroup bossGroup = new NioEventLoopGroup(); // (1)
        EventLoopGroup workerGroup = new NioEventLoopGroup();
        try {
            ServerBootstrap b = new ServerBootstrap(); // (2)
            b.group(bossGroup, workerGroup)
             .channel(NioServerSocketChannel.class) // (3)
             .childHandler(new ChannelInitializer<SocketChannel>() { // (4)
                 @Override
                 public void initChannel(SocketChannel ch) throws Exception {
                     ch.pipeline().addLast(new DiscardServerHandler());
                 }
             })
             .option(ChannelOption.SO_BACKLOG, 128)          // (5)
             .childOption(ChannelOption.SO_KEEPALIVE, true); // (6)
    
            // Bind and start to accept incoming connections.
            ChannelFuture f = b.bind(port).sync(); // (7)
    
            // Wait until the server socket is closed.
            // In this example, this does not happen, but you can do that to gracefully
            // shut down your server.
            f.channel().closeFuture().sync();
        } finally {
            workerGroup.shutdownGracefully();
            bossGroup.shutdownGracefully();
        }
    }
    
    public static void main(String[] args) throws Exception {
        int port;
        if (args.length > 0) {
            port = Integer.parseInt(args[0]);
        } else {
            port = 8080;
        }
        new DiscardServer(port).run();
    }
}
```

<!--
1. [`NioEventLoopGroup`] is a multithreaded event loop that handles I/O operation.  Netty provides various [`EventLoopGroup`] implementations for different kind of transports. We are implementing a server-side application in this example, and therefore two [`NioEventLoopGroup`] will be used. The first one, often called 'boss', accepts an incoming connection. The second one, often called 'worker', handles the traffic of the accepted connection once the boss accepts the connection and registers the accepted connection to the worker.  How many Threads are used and how they are mapped to the created [`Channel`]s depends on the [`EventLoopGroup`] implementation and may be even configurable via a constructor.
-->
<!--
1. [`ServerBootstrap`] is a helper class that sets up a server. You can set up the server using a [`Channel`] directly. However, please note that this is a tedious process, and you do not need to do that in most cases.
-->
<!--
1. Here, we specify to use the [`NioServerSocketChannel`] class which is used to instantiate a new [`Channel`] to accept incoming connections.
-->
<!--
1. The handler specified here will always be evaluated by a newly accepted [`Channel`]. The [`ChannelInitializer`] is a special handler that is purposed to help a user configure a new [`Channel`].  It is most likely that you want to configure the [`ChannelPipeline`] of the new [`Channel`] by adding some handlers such as `DiscardServerHandler` to implement your network application.  As the application gets complicated, it is likely that you will add more handlers to the pipeline and extract this anonymous class into a top level class eventually.
-->
<!--
1. You can also set the parameters which are specific to the `Channel` implementation. We are writing a TCP/IP server, so we are allowed to set the socket options such as `tcpNoDelay` and `keepAlive`. Please refer to the apidocs of [`ChannelOption`] and the specific [`ChannelConfig`] implementations to get an overview about the supported `ChannelOption`s.
-->
<!--
1. Did you notice `option()` and `childOption()`?  `option()` is for the [`NioServerSocketChannel`] that accepts incoming connections. `childOption()` is for the [`Channel`]s accepted by the parent [`ServerChannel`], which is [`NioServerSocketChannel`] in this case.
-->
<!--
1. We are ready to go now. What's left is to bind to the port and to start the server. Here, we bind to the port `8080` of all NICs (network interface cards) in the machine. You can now call the `bind()` method as many times as you want (with different bind addresses.)
-->


1. [`NioEventLoopGroup`]は、マルチスレッドのイベントループで、入出力処理を行います。Nettyは、転送の種類に応じてさまざまな[`EventLoopGroup`]実装を提供します。この例では、我々はサーバサイドアプリケーションを実装しています。したがって、２つの[`NioEventLoopGroup`]が使用されます。１つめはコネクションを受け付るもので、よく「ボス(boss)」と呼ばれます。２つめは、よく「ワーカー(worker)」と呼ばれるもので、ボスがコネクションを受け付けてこのワーカーに割り当てると、そのコネクションで通信を行います。どれだけのスレッドが使用されるか、生成された[`Channel`]にスレッドがどのように割り当てられるかは、[`EventLoopGroup`]の実装に依存します。これは、コンストラクタを通して設定を変更することができます。
2. [`ServerBootStrap`]はサーバをセットアップするヘルパークラスです。直接[`Channel`]を使ってサーバをセットアップすることは可能ですが、これは退屈な作業であり、ほとんどの場合、その必要はないということを覚えておいてください。
3. ここでは、[`NioServerSocketchannel`]クラスを指定しています。このクラスは、[`Channel`]を新たにインスタンス化して、入力(incoming)コネクションを受け入れるのに使用されます。
4. ここで指定されているハンドラは、新たに受け入れられた[`Channel`]によって、毎回評価されます。[`ChannelInitializer`]は特別なハンドラで、ユーザが新しい[`Channel`]を設定するのを助けるためのものです。ネットワークアプリケーションを実装するために、新しい[`Channel`]の[`ChannelPipeline`]に`DiscardServerHandler`のようなハンドラを追加する設定をしたい、というようなことはよくありそうなことです。アプリケーションが複雑になるにつれて、より多くのハンドラをパイプラインに追加していって、最終的には、この無名クラスをトップレベルクラスに引き上げるようになるでしょう。
5. `Channel`実装を指定するパラメータを設定することをもできます。我々はTCP/IPサーバを書いているので、`tcpNoDelay`や`keepAlive`といったソケットオプションを指定することができます。サポートされている`ChannelOption`を概要については、[`ChannelOption`]と[`ChannelConfig`]実装クラスのAPIドキュメントを参照してください。
6. `option()`、`childOption()`に気が付きましたか？`option()`は、入力(incoming)コネクションを受け付ける[`NioServerSocketChannel`]のためのものです。`childOption`のほうは、親の[`ServerChannel`]（この場合は、 [`NioServerSocketChannel`]）によって受け付けられた[`Channel`]のためのものです。
7. 準備が整いました。残りは、ポートにバインドしてサーバを開始するだけです。ここでは、マシン上の全NICのポート8080にバインドします。（異なるアドレスを使って）好きなだけ何度でも`bind()`メソッドを呼ぶことができます。

<!--
Congratulations! You've just finished your first server on top of Netty.
-->

おめでとうございます！Netty上に最初のサーバを完成させました。

<!--
### Looking into the Received Data
-->


### 受信したデータの中身を見る

<!--
Now that we have written our first server, we need to test if it really works. The easiest way to test it is to use the *telnet* command. For example, you could enter `telnet localhost 8080` in the command line and type something.
-->


最初のサーバを書き上げたので、これが本当に動くのかテストする必要があります。テストする最も簡単な方法は *telnet* コマンドを使うことです。例えば、コマンドラインで `telnet localhost 8080` と入力し、何かタイプしてみます。

<!--
However, can we say that the server is working fine? We cannot really know that because it is a discard server. You will not get any response at all. To prove it is really working, let us modify the server to print what it has received.
-->

しかし、このサーバは正しく動いているといえるのでしょうか？このサーバは受信したデータを捨てるので、動作しているのかわからないのです。何の応答も受けとることはありません。サーバが正しく動いていることを証明するため、受け取ったものをプリントするように修正しましょう。


<!-- We already know that `channelRead()` method is invoked whenever data is received. Let us put some code into the `channelRead()` method of the `DiscardServerHandler`: -->

データ受信時に`channelRead()`メソッドが起動されることは既にわかっています。`DiscardServerhandler`の`channelRead()`メソッドにコードを足してみましょう。

```java
@Override
public void channelRead(ChannelHandlerContext ctx, Object msg) {
    ByteBuf in = (ByteBuf) msg;
    try {
        while (in.isReadable()) { // (1)
            System.out.print((char) in.readByte());
            System.out.flush();
        }
    } finally {
        ReferenceCountUtil.release(msg); // (2)
    }
}
```

<!--
1. This inefficient loop can actually be simplified to: `System.out.println(in.toString(io.netty.util.CharsetUtil.US_ASCII))`
-->
<!--
1. Alternatively, you could do `in.release()` here.
-->
<!--
If you run the *telnet* command again, you will see the server prints what has received.
-->

1. この効率が良くないループ処理は、実際は次のように単純化できます。`System.out.println(in.toString(io.netty.util.CharsetUtil.US_ASCII))`
2. あるいは、ここで`in.release()`とすることもできます。

もう一度*telnet*コマンドを動かすと、サーバが受信したデータを見ることができるでしょう。

<!--
The full source code of the discard server is located in the [`io.netty.example.discard`] package of the distribution.
-->


DISCARDサーバの全ソースコードは、配布物の [`io.netty.example.discard`] パッケージにあります。

<!--
### Writing an Echo Server
-->

### Echoサーバを書く

<!--
So far, we have been consuming data without responding at all. A server, however, is usually supposed to respond to a request. Let us learn how to write a response message to a client by implementing the [`ECHO`](http://tools.ietf.org/html/rfc862) protocol, where any received data is sent back.
-->

これまでは、レスポンスを返却することなく受信データを消費していました。しかし、通常、サーバというのはリクエストに応答するものです。 [`ECHO`](http://tools.ietf.org/html/rfc862) プロトコルを実装して、応答メッセージをクライアントに返す方法を学びましょう。

<!--
The only difference from the discard server we have implemented in the previous sections is that it sends the received data back instead of printing the received data out to the console. Therefore, it is enough again to modify the `channelRead()` method:
-->

前節で実装したDISCARDサーバとのちがいは、受信したデータをコンソールに出力するのではなく送り返すという点だけです。したがって、同じように`channelRead()`メソッドを修正すれば十分です。

```java
    @Override
    public void channelRead(ChannelHandlerContext ctx, Object msg) {
        ctx.write(msg); // (1)
        ctx.flush(); // (2)
    }
```
<!--
1. A [`ChannelHandlerContext`] object provides various operations that enable you to trigger various I/O events and operations.  Here, we invoke `write(Object)` to write the received message in verbatim.  Please note that we did not release the received message unlike we did in the `DISCARD` example.  It is because Netty releases it for you when it is written out to the wire.
-->
<!--
1. `ctx.write(Object)` does not make the message written out to the wire.  It is buffered internally, and then flushed out to the wire by `ctx.flush()`.  Alternatively, you could call `ctx.writeAndFlush(msg)` for brevity.
-->

1. [`ChannelHandlerContext`]オブジェクトは、さまざまな入出力イベント、操作を引き起こすさまざまな操作を提供します。ここでは、`write(Object)`メソッドを起動して、受信したメッセージをそのまま書き出します。`DISCARD`の例とはちがって、受信したメッセージを解放していない点に注意してください。これは、メッセージが最後まで書きだされたときに、Nettyが代わりに解放してくれるからです。
2. `ctx.write(Object)` はメッセージを最後まで書き出すことはしません。これは内部的にバッファされ、`ctx.flush()`によって全てフラッシュされます。代わりに簡潔に、`ctx.writeAndFlush(msg)`と書くこともできます。


<!--
If you run the *telnet* command again, you will see the server sends back whatever you have sent to it.
-->

再び*telnet*コマンドを動かすと、送信したものをサーバが送り返してくるのがわかるでしょう。

<!--
The full source code of the echo server is located in the [`io.netty.example.echo`] package of the distribution.
-->

DISCARDサーバの全ソースコードは、配布物の [`io.netty.example.echo`] パッケージにあります。


<!-- ### Writing a Time Server -->

### Timeサーバを書く

<!--
The protocol to implement in this section is the [`TIME`](http://tools.ietf.org/html/rfc868) protocol. It is different from the previous examples in that it sends a message, which contains a 32-bit integer, without receiving any requests and loses the connection once the message is sent. In this example, you will learn how to construct and send a message, and to close the connection on completion.
-->

この節で実装するのは、[`TIME`](http://tools.ietf.org/html/rfc868) プロトコルです。今までの例と違うのは、何のリクエストも受信することなく、32bit integer型のメッセージを送信し、メッセージが送信されるとすぐにコネクションを閉じるという点です。この例では、送信メッセージを組み立てる方法と、完了時にコネクションをクローズする方法を学びます。

<!--
Because we are going to ignore any received data but to send a message as soon as a connection is established, we cannot use the `channelRead()` method this time. Instead, we should override the `channelActive()` method. The following is the implementation:
-->

受信データは無視して、コネクションが確立したら即座にメッセージを送信しようとしているのですから、今回は`channerRead()`メソッドは使えません。代わりに`channelActive()`メソッドをオーバライドする必要があります。次に実装例を示します。

```java
package io.netty.example.time;

public class TimeServerHandler extends ChannelInboundHandlerAdapter {

    @Override
    public void channelActive(final ChannelHandlerContext ctx) { // (1)
        final ByteBuf time = ctx.alloc().buffer(4); // (2)
        time.writeInt((int) (System.currentTimeMillis() / 1000L + 2208988800L));
        
        final ChannelFuture f = ctx.writeAndFlush(time); // (3)
        f.addListener(new ChannelFutureListener() {
            @Override
            public void operationComplete(ChannelFuture future) {
                assert f == future;
                ctx.close();
            }
        }); // (4)
    }
    
    @Override
    public void exceptionCaught(ChannelHandlerContext ctx, Throwable cause) {
        cause.printStackTrace();
        ctx.close();
    }
}
```

<!--
1. As explained, the `channelActive()` method will be invoked when a connection is established and ready to generate traffic.  Let's write a 32-bit integer that represents the current time in this method.
-->
<!--
1. To send a new message, we need to allocate a new buffer which will contain the message. We are going to write a 32-bit integer, and therefore we need a [`ByteBuf`] whose capacity is at least 4 bytes. Get the current [`ByteBufAllocator`] via `ChannelHandlerContext.alloc()` and allocate a new buffer.
-->
<!-- 1. As usual, we write the constructed message. -->


1. 説明したとおり、この`channelActive()`メソッドは、コネクションが確立して通信が可能となった時点で起動されます。このメソッドで、現在時刻を表す32bit整数型のデータを書き込みます。
2. 新しいメッセージを送信するために、メッセージを格納する新しいバッファをアロケートする必要があります。32bit整数型のデータを書こうとしているので、最低4バイトの容量を持つ[`ByteBuff`]が必要になります。`ChannelHandlerContext.alloc()`で[`ByteBufAllocator`]を取得して新しいバッファをアロケートします。
3. 今までのとおり、組み立てたメッセージを書き込みます。

   <!-- But wait, where's the flip? Didn't we used to call `java.nio.ByteBuffer.flip()` before sending a message in NIO? `ByteBuf` does not have such a method because it has two pointers; one for read operations and the other for write operations. The writer index increases when you write something to a `ByteBuf` while the reader index does not change. The reader index and the writer index represents where the message starts and ends respectively. -->

でもちょっと待ってください。、flipはどこにあるのでしょう？NIOでメッセージを送る前は、`java.nio.ByteBuffer.flip()` を呼ぶのではなかったのですか？`ByteBuf`にはflipのようなメソッドはありません。なぜなら、`ByteBuf`は２つのポインタをもっているからです。読み込み操作のためのものと、もうひとつは書き込み操作のためのものです。ライタのインデックスは、`ByteBuf`に何か書き込んだ時に増加します。いっぽう、リーダのインデックスは変化しません。リーダのインデックスとライタのインデックスはそれぞれが、メッセージがどこから始まってどこで終わるのかを表します。

<!--
In contrast, NIO buffer does not provide a clean way to figure out where the message content starts and ends without calling the flip method. You will be in trouble when you forget to flip the buffer because nothing or incorrect data will be sent. Such an error does not happen in Netty because we have different pointer for different operation types. You will find it makes your life much easier as you get used to it -- a life without flipping out!
-->

これに対し、NIOのバッファは、メッセージの始まりと終わりがどこであるかをflipメソッドを呼ぶことなしに知るためのクリーンな方法を提供していません。バッファをフリップするのを忘れた場合、苦労することでしょう。何のデータも送信されないかあるいは誤ったデータが送られてしまうのです。異なった操作に対して別々のポインタを持っているため、Nettyではこのような誤りは起こりません。あなたは、かつてそうであったように、生活がもっと楽に感じるでしょう。気がヘンになる(flipping out)ことがない生活を！

<!-- Another point to note is that the `ChannelHandlerContext.write()` (and `writeAndFlush()`) method returns a [`ChannelFuture`]. A [`ChannelFuture`] represents an I/O operation which has not yet occurred. It means, any requested operation might not have been performed yet because all operations are asynchronous in Netty. For example, the following code might close the connection even before a message is sent: -->

もうひとつ覚えておくべきことは、`ChannelHandlerContext.write()`（それに`writeAndFlush()`）メソッドは、[`ChannelFuture`]を返却するということです。[`ChannelFuture`]は、まだ行われいない入出力操作を表します。このことが意味するのは、要求された操作がまだ実行されていないかもしれないということです。Nettyでは全ての操作が非同期であるからです。例えば、次のコードは、メッセージが送信される前にクローズされるかもしれません。

   ```java
   Channel ch = ...;
   ch.writeAndFlush(message);
   ch.close();
   ```

<!-- Therefore, you need to call the `close()` method after the [`ChannelFuture`] is complete, which was returned by the `write()` method, and it notifies its listeners when the write operation has been done. Please note that, `close()` also might not close the connection immediately, and it returns a [`ChannelFuture`]. -->

したがって、[`ChannelFuture`]が完了した後に`close()`メソッドを呼ぶ必要があります。[`ChannelFuture`]は`write()`メソッドから返却され、書き込みの操作が完了した時、リスナーに通知をします。`close()`もまた、即座にコネクションをクローズするとは限らず、[`ChannelFuture`]を返却する、ということを覚えておいてください。

<!-- 1. How do we get notified when a write request is finished then? This is as simple as adding a [`ChannelFutureListener`] to the returned `ChannelFuture`. Here, we created a new anonymous [`ChannelFutureListener`] which closes the `Channel` when the operation is done. -->


4. 書き込み要求が完了したとき、どうやって通知を受け取るのでしょう？これは、返却された`ChannelFuture`に[`ChannelFutureListener`]を設定するだけです。ここでは、操作完了時に`Channel`をクローズする、新しい無名の[`ChannelFutureListener`]を作成します。

<!-- Alternatively, you could simplify the code using a pre-defined listener: -->

代わりに、予め定義されたリスナーを使ってコードを簡潔にすることもできます。

   ```java
   f.addListener(ChannelFutureListener.CLOSE);
   ```

<!-- To test if our time server works as expected, you can use the UNIX `rdate` command:  -->

TIMEサーバが期待通り動作するかテストするため、UNIXの`rdate`コマンドが使えます。

```
$ rdate -o <port> -p <host>
```

<!-- where `<port>` is the port number you specified in the `main()` method and `<host>` is usually `localhost`.  -->

`<port>`の箇所は、`main()`メソッドで指定したポート番号を、`<host>`には通常`localhost`を指定します。


<!-- ### Writing a Time Client -->

### Timeクライアントを書く

<!-- Unlike `DISCARD` and `ECHO` servers, we need a client for the `TIME` protocol because a human cannot translate a 32-bit binary data into a date on a calendar. In this section, we discuss how to make sure the server works correctly and learn how to write a client with Netty. -->

`DISCARD`サーバや`ECHO`サーバとは違い、`TIME`プロトコルの場合、クライントが必要になります。人間は32ビットのバイナリデータをカレンダー上の日付に変換できないからです。この節では、サーバが正確に動作しているかどうかを確かめる方法と、Nettyでクライアントを書く方法を考察します。


The biggest and only difference between a server and a client in Netty is that different [`Bootstrap`] and [`Channel`] implementations are used. Please take a look at the following code:

Nettyにおいて、サーバとクライアントとの最大かつ唯一の違いは、異なる[`Bootstrap`]と[`Channel`]実装が使われるという点です。次のコードを見てください。


```java
package io.netty.example.time;

public class TimeClient {
    public static void main(String[] args) throws Exception {
        String host = args[0];
        int port = Integer.parseInt(args[1]);
        EventLoopGroup workerGroup = new NioEventLoopGroup();
        
        try {
            Bootstrap b = new Bootstrap(); // (1)
            b.group(workerGroup); // (2)
            b.channel(NioSocketChannel.class); // (3)
            b.option(ChannelOption.SO_KEEPALIVE, true); // (4)
            b.handler(new ChannelInitializer<SocketChannel>() {
                @Override
                public void initChannel(SocketChannel ch) throws Exception {
                    ch.pipeline().addLast(new TimeClientHandler());
                }
            });
            
            // Start the client.
            ChannelFuture f = b.connect(host, port).sync(); // (5)

            // Wait until the connection is closed.
            f.channel().closeFuture().sync();
        } finally {
            workerGroup.shutdownGracefully();
        }
    }
}
```

<!-- 1. [`Bootstrap`] is similar to [`ServerBootstrap`] except that it's for non-server channels such as a client-side or connectionless channel. -->
<!-- 1. If you specify only one [`EventLoopGroup`], it will be used both as a boss group and as a worker group. The boss worker is not used for the client side though. -->
<!-- 1. Instead of [`NioServerSocketChannel`], [`NioSocketChannel`] is being used to create a client-side [`Channel`]. -->
<!-- 1. Note that we do not use `childOption()` here unlike we did with `ServerBootstrap` because the client-side [`SocketChannel`] does not have a parent. -->
<!-- 1. We should call the `connect()` method instead of the `bind()` method.  -->



1. [`Bootstrap`]は[`ServerBootstrap`]に似ていますが、クライアントサイドあるいはコネクションレスのチャネルといった、非サーバサイドのチャネル用である点が異なります。
2. 指定した[`EventLoopGroup`]が単一である場合、ボスグループとワーカグループ両方に使用されます。といってもボスはクライアント側では使用されませんが。
3. クライアント側の[`Channel`]を生成するために、[`NioServerSocketChannel`]の代わりに[`NioSocketChannel`]が使用されます。
4. `ServerBootstrap`でしたのとは違って、`childOption()`を使用しない点に注意してください。クライアント側の [`SocketChannel`]は親を持たないからです。
5. `bind()`メソッドの代わりに、`connect()`メソッドを呼ぶ必要があります。

<!-- As you can see, it is not really different from the the server-side code. What about the [`ChannelHandler`] implementation? It should receive a 32-bit integer from the server, translate it into a human readable format, print the translated time, and close the connection:  -->

見て分かる通り、サーバ側のコードとそれほど違いはありません。[`ChannelHandler`]実装のほうはどうでしょうか？この実装は、サーバから32ビット整数を受け取り、人間が読める形式に変換して表示し、コネクションをクローズしなければなりません。

```java
package io.netty.example.time;

import java.util.Date;

public class TimeClientHandler extends ChannelInboundHandlerAdapter {
    @Override
    public void channelRead(ChannelHandlerContext ctx, Object msg) {
        ByteBuf m = (ByteBuf) msg; // (1)
        try {
            long currentTimeMillis = (m.readUnsignedInt() - 2208988800L) * 1000L;
            System.out.println(new Date(currentTimeMillis));
            ctx.close();
        } finally {
            m.release();
        }
    }

    @Override
    public void exceptionCaught(ChannelHandlerContext ctx, Throwable cause) {
        cause.printStackTrace();
        ctx.close();
    }
}
```

<!-- 1. In TCP/IP, Netty reads the data sent from a peer into a [[`ByteBuf`]]. -->

1. TCP/IPでは、Nettyは通信相手から送られてきたデータを[[`ByteBuff`]]から読み取ります。

<!-- It looks very simple and does not look any different from the server side example. However, this handler sometimes will refuse to work raising an `IndexOutOfBoundsException`. We discuss why this happens in the next section.  -->

これはとても簡単なように見えますし、サーバ側の例と変わらないように見えます。しかし、このハンドラは、`IndexOutOfBoundsException`を発生させ動作しないことがあります。なぜこのようなことが起こるのか次の節で考察します。

<!-- ### Dealing with a Stream-based Transport  -->
### ストリーム型(Stream-based)の通信を扱う

<!-- #### One Small Caveat of Socket Buffer -->

#### ソケットバッファに関する注意書き

<!-- In a stream-based transport such as TCP/IP, received data is stored into a socket receive buffer. Unfortunately, the buffer of a stream-based transport is not a queue of packets but a queue of bytes. It means, even if you sent two messages as two independent packets, an operating system will not treat them as two messages but as just a bunch of bytes. Therefore, there is no guarantee that what you read is exactly what your remote peer wrote. For example, let us assume that the TCP/IP stack of an operating system has received three packets:  -->

TCP/IPのようなストリーム型の通信において、受信したデータはソケット受信バッファに蓄積されます。不運なことに、ストリーム型通信のバッファはパケットのキューではなく、バイト列のキューなのです。このことが意味するのは、２つのメッセージを２つの独立したパケットとして送信したとしても、OSはこれらを２つのメッセージとしてではなく、単にひとかたまりのバイト列として扱う、ということです。したがって、自分が読んだものと相手が書いたものが正確に同じである保証はありません。例えば、あるOSのTCP/IPスタックが３つのパケットを受信したとしましょう。

![Three packets received as they were sent](http://uml.mvnsearch.org/gist/82e3fbe0e2d4df28322b)


<!-- Because of this general property of a stream-based protocol, there's high chance of reading them in the following fragmented form in your application: -->

このストリーム型プロトコルの一般的な特性のために、アプリケーションで、次のように断片化した形で読んでしまう可能性があります。

![Three packets split and merged into four buffers](http://uml.mvnsearch.org/gist/b31c0bd7bbfc69fd82d6)

<!-- Therefore, a receiving part, regardless it is server-side or client-side, should defrag the received data into one or more meaningful frames that could be easily understood by the application logic. In case of the example above, the received data should be framed like the following: -->

したがって、受信する際は、サーバ側かクライント側かにかかわらず、受信したデータを、アプリケーションロジックが容易に理解できるような意味のあるフレームに、デフラグする必要があります。上記の例の場合、受信データを次のように組み立てる必要があります。


![Four buffers defragged into three](http://uml.mvnsearch.org/gist/82e3fbe0e2d4df28322b)

<!-- #### The First Solution -->

#### 一つ目の解

<!-- Now let us get back to the `TIME` client example. We have the same problem here. A 32-bit integer is a very small amount of data, and it is not likely to be fragmented often. However, the problem is that it can be fragmented, and the possibility of fragmentation will increase as the traffic increases. -->

`TIME`クライントの例に戻りましょう。ここで我々は同じ問題を抱えています。32ビット整数はとても小さいデータで、そうそう断片化しそうにはありません。しかし、問題は、それが断片化してしまう可能性があり、通信量が増加するにつれててその可能性が増加する、という点なのです。

<!-- The simplistic solution is to create an internal cumulative buffer and wait until all 4 bytes are received into the internal buffer. The following is the modified `TimeClientHandler` implementation that fixes the problem: -->

安直な解としては、累積的な内部バッファを作成し、この内部バッファに4バイト全てを受信するまで待つ、というものです。以下は、問題を解決した`TimeClientHandler`実装です。


```java
package io.netty.example.time;

import java.util.Date;

public class TimeClientHandler extends ChannelInboundHandlerAdapter {
    private ByteBuf buf;
    
    @Override
    public void handlerAdded(ChannelHandlerContext ctx) {
        buf = ctx.alloc().buffer(4); // (1)
    }
    
    @Override
    public void handlerRemoved(ChannelHandlerContext ctx) {
        buf.release(); // (1)
        buf = null;
    }
    
    @Override
    public void channelRead(ChannelHandlerContext ctx, Object msg) {
        ByteBuf m = (ByteBuf) msg;
        buf.writeBytes(m); // (2)
        m.release();
        
        if (buf.readableBytes() >= 4) { // (3)
            long currentTimeMillis = (buf.readUnsignedInt() - 2208988800L) * 1000L;
            System.out.println(new Date(currentTimeMillis));
            ctx.close();
        }
    }
    
    @Override
    public void exceptionCaught(ChannelHandlerContext ctx, Throwable cause) {
        cause.printStackTrace();
        ctx.close();
    }
}
```

<!-- 1. A [`ChannelHandler`] has two life cycle listener methods: `handlerAdded()` and `handlerRemoved()`.  You can perform an arbitrary (de)initialization task as long as it does not block for a long time. -->
<!-- 1. First, all received data should be cumulated into `buf`. -->
<!-- 1. And then, the handler must check if `buf` has enough data, 4 bytes in this example, and proceed to the actual business logic. Otherwise, Netty will call the `channelRead()` method again when more data arrives, and eventually all 4 bytes will be cumulated. -->

1. [`ChannelHandler`]は２つのライフサイクルリスナーメソッド、`handlerAdded()`と`handlerRemoved()`を持っています。長時間ブロックしない限り、初期化（終了）タスクを行うことができます。
2. まず、全ての受信データを`buf`に蓄積しなければなりません。
3. 次に、ハンドラは`buf`が十分なデータ、この例では4バイトを保持しているかをチェックし、それから実際のビジネスロジックに進まなければなりません。そうしないと、次のデータが到着するとNettyはまた`channelRead()`メソッドを呼び出し、最終的には4バイト全て蓄積されるでしょう。


<!-- #### The Second Solution -->

#### ２つめの解

<!-- Although the first solution has resolved the problem with the `TIME` client, the modified handler does not look that clean. Imagine a more complicated protocol which is composed of multiple fields such as a variable length field. Your [`ChannelInboundHandler`] implementation will become unmaintainable very quickly. -->

１つめの解は`TIME`クライントに関する問題を解決したけれども、修正後のハンドラはきれいには見えません。可変長フィールドのような複数のフィールドで構成されるもっと複雑なプロトコルを想像してください。 [`ChannelInboundHandler`]実装はすぐに保守できなくなるでしょう。

<!-- As you may have noticed, you can add more than one [`ChannelHandler`] to a [`ChannelPipeline`], and therefore, you can split one monolithic [`ChannelHandler`] into multiple modular ones to reduce the complexity of your application. For example, you could split `TimeClientHandler` into two handlers: -->

お気づきのように、ひとつの[`ChannelPipeline`]に対して複数の[`ChannelHandler`]を追加することができます。ゆえに、一枚岩の[`ChannelHandler`]を複数のモジュールで構成される[`ChannelHandler`]へと分割し、アプリケーションの複雑さを減らすことができるのです。例えば、`TimeClientHandler`は２つのハンドラに分割できるでしょう。

<!-- * `TimeDecoder` which deals with the fragmentation issue, and -->
<!-- * the initial simple version of `TimeClientHandler`. -->

* 断片化の問題を扱う`TimeDecoder`
* 最初のシンプルなバージョンの`TimeClientHandler`

<!-- Fortunately, Netty provides an extensible class which helps you write the first one out of the box: -->

幸運なことに、Nettyは、ひとつめのクラスを書くのを手助けする、すぐに使える拡張可能なクラスを提供しています。

```java
package io.netty.example.time;

public class TimeDecoder extends ByteToMessageDecoder { // (1)
    @Override
    protected void decode(ChannelHandlerContext ctx, ByteBuf in, List<Object> out) { // (2)
        if (in.readableBytes() < 4) {
            return; // (3)
        }
        
        out.add(in.readBytes(4)); // (4)
    }
}
```

<!-- 1. [`ByteToMessageDecoder`] is an implementation of [`ChannelInboundHandler`] which makes it easy to deal with the fragmentation issue. -->
<!-- 2. [`ByteToMessageDecoder`] calls the `decode()` method with an internally maintained cumulative buffer whenever new data is received. -->
<!-- 3. `decode()` can decide to add nothing to `out` where there is not enough data in the cumulative buffer.  [`ByteToMessageDecoder`] will call `decode()` again when there is more data received. -->
<!-- 4. If `decode()` adds an object to `out`, it means the decoder decoded a message successfully.  [`ByteToMessageDecoder`] will discard the read part of the cumulative buffer.  Please remember that you don't need to decode multiple messages. [`ByteToMessageDecoder`] will keep calling the `decode()` method until it adds nothing to `out`. -->

1. [`ByteToMessageDecoder`]は [`ChannelInboundHandler`]の実装で、断片化の問題を扱いやすくします。
2. [`ByteToMessageDecoder`]は、新しいデータが到着したら、内部的に管理されているバッファを引数にして`decode()`メソッドを呼び出します。
3. `decode()`メソッドでは、バッファに十分なデータがない場合は、`out`に何も設定しないようにすることができます。 [`ByteToMessageDecoder`]は、追加のデータがあれば再び`decode()`メソッドを起動するでしょう。
4. `decode()`メソッドで何かオブジェクトを`out`に追加した場合、デコーダがメッセージをデコードするのに成功したことを意味します。 [`ByteToMessageDecoder`]は、蓄積バッファの読み終わった部分を廃棄します。複数のメッセージを自分でデコードする必要がないということを覚えておいてください。 [`ByteToMessageDecoder`]は、`decode()`メソッドが`out`に何かを追加しない限り、`decode()`メソッドを起動し続けます。


<!-- Now that we have another handler to insert into the [`ChannelPipeline`], we should modify the [`ChannelInitializer`] implementation in the `TimeClient`: -->

[`ChannelPipeline`]に追加すべきハンドラがもうひとつのあるので、[`TimeClient`]の[`Channelinitializer`]実装を修正しなければなりません。

```java
b.handler(new ChannelInitializer<SocketChannel>() {
    @Override
    public void initChannel(SocketChannel ch) throws Exception {
        ch.pipeline().addLast(new TimeDecoder(), new TimeClientHandler());
    }
});
```

<!-- If you are an adventurous person, you might want to try the [`ReplayingDecoder`] which simplifies the decoder even more. You will need to consult the API reference for more information though. -->

あなたが冒険好きな人なら、デコーダをさらにシンプルにする[`ReplayingDecoder`]に挑戦したいと思うかもしれませんね。さらに情報を得るためにAPIリファレンスを参照する必要があるでしょうけども。


```java
public class TimeDecoder extends ReplayingDecoder<Void> {
    @Override
    protected void decode(
            ChannelHandlerContext ctx, ByteBuf in, List<Object> out) {
        out.add(in.readBytes(4));
    }
}
```

<!-- Additionally, Netty provides out-of-the-box decoders which enables you to implement most protocols very easily and helps you avoid from ending up with a monolithic unmaintainable handler implementation. Please refer to the following packages for more detailed examples: -->


さらに、Nettyはすぐに使えるデコーダ一式を提供しており、たいていのプロトコルをとても簡単に実装できるようになっています。これらのデコーダは、一枚岩のハンドラ実装を作って保守できなくなることを避けるよう手助けしてくれます。


<!-- * [`io.netty.example.factorial`] for a binary protocol, and -->
<!-- * [`io.netty.example.telnet`] for a text line-based protocol. -->

* バイナリプロトコル用の[`io.netty.example.factorial`] 
* テキストの行志向プロトコル用の[`io.netty.example.telnet`]

### Speaking in POJO instead of `ByteBuf`

### `ByteBuf`の代わりにPOJOで喋る

<!-- All the examples we have reviewed so far used a [`ByteBuf`] as a primary data structure of a protocol message. In this section, we will improve the `TIME` protocol client and server example to use a POJO instead of a [`ByteBuf`]. -->

今まで見てきた例は全て、[`ByteBuf`]をメッセージの主要なデータ構造として使っていました。この節では、[`ByteBuf`]の代わりにPOJOを使うように、`TIME`プロトコルのクライントとサーバの例を改良してみます。

<!-- The advantage of using a POJO in your [`ChannelHandler`]s is obvious; your handler becomes more maintainable and reusable by separating the code which extracts information from `ByteBuf` out from the handler. In the `TIME` client and server examples, we read only one 32-bit integer and it is not a major issue to use `ByteBuf` directly. However, you will find it is necessary to make the separation as you implement a real world protocol. -->

自作の[`ChannelHandler`]でPOJOを使うことの利点は明白です。[`ByteBuf`]から情報を抜き出すコードをハンドラから分離することにより、ハンドラの保守性、再利用性が高まるのです。`TIME`クライアント-サーバの例では、単に32ビット整数を読み取るだけで、`ByteBuf`を直接使用することは主要な問題ではありませんでした。しかし、実世界でのプロトコルを実装する場合、このような分離は必須であると気づくでしょう。



<!-- First, let us define a new type called `UnixTime`. -->

まず、`UnixTime`という新しい型を定義しましょう。

```java
package io.netty.example.time;

import java.util.Date;

public class UnixTime {

    private final long value;
    
    public UnixTime() {
        this(System.currentTimeMillis() / 1000L + 2208988800L);
    }
    
    public UnixTime(long value) {
        this.value = value;
    }
        
    public long value() {
        return value;
    }
        
    @Override
    public String toString() {
        return new Date((value() - 2208988800L) * 1000L).toString();
    }
}
```

<!-- We can now revise the `TimeDecoder` to produce a `UnixTime` instead of a [`ByteBuf`]. -->

これで、[`ByteBuf`]のかわりに`UnixTime`を生成するように`TimeDecoder`を改修することができます。

```java
@Override
protected void decode(ChannelHandlerContext ctx, ByteBuf in, List<Object> out) {
    if (in.readableBytes() < 4) {
        return;
    }

    out.add(new UnixTime(in.readUnsignedInt()));
}
```

<!-- With the updated decoder, the `TimeClientHandler` does not use [`ByteBuf`] anymore: -->

新しいデコーダでは、`TimeClientHandler`はもはや[`ByteBuf`]を使用しません。

```java
@Override
public void channelRead(ChannelHandlerContext ctx, Object msg) {
    UnixTime m = (UnixTime) msg;
    System.out.println(m);
    ctx.close();
}
```

<!-- Much simpler and elegant, right? The same technique can be applied on the server side. Let us update the `TimeServerHandler` first this time: -->

よりシンプルでエレガントになりましたね？同じテクニックはサーバ側にも当てはめることができます。まず、`TimeServerHandler`を改修しましょう。

```java
@Override
public void channelActive(ChannelHandlerContext ctx) {
    ChannelFuture f = ctx.writeAndFlush(new UnixTime());
    f.addListener(ChannelFutureListener.CLOSE);
}
```

<!-- Now, the only missing piece is an encoder, which is an implementation of [`ChannelOutboundHandler`] that translates a `UnixTime` back into a [`ByteBuf`]. It's much simpler than writing a decoder because there's no need to deal with packet fragmentation and assembly when encoding a message. -->

今、唯一足りていないのはエンコーダで、それは`UnixTime`を[`ByteBuf`]に戻す[`ChannelOutboundHandler`]実装です。これはデコーダを書くよりもずっと簡単です。というのも、メッセージをエンコードする際には、パケットの断片化やその組み立てについて取り扱う必要がないからです。


```java
package io.netty.example.time;

public class TimeEncoder extends ChannelOutboundHandlerAdapter {
    @Override
    public void write(ChannelHandlerContext ctx, Object msg, ChannelPromise promise) {
        UnixTime m = (UnixTime) msg;
        ByteBuf encoded = ctx.alloc().buffer(4);
        encoded.writeInt((int)m.value());
        ctx.write(encoded, promise); // (1)
    }
}
```

<!-- 1. There are quite a few important things in this single line. -->

1. この１行にはとても重要な点があります。

<!-- First, we pass the original [`ChannelPromise`] as-is so that Netty marks it as success or failure when the encoded data is actually written out to the wire. -->

第一には、エンコードされたデータが実際に書きだされるときに、Nettyがこれに成功・失敗を記録できるように、元の[`ChannelPromise`]をそのまま渡しています。

<!-- Second, we did not call `ctx.flush()`.  There is a separate handler method `void flush(ChannelHandlerContext ctx)` which is purposed to override the `flush()` operation. -->


第二に、`ctx.flush()`を呼んでいません。`flush()`操作をオーバライドする目的で用意された別のハンドラメソッド`void flush(ChannelHandlerContext ctx)`があるのです。

<!-- To simplify even further, you can make use of [`MessageToByteEncoder`]: -->

もっとシンプルにするのに、[`MessageToByteEncoder`]を使うことができます。

```java
public class TimeEncoder extends MessageToByteEncoder<UnixTime> {
    @Override
    protected void encode(ChannelHandlerContext ctx, UnixTime msg, ByteBuf out) {
        out.writeInt((int)msg.value());
    }
}

```

<!-- The last task left is to insert a `TimeEncoder` into the [`ChannelPipeline`] on the server side before the `TimeServerHandler`, and it is left as a trivial exercise. -->

残された最後のタスクは、サーバ側の[`ChannelPipeline`]へ、`TimeEncoder`を[`TimeServerHandler`]の前に追加することです。

<!-- ### Shutting Down Your Application -->

### アプリケーションをシャットダウンする

<!-- Shutting down a Netty application is usually as simple as shutting down all [`EventLoopGroup`]s you created via `shutdownGracefully()`.  It returns a [`Future`] that notifies you when the [`EventLoopGroup`] has been terminated completely and all [`Channel`]s that belong to the group have been closed. -->


Nettyアプリケーションをシャットダウンするのは、通常、作成した全ての[`EventLoopGroup`]を`shutdownGracefully()`を使ってシャットダウンするだけです。このメソッドは[`Future`]を返却し、この[`Future`]によって、[`EventLoopGroup`]が完全に終了されグループに属する全ての[`Channel`]がクローズされたことが通知されます。

<!-- ### Summary -->

### 要約


<!-- In this chapter, we had a quick tour of Netty with a demonstration on how to write a fully working network application on top of Netty. -->

この章では、完全に動作するネットワークアプリケーションをNetty上で書く方法について、実物を見ながら概観しました。

<!-- There is more detailed information about Netty in the upcoming chapters. We also encourage you to review the Netty examples in the [`io.netty.example`] package. -->



次の章ではNettyのより詳細な情報を提供します。[`io.netty.example`]パッケージにあるNettyのサンプルを調査することをお勧めします。


<!-- Please also note that [the community](http://netty.io/community.html) is always waiting for your questions and ideas to help you and keep improving Netty and its documentation based on your feedback. -->

また、[コミュニティ](http://netty.io/community.html)は、あなたを手助けし、フィードバックによってNettyとドキュメントを改善し続けるため、いつでも質問やアイデアを待っています。


[`Bootstrap`]: http://netty.io/4.0/api/io/netty/bootstrap/Bootstrap.html
[`ByteBuf`]: http://netty.io/4.0/api/io/netty/buffer/ByteBuf.html
[`ByteBufAllocator`]: http://netty.io/4.0/api/io/netty/buffer/ByteBufAllocator.html
[`ByteToMessageDecoder`]: http://netty.io/4.0/api/io/netty/handler/codec/ByteToMessageDecoder.html
[`Channel`]: http://netty.io/4.0/api/io/netty/channel/Channel.html
[`ChannelConfig`]: http://netty.io/4.0/api/io/netty/channel/ChannelConfig.html
[`ChannelFuture`]: http://netty.io/4.0/api/io/netty/channel/ChannelFuture.html
[`ChannelFutureListener`]: http://netty.io/4.0/api/io/netty/channel/ChannelFutureListener.html
[`ChannelHandlerContext`]: http://netty.io/4.0/api/io/netty/channel/ChannelHandlerContext.html
[`ChannelHandler`]: http://netty.io/4.0/api/io/netty/channel/ChannelHandler.html
[`ChannelInboundHandler`]: http://netty.io/4.0/api/io/netty/channel/ChannelInboundHandler.html
[`ChannelInboundHandlerAdapter`]: http://netty.io/4.0/api/io/netty/channel/ChannelInboundHandlerAdapter.html
[`ChannelInitializer`]: http://netty.io/4.0/api/io/netty/channel/ChannelInitializer.html
[`ChannelOption`]: http://netty.io/4.0/api/io/netty/channel/ChannelOption.html
[`ChannelOutboundHandler`]: http://netty.io/4.0/api/io/netty/channel/ChannelOutboundHandler.html

[`ChannelPipeline`]: http://netty.io/4.0/api/io/netty/channel/ChannelPipeline.html
[`ChannelPromise`]: http://netty.io/4.0/api/io/netty/channel/ChannelPromise.html
[`EventLoopGroup`]: http://netty.io/4.0/api/io/netty/channel/EventLoopGroup.html
[`Future`]: http://netty.io/4.0/api/io/netty/util/concurrent/Future.html
[`MessageToByteEncoder`]: http://netty.io/4.0/api/io/netty/handler/codec/MessageToByteEncoder.html
[`NioEventLoopGroup`]: http://netty.io/4.0/api/io/netty/channel/nio/NioEventLoopGroup.html
[`NioServerSocketChannel`]: http://netty.io/4.0/api/io/netty/channel/socket/nio/NioServerSocketChannel.html
[`NioSocketChannel`]: http://netty.io/4.0/api/io/netty/channel/socket/nio/NioSocketChannel.html
[`ReplayingDecoder`]: http://netty.io/4.0/api/io/netty/handler/codec/ReplayingDecoder.html
[`ServerBootstrap`]: http://netty.io/4.0/api/io/netty/bootstrap/ServerBootstrap.html
[`ServerChannel`]: http://netty.io/4.0/api/io/netty/channel/ServerChannel.html
[`SocketChannel`]: http://netty.io/4.0/api/io/netty/channel/socket/SocketChannel.html

[`io.netty.example`]: https://github.com/netty/netty/tree/4.0/example/src/main/java/io/netty/example
[`io.netty.example.discard`]: http://netty.io/4.0/xref/io/netty/example/discard/package-summary.html
[`io.netty.example.echo`]: http://netty.io/4.0/xref/io/netty/example/echo/package-summary.html
[`io.netty.example.factorial`]: http://netty.io/4.0/xref/io/netty/example/factorial/package-summary.html
[`io.netty.example.telnet`]: http://netty.io/4.0/xref/io/netty/example/telnet/package-summary.html
