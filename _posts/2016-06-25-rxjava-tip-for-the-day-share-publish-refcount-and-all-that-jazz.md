---
layout: post
title: RxJava Tip for the Day - Share, Publish, Refcount and All That Jazz
date: 2015-01-21 10:41
categories: android RxJava
---

_Originally posted this article on the Wedding Party tech blog._
<hr />


Ok, so in [my previous post](http://blog.kaush.co/2015/01/05/debouncedbuffer-with-rxjava/) I innocuously introduced the `.share()` operator.

{% codeblock lang:java linenos:false %}
Observable<Object> tapEventEmitter = _rxBus.toObserverable().share();
{% endcodeblock %}

## What is this share operator?

<!-- more -->

The `.share()` operator is basically just [a wrapper to the chained call `.publish().refcount()`](https://github.com/ReactiveX/RxJava/wiki/Connectable-Observable-Operators#connectableobservablerefcount).

You'll find the chained combo `.publish().refcount()` used in quite a few Rx examples on the web. It allows you to "share" the emission of the stream. Considering how powerpacked and frequently used this combo is, RxJava basically introduced the friendlier more useful operator `share()`. This mechanism is sometimes referred to as  "multicasting".

Let's dig into some of the basics first:

> "[ConnectedObservable](https://github.com/ReactiveX/RxJava/wiki/Connectable-Observable-Operators)" - This is a kind of observable which doesn't emit items even if subscribed to. It only starts emitting items after its `.connect()` method is called.

*It is for this reason that a connected obesrvable is also considered "cold" or "inactive" before the connect method is invoked.*

> `.publish()`- This method allows us to change an ordinary observable into a "ConnectedObservable". Simply call this method on an ordinary observable and it becomes a connected one.

We now know what 1/2 of the operator `share` does. Now why would you ever use a Connected Observable? [The docs say](https://github.com/ReactiveX/RxJava/wiki/Connectable-Observable-Operators):

> In this way you can wait for all intended Subscribers to subscribe to the Observable before the Observable begins emitting items.

This essentially means that a regular usecase for `publish` would involve more than one subscriber. When you have more than one subscriber, it can get tricky to handle each of the subscriptions and dispose them off correctly. To make this process easier, Rx introduced this magical operator called `refcount()`:

> `refcount()` - This operator keeps track of how many subscribers are subscribed to the resulting Observable and refrains from disconnecting from the source ConnectedObservable until all such Observables are unsubscribed.

It essentially maintains a reference counter in the background and accordingly takes the correct action when a subscription needs to be unsubscribed or disposed off. This is the second 1/2 of the operator `share`. You are now armed with knowledge of what each of those terms mean.

Let's look at the example from debouncedBuffer again and see how `share` was used there:

{% gist 53b6279aa77217514e611be1fd6f383d %}

We now have a "shareable" observable called "tapEventEmitter" and because it's sharable and still not yet 'live' (`publish` from the `share` call changes it to a ConnectedObservable), we can use it to compose our niftier Observables and rest assured that we always have a reference to the original observable (the original observable being `_rxBus.toObserverable()` in this case).


{% gist a7117fb7cc14f2eabec960287bb9a5ea %}

All this sounds good. There is however a possible race condition with this implementation (which [Ben pointed out through a comment on this gist](https://gist.github.com/benjchristensen/e4524a308456f3c21c0b#comment-1367814)). The race condition occurs because there are two subscribers here (debounce and buffer) and they may come and go at different points. Remember that the RxBus is backed by a hot/live Subject which is constantly emitting items. By using the `share` operator we guarantee a reference to the same source, but NOT that they'll receive the exact same items if the subscribers enter at different points of time. Ben explains this well:

> The race condition is when the two consumers subscribe. Often on a hot stream it doesn't matter when subscribers come and go, and refCount is perfect for that. The race condition refCount protects against is having only 1 active subscription upstream. However, if 2 Subscribers subscribe to a refcounted stream that emits 1, 2, 3, 4, 5, the first may get 1, 2, 3, 4, 5 and the second may get 2, 3, 4, 5.

> To ensure all subscribers start at exactly the same time and get the exact same values, refCount can not be used. Either ConnectableObservable with a manual, imperative invocation of connect needs to be done, or the variant of publish(function) which connects everything within the function before connecting the upstream.

In our usage it's almost immediate so it probably wouldn't matter a whole lot but considering the use case, it's ideal to have the exact same events emitted for the debouncedBuffer usecase. [I added a third improved implementation to handle this race condition](https://github.com/kaushikgopal/Android-RxJava/blob/master/app/src/main/java/com/morihacky/android/rxjava/rxbus/RxBusDemo_Bottom3Fragment.java):

{% gist cf3b5ebac5380632999d4c648fa359a0 %}
