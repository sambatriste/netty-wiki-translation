Since 4.0.16, Netty provides the native socket transport for Linux using JNI.  This transport has higher performance and produces less garbage, so you might want to give it a try.

## Using the native transport

Because the native transport is compatible with the NIO transport, you can just do the following search-and-replace:

* `NioEventLoopGroup` → `EpollEventLoopGroup`
* `NioEventLoop` → `EpollEventLoop`
* `NioServerSocketChannel` → `EpollServerSocketChannel`
* `NioSocketChannel` → `EpollSocketChannel`

Because the native transport is not part of the Netty core, you need to pull the `netty-transport-native-epoll` as a dependency in your Maven `pom.xml`:

```xml
  <build>
    <extensions>
      <extension>
        <groupId>kr.motd.maven</groupId>
        <artifactId>os-maven-plugin</artifactId>
        <version>1.2.3.Final</version>
      </extension>
    </extensions>
    ...
  </build>

  <dependencies>
    <dependency>
      <groupId>io.netty</groupId>
      <artifactId>netty-transport-native-epoll</artifactId>
      <version>${project.version}</version>
      <classifier>${os.detected.classifier}</classifier>
    </dependency>
    ...
  </dependencies>
```

Note that you must specify the proper classifier for the dependency.  `os-maven-plugin` in the `extensions` section in the `pom.xml` file will set `os.detected.classifier` property automatically.  For more information. refer to [the homepage of the `os-maven-plugin`](https://github.com/trustin/os-maven-plugin).

## Building the native transport

If you already have the JAR file of the native transport, you should not need to build the native transport by yourself because the JAR file already contains the necessary shared library files (e.g. `.so`, `.dll`, `.dynlib`) and they will be loaded automatically.

To build the native transport, you need to use Linux with 64-bit kernel 2.6 or higher.  Please also install the required tools and libraries:

```bash
# RHEL/CentOS/Fedora:
sudo yum install autoconf automake libtool glibc-devel.i686 glibc-devel libgcc.i686 make tar
# Debian/Ubuntu:
sudo apt-get install autoconf automake libtool make gcc-multilib tar
```