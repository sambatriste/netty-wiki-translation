## Before getting started

* [Read the OSSRH guide from Sonatype.](http://central.sonatype.org/pages/ossrh-guide.html)
* [Create a GnuPG key to sign the artifacts.](https://docs.sonatype.org/display/Repository/How+To+Generate+PGP+Signatures+With+Maven)
* Configure your `~/.m2/settings.xml` contains the following configuration:

    ```xml
    <?xml version="1.0" encoding="UTF-8"?>
    <settings>
      <servers>
        <server>
          <id>sonatype-nexus-snapshots</id>
          <username>myusername</username>
          <password>mypassword</password>
        </server>
        <server>
          <id>sonatype-nexus-staging</id>
          <username>myusername</username>
          <password>mypassword</password>
          <configuration>
            <httpHeaders>
              <!--
                Override User-Agent header which is used when deploying an artifact
                so that Sonatype Nexus does not create multiple staging repositories
                when artifacts are deployed from different platforms (e.g. Linux and OSX).
              -->
              <property>
                <name>User-Agent</name>
                <value>Apache-Maven</value>
              </property>
            </httpHeaders>
          </configuration>
        </server>
      </servers>
      ...
    </settings>
    ```

## Standard Maven release procedure

You must be familiar with the standard Maven release procedure, which uses [maven-release-plugin](http://maven.apache.org/maven-release/maven-release-plugin/):

1. [Stage the new release](https://docs.sonatype.org/display/Repository/Stage+a+Release) into the staging repository.
1. Verify the staged files are all good.  If not, [drop the staging repository](https://docs.sonatype.org/display/Repository/Dropping+a+Staging+Repository) and try again.
1. [Close the staging repository](https://docs.sonatype.org/display/Repository/Closing+a+Staging+Repository) so that no more modifications are made into the staging repository.
1. [Release the staging repository](https://docs.sonatype.org/display/Repository/Releasing+a+Staging+Repository) so that the new release is synchronized into the Maven central repository.

## Netty 3

Following the standard Maven release procedure should be enough.

## Netty 4 and beyond

The release procedure must be performed from 64-bit [RHEL 6.5 or its derivatives] so that we can easily ensure the ABI compatibility of the native libraries we ship.

## [[Netty/TomcatNative|Forked-Tomcat-Native]]

Because we ship both Linux and Mac OS X artifacts, the release procedure is even more complicated.

1. Perform a release from 64-bit [RHEL 6.5 or its derivatives] like you did for Netty 4.  However, do not close the staging repository just yet.
1. On your Mac OS X and 64-bit Windows, whose build environment is set up as documented [[here|Forked-Tomcat-Native]], run the following commands to deploy the release artifacts to the staging repository:

    ```
    $ git checkout netty-tcnative-[version]
    ... You'll get a warning about detached HEAD ...
    $ mvn -Psonatype-oss-release clean deploy
    ... The artifact with OSX or Windows native library will be deployed ...
    ```

1. Make sure both the Linux, Mac OS X, and Windows artifacts have been deployed into the same staging repositories.  If they are deployed into two or three different staging repositories, drop them all and figure out what was the problem.
1. If all three JARs (e.g. `netty-tcnative-1.1.30.Fork1-linux-x86_64.jar`, `netty-tcnative-1.1.30.Fork1-osx-x86_64.jar`, and `netty-tcnative-1.1.30-Fork1-windows-x86_64.jar`) exist in the same staging repository, close and release the staging repository.

## Close the milestone in Github

Go [here](https://github.com/netty/netty/issues/milestones) to close the released milestone.

## Upload a tarball

We use [bintray.com](https://bintray.com/netty/downloads/netty/view) for hosting tarballs.

1. Log in with your account.
1. [Add a new version](https://bintray.com/netty/downloads/netty/new/version).
   1. Type the version number and release date.  Note that the release date format is `dd/mm/yyyy`.
   1. Click the 'Create Version' button.
1. Put additional information to the newly created version.
   1. Type the name of the Git tag in the 'VCS tag' field. (e.g. `netty-4.1.0.Beta1`) 
   1. Click the 'Update Version' button.
1. Go to [the download page](https://bintray.com/netty/downloads/netty/view).
1. Click the link to the version you've just created, which will lead you to the version page.
1. Upload the tarball file.
   1. Click the 'Upload Files' link.
   1. Upload the tarball file (e.g. `netty-4.1.0.Beta1.tar.bz2`) by dragging and dropping the file into the 'Drag files here' area.
   1. Click the 'Save Changes' button once the upload is finished.
1. Publish the tarball file.
   1. Bintray will warn you that you have 1 unpublished item. Click the 'Publish' button.
1. Add the published tarball to the download list.
   1. You've uploaded the tarball successfully, but the page will say 'No direct downloads selected for this package.'  It is because you did not make the tarball file visible in the download list.
   1. Click the 'Files' tab.  You will see the tarball file you've uploaded.
   1. Move your mouse cursor over the 'Actions' link of the tarball file, which will display a submenu.
   1. Click the 'Show in download list'.
   1. Now go back to the 'General' tab. Your tarball should be visible in the direct download list.

## Update the 'new and noteworthy' page

We have a dedicated 'new and noteworthy' page for each minor versions.  Add or update it if necessary.  If you added one, add it also to [[New and noteworthy]] list.

## Update the web site

Our official web site is built with [Awestruct](http://awestruct.org/). Please make sure you are familiar with Awestruct and its related markup languages such as [HAML](http://haml.info/) and Markdown.

If you did not yet, clone the web site project:

```
git clone git@github.com:netty/netty-website.git
cd netty-website
```

`_config/site.yml` contains important metadata about the latest Netty versions.  For example, it has the following section:

```
releases:
  - version: 5.0.0.Alpha1
    date: 22-Dec-2013
    stable: false
    branch: master
  - version: 4.0.21.Final
    date: 01-Jul-2014
    stable: true
  - version: 3.9.2.Final
    date: 11-Jun-2014
    stable: true
```

Update it with the new version number and release date.  You could also update the stability and the Git branch if necessary.

Try to generate the web site to confirm that the new version shows up in the generated web site.  The generated web site is located in the `_site` directory and you can browse it from your favorite web browser by opening `_site/index.html`.

```
_bin/clean.sh
_bin/generate.sh
```

Update the Javadoc and Xref to the latest one.  You can copy them from your `netty/target/checkout/all/target` directory:

```bash
cd 4.1 # or the version you are working on
rsync -aiP --delete ~/Workspace/netty/target/checkout/all/target/api .
rsync -aiP --delete ~/Workspace/netty/target/checkout/all/target/xref .
git add -A .
cd ..
```

Also, don't forget to add `_config/site.yml` we updated to the Git index when you commit:

```
git add _config/site.yml
git commit -m "Release 4.1.0.Beta1"
git push
```

Now you are ready to push the web site:

```
_bin/deploy.sh
```

## Make an announcement

Write a blog post.  The blog posts are located in the `news` directory of the web site project.

Make sure the file name does not contains a dot (.) or a whitespace (e.g. OK: `2014-07-04-4-1-0-Beta1-released.html.md` NOT OK: `2014-07-04-4.1.0.Beta1-released.html.md`

Deploy the new blog post using the `_bin/deploy.sh` script.

Once the new blog post is up, make another announcement via your personal Twitter account, and retweet it using our official Twitter account.

[RHEL 6.5 or its derivatives]: http://en.wikipedia.org/wiki/Red_Hat_Enterprise_Linux_derivatives
