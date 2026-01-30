---
title: "`AndroidDateTimeFormatters` 3.0"
date: 2026-01-29T20:30:00-06:00
draft: true
---

It's been awhile since I last published a new version of my
[`AndroidDateTimeFormatters`](https://github.com/drewhamilton/AndroidDateTimeFormatters)
library. For a long time, I didn't think it would need any more new versions; it
does what it does and that's it.

But the Renovate PRs were piling up, and I decided to revisit the project to see
whether it needed anything new, or whether I could shut off Renovate and call
the library officially complete. What I found surprised me, and in fact had me
very close to declaring the library not just complete, but deprecated.

The main purpose of `AndroidDateTimeFormatters` is to provide a way to format
time in modern date/time libraries into a string that respect's Android's system
setting that chooses between a 12- and 24-hour clock.
`android.text.format.DateFormat.getTimeFormat` provides a `java.text.DateFormat`
object that does this, but there was no way to do it with modern date/time
libraries like `java.time` or ThreeTenBP. `AndroidDateTimeFormatters` was
created to fill this gap.

Over time, ThreeTenBP's utility on Android faded after [core library desugaring](https://developer.android.com/studio/write/java8-support#library-desugaring)
made it possible to use `java.time` on Android down to a very low `minSdk`. And
what I discovered upon revisiting this project after a few years was that, at
some point during its evolution, the desugared `java.time` library started
automatically using the system 12-/24-hour clock preference to format all short
times. `AndroidDateTimeFormatters`' main purpose was now default behavior, so
it wasn't needed anymore and could be deprecated.

But then I thought to check one thing, and I'm glad I did: If an app's `minSdk`
is 26 or higher, core library desugaring isn't used for `java.time`, because
that `java.time` is already built into the OS. I discovered that if I set my
app's `minSdk` to 26, the default formatter's behavior reverted to using the
locale's clock style preference, rather than Android's. The use of the latter
is built into the desugared version of `java.time`, but not the OS version.

`AndroidDateTimeFormatters` is still useful after all, but only when
`java.time` is used without desugaring, i.e. when an app's `minSdk` is 26+. So
today I'm releasing `AndroidDateTimeFormatters` 3.0 with a new `minSdk` of 26.

`AndroidDateTimeFormatters` 3.0 also enhances its behavior: The system clock
preference is now respected not just for `FormatStyle.SHORT` times, but for
`MEDIUM`, `LONG`, and `FULL` times too. I nerd-sniped myself into implementing
[this 6-year-old issue](https://github.com/drewhamilton/AndroidDateTimeFormatters/issues/23)
before putting out another release.

I also rewrote the library in Kotlin, but other than the behavior change
mentioned above, `AndroidDateTimeFormatters` 3.0 maintains compatibility with
`AndroidDateTimeFormatters` 2.x, so the package is not changed.
