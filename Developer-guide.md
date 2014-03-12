<div class="alert alert-danger"><strong>Looking for a tutorial?</strong> Visit [[the documentation home|Home]]. <strong>Got a question?</strong> Ask at <a href="https://stackoverflow.com/questions/tagged/netty">StackOverflow.com</a>.<br>Please note that this guide is <strong>not</strong> a 'user guide'.  It is not intended for the 'users' who build an application using Netty but for the contributors ('dev') who want to develop Netty itself.</div>

## Set up your development environment

### Install the necessary build tools

Your must have [Oracle JDK 7](http://java.oracle.com/) or above, [Apache Maven 3.0.5](http://maven.apache.org/) or above, and [Git](http://git-scm.com/) installed on your machine.  If you are on Linux, you have to install the additional packages mentioned in [[Native-transports]].

### Git line ending configuration

We use native line ending for all source code (i.e. '`\n`' for *nix and MacOS X, '`\r\n`' for Windows.) Please configure your Git installation so that your build does not fail and you push bad files, following the instructions below:

* [Dealing with line endings](https://help.github.com/articles/dealing-with-line-endings) by Github
* [Mind the End of Your Line](http://adaptivepatchwork.com/2012/03/01/mind-the-end-of-your-line/) by Tim Clem, for more information

### Set up IntelliJ IDEA

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

### Set up Eclipse with M2E and Java 7

Netty project can be imported into [Eclipse 3.7](http://www.eclipse.org/downloads/) or later with [M2E](http://eclipse.org/m2e/) integration out of the box.

Netty project Maven pom.xml settings dictate use of Java SE 1.6, while implicitly using Java 7 (1.7) features if present.  This may result in compilation errors in Eclipse.

One way to deal with this - look in ```Eclipse -> Window -> Preferences -> Installed JRE```

* make sure you have Java 7 installation available under ```Installed JRE```
* map this Java 7 installation onto Java 6 : ```Installed JRE -> Execution Environments -> Java SE 1.6```

Alternatively, Java 7 JRE can be selected on per-project basis for each Netty module.

## Write a meaningful commit message

After making changes on Netty, please make sure your commit message contains enough contextual information so that anyone understands the intention of the changes.  Unless the commit is trivial, please use the following form:

```plain
One line description of your change
 
Motivation:

Explain here the context, and why you're making that change.
What is the problem you're trying to solve.
 
Modifications:

Describe the modifications you've done.
 
Result:

After your change, what will change.
```

## Sign the contributor license agreement

First of all, please read and sign [the contributor license agreement](https://docs.google.com/spreadsheet/viewform?formkey=dHBjc1YzdWhsZERUQnhlSklsbG1KT1E6MQ) unless your contribution is trivial such as a single line change or a typo fix.

## Checklist

Please use the following checklist before pushing your commits or issuing a pull request:

* Does your work builds without any failure when you run '`mvn test`' from shell?
* Does your work introduce any new inspector warnings?
* Do your commit messages follow the format mentioned in the 'writing a meaningful commit message' section above?
* Did you sign the contributor license agreement?

## Work with a pull request

### Issue a pull request

Pull requests should be targeted at the branch for the latest stable releases.  If the pull request is for fixing a bug which also affects an old branch like `3.x`, we recommend you to submit another pull request for that branch, too.

1. [Rebase](http://git-scm.com/book/en/Git-Branching-Rebasing) your changes against the upstream branch.  Resolve any conflicts that arise.
1. Write JUnit test cases if possible. If not sure about how to write one, ask us how to write one.
1. Run `mvn test` before the initial submission or the subsequent pushes, and ensure the build succeeds.

### Merge a pull request

A pull request often contains multiple comments.  Those commits must be squashed into a small number of commits with explanatory comments.

Do not merge a pull request via the web UI unless there's a good reason doing that. Refer to the 'Patch and Apply' section in [the Github help page](https://help.github.com/articles/using-pull-requests#merging-a-pull-request).

## Update the official web site

### Write a new post

Our web site is generated from an Awestruct project [here](https://github.com/netty/netty-website).  To post a new article, create a new branch, write a new post, and issue a pull request like usual contributions.

## Release a new version

...

## Design ideas

* [[Thread model in 4.x|Thread model]]
