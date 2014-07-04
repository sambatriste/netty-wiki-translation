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

[RHEL 6.5 or its derivatives]: http://en.wikipedia.org/wiki/Red_Hat_Enterprise_Linux_derivatives
