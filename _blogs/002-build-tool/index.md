+++
title = "What are build automation tool ? And a closer look at the Maven and POM file"
description = "A beginner’s guide to build tools, Maven and POM file"
date = 2020-04-05T01:24:33+05:30

[taxonomies]
tags = ["build-tools", "maven", "pom"]

[extra]
toc = true
+++

![](https://cdn-images-1.medium.com/max/7000/1*VW7pDgspmF41A4AIZTB__g.jpeg)

Let us understand the build automation tool and it’s process through addressing the series of questions

## What are Build Automation Tool?

It is a process of automating an extensive range of tasks that one has to do in their day-to-day activity, right from source code to end-product

The overall process includes - downloading dependencies, compiling source code into binary code (machine-readable format), packaging binary code, running automated tests, code coverage, static code analysis, creating or updating database schema (migration), packaging the code into an executable format, deploying to production/testing/UAT environments, generate documentation out of source code.

## Different types of build tools

- Java - Ant, Maven, Gradle

- Ruby - Rake

- Javascript - Gulp, Grunt, Broccoli

- C# / .NET - Nant, MS Build

- Language agnostic - Make, Bazel, Buck

## What is Maven? How it works ?

It is one of the Build automation tool from the Java community based on POM (project object model). It is an XML file that has information about the project and configuration details used by Maven to build the project.

![](https://cdn-images-1.medium.com/max/2304/1*bSap54mpkjDBrzVJGXAhXw.jpeg)

**Maven build Lifecycle**

Maven build lifecycle goes through a series of stages, called as build **_phases_**. For example, the default lifecycle is

```
Validate
Compile
Test
Package
Verify
Install
Deploy
```

- **validate** — validate the project is correct and all necessary information is available. Also makes sure the dependencies are downloaded.

- **compile** — compile the source code of the project.

- **test** — runs the tests against the compiled source code using a suitable unit testing framework. These tests should not require the code be packaged or deployed.

- **package** — take the compiled code and package it in its distributable format, such as a JAR.

- **verify** — run any checks on results of integration tests to ensure quality criteria are met.

- **install** — install the package into the local repository for use as a dependency in other projects locally deployed.

- **deploy** — done in an integration or release environment, copies the final package to the remote repository for sharing with other developers and projects.

There are two other Maven lifecycles of note beyond the _default_ list above. They are

- **clean**: cleans up artifacts created by prior builds

- **site**: generates site documentation for this project

These phases executed sequentially when we run a maven build command.

**POM file**

{{ gist(url="https://gist.github.com/madhank93/53f5a4c138617f221b73a0488ca13aa4", class="gist") }}

Let’s break this POM file

**Project:**

`<project>` - is the root element and has the attributes XML namespace and Maven XML schema

`<modelVersion>` - element defines the POM version, currently the only supported value is `4.0.0`.

**GroupID, artifactID and version:**

`<groupId>` - element usually takes the unique ID of an organization, or a project. In this case I have used `io.github.madhank93`

`<artifactId>` - element has the name of the project `maventraining`

`<version>` - element contains the version number of the project `0.0.1-SNAPSHOT`

The above `groupId` and `artifactId` combination must be unique. Since this will be used in identifying the project in repository.

**Package:**

`<packaging>` - which specifies the type of artifact the Maven should produce

**Properties:**

Properties are serves as a placeholder. These values can be accessed anywhere within a POM file. By using the notation `${element_name}`or it can be used by plugins as default values

`<project.build.sourceEncoding>` - it is a property recognized by Maven.

**Plugins:**

Plugins are where much of the real action is performed, plugins are used to: create jar files, create war files, compile code, unit test code, create project documentation, and on and on. Almost any action that you can think of performing on a project is implemented as a Maven plugin.

In this article I have used `maven-compiler-plugin` , this compiler plugin is used to compile the source code of a Maven project. By default, it compiles with Java 5. We can manage the settings in `configuration` element, in this case, it is instructed to build artifacts to compatible with Java 1.8.

**Dependencies:**

There are two types of Maven dependencies:

- **Direct:** These dependencies defined in your pom.xml file under the <dependencies> section.

- **Transitive:** These are dependencies required by our direct dependencies

`<dependencies>` - this tag specifies the additional libraries that are needed for the project

`<scope>` - element indicates this dependency is only applicable for the testing phase of the Maven lifecycle.

[Maven central repository](https://mvnrepository.com/repos/central) is the default location to download all the dependent libraries. One of the dependency that used is in this article is [selenium-java](https://mvnrepository.com/artifact/org.seleniumhq.selenium/selenium-java/3.141.59).

<div align="center">* * * *</div>

<center>

Originally published on [Medium](https://medium.com/@madhankumaravelu93/what-are-build-automation-tool-and-a-closer-look-at-the-maven-and-pom-file-7b209a8a6c61)

</center>

# References

[1] [https://maven.apache.org/guides/getting-started/index.html](https://maven.apache.org/guides/getting-started/index.html)

[2] [https://maven.apache.org/guides/introduction/introduction-to-the-lifecycle.html](https://maven.apache.org/guides/introduction/introduction-to-the-lifecycle.html)

[3] [https://www.baeldung.com/maven](https://www.baeldung.com/maven)

[4] [http://tutorials.jenkov.com/maven/maven-tutorial.html](http://tutorials.jenkov.com/maven/maven-tutorial.html)
