---
layout: post
status: publish
published: true
title: Using DateTimeOffset
excerpt: "It's not a secret that `DateTime` structure in .NET is broken. You can find
  project like NodaTime that aims to fix some of the problems. In OpenTable we
  have resturants in multiple time zones. So using reservation date and time
  in form of `DateTime` structure is insufficient.

  Fortunatelly, .NET has `DateTimeOffset` structure which is a combination of
  `DateTime` and `Offset` properties. In this article I'll show how to use this
  structure in various scenarios."
author: "Łukasz Łysik"
excerpt:
categories:
- C#
tags:
- DateTime
---

It's not a secret that `DateTime` structure in .NET is broken. You can find
project like NodaTime that aims to fix some of the problems. In OpenTable we
have resturants in multiple time zones. So using reservation date and time
in form of `DateTime` structure is insufficient.

Fortunatelly, .NET has `DateTimeOffset` structure which is a combination of
`DateTime` and `Offset` properties. In this article I'll show how to use this
structure in various scenarios.

## Serialization

We should be able to represent `DateTimeOffset` in a string form for JSON
serialization.

{% highlight csharp %}
[Test]
public void DateTimeOffsetToString()
{
  var dto = new DateTimeOffset(2001, 3, 4, 14, 40, 21, TimeSpan.FromHours(4));
  to.ToString("o").Should().Be("2001-03-04T14:40:21.0000000+04:00");
}

[Test]
public void StringToDateTimeOffset()
{
  var dto = DateTimeOffset.Parse("2001-03-04T14:40:21.0000000+04:00");
  dto.Should().Be(new DateTimeOffset(2001, 3, 4, 14, 40, 21, TimeSpan.FromHours(4)));
}
{% endhighlight %}

I used `o` for a format, because without that current `DateTimeFormatInfo` would
be used which may be different in different machines.

## Getting local and UTC time

`DateTimeOffset` structure has 3 properties that represent `DateTime`:

- `UtcDateTime`
- `DateTime`
- `LocalDateTime`

Out of these, only `UtcDateTime` is obvious. Let's check what they return:

{% highlight csharp %}
[Test]
public void GetUtcDateTime()
{
  var dto = new DateTimeOffset(2001, 3, 4, 14, 40, 21, TimeSpan.FromHours(4));
  dto.UtcDateTime.Should().Be(new DateTime(2001, 3, 4, 10, 40, 21));
}

[Test]
public void GetDateTime()
{
  var dto = new DateTimeOffset(2001, 3, 4, 14, 40, 21, TimeSpan.FromHours(4));
  dto.DateTime.Should().Be(new DateTime(2001, 3, 4, 14, 40, 21));
}

[Test]
public void GetLocalDateTime()
{
  var dto = new DateTimeOffset(2001, 3, 4, 14, 40, 21, TimeSpan.FromHours(4));
  // It fails for different time zone settings
  dto.LocalDateTime.Should().Be(new DateTime(2001, 3, 4, 10, 40, 21));
}
{% endhighlight %}

`DateTime` returns date and time local to a specified offset. `LocalDateTime`
returns date and time in the current machine's time zone.

## Changing time

One of the requirements in OpenTable's mailing system is that we send one of the
email type at noon local to a restaurant. This date have to be transformed to UTC
because our scheduling system accepts datetime in UTC only. With `DateTimeOffset`
the task is very simple:

{% highlight csharp %}
[Test]
public void ChangeHour()
{
  var dto = new DateTimeOffset(2001, 3, 4, 14, 40, 21, TimeSpan.FromHours(4));

  var newHour = new TimeSpan(12, 0, 0);
  var dto1 = dto.Subtract(dto.TimeOfDay).Add(newHour);

  dto1.Should().Be(new DateTimeOffset(2001, 3, 4, 12, 0, 0, TimeSpan.FromHours(4)));
}
{% endhighlight %}
