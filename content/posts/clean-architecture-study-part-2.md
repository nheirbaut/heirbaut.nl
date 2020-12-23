---
title: "Clean Architecture Study - Part 2"
date: 2020-12-22T17:16:14+01:00
draft: false
---

In our [previous installment]({{< ref "clean-architecture-study-part-1" >}}) of the Clean Architecture Study we implemented the first of our initial requirements:
* A user can create an event
* A user can propose dates and times for the event
* A user can invite others to indicate their availability on the proposed dates an times

This installment continues with the second requirement 'A user can propose dates and times for the event'. As with the other domain code we will start with the test first:

```cs
[Fact]
public void User_can_propose_date_and_time_for_an_existing_event()
{
    var user = "testuser";
    var name = "testname";
    var initialDateTime = DateTime.Now + TimeSpan.FromDays(1);
    var dateTimeToPropose = DateTime.Now + TimeSpan.FromDays(2);
    var sut = new Event(user, name, initialDateTime);

    sut.AddProposedDateAndTime(dateTimeToPropose);

    sut.ProposedDateTimes.ShouldBe(
        new [] { initialDateTime, dateTimeToPropose },
         ignoreOrder: true);
}
```

There are two big changes here:
1. A method, `AddProposedDateAndTime()`, has been added to `Event` to add new date and time proposals.
2. The event now holds a collection of all proposed dates and times.

Let's add the collection of proposals first to `Event.cs`. In the body of the class add the following:

```cs
private ICollection<DateTime> _proposedDateTimes;
public IEnumerable<DateTime> ProposedDateTimes
    => _proposedDateTimes;
```

Why use a backing field? Because I want to be able to add to the collection later, but only want to expose a readonly collection to the client code to ensure strict encapsulation.

Now the new method can be added to `Event.cs`:

```cs
public void AddProposedDateAndTime(DateTime proposedDateTime)
{
    _proposedDateTimes.Add(proposedDateTime);
}
```

All should build now, but the test still fails. This is because collection is not yet initialized and does not contain the initial date. So to the bottom of the `Event` constructor add the following line:

```cs
_proposedDateTimes = new List<DateTime> { dateTime };
```

You probably also need to add `using System.Collections.Generic;` to the using statements. When run, all tests should now be green. But we are only in the [second stage of the nano-cycle of Test-Driven Development (TDD)](https://blog.cleancoder.com/uncle-bob/2014/12/17/TheCyclesOfTDD.html). We still need to refactor. As it now stands there are two places where the initial date and time for the event are stored, in the `DateTime` property and in the `ProposedDateTimes` collection property. Let's remove the `DateTime` property. This breaks the build for the unit tests. To fix this, change all places (1!) in `EventTests.cs` where the line says:

```cs
sut.DateTime.ShouldBe(dateTime);
```

to:

```cs
sut.ProposedDateTimes.ShouldBe(new [] { dateTime });
```

Running `dotnet test` now should now show all tests are green.

Unfortunately now it would be possible again to proposed past dates and times. So let's add a test for that again:

```cs
[Fact]
public void User_cannot_propose_past_date_and_time()
{
    var user = "testuser";
    var name = "testname";
    var initialDateTime = DateTime.Now + TimeSpan.FromDays(1);
    var dateTimeToProposeInThePast = DateTime.Now - TimeSpan.FromDays(1);
    var sut = new Event(user, name, initialDateTime);

    Should.Throw<ArgumentException>(() =>
        sut.AddProposedDateAndTime(dateTimeToProposeInThePast));
}
```

When trying to run the tests this test obviously fails. There is no check for past dates and times. Let's add that to the `AddProposedDateAndTime()` method by adding these lines at the beginning of the method:

```cs
if (proposedDateTime < DateTime.Now)
    throw new ArgumentException(
        $"Cannot create an event with a date in the past: {proposedDateTime}",
        nameof(proposedDateTime));
```

All tests should now be green. But by adding these lines we go against the '[Don't Repeat Yourself (DRY)](https://wiki.c2.com/?DontRepeatYourself)' principle as a similar check is done in the constructor of `Event`. To fix this we can centralize this by adding the following method to `Event`:

```cs
private static void ValidateProposedDateAndTime(DateTime proposedDateTime)
{
    if (proposedDateTime < DateTime.Now)
        throw new ArgumentException(
            $"Cannot create an event with a date in the past: {proposedDateTime}",
            nameof(proposedDateTime));
}
```

So in the constructor and `AddProposedDateAndTime()` both checks can now be replaced with a call to `ValidateProposedDateAndTime()`. All tests are still green after this.

This concludes the second part of the series, we got as far as the second requirement, the next installments we will continue with the requirements for the domain model.

All code so far can be found by creating a clone of the repository and then checking out the last for this article commit like this:

```posh
git checkout 4251b579
```
