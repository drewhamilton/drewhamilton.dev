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
times. That was `AndroidDateTimeFormatters`' main purpose! So I almost
deprecated the whole library.

But then I thought to check one thing, and I'm glad I did: If an app's `minSdk`
is 26 or higher, core library desugaring isn't used for `java.time`, because
that library is already built into the OS. And I discovered that if I set my
app's `minSdk` to 26, that automatic use of the system clock preference stopped
happening.

`AndroidDateTimeFormatters` is still useful after all, but only for `minSdk`
26+. So `AndroidDateTimeFormatters` 3.0 bumps its `minSdk` up to 26.

It also now uses the system clock setting for all standard time formats—that is,
medium, long, and full format styles in addition to short—because I nerd-sniped
myself into implementing [this 6-year-old issue](https://github.com/drewhamilton/AndroidDateTimeFormatters/issues/23) before putting
out another release.
