---
layout: post
title: Primer on Threading and Handlers in Android
date: 2014-06-20 04:00
categories: android java
---

--------------------------------------
<br />
Originally posted this article on the [Wedding Party tech blog](nerds.weddingpartyapp.com).

--------------------------------------

<br />

Mobile devices are getting pretty fast, but they aren't infinitely fast yet. If you want your app to be able to do any serious work without affecting the user experience by locking up the interface, you'll have to resort to running things in parallel. On Android, this is done with "threads".

Grab yourself a cup of coffee and read this post line by line. I'll introduce you to the concept of threads, talk about how Java uses threads and explain how "Handlers" in Android help with threading.

<!-- more -->

Whenever you want to do _asynchronous/parallel processing_, you do it with __threads__.

## Threads you say ?

A thread or "thread of execution" is basically a sequence of instructions (of program code), that you send to your operating system.

<div class="tac"><img src="http://upload.wikimedia.org/wikipedia/commons/a/a5/Multithreaded_process.svg" alt='Multi-threaded process' /></div>

<div class="tac" style="font-style:italic">Image Courtesy: Wikipedia.</div><br />

"Typically" your CPU can process one thread, per core, at any time. So a multi-core processor (most Android devices today) by definition can handle multiple-threads of execution (which is to say, they can do multiple things at once).

## Truth to multi-core processing and single-core multitasking

I say "typically" because the corollary to the above statement is not necessarily true. Single-core devices can "simulate" multithreading using multitasking.

Every "task" that's run on a thread can be broken down into multiple instructions. These instructions don't have to happen all at once. So a single-core device can switch to a thread "1" finish an instruction 1A, then switch to thread "2" finish an instruction 2A, switch back to 1 finish 1B, 1C, 1D, switch to 2, finish 2B, 2C and so on...

This switching between threads happens *so fast* that it *appears*, even on a single-core device, that all the threads are making progress at exactly the same time.  It's an illusion caused by speed, much like Agent Brown appearing to have multiple heads and arms.

<img src="/images/agent_brown_dodging_bullets.gif" /><br />

Now on to some code.

## Threads in core Java

In Java, when you want to do parallel processing, you execute your code in a `Runnable` either by extending the `Thread` class or implementing the Runnable interface

{% gist 859db79a6d1519a92dff0dcb7c3ce9cd %}

Both of these approaches are fundamentally very similar. Version 1 involves creating an actual thread while Version 2 involves creating a runnable, which in-turn has to be called by a Thread.

