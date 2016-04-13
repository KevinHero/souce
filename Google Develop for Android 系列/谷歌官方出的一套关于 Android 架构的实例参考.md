---
title: 谷歌官方出的一套关于 Android 架构的实例参考
date: 2016-03-29 13:12:17
tags: 架构的实例
categories: 技术
---

原文 [github.com](https://github.com/googlesamples/android-architecture)

The Android framework offers a lot of flexibility when it comes to defining how to organize and *architect* an Android app. This freedom, whilst very valuable, can also result in apps with large classes, inconsistent naming and architectures (or lack of) that can make testing, maintaining and extending difficult.

Android Architecture Blueprints is meant to demonstrate possible ways to help with these common problems. In this project we offer the same application implemented using different architectural concepts and tools.

You can use these samples as a reference or as a starting point for creating your own apps. The focus here is on code structure, architecture, testing and maintainability. However, bear in mind that there are many ways to build apps with these architectures and tools, depending on your priorities, so these shouldn't be considered canonical examples. The UI is deliberately kept simple.

**What does *****beta***** mean?**

We're still making decisions that could affect all samples so we're keeping the initial number of variants low before the stable release.

**Samples**

All projects are released in their own branch. Check each project's README for more information.
<!--more-->
In progress:

* todo-mvp-contentproviders - Based on todo-mvp-loaders, uses Content Providers
* todo-mvp-clean - Based on todo-mvp, uses concepts from Clean Architecture.
* todo-mvp-dagger - Based on todo-mvp, uses Dagger2 for Dependency Injection

Also, see ["New sample" issues](https://github.com/googlesamples/android-architecture/issues?q=is%3Aissue+is%3Aopen+label%3A%22New+sample%22) for planned samples.

**Why a to-do application?**

The aim of the app is to be simple enough that it's understood quickly, but complex enough to showcase difficult design decisions and testing scenarios. Check out the [app's specification](https://github.com/googlesamples/android-architecture/wiki/To-do-app-specification).

[![Screenshot](https://raw.githubusercontent.com/wiki/googlesamples/android-architecture/images/tasks2.png)](https://github.com/googlesamples/android-architecture/wiki/images/tasks2.png)

Also, a similar project exists to compare JavaScript frameworks, called [TodoMVC](https://github.com/tastejs/todomvc).

**Which sample should I choose for my app?**

That's for you to decide: each sample has a README where you'll find metrics and subjective assessments. Your mileage may vary depending on the size of the app, the size and experience of your team, the amount of maintenance that you foresee, whether you need a tablet layout or support multiple platforms, how compact you like your codebase, etc.

**Who is behind this project?**

This project is made by the [community](https://github.com/googlesamples/android-architecture/graphs/contributors) and curated by Google and core maintainers. Each sample has a group of owners that look after it keeping it up to date and handling issues and pull requests.

Want to be part of it? Read [how to become a contributor](https://github.com/googlesamples/android-architecture/blob/master/CONTRIBUTING.md) and the [contributor's guide](https://github.com/googlesamples/android-architecture/wiki/Contributions)