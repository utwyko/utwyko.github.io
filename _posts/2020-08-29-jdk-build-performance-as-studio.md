---
layout: post
title:  "Comparing Build Performance between JDKs in Android Studio"
date:   2020-08-29
categories: Dev
tags: [kotlin, android, mobile]
---

By default Android Studio builds projects using its embedded JDK8. IntelliJ IDEA product manager Dmitry Jemerov says [you should use the embedded JDK in Android Studio][yole]. My typical workflow while developing is:
- Write code in Android Studio.
- Verify the result by either running the app or running a specific set of tests in Android Studio.
- If everything is OK, run all tests and the linter through the terminal.

In the terminal I use the JDK set as `JAVA_HOME` by [jenv][jenv]. I want my JDK setup decoupled from Android Studio, using the embedded JDK as the default Java is not an option.

Using two different JDKs creates two separate [Gradle Daemons][daemon]. The Daemon contains the build cache, so one JDK cannot use the build cache of the other JDK.
In this case, running all tests and the linter cannot reuse the build cache from the embedded JDK so this step takes a lot longer than necessary.

Using the same JDK in both Android Studio and the terminal solves this, although this goes against Dmitry's recommendation. The project I work on is compatible with JDK14, so for a few weeks I used JDK14 in both places. I had the feeling that builds run in Android Studio were taking longer than with the embedded JDK, so I ran a quick, non conclusive experiment comparing the build performance of Android Studio (see test setup below). 

My simple test does not show any significant difference between JDKs. So until there is a good reason to switch back to the embedded JDK, I will continue using JDK14 in Android Studio.

{% include jdk_comparison_bar_chart.html %}
(Values are in seconds.)

### Test Setup
I rebuilt the project from Android Studio using each JDK six times. I excluded the results of the first rebuild after switching JDKs as rebuilding reuses some of the previous builds config.

#### Project
- 37 gradle modules
- KAPT with Dagger
- 90% Kotlin, 10% Java
- Jetifier enabled 

#### Machine
- MacBook Pro (15-inch 2018)
- Processor: 2,5 GHz 6-core Intel  Core i7
- Memory: 16GB 240MHz DDR4
- Graphics: Intel UHD Graphics 650 1536 MB
- OS: macOS Catalina 10.15.6

#### Android Studio
Version 4.0.1.
Custom VM options:
```
-Xms1G
-Xmx2048m
-XX:MetaspaceSize=512m
```

[yole]: https://stackoverflow.com/a/52914187/2999211
[jenv]: https://github.com/jenv/jenv
[daemon]: https://docs.gradle.org/current/userguide/gradle_daemon.html