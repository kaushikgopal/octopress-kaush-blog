---
layout: post
title: RxJava 2 for the RxJava 1 luddites
date: 2017-04-08 16:20
comments: false
published: false
---

In case you haven't heard: [RxJava2](https://github.com/ReactiveX/RxJava/wiki/What%27s-different-in-2.0https://github.com/ReactiveX/RxJava/wiki/What%27s-different-in-2.0) was released sometime back. RxJava 2 was a massive rewrite with  breaking apis. But it was a good thing and most dependent libraries have upgraded by now, so you're safe to pull that migration trigger with your respective codebases. 

This guide:
  
 * is aimed at folks who've used RxJava 1 but haven't pushed themselves to upgrade to 2.x yet
 * does not explain "why" things changed with RxJava2
 * gives you the most important search/replace combos for a quick and sane migration
 * contains most of the basic changes you'll need to finish an RxJava 1 -> 2 migration

other guides have been written, but i wanted somethign that would help me make sense of the changes. Why did things change in the way they did.
should help you understand why the changes were made and how to go about thinking.

The objective of this guide is

Keanu Reeves Meme - i now understand RxJava 2

# Why things changed with RxJava2

Nope, not explaining that with this guide. But if you're curious about this and want to learn about the details (highly recommended at some point), you should go through these resources in order:
 
