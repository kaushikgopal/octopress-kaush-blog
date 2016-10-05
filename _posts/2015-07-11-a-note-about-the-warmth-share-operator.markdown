---
layout: post
title: A note about the warmth of the share and replay operators
date: 2015-07-11T14:27:42-07:00
categories:
- dev
tags:
- rxjava
- android
---

A common question most android developers have when using RxJava is: how do I cache or persist the work done by my observables over a configuration change? If you start a network call and the user decides to rotate the screen, the work spent in executing that network call would have been in vain (considering the OS would re-create your activity).

There are two widely accepted solutions to this problem:

1. store the observable somewhere as a singleton, possibly apply the [cache](https://github.com/ReactiveX/RxJava/wiki/Observable-Utility-Operators) operator and re-subscribe on activity recreation
2. house your observable in a retained "worker" fragments [^1]

I [whipped up a quick example](https://github.com/kaushikgopal/RxJava-Android-Samples#rotation-persist) on [github](https://github.com/kaushikgopal/RxJava-Android-Samples) to demonstrate the second technique (which is generally my choice of poison). Now I've used the technique of worker fragments (successfully) a bunch of times before so I was a little surprised to see the example not work.

Let's go over the use case again:

You have an observable that executes a long running network call. Before the call completes, you perform an activity rotation. After the activity is recreated, you continue the network call from where it left off or just use the result if it completes before your activity recreation process.

Instead of simulating this network call use case I decided to just fake it with a "hot" observable instead (which makes the use case a tad bit different but would help demonstrate the solution equally well). If you're looking for a quick simple example that demonstrates the difference between a hot and cold observable, I strongly recommend watching this [egghead.io video on the subject](https://egghead.io/lessons/rxjs-demystifying-cold-and-hot-observables-in-rxjs).

In fact, I used the exact same concoction of operators for my example:

{% gist 76628d4a10eb82f55aad %}

A little more investigation revealed that the share operator I used to fake the source stream was not really hot but "warm". This is easier explained with Marble diagrams:

Here's how we expect share to behave (and it does):

![share marble diag 1](/images/marble_diag_share_1.jpg "Marble Diagram share 1")

But owing to the activity recreation, what really happens is that the first subscriber (S1) unsubscribes from the source observable (O1 - housed in the worker fragment), after which a similar subscriber (S2) from the recreated activity subscribes again to O1 from the same worker fragment. So the marble diagram really lands up looking like this:

![share marble diag 2](/images/marble_diag_share_2.jpg "Marble Diagram share 2")

Notice that re-subscription? That changes things a little.

![share marble diag 3](/images/marble_diag_share_3.jpg "Marble Diagram share 3")

In this way, the share operator is "warm". It behaves cold to first time subscribers but hot to subsequent ones.

> the share operator behaves cold to first time subscribers but hot to subsequent ones

## Epilogue:

So how did I circumvent the problem? I added a fake subscriber that never unsubscribes.

{% gist 1f1a3aa37774ebe3032d %}

Clever but very hacky[^2]. Think carefully before you write code like this in production.

## Epilogue (Part 2):

[@fabioCollini](https://twitter.com/fabioCollini/status/620664072770592768) on twitter said he preferred a replay().connect() approach over the publish().refcount() one which is [what the share operator really is](http://blog.kaush.co/2015/01/21/rxjava-tip-for-the-day-share-publish-refcount-and-all-that-jazz/).

This reminded me of a tip that [Dan](https://twitter.com/danlew42) mentioned once, replay is similar to share in that it’s hot to the first subscriber and cold to every other subscriber after the first item emits[^3].

> replay is hot to the first subscriber and cold to every other subscriber after the first item emits

Because marble diagrams are amazing:


![replay marble diag 1](/images/marble_diagram_replay.001.jpg "Marble Diagram replay 1")

![replay marble diag 1](/images/marble_diagram_replay.002.jpg "Marble Diagram replay 2")

![replay marble diag 1](/images/marble_diagram_replay.003.jpg "Marble Diagram replay 3")


I've since [modified the example to use a ConnectableObservable ](https://github.com/kaushikgopal/RxJava-Android-Samples/commit/1812c18064a1a508b3d704be21137b1e3ab868f0?diff=split) instead and it works pretty much the same way.


[^1]: Alex Lockwood wrote about [this approach](http://www.androiddesignpatterns.com/2013/04/retaining-objects-across-config-changes.html) here.
[^2]: I can't take credit for this idea. The comments in the code reveal the identity of the troublemaker.
[^3]: Depending on what it’s replaying, it could also be cold to the first subscriber (he is quick to point out)
