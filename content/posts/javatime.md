---
title: "The inadequacy of java.util.Date"
date: 2020-01-28T12:03:00+01:00
showDate: true
draft: false
---

## Or how I learned to stop crying and love the complexities of `java.time`

The `java.time` package, along with the rest of Java 8, was released to the
Android world almost 6 years ago now. It reached Android devices 3.5 years
later—but only on devices that run it natively, i.e. devices with Android 8 or
higher. Since the vast majority of Android apps must support back to Android
6 or earlier, few Android developers actually get to use `java.time` on a
regular basis.

That doesn't mean that Android devs haven't used more advanced date/time APIs
than those in `java.util`, though. [Joda Time](https://www.joda.org/joda-time/)
was in wide use before Java 8 was released, and later
[ThreeTenBP](https://www.threeten.org/threetenbp/) backported almost the entire
`java.time` API (with a different package name) to Java 6, meaning it (with the
help of [ThreeTenABP](https://github.com/JakeWharton/ThreeTenABP)) could be used
on Android. But for one reason or another, plenty of Android devs haven't
adopted these libraries, and continue to use `java.util.Date`.

### What's wrong with `java.util.Date`?

Let's look at a few problems with Java's original `Date` class.

#### Data is lost when converting to `Date`

A given `Date` instance can represent quite a variety of actual data types: it's
used for dates, times, and date-times; and may or may not include a
time zone offset. But when you convert some backend string representing one of
these things to a `Date`, all you have left is a `long`: the number of
milliseconds since January 1, 1970, at midnight UTC. Nothing is left to indicate
whether the original data was a date-time or just a date. Nothing is left to
tell which time zone, if any, the original data referenced.

#### Data is fabricated when converting to `Date`

As mentioned above, a `Date` is backed only by a `long`, representing
milliseconds. So what happens if the original data doesn't represent
milliseconds? The formatter (`java.util.DateFormat`) makes them up.

##### "What?"

I mean, it uses reasonable-sounding defaults. If your data string is only
specific to the second, typically the `DateFormat` will add `000` for the
unknown milliseconds value. This is probably fine for most use cases.

But if the data string is only specific to the day (e.g. "2020-09-02"), then
there are a whole lot more digits to make up: not just the millisecond, but also
the second, minute, and hour.

##### "OK, but then just use midnight of that date by convention."

That's exactly what a typical `DateFormat` does! But which midnight? If you use
the device's default time zone for this parsing, you'll end up with a different
`Date` value han devices in other time zones. Or worse, you could end up with a
different value than the exact same device parses later, if it changes time
zones!

##### "...OK, then always use UTC for dates by convention"

This is in fact the convention for date-only data converted to `Date`s. But the
problem is the consumer of such a `Date` must know that it represents only a
date, not a date-time (because for a date-time they will want to display it in
the user's time zone); and that this convention exists (or they will format it
with the user's time zone out of habit).
