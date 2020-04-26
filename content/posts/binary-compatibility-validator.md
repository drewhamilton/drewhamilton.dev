---
title: "Monitoring binary compatibility on a pre-stable project"
date: 2020-04-26T11:35:00+02:00
draft: true
---

The open-source Kotlin [binary compatibility
validator](https://github.com/Kotlin/binary-compatibility-validator) is a great
tool. It takes about 3 minutes (seriously!) to add it to a project, and when
enabled forces build failures on changes to the public API service. If you
acknowledge the API changes are intentional, you fix the build by changing the
readable summary of the public API in source control. If you maintain a library
that is consumed as a compiled binary, there is no good reason not to use binary
compatibility validation, and this is the easiest tool I've found to do that.

My main project at work for a few months has been a new set of Android
libraries. They're not stable yet; we aim to stabilize them within a few more
months. Last week I enabled the binary compatibility validator on the entire
project that comprises these libraries.

Why would I do this when I just said the libraries aren't stable yet? Every time
the public API changes—which is still a lot—the build fails until the developer
runs `./gradlew apiDump`.

But that's a small drawback. It's an easy, fast command to run. The benefit is
that for each library, we have a single `.api` file defining the entire
library's public API. With each PR we can check the diff on that file and be
sure the API is moving in the right direction.

At any time we can review the whole `.api` file and make sure there's nothing in
there that we don't expect or don't want. This is a big improvement on the
previous status quo which required going through each source file one-by-one—
tedious and prone to review fatigue. The initial API dump when enabling this
tool already uncovered a few things we don't want to be public and had let slip
by in previous code reviews!

With the binary compatibility validator enabled, I am more confident of two
things when we approach our stable release date later this year:
1. We won't discover API errors at the last second that cause stable release
   delays: we're catching these things now, instead, with plenty of time to fix
   them.
2. We won't accidentally release something in the public API that should not
   have been public: it's easy to review the entire public API in one file.
