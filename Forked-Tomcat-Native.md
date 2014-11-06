[netty-tcnative](https://github.com/netty/netty-tcnative) is a fork of [Tomcat Native](http://tomcat.apache.org/native-doc/).  It includes a set of changes contributed by Twitter, Inc, such as:

* Simplified distribution and linkage of native library
* Complete mavenization of the project
* Improved OpenSSL support

To minimize the maintenance burden, we create a dedicated branch for each stable upstream release and apply our own changes on top of it, while keeping the number of maintained branches to minimum.

## How to use

In your application's `pom.xml`, add the `os-maven-plugin` extension and `netty-tcnative` dependency:

```xml
<project>
  ...
  <dependencies>
    ...
    <dependency>
      <groupId>io.netty</groupId>
      <artifactId>netty-tcnative</artifactId>
      <version>1.1.30.Fork1</version>
      <classifier>${os.detected.classifier}</classifier>
    </dependency>
    ...
  </dependencies>
  ...
  <build>
    ...
    <extensions>
      <extension>
        <groupId>kr.motd.maven</groupId>
        <artifactId>os-maven-plugin</artifactId>
        <version>1.2.3.Final</version>
      </extension>
    </extensions>
    ...
  </build>
  ...
</project>
```

To use the OpenSSL-based `SSLEngine` implementation, use `io.netty.handler.ssl.SslContext` class:

```java
public static void main(String[] args) throws Exception {
  SslContext sslCtx = SslContext.newServerContext(SslProvider.OPENSSL, ...);
  ...
}

public class MyChannelInitializer extends ChannelInitializer<SocketChannel> {

  private final SslContext sslCtx;
  
  public MyChannelInitializer(SslContext sslCtx) {
    this.sslCtx = sslCtx;
  }
  
  @Override
  public void initChannel(SocketChannel ch) throws Exception {
    ChannelPipeline p = ch.pipeline();
    if (sslCtx != null) {
      p.addLast(sslCtx.newHandler(ch.alloc()));
    }
    ...
  }
}
```

Netty will unpack the shared library provided by the `netty-tcnative-<version>-<os.detected.classifier>.jar` into a temporary directory and load it using `java.lang.System.load(String)`.

Note that the shared library is dynamically linked against Apache Portable Runtime (APR) and OpenSSL.  They must be in the library load path such as system library directories, `$LD_LIBRARY_PATH`, and `%PATH%`.

* If you are on Linux, you probably don't need to do anything because you can install them using your system's package manager.
* If you are on Mac, you have to install `openssl` package using [Homebrew](http://brew.sh/).
* If you are on Windows, you have to:
  * Build APR by yourself (or get it from [our CI server](http://clinker.netty.io/jenkins/view/TCNative/job/apr-1.5.1-windows-x86_64/lastSuccessfulBuild/artifact/dist/bin/)),
  * Install [OpenSSL for Windows](http://slproweb.com/products/Win32OpenSSL.html), and
  * Add the directories that contain the .DLL files to `%PATH%`.

## How to build

You don't usually need to build `netty-tcnative` by yourself because we officially ship the JARs with the native libraries for Linux x86_64, Mac OS X x86_64, and Windows x86_64.  If you are looking for a SNAPSHOT build, browse [our CI server](http://clinker.netty.io/jenkins/view/TCNative/).

If you are on a platform we don't ship the JARs with the native libraries, such as Windows x86_32, follow the instructions in this section.

### Building on Linux

Install the required packages:

```bash
# RHEL/CentOS/Fedora:
sudo yum install autoconf automake libtool glibc-devel make tar apr-devel openssl-devel
# Debian/Ubuntu:
sudo apt-get install autoconf automake libtool make tar libapr1-dev libssl-dev
```

Build the package and install it into your local Maven repository:

```bash
git clone https://github.com/netty/netty-tcnative.git
cd netty-tcnative
# To build a specific version: (e.g. netty-tcnative-1.1.30.Fork2)
git checkout netty-tcnative-[version]
# To build a snapshot: (e.g. 1.1.30)
git checkout [branch]
mvn clean install
```

### Building on Mac OS X

First of you need to install [Xcode](https://itunes.apple.com/de/app/xcode/id497799835?mt=12). After this make sure you install the command line tools as well:
```bash
xcode-select --install
```

Use [Homebrew](http://brew.sh/) to install the required packages:

```bash
brew install autoconf automake libtool openssl
```

Add the following exports to your `~/.bash_profile`, so that the compiler and linker finds OpenSSL built by Homebrew.

```bash
export CPPFLAGS="-I/usr/local/opt/openssl/include $CPPFLAGS"
export LDFLAGS="-L/usr/local/opt/openssl/lib $LDFLAGS"
```

Build the package and install it into your local Maven repository:

```bash
git clone https://github.com/netty/netty-tcnative.git
cd netty-tcnative
# To build a specific version: (e.g. netty-tcnative-1.1.30.Fork2)
git checkout netty-tcnative-[version]
# To build a snapshot: (e.g. 1.1.30)
git checkout [branch]
mvn clean install
```

### Building on Windows

Install the following packages:

* [Visual C++ 2010 Express](http://www.visualstudio.com/en-us/downloads#d-2010-express)
* [Windows SDK for Windows 7](http://www.microsoft.com/en-us/download/details.aspx?id=8279)
* [Visual Studio 2010 Service Pack 1](http://www.microsoft.com/en-us/download/details.aspx?id=23691)
* [Visual C++ 2010 Service Pack 1 Compiler Update](http://www.microsoft.com/en-us/download/details.aspx?id=4422)
* OpenSSL for Windows
  * If you are on 32-bit Windows:
    * [Visual C++ 2008 Redistributable Package (x86)](http://www.microsoft.com/en-us/download/details.aspx?id=29)
    * [Win32 OpenSSL 1.0.1g](http://slproweb.com/download/Win32OpenSSL-1_0_1g.exe)
  * If you are on 64-bit Windows:
    * [Visual C++ 2008 Redistributable Package (x64)](http://www.microsoft.com/en-us/download/details.aspx?id=15336)
    * [Win64 OpenSSL 1.0.1g](http://slproweb.com/download/Win64OpenSSL-1_0_1g.exe)
  * Set the installation directory to: `C:\Workspaces\openssl-1.0.1g-dist`
  * Choose 'Copy OpenSSL DLLs to: the OpenSSL binaries (/bin) directory'
* [Apache Portable Runtime (APR) 1.5.1](http://www.us.apache.org/dist//apr/apr-1.5.1-win32-src.zip)
  * Unzip into: `C:\Workspaces\apr-1.5.1`

Now, start the command prompt (`cmd.exe`) and load the environment variables required for Visual C++:

```bat
REM 32-bit Windows:
"C:\Program Files\Microsoft Visual Studio 10.0\VC\bin\vcvars32.bat"
REM 64-bit Windows:
CALL "C:\Program Files\Microsoft SDKs\Windows\v7.1\Bin\SetEnv.cmd" /Release /x64 /xp
```

Build APR:

```bat
cd "C:\Workspaces\apr-1.5.1"
REM 32-bit Windows
nmake /f Makefile.win ARCH="Win32 Release" PREFIX=..\apr-1.5.1-dist buildall install
REM 64-bit Windows
nmake /f Makefile.win ARCH="x64 Release" PREFIX=..\apr-1.5.1-dist buildall install
```

Set the required environment variables for building `netty-tcnative` on Windows:

```bat
set APR_INCLUDE_DIR=C:\Workspaces\apr-1.5.1\include
set APR_LIB_DIR=C:\Workspaces\apr-1.5.1-dist\lib
set OPENSSL_INCLUDE_DIR=C:\Workspaces\openssl-1.0.1g-dist\include
set OPENSSL_LIB_DIR=C:\Workspaces\openssl-1.0.1g-dist\lib\VC
```

Clone `netty-tcnative` and build it:

```bat
git clone https://github.com/netty/netty-tcnative.git
cd netty-tcnative
REM To build a specific version: (e.g. netty-tcnative-1.1.30.Fork2)
git checkout netty-tcnative-[version]
REM To build a snapshot: (e.g. 1.1.30)
git checkout [branch]
mvn clean install
```

## Creating a new fork

Checking out the branch `bootstrap` and running the `new-fork` script with the upstream tcnative version will bring you to a new fully mavenized branch whose name is identical to the upstream tcnative version.  For example:

```
$ git checkout bootstrap
$ ./new-fork 1.1.29 1
```

will create a new branch called `1.1.29`, which contains a mavenized fork of `tcnative-1.1.29`.  Please note that the fork does not contain any patches that will change the main source code of `tcnative`.  You will probably want to cherry-pick some commits from other branches that were patched already.
