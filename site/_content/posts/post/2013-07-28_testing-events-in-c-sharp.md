---
title: Testing events in C#
tags: [ c#, event-handling ]
---
## TL;DR

This is a story about the pain of testing events in C#. If you don't have time to read the whole story, but you _are_ looking for an easy, reliable way to test (asynchronous) events, you can skip straight to [the end](#solution).


## The story

So I've been unit-testing events in C# lately, and it's been kind of a pain. Let's say we have this code:

<pre class="prettyprint">
public delegate void AnswerHandler(object sender, AnswerEventArgs e);
public class AnswerEventArgs : EventArgs
{
    public int Answer { get; set; }
}

public class AnswerFinder
{
    public event AnswerHandler AnswerEvent;

    public void FindTheAnswer()
    {
        int? answer = DeepThought.Compute();
        if (answer.HasValue)
        {
            var eventArgs = new AnswerEventArgs { Answer = answer.Value };
            AnswerEvent(this, eventArgs);
        }
    }
}
</pre>

And we want to know whether we got the correct answer. We could write a unit test that looks something like this:

<pre class="prettyprint">
[Test]
public void Attempt1()
{
    var finder = new AnswerFinder();
    finder.AnswerEvent += (sender, e) =>
    {
        Assert.That(e.Answer, Is.EqualTo(42));
    };
    finder.FindTheAnswer();
}
</pre>

Or rather, we could write something like this, because while in this particular case no memory leak would occur, it's usually good practice to clean up after yourself:

<pre class="prettyprint">
[Test]
public void Attempt2()
{
    var finder = new AnswerFinder();
    EventHandler<AnswerEventArgs> handler = (sender, e) =>
    {
        Assert.That(e.Answer, Is.EqualTo(42));
    };
    finder.AnswerEvent += handler;
    finder.FindTheAnswer();
    finder.AnswerEvent -= handler;
}
</pre>

That will do the trick, right? Wrong. There's no guarantee an answer will be found. `DeepThought.Compute()` may simply give up at some point and return `null`, and the event will never be raised. Unfortunately, in that case, our assert is never executed, and the test will happily report success. So we have to guard against that:

<pre class="prettyprint">
[Test]
public void Attempt3()
{
    var finder = new AnswerFinder();
    int answer = -1;
    AnswerHandler handler = (sender, e) =>
    {
        answer = e.Answer;
    };
    finder.AnswerEvent += handler;
    finder.RaiseTheAnswer();
    Assert.That(answer, Is.EqualTo(42));
    finder.AnswerEvent -= handler;
}
</pre>

Ouch. Also, what does that `-1` mean? Assuming that the answer is always an integer, are we sure it can't be negative? I'm certainly not. Also, we're conflating two properties here: whether or not the event was raised, and the actual answer. Let's fix that.

<pre class="prettyprint">
[Test]
public void Attempt4()
{
    var finder = new AnswerFinder();
    bool eventWasCalled = false;
    AnswerHandler handler = (sender, e) =>
    {
        eventWasCalled = true;
        Assert.That(e.Answer, Is.EqualTo(42));
    };
    finder.AnswerEvent += handler;
    finder.RaiseTheAnswer();
    Assert.That(eventWasCalled, Is.True);
    finder.AnswerEvent -= handler;
}
</pre>


That's an awful lot of boilerplate code for such a simple test! In fact, I'm having trouble seeing the actual test logic through all the boilerplate. Now, imagine if you have a comprehensive test suite with many tests like these. Or actually: please don't. It makes me sad.

What makes me even sadder, is that `DeepThought.Compute()` is quite an expensive operation. We don't want it to freeze the GUI; we should off-load it to a background thread. Great: now we have an asynchronous event. Let's replace the `bool` with a `ManualResetEventSlim`:

<pre class="prettyprint">
[Test]
public void Attempt5()
{
    var finder = new AnswerFinder();
    var resetEvent = new ManualResetEventSlim(false);
    AnswerHandler handler = (sender, e) =>
    {
        resetEvent.Set();
        Assert.That(e.Answer, Is.EqualTo(42));
    };
    finder.AnswerEvent += handler;
    finder.RaiseTheAnswer();
    Assert.That(resetEvent.Wait(TimeSpan.FromMilliseconds(500)), Is.True);
    finder.AnswerEvent -= handler;
}
</pre>

OK, that's not so bad, right? But wait: `ManualResetEventSlim` implements `IDisposable`!

<pre class="prettyprint">
[Test]
public void Attempt6()
{
    var finder = new AnswerFinder();
    using (var resetEvent = new ManualResetEventSlim(false))
    {
        AnswerHandler handler = (sender, e) =>
        {
            resetEvent.Set();
            Assert.That(e.Answer, Is.EqualTo(42));
        };
        finder.AnswerEvent += handler;
        finder.RaiseTheAnswer();
        Assert.That(resetEvent.Wait(TimeSpan.FromMilliseconds(100)), Is.True);
        finder.AnswerEvent -= handler;
    }
}
</pre>

Of course we could lessen that load by turning `resetEvent` into a field, and using `[SetUp]` and `[TearDown]` methods to initialize it and clean it up. But still a lot of boilerplate remains.

Thankfully, we can do better. A lot better.

<a name='solution'></a>
## A solution

I've written a small class that uses some clever reflection tricks to handle most of the boilerplate I've shown above. Behold:

<pre class="prettyprint">
[Test]
public void FinalAttempt()
{
    var finder = new AnswerFinder();
    AnswerHandler handler = (sender, e) =>
    {
        Assert.That(e.Answer, Is.EqualTo(42));
    };
    using (new EventMonitor(finder, "AnswerEvent", handler))
    {
        finder.RaiseTheAnswer();
    }
}
</pre>

Neat, isn't it? You can find the code [below](#code). Maybe I'll do a proper version on GitHub some day, with its own test suite and proper documentation. Until then, this blog post will have to do :).

