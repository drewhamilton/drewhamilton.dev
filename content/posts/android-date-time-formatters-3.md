---
title: "`AndroidDateTimeFormatters` 3.0"
date: 2026-02-01T17:00:00-06:00
draft: true
---

[`AndroidDateTimeFormatters`](https://github.com/drewhamilton/AndroidDateTimeFormatters)
went a long time without needing an update. It formats times in either a 12- or
24-hour style, depending on the system preference—what could change? But it
built up a pile of unmerged dependency update PRs that was starting to make me
anxious, so I decided to revisit the project to see whether it needed anything
new, or whether I could shut off Renovate for good.

The Android API already provides a way to format clock times in the preferred
style: `android.text.format.DateFormat.getTimeFormat` returns a
`java.text.DateFormat` instance that does this. But there was no way to do it
in modern date/time libraries like `java.time` or ThreeTenBP.
`AndroidDateTimeFormatters` was created to fill that gap. ThreeTenBP's utility
on Android faded over time as [core library desugaring](https://developer.android.com/studio/write/java8-support#library-desugaring)
took over, so the `java.time` version of `AndroidDateTimeFormatters` was the
main remaining use case.

What I discovered upon revisiting this project after a few years was that
Android's desugared `java.time` formatters now do what
`AndroidDateTimeFormatters` was created to do: they format short times based on
the user's preferred 12- or 24-hour style. Android app developers now get this
behavior in desugared `java.time` formatters by default, without the need to
import or use `AndroidDateTimeFormatters`.

So is there any point in using `AndroidDateTimeFormatters` anymore? As it turns
out, there is! If an app's `minSdk` is 26 or higher, `java.time` isn't
desugared, because it's already included in the OS. And the version of
`java.time` that comes preinstalled on Android 26+—unlike the desugared
version of `java.time`—*doesn't* use the system clock style preference. It uses
the formatting locale's standard preference, regardless of the user's
preference, like the desugared version used to do back when I first published
`AndroidDateTimeFormatters`.

So `AndroidDateTimeFormatters` is still useful after all, but only when
`java.time` is used without desugaring, i.e. when an app's `minSdk` is 26+.
Today I'm releasing `AndroidDateTimeFormatters` 3.0 with a new `minSdk` of 26.

`AndroidDateTimeFormatters` 3.0 also enhances its behavior: The system clock
preference is now respected not just for `FormatStyle.SHORT` times, but for
`MEDIUM`, `LONG`, and `FULL` styles as well. It's also rewritten in Kotlin, but
retains full binary compatibility with the previous 2.x releases.
