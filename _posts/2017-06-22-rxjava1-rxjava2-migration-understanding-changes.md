---
layout: post
title: RxJava 1 -> RxJava 2 (Understanding the changes)
date: 2017-06-21T13:09:50-07:00
---

In case you haven't heard: [RxJava2](https://github.com/ReactiveX/RxJava/wiki/What%27s-different-in-2.0https://github.com/ReactiveX/RxJava/wiki/What%27s-different-in-2.0) was released sometime back. RxJava 2 was a massive rewrite with  breaking apis (but for good reasons). Most dependent libraries have upgraded by now though, so you're safe to pull that migration trigger with your codebases. 

_Folks starting out directly with Rx2 might enjoy this guide but it's the ones that started with Rx 1 that will probably appreciate it the most_.

I've split this guide into 3 parts:

1. [Understanding the changes](http://blog.kaush.co/2017/06/21/rxjava1-rxjava2-migration-understanding-changes/)
2. [Disposing subscriptions](http://blog.kaush.co/2017/06/21/rxjava-1-rxjava-2-disposing-subscriptions/)
3. Miscellaneous changes

Let's get started.

# Why things changed with RxJava2

There are a couple of other guides explaining how one should migrate to Rx 2 but I wanted something that would help me make "sense" of the changes. Why exactly did things change and how should I now start thinking about them.

## tl;dr- [Reactive Streams](http://www.reactive-streams.org/) spec

[Reactive Streams](http://www.reactive-streams.org/) is a standard for doing "reactive" programming and RxJava now implements the Reactive Streams specs with version 2.X. RxJava was sort of a trailblazer in reactive programming land, but it wasn't alone. There were other libraries that also had reactive paradigms. But with all the libraries adhering to the Reactive Streams spec now, interop between the libraries is a tad bit easier. 

The [spec](https://github.com/reactive-streams/reactive-streams-jvm/tree/v1.0.0/api/src/main/java/org/reactivestreams) per say is pretty straightforward with just 4 interfaces:

1. `Publisher` (anything that publishes events, so `Observable`,`Flowable` - more on this later etc.)
2. `Subscriber` (anything that listens to a Publisher)
3. `Subscription` (`Publisher.subscribe(Subscriber) => Subscription` when you join a Publisher and a Subscriber, you are given a connection also called a `Subscription`)
4. `Processor` (a Publisher + a Subscriber, sound familiar? yep `Subject`s for us RxJava 1 luddites)

_________________

If you're more curious about the design and goals, I also suggest the following resources:

* [What's different in 2.0](https://github.com/ReactiveX/RxJava/wiki/What%27s-different-in-2.0)  wiki page - this is really the place I kept coming back to and referencing when I needed to understand the details
* [Fragmented Ep #53 with JakeWharton](http://fragmentedpodcast.com/episodes/053-jake-wharton-on-rxjava-2/) (forgive the shameless promotion) - ultimate lazy person's guide to understand why/what things changed with RxJava2, as explained by an actual demigod. This was the one that really made it first click for me.
* [Thought process behind the 2.0 design](https://github.com/ReactiveX/RxJava/issues/2787) for the truly loyal
* [Ep 11 of The Context ](https://github.com/artem-zinnatullin/TheContext-Podcast/blob/master/show_notes/Episode_11.md) —  by my friends Hannes and Artem :)
<!-- * [Exploring RxJava2 for Android](https://youtu.be/htIXKI5gOQU?t=14m49s) - if you'd rather "see" said demigod, check this video out. I would start at the 14:50 mark. --> 

________________________

We should probably get one important change out of the way:

# Dependency change

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

A minor change in your gradle dependency pull. Notice the package change there's a "2" suffix on import and the actual classes are now in the `io.reactivex` pacakge vs `rx`.

Also you "could" theoretically have both versions running simultaneously. But this is a *bad* idea because certain primary constructs like `Observable`s have a very different notion of handling streams (backpressure). It can be a nightmare during that interim period where you have to remember the behavior differences. Also if you happen to use both Rx1 and Rx2 `Observable`s you have to be careful about qualifying explicitly with the right package name (`rx.Observable` or `io.reactivex.Observable`). Very easy to mix and get wrong.

> Bite the bullet and migrate it all in one shot.

Another super important change:

# Observable -> Flowable 

Search : `import rx.Observable;`
Replace: `import io.reactivex.Flowable;`

* Flowable is the new Observable. Succinctly - it’s a backpressure-enabled base reactive class.
* you want to be using Flowable everywhere now, **not** Observable. Use this as your default.
* Observables are still available for use, but unless you really understand backpressure, you probably don't want to be using them anymore.

> Flowable = Observable + backpressure handling

# Note on "Publisher"s:

Remember `Publisher`? it's basically the Reactive Streams interface for anything that produces events (_Reread the Reactive Streams spec section above if this is not making sense_).

`Flowable` implements `Publisher`. This is our new base default reactive class and implements the Reactive Streams spec 1 <-> 1. Think of of Flowable as primero uno "Publisher".

The other base reactive classes that are Publishers include `Observable`, `Single`, `Completable` and `Maybe`. But they don't implement the `Publisher` interface directly. 

Why you ask? 

Well the other base classes are now considered "Rx" specific constructs with specialized behavior pertaining to Rx. These are not necessarily notions you would find in the Reactive Streams specs.

We can look at the interface declarations and it'll be clear.

# Note on "Subscriber"s:

So if `Publisher` is the Reactive Streams event producer. `Subscriber` is the Reactive Streams event "listener" (it's extremely helpful to keep these terms firmly grounded in our heads, hence the incessant repetition).

Looking at the actual interface code declaration should offer more clarity to the above two sections.

    // Reactive Streams spec 

    // Flowable implements Publisher
    
    interface Publisher<T> {
        void subscribe(Subscriber<? super T> s);
    }

As noted before, `Publisher` and `Subscriber` are part of the Reactive Streams spec. `Flowable` which is now going to be our numero uno base reactive class of choice implements `Publisher`. All good so far. 

But what about the other base reactive classes like `Observable`, `Single` etc. that we've come to love and use?

On the publishing side, instead of implementing the standard "Publisher" interface the other event producers implement corresponding RxJava specific sources.

    // RxJava specific constructs
    
    // Observable implements "ObservableSource"
    interface ObservableSource<T> {
        void subscribe(Observer<? super T> observer);
        // notice "Observer" here vs the standard "Subscriber"
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

Notice: instead of having the standard `Subscriber` (Reactive Streams standard) the other base reactive classes (`Observable`, `Single` etc.) now have corresponding "special" Rx specific `Subscriber` or event listeners called "Observer"s.

That's it for Part 1. In the [next part](http://blog.kaush.co/2017/06/21/rxjava-1-rxjava-2-disposing-subscriptions/), we'll look at disposing subscriptions.