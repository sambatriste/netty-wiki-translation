<div class="alert alert-danger"><strong>Looking for a tutorial?</strong> Visit [[the documentation home|Home]]. <strong>Got a question?</strong> Ask at <a href="https://stackoverflow.com/questions/tagged/netty">StackOverflow.com</a>.<br>Please note that this guide is <strong>not</strong> a 'user guide'.  It is not intended for the 'users' who build an application using Netty but for the contributors ('dev') who want to develop Netty itself.</div>

## Working with the source code

### Important!

Before pushing your commits or issuing a pull request, make sure:

* your work builds without any failure - run '`mvn test`' from shell.
* your work does not introduce any new inspection warning - see below.

### Setting up IntelliJ IDEA

Netty project team uses [IntelliJ IDEA](http://www.jetbrains.com/idea/) as the primary IDE, although we are fine with using other development environments as long as you adhere to our coding style.

#### Code style

Download [this code style configuration](http://netty.io/files/IntelliJ%20IDEA%20Code%20Style.zip) and unzip `Netty project.xml` into `<IntelliJ config directory>/codestyles` directory.  Choose 'Netty project' as the default code style.

#### Inspection profile

Download, unzip, and import [this inspection profile](http://netty.io/files/IntelliJ%20IDEA%20Inspection%20Profile.xml.zip) into your IntelliJ IDEA and use it as the default.  See [here](http://www.jetbrains.com/idea/webhelp/customizing-profiles.html#d1372841e358) to learn how to import an inspection profile.

Ensure your modification does not introduce any inspection warning. If you think it's a false positive, suppress the warning by using the `@SuppressWarnings` annotation or `noinspection` line comment as guided by the IDE.  For more information about using the inspector, please refer to [the web help pages](http://www.jetbrains.com/idea/webhelp/inspecting-source-code.html).

#### Copyright profile

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

### Setting up Eclipse with M2E and Java 7

Netty project can be imported into [Eclipse 3.7](http://www.eclipse.org/downloads/) or later with [M2E](http://eclipse.org/m2e/) integration out of the box.

Netty project Maven pom.xml settings dictate use of Java SE 1.6, while implicitly using Java 7 (1.7) features if present.  This may result in compilation errors in Eclipse.

One way to deal with this - look in ```Eclipse -> Window -> Preferences -> Installed JRE```

* make sure you have Java 7 installation available under ```Installed JRE```
* map this Java 7 installation onto Java 6 : ```Installed JRE -> Execution Environments -> Java SE 1.6```

Alternatively, Java 7 JRE can be selected on per-project basis for each Netty module.

### Git line ending configuration

We use native line ending for all source code (i.e. '`\n`' for *nix and MacOS X, '`\r\n`' for Windows.) Please configure your Git installation so that your build does not fail and you push bad files, following the instructions below:

* [Dealing with line endings](https://help.github.com/articles/dealing-with-line-endings) by Github
* [Mind the End of Your Line](http://adaptivepatchwork.com/2012/03/01/mind-the-end-of-your-line/) by Tim Clem, for more information

### Merging a pull request

A pull request often contains multiple comments.  Those commits must be squashed into a small number of commits with explanatory comments.

Do not merge a pull request via the web UI unless there's a good reason doing that. Refer to the 'Patch and Apply' section in [the Github help page](https://help.github.com/articles/using-pull-requests#merging-a-pull-request).

## Writing a new post

Our web site is generated from an Awestruct project [here](https://github.com/netty/netty-website).  To post a new article, create a new branch, write a new post, and issue a pull request like usual contributions.

## Design ideas

* [[Thread model in 4.x|Thread model]]
