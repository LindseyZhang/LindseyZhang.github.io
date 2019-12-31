---
layout: post
title: 在 Java 代码中调用 Gradle Task
categories: [Tech]
tags: [gradle-tooling-api]
description: gradle 除了提供强大的依赖管理方式，同时拥有完善的生态，大量即插即用的 plugin 提供了大量好用的方法，如代码风格检查，生成契约测试等。
---





Gradle tooling api 是[官方](https://docs.gradle.org/current/userguide/embedding.html提供的一种可以在程序中调用 gradle   task  的方式。

例子：

https://github.com/gradle/gradle/tree/master/subprojects/docs/src/samples/toolingApi