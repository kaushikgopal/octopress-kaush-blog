---
layout: post
title: RxJava 1 -> RxJava 2 (Disposing Subscriptions)
date: 2017-06-21T13:09:54-07:00
---

This is a continuation post in a 3 part series:

1. Understanding the changes
2. Disposing subscriptions
3. Miscellaneous changes



# Disposing Subscriptions

This was the part that I initially found most tricky to grasp but also most important to know as an AndroidDev (memory leak and all).

Jedi master Karnok explains this best in the wiki:

> In RxJava 1.x, the interface rx.Subscription was responsible for stream and resource lifecycle management, namely unsubscribing a sequence and releasing general resources such as scheduled tasks. The Reactive-Streams specification took this name for specifying an interaction point between a source and a consumer: org.reactivestreams.Subscription allows requesting a positive amount from the upstream and allows cancelling the sequence.

From that definition alone, it would appear like nothing's changed, but that is definitely not the case. In my first post, I pointed out:

`Publisher.subscribe(Subscriber) => Subscription`

The use of `=>` vs `=` was intentional. If you look at the [source code for `Publisher`'s subscribe method](https://github.com/reactive-streams/reactive-streams-jvm/blob/v1.0.0/api/src/main/java/org/reactivestreams/Publisher.java#L28) again, you'll notice a return type of `void` viz. it doesn't return a Subscription for you to tack on to a CompositeSubscription (which you can then conveniently dispose off onStop/onDestroy).

    interface Publisher<T> {
        // return type void (not Subscription like before)
        void subscribe(Subscriber<? super T> s);
    }

Karnok again:

> Because Reactive-Streams base interface, org.reactivestreams.Publisher defines the subscribe() method as void, Flowable.subscribe(Subscriber) no longer returns any Subscription (or Disposable). The other base reactive types also follow this signature with their respective subscriber types.

So if you look at the declarations again

    // RxJava specific constructs    

    // Observable implements "ObservableSource"
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

Notice the return type `void` in all of them.

## Getting hold of Subscriptions

So you may ask how do I get a hold off that Subscription then (so that you might rightly cancel or dispose it off like a responsible citizen)? 

Let's take a look at the [declaration code for `Subscriber`'s onSubscribe](https://github.com/reactive-streams/reactive-streams-jvm/blob/v1.0.0/api/src/main/java/org/reactivestreams/Subscriber.java#L31) method:

    public interface Subscriber<T> {
  
        public void onSubscribe(Subscription s);
      
        // Subscriptions are additionally cool cause they have:
        // s.request(n) -> request data
        // s.cancel()   -> cancel this connection
        
        public void onNext(T t);
        public void onError(Throwable t);
        public void onComplete();
    }

You are now given the Subscription class as a parameter in your onSubscribe callback. So within the OnSubscribe method, you have a hold of the subscription and can then conveniently dispose off the Subscription inside the `onSubscribe` callback. 

This was actually a pretty well thought off change because this really makes the interface for a Subscriber lightweight. In RxJava 1 land, Subscribers were more "heavy" cause they had to deal with a lot of the internal state handling.  

... but who are we kidding: that is not convenient at all (atleast for those of us who need the subscriber to depend on our lifecycle). I'd rather just shove everything into a CompositeSubscription like before and be on my merry way. But such are the rulings of the Reactive Streams spec. 

Thankfully the maintainers of RxJava in all their benevolence realized this trade-off and have remedied this with convenient helpers. 

But first, some more definitions:

## `Disposable` is the new `Subscripton`

What we called `Subscription` in RxJava 1 is now called `Disposable`. 

Why couldn't we just keep the name `Subscription`? (per my understanding):

1. You have to remember the Reactive Streams spec already has this name reserved and the maintainers of RxJava 2 are serious about the spec adherence. We don't want confusion about there being more functionality with an Rx Subscription vs other Reactive Stream spec adhering libraries
2. We still want some of the behaviors and conveniences of RxJava 1 like CompositeSubscriptions.

So if `Disposable`s is what we're using now, by that token we have a `CompositeDisposable` which is the object you want to be using and tacking all your Disposables onto. It functions pretty similarly to how we used `CompositeSubscription` before. 

Ok, back to the original question: how do I get a hold of the Disposable?

## Getting hold of Disposables


Now before we go any further, if you're adding your callbacks directly in the form of lambdas, this is not a problem as most observable sources return a Disposable with their `subscribe` method call when not provided with a subscriber object:

    Publisher.subscribe(Subscriber)
    // void return type

    Publisher.subscribe(nextEvent -> {}, error -> {}, () -> {})
    // return Disposable so we're good

So if you look at some sample code, the below works fine no problem:

    disposable =
        myAwesomeFlowable
            .observeOn(AndroidSchedulers.mainThread())
            .subscribe(event -> {
               // onNext
            }, throwable -> {
               // onError
            }, () -> {
                // onComplete
            });
    
    compositeDisposable.add(disposable);

However if I rewrote that code just a little differently:

    disposable =
        myAwesomeFlowable
            .observeOn(AndroidSchedulers.mainThread())
            .subscribe(new FlowableSubscriber<TextViewTextChangeEvent>() {
              @Override
              public void onSubscribe(Subscription subscription) {
              }

              @Override
              public void onNext(TextViewTextChangeEvent textViewTextChangeEvent) {
              }

              @Override
              public void onError(Throwable t) {
              }

              @Override
              public void onComplete() {
              }
            });
    // ^ THIS IS WRONG. Won't work        
    // compositeDisposable.add(disposable);

The above code won't compile. If you want to pass a Subscriber object (like the above `FlowableSubscriber` or an `ObservableSource`) then this strategy won't work. 

A lot of existing RxJava 1 code uses this strategy a lot, so the RxJava maintainers very kindly added a handy method on most Publishers called `subscribeWith`. From the wiki:

> Due to the Reactive-Streams specification, Publisher.subscribe returns void and the pattern by itself no longer works in 2.0. To remedy this, the method E subscribeWith(E subscriber) has been added to each base reactive class which returns its input subscriber/observer as is. 

    E subscribeWith(E subscriber)

If you're still following closely, you'd ask... wait! that doesn't return a Disposable! why the hell is this even remotely more convenient?

Well... it says that the `Subscriber` you pass is sent back to you with `subscribeWith`. But what if your Subscriber itself "implemented" the Disposable interface? If you had a DisposableSubscriber, you could for all practical purposes treat it as a disposable and tack it on to a CompositeDisposable, while still using it as a Subscriber. That's typically the pattern you want to adopt. Here's some code that should make these techniques clear:

    disposable =
        myAwesomeFlowable
            .observeOn(AndroidSchedulers.mainThread())
            .subscribeWith(new getDisposableSubscriber<TextViewTextChangeEvent>() {
                @Override
                public void onNext(TextViewTextChangeEvent event) {}

                @Override
                public void onError(Throwable e) {}

                @Override
                public void onComplete() { }
            });
    
    compositeDisposable.add(disposable);

Apart from `DisposableSubscriber`, there's also a `ResourceSubscriber` which implements Disposable (no idea why you would use one over the other). 

There's also a `DefaultSubscriber` which doesn't implement the Disposable interface, so you can't use it with `subscribeWith`.

# to .clear or to .dispose

There's no longer an `unsubscribe` call on CompositeDisposable. It's been renamed to `dispose` ☝️️ but you don't want to be using either of those anyway. The `clear` method remains and is most likely the method you want to use. 

## What's the difference? 

unsubscribe/dispose [terminates even future subscriptions while clear doesn't](https://github.com/kaushikgopal/RxJava-Android-Samples/commit/1e7d4b2f867a97b32a0cde81cb488c3d17d4952f) allowing you to reuse the CompositeDisposable.

In the next and final part, we'll look at some of the miscellaneous changes.
