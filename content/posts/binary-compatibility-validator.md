---
title: "Monitoring binary compatibility on a pre-stable project"
date: 2020-04-28T12:35:00+02:00
draft: false
---

The open-source Kotlin [binary compatibility
validator](https://github.com/Kotlin/binary-compatibility-validator) is a great
tool. When added to a project, it fails the build on any change to the public
API surface. If a public API change is intentional, `./gradlew apiDump` fixes
the build by updating the summary of the public API in source control.

If you maintain a library that is consumed as a compiled binary, it is wise to
use some form of binary compatibility validation, and this is the easiest such
tool I know of. It takes maybe 3 minutes to add it to a project, and the public
API summary it produces is easy for a developer to read.

My main project at work for a few months has been a new set of Android
libraries. They're not stable yet; we aim to stabilize them within a few more
months. Last week I enabled this binary compatibility validator on the entire
project that comprises these libraries.

Why would I do this when I just said the libraries aren't stable yet? Isn't it
annoying that the build now fails every time the public API changes (which is a
lot)? Sure, a little, but it's a small drawback. Running  `./gradlew apiDump` is
easy and fast.

The benefit is that for each library, we have a single `.api` file defining the
entire public API. With each PR we can check the diff on that file and be sure
the API is moving in the right direction. And at any time, we can review the
whole `.api` file and make sure there's nothing in there that we don't expect or
don't want. This is a big improvement on the previous status quo, wherein seeing
the entire public API surface requires going through each source file
one-by-one—tedious and prone to review fatigue. The initial API dump when
enabling this tool already uncovered a few things we don't want to be public and
had let slip by in previous code reviews!

With this binary compatibility validator enabled, I am more confident of two
things when we approach our stable release date later this year:

1. We won't discover API errors that cause stable release delays at the last
   second: we're catching these things now, instead, with plenty of time to fix
   them.
2. We won't accidentally release something in the public API that should not
   have been public: it's easy to review the entire public API in one file.

---

### Notes

Your library does not need to be written in Kotlin to use this tool, but the
Kotlin plugin does need to be applied to the Gradle module.

See [this PR](https://github.com/drewhamilton/InlineDimens/pull/21/files?file-filters%5B%5D=.api&file-filters%5B%5D=.gradle)
in one of my open-source libraries for an example of how to enable this tool.
