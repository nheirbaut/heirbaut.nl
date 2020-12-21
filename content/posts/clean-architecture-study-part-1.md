---
title: "Clean Architecture Study - Part 1"
date: 2020-12-21T01:45:29+01:00
draft: false
---

For quite some time I have been a fan of [Clean Architecture](https://blog.cleancoder.com/uncle-bob/2012/08/13/the-clean-architecture.html). I used in in several projects and found that it made my live as a developer a lot easier. To enable myself to play around with, and study, different techniques and technologies and want to create a series of projects around this Clean Architecture. In this post we will start of with the first part, where we set up our initial development structure, including initial domain model and tests.

All code will currently be written in C# using .Net 5. As I really like using the command line and feel like I need to practice using it a lot more on Windows all my examples will show how to do stuff using the command line as much as possible. It also saves me from creating a lot of screenshots :).

No let's start cranking. The aim of this article and the ones that follow is to create a simple date picker application for events. Initially the requirements will be:
* A user can create an event
* A user can propose dates and times for the event
* A user can invite others to indicate their availability on the proposed dates an times

This is just a bare minimum list of requirements. In the future more requirements will be added, showing the advantages of applying a Clean Architecture. All resulting code can be found in its [Github repository](https://github.com/nheirbaut/DatePicker).

## Initial setup

Open a PowerShell command shell at your favorite development code storage location and create a folder that will create all (future) projects of our study and move to that folder:

```posh
New-Item -ItemType "directory" DatePicker
cd DatePicker
```

Now we can create our initial project that will contain the domain model:

```posh
dotnet new classlib -n DatePicker.Domain
```

There is an initial class definition file in there called `Class1.cs`. Let's remove it:

```posh
Remove-Item DatePicker.Domain\Class1.cs
```

Since I really like Test Driven Design we will also create a unit test project. This will be based on [xUnit](https://xunit.net/) as I really like this framework and it seems to have become the de-factor standard in the .Net world. To get some more information on the standards, methods and techniques I use in later on when creating the tests I can heartily recommend the book [Unit Testing Principles, Practices, and Patterns](https://www.amazon.com/gp/product/1617296279) by [Vladimir Khorikov](https://enterprisecraftsmanship.com/).

Create the test project:

```posh
dotnet new xunit -n DatePicker.Domain.UnitTests
```

This will add a default test file which we can remove as we will add our own ones later on:

```posh
Remove-Item DatePicker.Domain.UnitTests\UnitTest1.cs
```

To enable the unit tests to use the code from the domain project add the domain project as a reference to the unit test project:

```posh
dotnet add DatePicker.Domain.UnitTests reference DatePicker.Domain
```

At this point it is not strictly necessary, but we can create a Visual Studio solution file that will reference both created projects. I myself will use Visual Studio Code as much as possible, but someone else might feel more conformable using Visual Studio.

Create the solution file:

```posh
dotnet new sln -n DatePicker
```

Add references to our projects to that solution file:

```posh
dotnet sln add DatePicker.Domain
dotnet sln add DatePicker.Domain.UnitTests
```

Now doing a build should succeed without errors or warnings with the following command:

```posh
dotnet build
```

This should be enough initial setup to start working on the actual problem domain. Now might be a good time to initialize a local Git repository. Do not forget to add a `.gitignore` file to make sure not too much gets stored in the repository. I can really recommend using [Toptal](https://www.toptal.com/developers/gitignore) to create an initial `.gitignore`. The one I would use now [looks like this](https://www.toptal.com/developers/gitignore/api/csharp,visualstudiocode,visualstudio). In the top of the file you can see which values I used.

## Create an initial unit test

For we begin we will add [Shouldly](https://shouldly.io/) as a dependency to our unit test project. Why? Because I think it will improve the readability of our tests. So issue the following command on the command line:

```posh
dotnet add DatePicker.Domain.UnitTests package Shouldly
```

So now we are ready to create an initial unit test. Why not start immediately with coding the application? Because I like [Test-Driven Development (TDD)](https://en.wikipedia.org/wiki/Test-driven_development), and think it is a good way of keeping a software development under control. Perhaps at at later stage the tests will be extended or replaced with [Behavior-Driven Development](https://en.wikipedia.org/wiki/Behavior-driven_development).

What does the first requirement say? It says: A user can create an event. That should be simple enough. First create a new file for the tests:

```posh
New-Item .\DatePicker.Domain.UnitTests\EventTests.cs
```

Open the file and add the following initial content:

```cs
using Xunit;

namespace DatePicker.Domain.UnitTests
{
    public class EventTests
    {
    }
}
```

So let's add our first test for the initial creation of an event:

```cs
[Fact]
public void Event_can_be_created_for_user_with_event_name_and_date()
{
    var user = "testuser";
    var dateTime = DateTime.Now + TimeSpan.FromDays(1);
    var name = "testname";

    var sut = new Event(user, name, dateTime);

    sut.User.ShouldBe(user);
    sut.DateTime.ShouldBe(dateTime);
    sut.Name.ShouldBe(name);
}
```

In a test I use the following order of events:
1. Arrange
2. Act
3. Assert

Some people like to make that explicit by adding comments, but I prefer just clearly having short tests with code blocks indicating the separations. I also like to make the name of the items I am testing `sut` as to make it more easier to rename the item if further analysis shows that that new name might be more descriptive.

To run this test a command `dotnet test` can be given on the command line. This will build the sources and run all available tests for us. At this moment they will even fail to build as we haven't added our `Event` class yet. So looking at the test we should add a file named `Event.cs` to our folder `DatePicker.Domain` with the following contents:

```cs
using System;

namespace DatePicker.Domain
{
    public class Event
    {
        public string User { get; }
        public string Name { get; }
        public DateTime DateTime { get; }

        public Event(string user, string name, DateTime dateTime)
        {
            User = user;
            Name = name;
            DateTime = dateTime;
        }
    }
}
```

Revisit `EventTests.cs` and make sure all correct `using`-statements are added. If you now run `dotnet test` the test should succeed.

## Adding some more tests

We created a test for the happy path, but to be sure the code also responds to invalid data some tests will be added for that as well:
* A test for invalid user names
* A test for invalid event names
* A test for invalid dates

Of course extra tests my be added in the future, these are just the tests as can see them now. For the testing of the reaction to invalid names we will first create a helper class that will contain some of these invalid names:

```posh
New-Item -Force  .\DatePicker.Domain.UnitTests\Helpers\InvalidNamesData.cs
```

The `-Force` flag will make sure that the `Helpers` folder is also created. Now add the following code to the file `InvalidNamesData.cs`:

```cs
using System.Collections;
using System.Collections.Generic;

namespace TestUtilities.Helpers
{
    public class InvalidNamesData : IEnumerable<object[]>
    {
        public IEnumerator<object[]> GetEnumerator()
        {
            yield return new object[] {null};
            yield return new object[] {""};
            yield return new object[] {" "};
            yield return new object[] {"\t"};
            yield return new object[] {"\r"};
            yield return new object[] {"\n"};
            yield return new object[] {" \t"};
            yield return new object[] {" \r"};
            yield return new object[] {" \n"};
            yield return new object[] {"\t "};
            yield return new object[] {"\r "};
            yield return new object[] {"\n "};
        }

        IEnumerator IEnumerable.GetEnumerator()
            => GetEnumerator();
    }
}
```

As you can see it defines an enumerable object that with each iteration will return a new invalid name. This can be used with xUnit in a so called [parameterized test](https://andrewlock.net/creating-parameterised-tests-in-xunit-with-inlinedata-classdata-and-memberdata/). The following code shows usage of the `InvalidNamesData` class in the unit tests. Add that code to `EventTests.cs`.

```cs
[Theory]
[ClassData(typeof(InvalidNamesData))]
public void Event_cannot_be_created_with_invalid_username(string invalidUserName)
{
    var dateTime = DateTime.Now + TimeSpan.FromDays(1);
    var name = "testname";

    Should.Throw<ArgumentException>(() =>
        new Event(invalidUserName, name, dateTime));
}

[Theory]
[ClassData(typeof(InvalidNamesData))]
public void Event_cannot_be_created_with_invalid_event_name(string invalidName)
{
    var user = "testuser";
    var dateTime = DateTime.Now + TimeSpan.FromDays(1);

    Should.Throw<ArgumentException>(() =>
        new Event(user, invalidName, dateTime));
}
```

To get this to build you probably have to add `using DatePicker.Domain.UnitTests.Helpers;` to the using statements at the top. If you now try `dotnet test` the tests will fail because invalid names are not yet checked. Let's add those checks by adding the following lines to the top of the constructor of `Event`:

```cs
if (string.IsNullOrWhiteSpace(user))
    throw new ArgumentException(
        "The user name cannot be only whitespace or null",
        nameof(user));

if (string.IsNullOrWhiteSpace(name))
    throw new ArgumentException(
        "The event name cannot be only whitespace or null",
        nameof(name));
```

Running `dotnet test` should now show all tests passing again. One more test to add, namely, what would happen if you wanted to create an event in the past. That would not be of much use. So add the following tests to `EventTests.cs`:

```cs
[Fact]
public void Event_cannot_be_created_with_past_date()
{
    var user = "testuser";
    var dateTimeInThePast = DateTime.Now - TimeSpan.FromDays(1);
    var name = "testname";

    Should.Throw<ArgumentException>(() =>
        new Event(user, name, dateTimeInThePast));
}
```

Running `dotnet test` should show a failing test. This can be fixed by adding the following check to the constructor of `Event`:

```cs
if (dateTime < DateTime.Now)
    throw new ArgumentException($"Cannot create an event with a date in the past: {dateTime}", nameof(dateTime));
```

This concludes the first part of the series. In the next part we will make a careful start with the application layer. This will probably mean extending the domain layer further.

All code so far can be found by creating a clone of the repository and then checking out the last for this article commit like this:

```posh
git checkout 37038eac
```
