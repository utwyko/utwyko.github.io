---
layout: post
title:  "Introducing Kotlin Multiplatform incrementally"
date:   2020-08-14
categories: Dev
tags: [kotlin, multiplatform, mobile]
---

You can use Kotlin for cross-platform development using [Kotlin Multiplatform][mp]. Its approach differs from other cross-platform solutions such as Flutter and React Native: you can add it to existing codebases and it does not replace all platform-native code. 

At Blendle, we wanted to experiment with Kotlin Multiplatform on the Android, Web and iOS clients. Using a tool to build an actual production feature is usually the quickest way to evaluate a tool, so we tried to figure out the easiest and fastest way to do this. The setup also needed to be easily removeable. This was especially important for the web/iOS developers. Merging Multiplatform code back into the Android app is trivial as the majority of the Android codebase was already written in Kotlin, but this is not the case for the other platforms.

We evaluated several approaches. I recommend checking out [this talk by Ben Asher and Alec Strong][kcmp] for an in depth evaluation of these approaches:
- Merge Android/Web/iOS codebases into a single Git repository (a mono-repo), add multiplatform code there.
- Duplicate the Multiplatform code in each client codebase.
- Create a separate repo for Multiplatform code.

We created a separate repo with its own CI pipeline. This isolated all Multiplatform code, apart from the integration into the clients, into a removable component. 

The first feature we build in Multiplatform was email verification: when a user signs-up with a suspicious email address (e.g. `user@gnail.com`) we suggest a corrected email address before the user confirms their registration. A simplified version of this already worked: we fetched a verification regex from the backend, but we were not satisfied with that implementation. Email verification was a perfect use-case for Multiplatform because:
* The logic should be identical for each client.
* The email address should be verified after each key stroke. Doing the verification server-side requires a lot of traffic and adds latency.
* It is easily testable in isolation.

After writing the Multiplatform code, we integrated the library into the Android, Web and iOS clients.

On Android, we assembled a library JAR by running the gradle `assemble` task. The Android repository included the output as a local library. This is suboptimal as releasing new versions requires quite some manual work and it increases the size of the Android git repo. However, we could start integrating the Multiplatform code within a few minutes.

On iOS we started out with a similar approach by producing a `.framework` file and including that in the iOS repository. This did not work as iOS requires different frameworks for different processor architectures. So we added a Gradle task producing a Fat Framework supporting both architecture ([see this post for details][fat_framework]). We did not want to use the Fat Framework for release builds though as it increases the size of the binary, so we created a BuildPhase in the iOS build that chooses the correct framework based on the build configuration (FAT for debug or the proper processor architecture framework for release builds).

On Web, we could not get the setup to work. The functions in the `commonMain` module were not properly exported to the Javascript outputs and the output file size was too large to include for such a simple feature. Since there are so many [changes to targeting Javascript with Multiplatform][kotlin_14m1] in the upcoming Kotlin 1.4 we decided to postpone the integration with Web until after the Kotlin 1.4 release.

Apart from Web, we were quite happy with the resulting setup. Using the Multiplatform Email Verification API from iOS and Android was easy and we were even able to use a [Sealed Class][sealed_class] as a result type in the API. Since the first iteration we've build several more features in the Multiplatform library, with a majority being build in Kotlin by iOS developers! We want to make improvements to the setup, such as:

* Pushing the JVM binaries to an artifactory. We do not use a custom artifactory at Blendle but GitHub actions (which we already use) offers [an easy way to publish packages][actions_packages].
* Including the Multiplatform repository into the Android build process for debug builds so that we can test changes to the Multiplatform code without having to produce a JAR.

[mp]: https://www.jetbrains.com/lp/mobilecrossplatform/
[kcmp]: https://www.youtube.com/watch?v=je8aqW48JiA
[fat_framework]: https://medium.com/@saschpe/kotlin-multiplatform-fat-framework-for-ios-cdd05ec479cb
[actions_packages]: https://docs.github.com/en/packages/learn-github-packages/publishing-a-package
[kotlin_14m1]: https://blog.jetbrains.com/kotlin/2020/03/kotlin-1-4-m1-released/
[sealed_class]: https://kotlinlang.org/docs/sealed-classes.html