Here's a few things `EventMonitor` can do:

* It works both for synchronous and asynchronous events.
* You can give it a custom timeout in an optional constructor parameter. By default, it'll wait for 500 milliseconds.
* Even if you decide not to use the conventional `object sender, EventArgs e` delegates, it will work with any delegate with up to 4 parameters, as long as they have a `void` return type.
* If you want to check that the same event is fired twice, you can do that as follows:

<pre class="prettyprint">
[Test]
public void TheSameEventTwice()
{
    var finder = new AnswerFinder();
    AnswerHandler handler = (sender, e) =>
    {
        Assert.That(e.Answer, Is.EqualTo(42));
    };
    using (var monitor = new EventMonitor(finder, "AnswerEvent",
                                          handler, Mode.MANUAL))
    {
        finder.RaiseTheAnswer();
        monitor.Verify();

        // Let's do it again!
        finder.RaiseTheAnswer();
        monitor.Verify();
    }
}
</pre>

Unfortunately, it doesn't seem possible in C# to pass an event as a parameter. That's why I opted for the "stringly typed" option of passing the event's name as a string. If you know of a better way to handle this, I would very much like to hear about it!

<a name='code'></a>
## Full code of `EventMonitor`

<pre class="prettyprint">
using System;
using System.Linq;
using System.Reflection;
using System.Threading;
using NUnit.Framework;

namespace EventMonitor
{
	public enum Mode
	{
		MANUAL,
		AUTOMATIC
	}

	public class EventMonitor : IDisposable
	{
		private static readonly int MaximumArity = 4;

		private readonly object objectUnderTest;
		private readonly EventInfo eventInfo;
		private readonly Delegate wrappedHandler;
		private readonly TimeSpan timeout;
		private readonly Mode mode;
		private readonly ManualResetEventSlim resetEvent;

		public EventMonitor(object objectUnderTest, string eventName, Delegate handler, Mode mode = Mode.AUTOMATIC)
			: this(objectUnderTest, eventName, handler, TimeSpan.FromMilliseconds(500), mode)
		{ }

		public EventMonitor(object objectUnderTest, string eventName, Delegate handler, TimeSpan timeout, Mode mode = Mode.AUTOMATIC)
		{
			this.objectUnderTest = objectUnderTest;
			this.timeout = timeout;
			this.mode = mode;
			this.resetEvent = new ManualResetEventSlim(false);

			this.eventInfo = objectUnderTest.GetType().GetEvent(eventName);
			Assert.That(eventInfo, Is.Not.Null, string.Format("Event '{0}' not found in class {1}", eventName, objectUnderTest.GetType().Name));

			this.wrappedHandler = Delegate.Combine(handler, GenerateRecordingDelegate(eventInfo.EventHandlerType));
			eventInfo.AddEventHandler(objectUnderTest, wrappedHandler);
		}

		public virtual void Dispose()
		{
			if (mode == Mode.AUTOMATIC)
			{
				Verify();
			}
			eventInfo.RemoveEventHandler(objectUnderTest, wrappedHandler);
			resetEvent.Dispose();
		}

		public void Verify()
		{
			Assert.That(resetEvent.Wait(timeout), Is.True, string.Format("Event '{0}' was not raised!", eventInfo.Name));
			resetEvent.Reset();
		}

		private Delegate GenerateRecordingDelegate(Type eventHandlerType)
		{
			var method = eventHandlerType.GetMethod("Invoke");
			int arity = method.GetParameters().Count();
			Assert.That(arity, Is.LessThanOrEqualTo(MaximumArity), string.Format("Events of arity up to {0} supported; this event has arity {1}", MaximumArity, arity));
			var methodName = string.Format("Arity{0}", arity);
			var eventRegisterMethod = typeof(EventMonitor).GetMethod(methodName, BindingFlags.NonPublic | BindingFlags.Instance);
			return Delegate.CreateDelegate(eventHandlerType, this, eventRegisterMethod);
		}

		private void Arity0()
		{
			resetEvent.Set();
		}

		private void Arity1(object arg1)
		{
			resetEvent.Set();
		}

		private void Arity2(object arg1, object arg2)
		{
			resetEvent.Set();
		}

		private void Arity3(object arg1, object arg2, object arg3)
		{
			resetEvent.Set();
		}
		
		private void Arity4(object arg1, object arg2, object arg3, object arg4)
		{
			resetEvent.Set();
		}
	}
}
</pre>
