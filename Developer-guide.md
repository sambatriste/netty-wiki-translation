## Working with the source code

### Important!

Before pushing your commits or issuing a pull request, make sure:

* your work builds without any failure - run '`mvn test`' from shell.
* your work does not introduce any new inspection warning - see below.

### Setting up IntelliJ IDEA inspection profile

Netty project team uses [IntelliJ IDEA](http://www.jetbrains.com/idea/) as the primary IDE. Download, unzip, and import [this inspection profile](http://netty.io/files/IntelliJ%20IDEA%20Inspection%20Profile.xml.zip) into your IntelliJ IDEA and use it as the default for Netty project. Ensure your modification does not introduce any inspection warning. If you think it's a false positive, suppress the warning by using the `@SuppressWarnings` annotation or `noinspection` line comment as guided by the IDE.  For more information about using the inspector, please refer to [the web help pages](http://www.jetbrains.com/idea/webhelp/inspecting-source-code.html).

### Git line ending configuration

We use native line ending for all source code (i.e. '`\r`' for *nix and MacOS X, '`\r\n`' for Windows.) Please configure your Git installation so that your build does not fail and you push bad files, following the instructions below:

* [Dealing with line endings](https://help.github.com/articles/dealing-with-line-endings) by Github
* [Mind the End of Your Line](http://timclem.wordpress.com/2012/03/01/mind-the-end-of-your-line/) by Tim Clem, for more information

### Merging a pull request

A pull request often contains multiple comments.  Those commits must be squashed into smaller number of commits with explanatory comments.

Do not merge a pull request via the web UI unless there's a good reason doing that. Refer to the 'Patch and Apply' section in [the Github help page](https://help.github.com/articles/using-pull-requests#merging-a-pull-request).

### Design ideas

* [[Thread model in 4.x|Thread model]]
