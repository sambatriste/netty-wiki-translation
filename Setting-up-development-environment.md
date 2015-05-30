<div class="alert alert-danger"><strong>Looking for a tutorial?</strong> Visit [[the documentation home|Home]]. <strong>Got a question?</strong> Ask at <a href="https://stackoverflow.com/questions/tagged/netty">StackOverflow.com</a>.<br>Please note that this guide is <strong>not</strong> a 'user guide'.  It is not intended for the 'users' who build an application using Netty but for the contributors ('dev') who want to develop Netty itself.</div>

## Install the necessary build tools

Your must have [Oracle JDK 7](http://java.oracle.com/) or above, [Apache Maven 3.1.1](http://maven.apache.org/) or above, and [Git](http://git-scm.com/) installed on your machine.  If you are on Linux, you have to install the additional packages mentioned in [[Native-transports]].

## Git line ending configuration

We use native line ending for all source code (i.e. '`\n`' for *nix and MacOS X, '`\r\n`' for Windows.) Please configure your Git installation so that your build does not fail and you push bad files, following the instructions below:

* [Dealing with line endings](https://help.github.com/articles/dealing-with-line-endings) by Github
* [Mind the End of Your Line](http://adaptivepatchwork.com/2012/03/01/mind-the-end-of-your-line/) by Tim Clem, for more information

## Set up IntelliJ IDEA

Netty project team uses [IntelliJ IDEA](http://www.jetbrains.com/idea/) as the primary IDE, although we are fine with using other development environments as long as you adhere to our coding style.

### Use the same-bit version with your OS

If you are on 64-bit operating system, use the 64-bit version of IntelliJ IDEA.  For an instance, the start menu shortcut points to the 32-bit binary even if you are using 64-bit Windows.  You'll have to find the `idea64.exe` in the installation directory and use it instead.  Otherwise, you'll see IntelliJ IDEA complains that it cannot find `io.netty:netty-tcnative:windows-x86_32`.

### Code style

Download [this code style configuration](http://netty.io/files/IntelliJ%20IDEA%20Code%20Style.zip) and unzip `Netty project.xml` into `<IntelliJ config directory>/codestyles` directory.  Choose 'Netty project' as the default code style.

### Inspection profile

Download, unzip, and import [this inspection profile](http://netty.io/files/IntelliJ%20IDEA%20Inspection%20Profile.xml.zip) into your IntelliJ IDEA and use it as the default.  See [here](http://www.jetbrains.com/idea/webhelp/customizing-profiles.html#d1372841e358) to learn how to import an inspection profile.

Ensure your modification does not introduce any inspection warning. If you think it's a false positive, suppress the warning by using the `@SuppressWarnings` annotation or `noinspection` line comment as guided by the IDE.  For more information about using the inspector, please refer to [the web help pages](http://www.jetbrains.com/idea/webhelp/inspecting-source-code.html).

### Copyright profile

Copyright text:

```plain
Copyright $today.year The Netty Project

The Netty Project licenses this file to you under the Apache License,
version 2.0 (the "License"); you may not use this file except in compliance
with the License. You may obtain a copy of the License at:

  http://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS, WITHOUT
WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied. See the
License for the specific language governing permissions and limitations
under the License.
```

Keyword to detect copyright in comments:

```
The Netty project licenses
```

Allow replacing copyright if old copyright contains:

```
The Netty project licenses
```

## Set up Eclipse with M2E and Java 7

1. [Download os-maven-plugin](http://repo1.maven.org/maven2/kr/motd/maven/os-maven-plugin/1.2.0.Final/os-maven-plugin-1.2.0.Final.jar) and put it into `<ECLIPSE_HOME>/plugins` directory to work around the problem where m2e does not evaluate an extension specified in our `pom.xml`.  (Unlike its name, it's both a Maven plugin and an Eclipse plugin.)
1. Import the project via the 'File -> Import... -> Existing Maven Projects' menu.
1. Netty project Maven `pom.xml` settings dictate use of Java SE 1.6, while implicitly using Java 7 (1.7) features if present.  This may result in compilation errors in Eclipse.  There are two ways to work around this problem:
  1. Look in the 'Window -> Preferences -> Installed JRE' menu:
    * Make sure you have Java 7 installation available under 'Installed JRE'
    * Map this Java 7 installation onto Java 6: 'Installed JRE -> Execution Environments -> Java SE 1.6'
  1. Alternatively, Java 7 JRE can be selected on per-project basis for each Netty module.
