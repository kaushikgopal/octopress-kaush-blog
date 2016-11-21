---
date: 2015-01-05 07:02
layout: post
title: DebouncedBuffer With RxJava
categories: android RxJava
---

<div class="sidenote">Originally posted this article on the Wedding Party tech blog.</div>

This is a bonus RxJava post that I landed up writing along with my [previous post on creating an event bus with RxJava](http://blog.kaush.co/2014/06/20/implementing-an-event-bus-with-rxjava-rxbus). If you went through the [code in the actual repo](https://github.com/kaushikgopal/Android-RxJava/tree/master/app/src/main/java/com/morihacky/android/rxjava/rxbus) you would have noticed more than one version of the bottom fragment in the RxBus demo.

Originally I envisioned the RxBus example being a tad bit fanicer however as I coded up the example, I realized that too many concepts were getting conflated. The ridiculous simplicity of the RxBus implementation was lost. So I dumbed down the original example but left in the original code for the Rx padawans.

Original example:

![Simple RxBus example](/images/rxbus_simple.gif)

Fancier one:

![Fancy RxBus example](/images/rxbus_fancy.gif)

<!-- more -->

The fanciness is basically in the numbers being accumulated and shown in "chunks". In my head I thought I could simply use the `debounce` operator (like the [debounce search example](https://github.com/kaushikgopal/Android-RxJava/blob/master/app/src/main/java/com/morihacky/android/rxjava/SubjectDebounceSearchEmitterFragment.java)) and be on my jolly way, but this was not to be...  From the always helpful [RxJava wiki](https://github.com/ReactiveX/RxJava/wiki/Alphabetical-List-of-Observable-Operators):

> `debounce` - only emit an item from the source Observable after a particular timespan has passed without the Observable emitting any other items

I wanted the "*after a particular timespan has passed without the Observable emitting any other items*" part of debounce, but I needed the whole *list of emitted items* (not just a single item). The`buffer`operator also seemed promising:

> `buffer` - periodically gather items from an Observable into bundles and emit these bundles rather than emitting the items one at a time

"emit as bundles", perfect! uh... not exactly, it says "periodically" and that literally means periodically. So EVERY X seconds/minutes it will emit a list of objects regardless of whether there were any taps/events. This would mean a bunch of empty lists would periodically be emitted when no taps were registered. I could get the example working if I tweaked the time component for `buffer` just enough and `filter` out the empty results, but it was a hack and not the way of the Rx.

What I really needed was a selective combination of debounce and buffer - a "debouncedBuffer" operator. Such an operator doesn't exist but you could achieve something similar by calling `.buffer()` with a special "boundry observable" parameter, as Jedi master Ben Christensen points out in this [Stack Overflow post](http://stackoverflow.com/questions/24828897/how-to-group-events-by-idle-periods-using-reactive-extensions) (the Rx is strong with this one).

Essentially, if you pass an observable as a parameter to `buffer`, every time this observable emits an item, `buffer` will take the source observable and emit a *list* of items from the *source* observable (instead of the items emitted by the boundary observable).

{% codeblock lang:java linenos:false %}
<Source Observable For Actual Events>.buffer(<Boundary Observable that only tells "when" to take items from the Source>)
{% endcodeblock %}

So what we're going to do is use a "debouncedEventEmitter" as our boundary observable. The "debouncedEventEmitter" will basically emit a single item-which we really don't care about-only after a certain time has elapsed from the emission of the first item.

{% codeblock lang:java linenos:false %}
Observable<Object> debouncedEventEmitter = tapEventEmitter.debounce(1, TimeUnit.SECONDS);
{% endcodeblock %}

We now buffer our source observable again to give us a list of items (vs a single item) everytime the debouncedEventEmitter emits a single item.

{% codeblock lang:java linenos:false %}
Observable<List<Object>> debouncedBufferEmitter = tapEventEmitter.buffer(debouncedEventEmitter);
{% endcodeblock %}

Altogether now:

{% gist 76a6b4c97a193a7461e0b84485913c5e %}

Notice the `.share()` operator? In [my next post](http://blog.kaush.co/2015/01/21/rxjava-tip-for-the-day-share-publish-refcount-and-all-that-jazz/), I'll go into the details of that operator along with `.publish()` and `refcount()`.

*[UPDATE: [Ben pointed](https://twitter.com/benjchristensen/status/552709457856049152) me to a [niftier implementation of debounced buffer](https://gist.github.com/benjchristensen/e4524a308456f3c21c0b). I've added a [third variant of the Bottom fragment](https://github.com/kaushikgopal/Android-RxJava/blob/master/app/src/main/java/com/morihacky/android/rxjava/rxbus/RxBusDemo_Bottom3Fragment.java) that uses this approach. A [subsequent post](http://blog.kaush.co/2015/01/21/rxjava-tip-for-the-day-share-publish-refcount-and-all-that-jazz/) will go into the details.]*

_Follow discussion on [Reddit](http://www.reddit.com/r/androiddev/comments/2rffd2/debouncedbuffer_with_rxjava/)._