* [Fragmented Ep #53 with JakeWharton](http://fragmentedpodcast.com/episodes/053-jake-wharton-on-rxjava-2/) (forgive the shameless promotion) - ultimate lazy person's guide to understand why/what things changed with RxJava2, as explained by an actual demigod. This was the one that really made it click for me.
<!-- * [Exploring RxJava2 for Android](https://youtu.be/htIXKI5gOQU?t=14m49s) - if you'd rather "see" said demigod, check this video out. I would start at the 14:50 mark. --> 
* The [aforelinked wiki](https://github.com/ReactiveX/RxJava/wiki/What%27s-different-in-2.0) - this is really the place I kept coming back to and referencing when I needed to understand the details
* [Thought process behind the 2.0 design](https://github.com/ReactiveX/RxJava/issues/2787) for the truly loyal


# Reactive Streams spec

Ok I lied a little, but I think a brief explanation of the intention behind the move will actually help you with the migration especially since there's an interim period where we're juggling with both Rx1 and Rx2 terminology.

### Why was RxJava2 such a massive change? 

tl;dr- [Reactive Streams](http://www.reactive-streams.org/). 

This is a standard for doing "reactive" programming and now RxJava 2 implements the Reactive Streams specs. This allows interop with other Reactive libraries and just makes the world a more friendlier and reactive place. 

The [Reactive Streams spec](https://github.com/reactive-streams/reactive-streams-jvm/tree/v1.0.0/api/src/main/java/org/reactivestreams) is pretty straightforward with just 4 interfaces:

* `Publisher` (anything that publishes events, so `Observable`,`Flowable` etc.)
* `Subscriber` (anything that listens to a Publisher)
* `Subscription` (`Publisher.subscribe(Subscriber) => Subscription` when you join a Publisher and a Subscriber, you are given a connection also called a `Subscription`)
* `Processor` (a Publisher + a Subscriber, sound familiar? yep `Subject`s for us RxJava folks)

# Show me the changes already! :

## dependencies

Search:

    compile "io.reactivex:rxjava:${rxJavaVersion}"
    compile "io.reactivex:rxandroid:${rxAndroidVersion}"
    
    compile "com.jakewharton.rxbinding:rxbinding:${rxBindingsVersion}"
    compile "com.squareup.retrofit2:adapter-rxjava:${retrofit2Version}"

Replace:

     compile "io.reactivex.rxjava2:rxjava:${rxJavaVersion}"
     compile "io.reactivex.rxjava2:rxandroid:${rxAndroidVersion}"
     
     compile "com.jakewharton.rxbinding2:rxbinding:${rxBindingsVersion}"
     compile "com.squareup.retrofit2:adapter-rxjava2:${retrofit2Version}"

    
* note the package change
* you "could" have both versions running simultaneously, but this is a *bad* *bad* idea. bite the bullet and migrate it all in one shot

<Explain why>

    
## Observable -> Flowable 

Search : `import rx.Observable;`
Replace: `import io.reactivex.Flowable;`

Flowable = Observable + backpressure handling

* Flowable is the new Observable. Succinctly - it’s a backpressure-enabled base reactive class.
* you want to be using Flowable everywhere now, **not** Observable. Use this as your default.
* Observables are still available for use, but unless you really understand backpressure, you probably don't want to be using them anymore.


### note on Publishers:

Remember `Publisher`? it's basically the Reactive Streams interface for the thing that produces events (_See the Reactive Streams spec section above if this is not making sense_).

`Flowable`s implement `Publisher`. They are our new base default reactive class and implement the Reactive Streams spec 1:1. Think of of Flowable as primero uno `Publisher`.

The other base reactive classes that are Publishers include `Observable`s, `Single`s, `Completable`s and `Maybe`s. But they don't implement the `Publisher` interface directly. Why you ask? 

Well the other base classes are now considered Rx specific things. Instead of having the standard `Subscriber` (according to the Reactive Streams standard) listen to them, they have corresponding special `Subscriber`s that listen to them. The interface though is very very similar to a Publisher.

    // Reactive Strems spec
    // Flowable implements Publisher
    interface Publisher<T> {
        void subscribe(Subscriber<? super T> s);
    }

    // RxJava equivalents
    
    // Observable implements ObservableSource
    interface ObservableSource<T> {
        void subscribe(Observer<? super T> observer);
    }
    
    // Single implements SingleSource
    interface SingleSource<T> {
        void subscribe(SingleObserver<? super T> observer);
    }
    
    interface CompletableSource {
        void subscribe(CompletableObserver observer);
    }
    
    interface MaybeSource<T> {
        void subscribe(MaybeObserver<? super T> observer);
    }

<!-- If you forget the Rx 1 terminology and remember the Reactive Streams spec now, all of this should make sense. -->

### note on Subscribers

`Publisher` is the Reactive Streams event producer. `Subscriber` is the Reactive Streams event listener. It's extremely helpful to keep these terms firmly grounded in our heads, hence the incessant repetition.

Search : `onCompleted()`
Replace: `onComplete()`

The Reactive Streams spec has [defined Subscribers to have "onComplete" events](https://github.com/reactive-streams/reactive-streams-jvm/blob/v1.0.0/api/src/main/java/org/reactivestreams/Subscriber.java) (notice the absence of past tense).

# Disposing Subscriptions

This was the part that I initially found most tricky but is also probably the most important part to know well (as AndroidDevs - memory leak and all).

Jedi master Karnok explains this best in the wiki:

> In RxJava 1.x, the interface rx.Subscription was responsible for stream and resource lifecycle management, namely unsubscribing a sequence and releasing general resources such as scheduled tasks. The Reactive-Streams specification took this name for specifying an interaction point between a source and a consumer: org.reactivestreams.Subscription allows requesting a positive amount from the upstream and allows cancelling the sequence.

From that definition alone, it would appear like nothing's changed, but that is definitely not the case. Early on, I pointed out 

`Publisher.subscribe(Subscriber) => Subscription`

The use of `=>` vs `=` was intentional. If you look at the [source code for `Publisher`'s subscribe method](https://github.com/reactive-streams/reactive-streams-jvm/blob/v1.0.0/api/src/main/java/org/reactivestreams/Publisher.java#L28), you'll notice a return type of void viz. it doesn't return a Subscription for you to tack on to a CompositeSubscription (which you can then conveniently dispose off onStop/onDestroy).

> Because Reactive-Streams base interface, org.reactivestreams.Publisher defines the subscribe() method as void, Flowable.subscribe(Subscriber) no longer returns any Subscription (or Disposable). The other base reactive types also follow this signature with their respective subscriber types.

So you may ask how do I get a hold off that Subscription then (so that you might rightly cancel or dispose it off like a responsible citizen)? Have a look at the [source code for `Subscriber`'s onSubscribe](https://github.com/reactive-streams/reactive-streams-jvm/blob/v1.0.0/api/src/main/java/org/reactivestreams/Subscriber.java#L31):

    public interface Subscriber<T> {
  
        public void onSubscribe(Subscription s);
      
        // Subscriptions are additionally cool cause they have:
        // s.request(n) -> request data
        // s.cancel()   -> cancel this connection
        
        public void onNext(T t);
        public void onError(Throwable t);
        public void onComplete();
    }

You are now given the Subscription class as a parameter in your onSubscribe callback. You can then conveniently dispose off the Subscription inside the `onSubscribe` callback. This was actually a pretty well thought off change because this really makes the interface for a Subscriber lightweight. In RxJava 1 land, Subscribers were more heavy cause they had to deal with a lot of the internal state handling.  

... who are we kidding: that is not convenient at all (for us lifecycle dependents). I'd rather just shove everything into a CompositeSubscription like before and be on my merry way. But such are the rulings of the Reactive Streams spec. 

Thankfully the maintainers of RxJava in all their benevolence realized this trade-off and have remedied this with convenient helpers. But first, some more definitions:

## `Disposable` is the new `Subscripton`

What we called `Subscription`s in RxJava 1 are now called `Disposable`s. 

Why couldn't we just keep the name `Subscription`? Per my understanding:

1. You have to remember the Reactive Streams spec already has this name reserved and the maintainers of RxJava 2 are serious about the spec adherence. We don't want confusion about there being more functionality with an Rx Subscription vs other Reactive Stream spec adhering libraries
2. We still want some of the behaviors and conveniences of RxJava 1 like CompositeSubscriptions.

So if `Disposable`s is what we're using now, by that token we have a super charged `CompositeDisposable`. This is the object you want to be using and tacking all your Disposables on. It functions very similarly to how we used `CompositeSubscription` before. 

If you were following closely though, you'd still ask the question? ok... but how does that make it anymore convenient? `Publisher.subscribe(Subscriber)` still doesn't return a Disposable/Subscription for us to tack on to the CompositeDisposable.

That's why the RxJava maintainers added a handy method on most Publishers called `subscribeWith`. From the wiki:

> Due to the Reactive-Streams specification, Publisher.subscribe returns void and the pattern by itself no longer works in 2.0. To remedy this, the method E subscribeWith(E subscriber) has been added to each base reactive class which returns its input subscriber/observer as is. 

If you're still following closely, you'd again ask... wait! that doesn't return a Subscription! it doesn't event return a Disposable! why the hell is this even remotely more convenient?

Well.. it says that the `Subscriber` you pass is sent back to you with `subscribeWith`. What if your Subscriber "implemented" the Disposable interface? You could then conveniently just tack it on to a CompositeDisposable right?

RxJava 2 conveniently provides you with some Subscriber helper classes that do this for you. We now have 

* `DisposableSubscriber` - the one I recommend using when you can
* `ResourceSubscriber` - 

Other versions also exist like `DefaultSubscriber` and `FlowableSubscriber` but those don't implement the Disposable interface.

* `Processor`s do not return a `Disposable`/`Subscription` anymore. So disposing the connection is done a tad bit differently 


## Subject -> Processor

Search : `import rx.subjects.PublishSubject;`
Replace: `import io.reactivex.processors.PublishProcessor;`

Processor = Subject + backpressure handling. 

* Just like how Observables continue to exit and don't handle backpressure, Subjects continue to exist and don't handle backpressure.
* In the same vein, you should be defaulting to Processors instead of Subjects now.
* The equivalent variants (AsyncProcessor, BehaviorProcessor, PublishProcessor, ReplayProcessor etc.) all exist.
* Did you know there was this Subject called a [UnicastSubject](http://reactivex.io/RxJava/javadoc/rx/subjects/UnicastSubject.html)?

note:
 mDateSubject.asObservable();
mDateSubject.cast(Date.class);

# Operators

## Observable.create unsafeCreate
 
 this is a win


## Observable.from has become more specific

Search : `Observable.from()`
Replace:  `Flowable.from<paramter-type>()`

This is a super common and useful operator that you most likely used. It's changed to be more specific. This operator now exists only with specific variants like `Flowable.fromIterable` for (Array)Lists, `Flowable.fromArray` for (yes you guessed that right), `Flowable.fromPublisher` (everything has been modelled after the Reactive-Streams spec), , `Flowable.fromCallable` (when you just want to create a stream from a block of code) and some other ones we probably don't use (as #AndroidDev) like `Flowable.fromFuture` etc.

# Functions

The previous `Func`ky names have been changed. I'm not entirely sure why these changed, but my guess is the same: model the Java 7 + Reactive Streams world a little more closely.

Search : `rx.functions.Func0                  call`
Replace: `java.util.concurrent.Callable       call`

^ notice the Java 7 native Callable reference

Search : `rx.functions.Func1                  call`
Replace: `io.reactivex.functions.Function     apply`

Search : `rx.functions.Func2                  call`
Replace: `io.reactivex.functions.BiFunction   apply`


Search : `rx.functions.Func<3..9>                call`
Replace: `io.reactivex.functions.Function<3..9>  apply`

# Actions

We got Actions and Consumers now

Search : `rx.functions.Action0                call`
Replace: `io.reactivex.functions.Action       run`

Search : `rx.functions.Action1                call`
Replace: `io.reactivex.functions.Consumer     accept`

Search : `rx.functions.Action2                call`
Replace: `io.reactivex.functions.BiConsumer   accept`

* if you made that switch to retrolambda, this part of the migration would have been significantly simpler.

# Schedulers

The only difference here is the package move.

Search : `import rx.schedulers.Schedulers;`
Replace: `import io.reactivex.schedulers.Schedulers;`

Search : `import rx.Scheduler;`
Replace: `import io.reactivex.Scheduler;`

Search : `import rx.android.schedulers.AndroidSchedulers;`
Replace: `import io.reactivex.android.schedulers.AndroidSchedulers;`

:v: you're all set.
 

# Transformers

Search : `rx.Observable.Transformer`
Replace:  `io.reactivex.<reactive-object>Transformer;`

Transformers are first class citizens now and require to be imported separately. Since you're now using Flowables the common one is going to be `FlowableTransformer`. There are other variants like `CompletableTransformer`, `MaybeTransformer`, `ParallelTransformer`, `SingleTransformer` and yes`ObservableTransformer` (but c'mon dude stop with the Observable already).

Don't know what Transformers are?

* Transformers are cool (yup, those ones too but fu Michael Bay)
* look up the `.compose` operator, it's the :bomb:
* demigod dlew [explains Transformers in this blog post really well (Don't break the chain)](http://blog.danlew.net/2015/03/02/dont-break-the-chain/)


## Performance


The complete rewrite of 2.x improved our memory consumption and performance considerably; here is a benchmark that compares various versions and libraries.
https://twitter.com/akarnokd/status/752465265091309568
https://github.com/akarnokd/akarnokd-misc/tree/master/src/jmh/java/hu/akarnokd/comparison/scrabble

# No more null in your streams

* Nulls ain't allowed, this can potentially be painful. See this discussoin https://github.com/ReactiveX/RxJava/issues/4644. I personally am a fan of this choice though.

This is mildly controversial. 

I used to do a whole bunch of Observable<Void> in my early days, this came to bite me here. cause you basically can't ever allow a null object in your stream

Observable.fromCallable({
    return null;
});

you want to change these to something like this from hereon:

Completable.fromAction(() -> {
   // no return necessary
});

# Miscellaneous thoughts:



https://medium.com/@theMikhail/rxjava2-an-early-preview-5b05de46b07

* `TestSubject`s are dead. But you should be using [`TestScheduler`s and `TestSubscriber`s](https://github.com/Froussios/Intro-To-RxJava/blob/master/Part%204%20-%20Concurrency/2.%20Testing%20Rx.md) anyway :point_left: great resource on testing Rx btw.

Testing is now super easy.


* The previous marble diagrams and docs that we've come to love haven't all been updated yet, so [keep these javadocs handy](http://reactivex.io/RxJava/2.x/javadoc/) when you start
* If you did the due diligence of migrating to retrolambda, the search-replacing during this migration is significantly reduced


# Migration diffs for some open source repositories:

* [RxJava-Android-Samples](https://github.com/kaushikgopal/RxJava-Android-Samples/pull/83)
* [TrueTime Android](https://github.com/instacart/truetime-android/pull/51/files)

_(If you've seen other repos that have clubbed all the Rx2 migrations to a single PR, please do send em over and i'll try to include them here)_

Tried to make this fun and embedded some gems/concepts for the intermediate Rx user too.
Some of those links should take you to interesting source code locations