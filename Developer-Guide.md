This page explains the typical workflow that a developer follows to make a contribution to the Netty project.

## How to build

### Check out the source code

Netty project uses Git to manage its source code.  If you are not familiar with Git, you will find the book '[Pro Git](http://progit.org/book/)' a very good starting point.

The source code repository can be cloned from the anonymous read-only repository with the following command:

    $ git clone git://github.com/netty/netty.git netty

Please note, if you are going to contribute to the project, you have to [fork the repository](http://help.github.com/forking/), clone the fork into your local disk, commit and push your work to your fork, and [send a pull request](http://help.github.com/pull-requests/):

    $ git clone git@github.com/<username>/netty.git netty

### Update the global Maven settings

Netty project uses [Apache Maven](http://maven.apache.org/) for builds.  Since it requires some artifacts that is not available in the central Maven repository, you need to configure your global Maven settings file (`~/.m2/settings.xml`) like the following:

    <?xml version="1.0" encoding="UTF-8"?>
    <settings>
      <profiles>
        <profile>
          <id>jboss-nexus</id>
          <repositories>
            <repository>
              <id>jboss-public-repository-group</id>
              <name>JBoss Public Repository Group</name>
              <url>http://repository.jboss.org/nexus/content/groups/public/</url>
              <layout>default</layout>
              <releases>
                <enabled>true</enabled>
                <updatePolicy>never</updatePolicy>
              </releases>
              <snapshots>
                <enabled>true</enabled>
                <updatePolicy>never</updatePolicy>
              </snapshots>
            </repository>
          </repositories>
          <pluginRepositories>
            <pluginRepository>
              <id>jboss-public-repository-group</id>
              <name>JBoss Public Repository Group</name>
              <url>http://repository.jboss.org/nexus/content/groups/public/</url>
              <releases>
                <enabled>true</enabled>
              </releases>
              <snapshots>
                <enabled>true</enabled>
              </snapshots>
            </pluginRepository>
          </pluginRepositories>
        </profile>
      </profiles>

      <activeProfiles>
        <activeProfile>jboss-nexus</activeProfile>
      </activeProfiles>
    </settings>

Once the `settings.xml` file has been configured, you have to set up the `MAVEN_OPTS` environment variable to avoid `OutOfMemoryError` during the build.  (You will probably want to set it up in your shell profile such as `~/.bash_profile` or `~/.profile`.)

    $ export MAVEN_OPTS="-server -ea:org.jboss.netty... -Xmx512m"

### Build the distribution

You are all set now. `mvn package` command will generate a JAR (and tarballs) in the `target` directory like any other ordinary Maven projects.

## Watching project activities

If you are a regular contributor, you'd better subscribe to the [project activity feed](http://feeds2.feedburner.com/netty_project_activities) using your RSS reader.  Alternatively, you can watch the project so that the project activities appear in your Github dashboard, by clicking the 'Watch' button on the [project page](..).

## Making a contribution

1. File a [CLA (contributor license agreement)](https://cla.jboss.org/) if you did not yet.
1. File a pull request for your code changes.
1. Someone will review it, ask for additional changes, and eventually merge it once the proposed change is accepted.

Please note, even if you have the permission to push the changes directly to the upstream repository, it is advised to file a pull request and wait at least 24 hours if the change is likely to impact other people's work, so that as many people can review the change and give useful feed back.