Version 2, is generally the preferred approach (and is a much [larger subject](http://en.wikipedia.org/wiki/Composition_over_inheritance) [of discussion](http://stackoverflow.com/questions/541487/implements-runnable-vs-extends-thread), beyond the scope of this post).

## Threads in Android

Whenever your app starts up in Android, all components are run on a single primary thread (by default) called the "main" thread. The primary role of this thread though, is to handle the user interface and dispatch events to the appropriate widgets/views. For this reason, the main thread is also referred to as the "UI" thread.

[If you have a long running operation on your UI thread](http://android-developers.blogspot.com/2009/05/painless-threading.html), the user interface is going to get locked up until this operation is complete. This is bad for your users! That's why it's important to understand how threads work in Android specifically, so you can offload some of the work to parallel threads. Android is pretty merciless about keeping things off the UI thread. If you have a long running operation on the UI thread you're probably going to run into the infamous [ANR](http://developer.android.com/training/articles/perf-anr.html) that will conveniently allow your users to kill your app!

Now Android is all Java, so it supports the usage of the core Java `Thread` class to perform asynchronous processing. So you could use code very similar to the one shown in the "Threads in Java" section above, and start using threads in Android right away. But that can be a tad bit difficult.

### Why is using core Java threads in Android difficult?

Well, parallel processing is not as easy as it sounds because you have to maintain "concurrency" across the multiple threads. In the words of the [very wise Tim Bray](https://www.tbray.org/ongoing/When/201x/2014/01/01/Software-in-2014#p-2):

> ordinary humans can't do concurrency at scale (or really at all) ...

Specifically for Android, the following is additionally cumbersome:

1. Synchronization with the UI thread is a major [PITA](http://www.urbandictionary.com/define.php?term=pita) (you typically want to do this, when you want to send progress updates to the UI, for your background operation)
2. Things change even more weirdly with orientation/configuration changes because an orientation change causes an activity to be recreated (so background threads may be trying to change the state of a destroyed activity, and if the background thread isn't on the UI thread, well that complicates things even more because of point 1).
3. There's also no default handling for thread pooling
4. Canceling thread actions requires custom code

###  Arr...ok, so how DO we do parallel processing in Android?

Some (in)famous Android constructs you've probably come across:

1. [`Handler`](http://developer.android.com/reference/android/os/Handler.html)

    This is the subject of our detailed discussion today

2. [`AsyncTask`](http://developer.android.com/reference/android/os/AsyncTask.html)

    Using AsyncTasks are truly the simplest way to handle threads in Android. That being said, they are also the [most](http://blog.danlew.net/2014/06/21/the-hidden-pitfalls-of-asynctask/) [error](http://www.jayway.com/2012/11/28/is-androids-asynctask-executing-tasks-serially-or-concurrently/) [prone](http://bon-app-etit.blogspot.com/2013/04/the-dark-side-of-asynctask.html).

3. [`IntentService`](http://developer.android.com/reference/android/app/IntentService.html)

    It requires more boiler plate code, but this is generally my preferred mechanism for off-loading long-running operations to the background. Armed with an EventBus like [Otto](http://square.github.io/otto/), IntentServices become amazingly easy to implement.

4. [`Loader`](http://developer.android.com/guide/components/loaders.html#summary)

    These are geared more towards performing asynchronous tasks, that have to deal with data from databases or content providers.

5. [`Service`](http://developer.android.com/guide/components/services.html) (honorable mention)

    If you've worked with Services closely, you should know that this is actually a little misleading. A common misconception is that Services run on the background thread. Nope! they "appear" to run in the background because they don't have a UI component associated with them. They actually run on the UI thread (by default).... So they run on the UI thread by default, even though they don't have a UI component?


<div class="tac"><a href="http://martinvalasek.com/blog/pictures-from-a-developers-life"><img src="http://www.topito.com/wp-content/uploads/2013/01/code-24.gif" /></a></div>
<br />
<div class="tac">Naming has never been Google's strong suit. ActivityInstrumentationTestCase ... wait for it .... 2! Spinner anyone? </div>
<br />

If you want your service to run on a background thread, you'll have to manually spawn another thread and execute your operations in that thread (similar to an approach discussed above). Really you should just use IntentServices but that is a subject for another post.

# Android Handlers:

From the not-so-dummy-friendly [Android developer documentation for Handlers](http://developer.android.com/reference/android/os/Handler.html):

> A Handler allows you to send and process Message and Runnable objects associated with a thread's MessageQueue. Each Handler instance is associated with a single thread and that thread's message queue. When you create a new Handler, it is bound to the thread/message queue of the thread that is creating it -- from that point on, it will deliver messages and runnables to that message queue and execute them as they come out of the message queue.

<div class="tac"><a href="http://martinvalasek.com/blog/pictures-from-a-developers-life"><img src="http://www.topito.com/wp-content/uploads/2013/01/code-02.gif" /></a></div>
<br />

To understand that better, you probably need to know what __Message Queues__ are.

## Message Queues:

Threads basically have something called a "Message Queue". These message queues allow communication between threads and is a sort of pattern, where control (or content) is passed between the threads.

It's actually a wonderful name, because it's exactly just that: a queue of messages or sequence of instructions, for the thread, to perform one by one. This additionally allows us to do two more cool things:

* "schedule" Messages and Runnables to be executed at some point in the future
* enqueue an action to be performed on a different thread other than your own

_Note: when I mention 'message' from here onwards, it's the same as a runnable object or a sequence of instructions._

So going back to Handlers in Android.... if you read and pause at every single line, the docs make much more sense now:

> A Handler allows you to send and process Message and Runnable objects associated with a thread's MessageQueue.

So, a handler allows you to send messages to a thread's message queue. check ✔ !

> Each Handler instance is associated with a single thread and that thread's message queue.

A handler can only be associated with a single thread. ✔

> When you create a new Handler, it is bound to the thread / message queue of the thread that is creating it

So, which thread is a handler associated with? The thread that creates it. ✔

> -- from that point on, it will deliver messages and runnables to that message queue and execute them as they come out of the message queue.

Yeah yeah we already know that. Moving on...

*Tip: Here's something you probably didn't know : in Android, every thread is associated with an instance of a Handler class, and it allows the thread to run along with other threads and communicate with them through messages.*

*Another Tip (if you've dealt with the more common AsyncTasks): AsyncTasks use Handlers but don't run in the UI thread. They provide a channel to talk back to the UI, using the postExecute method.*

## This is all cool yo, so how do I create them Handlers?

Two ways:

1. using the default constructor: new Handler()
2. using a parameterized constructor that takes a runnable object or callback object

## What useful methods does the Handler API give me?

Remember:

* Handlers simply send messages or "posts" to the message queues.
* They are convenience methods that help syncing back with the UI thread.

If you look at the [API for Handlers](http://developer.android.com/reference/android/os/Handler.html) now, the main methods provided make sense:

1. post
2. postDelayed
3. postAtTime


## Code samples:

The examples below are rudimentary, what you actually want to be closely following are the comments.

### Example 1: using "post" method of the Handler

{% gist 19268bef99a644aea4728de7c2ebb3bf %}

If I didn't have a handler object, posting back to the UI thread would have been pretty tricky.

### Example 2: using postDelayed

In a recent feature for Wedding Party, I had to emulate the auto-complete functionality with an EditText. Every change in text triggered an API call to retrieve some data from our servers.

I wanted to reduce the number of API calls shot out by the app, so I used the Handler's postDelayed method to achieve this.

This example doesn't focus on parallel processing, but rather the ability for the Handler to function as a Message Queue and schedule messages to be executed at some later point in the future

{% gist ea0d81206815734a9964eb984c6d5fb7 %}

I leave the "postAtTime" as an exercise for the reader. Got a grip on Handlers? Happy threading!


_Follow the discussion on [Hacker News](https://news.ycombinator.com/item?id=7921979) or [Reddit](http://www.reddit.com/r/androiddev/comments/28nsty/primer_on_threading_and_handlers_in_android/) or [Google Plus](https://plus.google.com/106712246601366256750/posts/KMU37xoYGuY)_.
