# Maven Release Tutorial

Here are some notes on the steps to take to distribute an artifact on Maven Central

## 1: Add required information to the project `pom.xml`

In addition to the typical dependencies and build sections add the following info to the pom.

```
...
<name>A human readable project name</name>
<description>A human readable description</description>
<url>The project homepage (e.g. its public source control repository page)

<licenses>
  <license>
    <name>The common license name, e.g. MIT, Apache 2, etc</name>
    <url>Where the license text is located, could be a link to the licens included in the source control</url>
    <distribution>repo</distribution>
  </license>
</licenses>

<!--
   Source Control Management for linking the source to the distributed artifcat
   See https://maven.apache.org/pom.html#SCM
-->
<scm>
  <!-- e.g. scm:git:https://github.com/
  <!-- for read access -->
  <connection>scm:SOURCE_CONTROL_IMPL:SOURCE_REPO_URL</connection>
  <!-- for write access -->
  <developerConnection>scm:SOURCE_CONTROL_IMPL:SOURCE_REPO_URL</developerConnection>
  <tag>the tag identifying the commit to release</tag>
</scm>

<!-- at least one Developer is required -->
<developers>
  <developer>
    <id>developer id</id>
    <name>developer name</name>
    <email>developer email</email>
    <organization>some org</organization>
    <roles><role>developer</role></roles>
    <timezone>time zone offset</timezone>
  </developer>
</developers>

<!--
  Deploy to the central repository
-->
<distributionManagement>
  <snapshotRepository>
    <id>ossrh</id>
    <url>https://oss.sonatype.org/content/repositories/snapshots</url>
  </snapshotRepository>
  <repository>
     <id>ossrh</id>
     <url>https://oss.sonatype.org/service/local/staging/deploy/maven2/</url>
  </repository>
</distributionManagement>
...
```

## 2. Add maven plugins for managing releases

```
...
<build>
  <plugins>
    ...
    <plugin>
      <groupId>org.sonatype.plugins</groupId>
      <artifactId>nexus-staging-maven-plugin</artifactId>
      <version>1.6.7</version>
      <extensions>true</extensions>
      <configuration>
        <!-- should match server entry in settings.xml -->
        <serverId>ossrh</serverId>
        <nexusUrl>https://oss.sonatype.org/</nexusUrl>
        <autoReleaseAfterClose>true</autoReleaseAfterClose>
      </configuration>
    </plugin>
    
    <plugin>
      <groupId>org.apache.maven.plugins</groupId>
      <artifactId>maven-release-plugin</artifactId>
      <version>2.5.3</version>
      <configuration>
        <autoVersionSubmodules>true</autoVersionSubmodules>
        <useReleaseProfile>false</useReleaseProfile>
        <releaseProfiles>release</releaseProfiles>
        <goals>deploy</goals>
      </configuration>
    </plugin>
    ...
  <plugins>
...
</build>
...
```

## 3. Optionally add plugins to include sources and javadoc in the distribution

```
<build>
...
  <plugins>
  ...
    <plugin>
        <groupId>org.apache.maven.plugins</groupId>
        <artifactId>maven-source-plugin</artifactId>
        <version>2.2.1</version>
        <executions>
            <execution>
                <id>attach-sources</id>
                <goals>
                    <goal>jar-no-fork</goal>
                </goals>
            </execution>
        </executions>
    </plugin>
    
    <plugin>
        <groupId>org.apache.maven.plugins</groupId>
        <artifactId>maven-javadoc-plugin</artifactId>
        <version>2.9.1</version>
        <executions>
            <execution>
                <id>attach-javadocs</id>
                <goals>
                    <goal>jar</goal>
                </goals>
            </execution>
        </executions>
    </plugin>
  ...
  </plugins>
...
</build>  
```

## 4. Sign the distributed artificat

If you don't already have a PGP key then [create one](http://central.sonatype.org/pages/working-with-pgp-signatures.html) with 

```
gpg --gen-key
```

Distribute the public key [here](https://pgp.mit.edu/). 
You can [create the "ASCII-armored" PGP key](https://access.redhat.com/documentation/en-US/Red_Hat_Enterprise_Linux/4/html/Step_by_Step_Guide/s1-gnupg-export.html) with

```
gpg --armor --export $key_email_or_uid
```

Add the maven plugin to automatically sign the distributed artifact

```
<build>
  <plugins>
  ...
    <plugin>
        <groupId>org.apache.maven.plugins</groupId>
        <artifactId>maven-gpg-plugin</artifactId>
        <version>1.5</version>
        <executions>
            <execution>
                <id>sign-artifacts</id>
                <phase>verify</phase>
                <goals>
                    <goal>sign</goal>
                </goals>
            </execution>
        </executions>
    </plugin>
    ...
  </plugins>
  ...
</build>
```

## 5. Create a repository in [OSSRH](https://central.sonatype.org/pages/ossrh-guide.html#initial-setup). 
This could take a few minutes to a few days depending on how quickly someone responds to your request.

## 6. Add the central repository and your JIRA credentials to your local `settings.xml` 
(probably at `$HOME/.m2/settings.xml`)

```
<settings xmlns="http://maven.apache.org/SETTINGS/1.0.0"
      xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
      xsi:schemaLocation="http://maven.apache.org/SETTINGS/1.0.0
                          https://maven.apache.org/xsd/settings-1.0.0.xsd">
  ...                          
  <servers>
    ...
    <server>
      <!--
        http://central.sonatype.org/pages/ossrh-guide.html
        http://central.sonatype.org/pages/apache-maven.html
      -->
      <id>ossrh</id>
      <!-- JIRA credentials -->
      <username>username</username>
      <password>password</password>
    </server>
    ...
  </servers>
  ...
</settings>

```

## 7. [Release](https://central.sonatype.org/pages/apache-maven.html#performing-a-release-deployment-with-the-maven-release-plugin)

First make sure your changes are all committed and tests are passing, then follow the prompts while running
```
mvn release:clean release:prepare
```

If you see an error with `gpg` try running `export GPG_TTY=$(tty)` prior to invoking the above command

Then to actually upload the artifacts run

```
mvn release:perform
```

## 8. Check for your artifact in maven central

It may take a while for the artifact to be available from [search](https://search.maven.org/#search), but you can browse for
your release by group id [here](https://repo1.maven.org/maven2/)

# Appendix

Some helpful links

* [Maven pom.xml reference](https://maven.apache.org/pom.html)
* [Maven central repository reference](http://central.sonatype.org/pages/apache-maven.html)
* [Sonatype JIRA login](https://issues.sonatype.org/secure/Dashboard.jspa)
* [GNU Privacy Guard reference (gpg)](https://access.redhat.com/documentation/en-US/Red_Hat_Enterprise_Linux/4/html/Step_by_Step_Guide/ch-gnupg.html)
* [Maven versions plugin](https://www.mojohaus.org/versions-maven-plugin/)

