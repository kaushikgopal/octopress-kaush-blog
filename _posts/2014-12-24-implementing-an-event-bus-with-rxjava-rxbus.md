---
layout: post
title: Implementing an Event Bus With RxJava - RxBus
date: 2014-12-24 18:43
categories: android RxJava
---

_Originally posted this article on the Wedding Party tech blog._
<hr />

This post has three parts:

1. quick primer on what an event bus is
2. implementing the event bus with RxJava
3. parting thoughts on this approach

"RxBus" is not going to be a library. Implementing an event bus with RxJava is so ridiculously easy that it doesn't warrant the bloat of an independent library.

<!-- more -->

# Part 1: What is an event bus?

Let's talk about two concepts that seem similar: the Observer pattern and the Pub-sub pattern.

## Observer pattern

This is a pattern of development in which your class or primary object (known as the Observable) notifies other interested classes or objects (known as Observers) with relevant information (events).

## Pub-sub pattern

The objective of the pub-sub pattern is exactly the same as the Observer pattern viz. you want some other class to know of certain events taking place.

There's an important semantic difference between the Observer and Pub-sub patterns though: in the pub-sub pattern the focus is on "broadcasting" messages outside. The Observable here doesn't want to know who the events are going out to, just that they've gone out. In other words the Observable (a.k.a Publisher) doesn't want to know who the Observers (a.k.a Subscribers) are.

## Why the anonymity?

It allows for this thing called "decoupling", which is a good word in computer programming. You want to keep coupling as low as possible in your design.

Typically, you would expect the publisher to have direct knowledge of each of the many subscribers that it needs to notify, so it can go about notifying each of them, once the "event" or message is ready. But with an event bus, the publisher is relieved of such duties and this independence helps, because the publisher and subscriber need not have logic coded in them that establish the dependencies between the two.

In other words "consciously decouple" your code whenever you can*.

## How the anonymity?

Ok, so a natural question with the pub-sub pattern is: how do you actually achieve that anonymity between publisher and subscriber? An easy way is to just get hold of a middleman and let that middleman take care of all the communication. An event bus is one such middleman.

That's it. An event bus is as simple as that.

Two event bus libraries that are commonly used in Android are [Otto](http://square.github.io/otto/) and Green Robot's [EventBus](https://github.com/greenrobot/EventBus). There are plenty of posts online that explain how to implement them in your app.

# Part 2: Implementing the event bus with RxJava

I've been posting real world examples of using [RxJava for Android in this github repo](https://github.com/kaushikgopal/Android-RxJava), so i'll stick to posting the complete implementation in that repo. Here are the interesting parts of the implementation:

{% gist 66d5b3d1a8f630811371910150dca43d %}

That's it. You now have an event bus ready to use.

Here's how you post an event to the bus:

{% gist ecb186475c697410a6b4ee03d5a507ad %}

and here's how you listen to those events from other fragments/services etc.


{% gist 683d9fd1b307ff4ce4500e721188bafc %}

In the example we post events from the top fragment (green part) and listen in (through the bus) from the bottom fragment (blue part).

![Simple RxBus example](/images/rxbus_simple.gif)

# Part 3  - parting thoughts

## Dead events

There are some cases where it's useful to know if there are any observers currently listening to the bus. For e.g. [if you use an event bus to handle your GCM push notifications](http://markhudnall.com/2013/11/13/gcm-foreground-and-background/)) and don't want to send a push notification if the app is in the foreground, then it's important to listen to "[dead events](https://github.com/square/otto/blob/master/otto/src/main/java/com/squareup/otto/DeadEvent.java)".

For e.g in a recent release for Wedding Party, we added "Messaging" to our app. If the user has the app open (thus having atleast 1 or more listeners to the bus) we don't send push notifications but if they have the app in the background, then we send a push notification to let them know of chat messages. After an event is posted to the event bus, if no subscriber is listening, a dead event is returned. If we get back a dead event a push notification is shot out.

How would you do this with the RxBus implementation?

It's pretty easy actually. `Subject`s  have this helpful method on them `hasObservers()` that would tell us exactly just that. This was added in the [1.x release for RxJava](https://github.com/ReactiveX/RxJava/pull/1802), so you have to be on the latest version of RxAndroid (0.23.0 as of this writing) in order to see this method.

## So should I use RxBus instead of Otto/EventBus?

If you simply want to use an event bus with your Android app, you're probably better off with a library like [Otto](http://square.github.io/otto/) (which i highly recommend) or [EventBus](https://github.com/greenrobot/EventBus). Otto has a clean api driven by annotations and is probably far more simpler to use.

If you're familiar with Rx, already use RxJava in your app and want to get rid of an additional library, definitely try out the RxBus approach!

_Did i miss anything? Do you have more suggestions on improving the RxBus implementation? Follow the discussion on [Reddit](http://www.reddit.com/r/androiddev/comments/2qebdo/rxbus_implementing_an_event_bus_with_rxjava/), [Google Plus](https://plus.google.com/105979641354189463768/posts/Au1xZ8RGLdG), feel free to tweet your comments @[kaushikgopal](http://twitter.com/kaushikgopal) or just raise an issue in [the repo](https://github.com/kaushikgopal/Android-RxJava/issues)._

_Pssst. there are two bonus posts that delve a little more into the RxJava trickery I used to make the RxBus example a tad bit fancier. Stay tuned._

<br />

![conscious decoupling](/images/uncopule-decouple.gif)
