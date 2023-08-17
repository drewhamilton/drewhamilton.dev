---
title: "Poko goes multiplatform"
date: 2023-08-09T12:55:00-07:00
draft: false

external: true
externalUrl: https://code.cash.app/poko-multiplatform
---

[Poko](https://github.com/drewhamilton/Poko) is a Kotlin compiler plugin that generates `equals`,
`hashCode`, and `toString` functions for annotated classes based on their properties. Inspired by
Jake Wharton’s [blog post on maintaining compatibility in public
APIs](https://jakewharton.com/public-api-challenges-in-kotlin), it partially mimics the behavior of
Kotlin data classes while leaving out the `copy` and `componentN` functions, which are difficult to
maintain without breaking API compatibility. Our [Paraphrase](https://github.com/cashapp/paraphrase)
and [Redwood](https://github.com/cashapp/redwood) runtimes use Poko.

The `equals`, `hashCode`, and `toString` functions are familiar to all JVM developers. But Kotlin
generates the exact same set of functions for data classes on all of its multiplatform targets—not
just the JVM. It stands to reason that Poko should do the same. And thanks to a slew of recent
contributions from Jake, now it does! In addition to the JVM and Android platforms it has supported
since its first release 3 years ago, Poko 0.15.0 now supports native and JS targets as well.

The generated functions behave the same on all platforms, and by default implement the same behavior
that data classes implement: `equals` and `hashCode` depend on each of the class’s primary
constructor properties, and `toString` prints the values of each of these properties.

In addition to standard data class behavior, Poko also recently added support for respecting array
content via an `@ArrayContentBased` annotation. Normally, data classes and Poko classes compare
array properties only by reference (unlike collections, which always compare contents), because this
mirrors the JVM’s default treatment of arrays. But in some cases, the performance gains of using
arrays are considerable—particularly on JS, where `List` types involve several wrappers around the
native `Array`. In runtimes like those of Paraphrase and Redwood, which are only invoked by
generated code, the awkwardness of using array properties is less important than this performance
difference. `@ArrayContentBased` makes it possible to opt in to array use in these cases. Like the
rest of Poko, it now works for all target platforms.

Whether using arrays or not, it’s important for library authors to maintain API compatibility across
releases. Now, Kotlin multiplatform library authors can easily ship public data models with minimal
concern about breaking their APIs.

_This post concludes Cash App’s [Summer of Kotlin
Multiplatform](https://code.cash.app/kotlin-multiplatform-summer) series._