[netty-tcnative](https://github.com/netty/netty-tcnative) is a fork of [Tomcat Native](http://tomcat.apache.org/native-doc/).  It includes a set of changes contributed by Twitter, Inc, such as:

* Complete mavenization of the project
* Improved OpenSSL support

To minimize the maintenance burden between the upstream and the fork, we create a dedicated branch for each stable upstream release and apply our own changes on top of it, while keeping the number of maintained branches to minimum.

## Creating a new fork

Checking out the branch `bootstrap` and running the `new-fork` script with the upstream tcnative version will bring you to a new fully mavenized branch whose name is identical to the upstream tcnative version.  For example:

```
$ git checkout bootstrap
$ ./new-fork 1.1.29
```

will create a new branch called `1.1.29`, which contains a mavenized fork of `tcnative-1.1.29`.  Please note that the fork does not contain any patches that will change the main source code of `tcnative`.  You will probably want to cherry-pick some commits from other branches that were patched already.

## Using `netty-tcnative` to accelerate SSL performance

Coming soon.