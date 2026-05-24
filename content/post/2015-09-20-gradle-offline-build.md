---
date: 2015-09-20T08:38:00.000Z
lastmod: 2017-07-30T08:40:44.000Z
title: Gradle Offline Build
draft: false
slug: gradle-offline-build
tags: ["Java","Gradle","Offline build"]
image: /content/images/2017/07/gradle-logo.png
description: 
---

# The Problem

In a recent project, I have to do Gradle build on a CI server in a very constrained environment. The constraints are:

- NO Internet connection
- NO Gradle installed
- NO local Maven repo

The obvious problem that these constraints cause is that the project can't build because:

- Gradle is not installed
- Can't get dependencies

# The Solution

This solution will achieve the following:

1. Use Gradle to build
2. Use Gradle to manage dependencies

Let's solve the problem step by step.

## Gradle Wrapper

Put simply, Gradle Wrapper is *the* way to let you run Gradle without installing it. It's done by running a shell/batch script to *download* Gradle binary (More on Gradle Wrapper [here](https://docs.gradle.org/current/userguide/gradle_wrapper.html)).

But the CI server has no Internet connection, how to download Gradle binary? Simply modify `distributionUrl` in `gradle-wrapper.properties` to a local path, e.g.

    # Sample gradle-wrapper.properties
    # Other properties omitted
    distributionUrl=gradle-2.4-bin.zip
    

And put `gradle-2.4-bin.zip` (You can get all Gradle distributions from [here](https://services.gradle.org/distributions)) in `${projectDir}/gradle/wrapper`. You should add this file in source control.

From now on, you can build using Gradle without Gradle installed, e.g. (assuming on Mac or Linux) `./gradlew clean build`.

## Local Dependencies

To let Gradle manage dependencies and smartly load them from the right place, we need a way to store the dependencies and load them conditionally.

In `build.gradle`,

    ext.libs = "$projectDir/libs"
    ext.compileLib = "${libs}/compile"
    ext.runtimeLib = "${libs}/runtime"
    ext.testCompileLib = "${libs}/testCompile"
    ext.testRuntimeLib = "${libs}/testRuntime"
    
    dependencies {
        if (gradle.startParameter.isOffline()) {
            compile fileTree(dir: compileLib)
            runtime fileTree(dir: runtimeLib)
            testCompile fileTree(dir: testCompileLib)
            testRuntime fileTree(dir: testRuntimeLib)
        } else {
            compile 'com.google.guava:guava:18.0'
            testCompile group: 'junit', name: 'junit', version: '4.11'
        }
    }
    

By doing this, if Gradle is running in `offline` mode, it will load dependencies from local subdirectories in the project. Otherwise, it will try to get them from Gradle home or download from the Internet. I choose the parameter `offline` because I think it's the best parameter that fits the purpose. Of course, you can use other parameter.

How to store dependent jars in the local subdirectories? A simple `copyToLibs` task would do the job.

    task deleteLibs(type: Delete) {
        delete 'libs/compile'
        delete 'libs/runtime'
        delete 'libs/testCompile'
        delete 'libs/testRuntime'
    }
    
    task copyToLibs(dependsOn: 'deleteLibs') << {
        ['compile', 'runtime', 'testCompile', 'testRuntime'].each { scope ->
            copy {
                from configurations.getByName(scope).files
                into "${libs}/${scope}"
            }
        }
    }
    

To make this whole thing work, when developing on your local machine, before pushing to Git server, you should run `copyToLibs` to make sure that the required dependencies are copied to the local subdirectories and add those dependencies to Git.

On the CI server, run `./gradlew --offline clean build`. Your tasks may vary but make sure that `--offline` is passed in because that's the way to tell Gradle to find dependencies from local dirs, instead of Gradle home or the Internet.

## Summary

By using Gradle Wrapper and storing dependencies in local directories of the project, we can still build and manage dependencies using Gradle. I think this is a simple and elegant solution. If you know a better one, leave a comment or drop me an email (me [at] this domain).

A demo app is available [here](https://github.com/ChengLong/GradleOfflineBuildDemo).
