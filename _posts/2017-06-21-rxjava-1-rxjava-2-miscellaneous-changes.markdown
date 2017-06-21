---
layout: post
title: RxJava 1 -> RxJava 2 (Miscellaneous Changes)
date: 2017-06-21T13:10:05-07:00
published: false
---

This is the final post in a 3 part series:

1. Understanding the changes
2. Disposing subscriptions
3. Miscellaneous changes


Let's look at some of the other changes that came about.

<h2> onComplete<strike>&nbsp;d&nbsp;</strike></h2>

    Search : `onCompleted()`
    Replace: `onComplete()`

The Reactive Streams spec has [defined Subscribers to have "onComplete" events](https://github.com/reactive-streams/reactive-streams-jvm/blob/v1.0.0/api/src/main/java/org/reactivestreams/Subscriber.java). RxJava 1.x previously used the past tense onComplete"d", so you'll have to do a quick search and replace on those.

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



* `Processor`s do not return a `Disposable`/`Subscription` anymore. So disposing the connection is done a tad bit differently 


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
