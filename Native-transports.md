Since 4.0.16, Netty provides the native socket transport for Linux using JNI.  This transport has higher performance and produces less garbage, so you might want to give it a try.

## Using the native transport

Because the native transport is compatible with the NIO transport, you can just do the following search-and-replace:

* `NioEventLoop` -> `EpollEventLoop`
* `Nio*Channel` -> `Epoll*Channel`

## Building the native Transport

To build the native transport you need to use Linux with 64-bit kernel 2.6 or higher.

Please install the required tools and libraries:

```
RHEL/CentOS/Fedora:
# sudo yum install autoconf automake libtool glibc-devel.i686 glibc-devel libgcc.i686 make
Debian/Ubuntu:
# sudo apt-get install autoconf automake libtool make gcc-multilib
```