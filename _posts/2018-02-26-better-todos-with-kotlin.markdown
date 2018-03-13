---
layout: post
title: Smarter ToDos With Kotlin
date: 2018-02-26T20:45:09-08:00
---

Kotlin already has [TODO](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin/-t-o-d-o.html)s. That's awesome! but it's a tad bit aggressive. 

Take this piece of code for instance:

    fun finishAwesomeFeature() {
        callAwesomeFeature()
        TODO("add analytics so data scientists stop harassing me")
    }

If your app happens to call awesome feature, Kotlin will blow it up! 

    Exception in thread "main" kotlin.NotImplementedError: An operation is not implemented: add analytics so data scientists stop harassing me
        at AwesomeFeatureKt.finishAwesomeFeature

I wanted something similar, but a tad bit gentler:

* A reminder "in code" that I needed to get something done - the code is my documentation, to-do list etc. I don't need to sign in or go to another tool to check my code. I'm always in it.
* Never EVER blow up for my user or in production - I can't/won't allow my users to suffer for my tardiness.
* Blow up aggressively for me and my fellow developers - They're tough and can take it (rather they'll make sure to be tough on me).
* Provide a "due date" for said blow up - Todo's have due dates, I don't always _have_ to do it right now\*. I mean the analytics can wait\*\*.

Kotlin -the nifty devil that it is- allowed me to make some minor tweaks and get all of the above:

    // in an appropriately named file: DevCop.kt

    fun ToDo(reason: String, date: Date) {
        if (BuildConfig.DEBUG && Date().after(date)) {
            TODO(reason)
        }

        Timber.v("operation $reason not implemented (yet)")
    }

There you have it!

* won't blow up in production
* will blow up for developers
* but only if due date has passed

To make it even more nifty, there's a quick date creator extension as well that allows us to do something like this:

    ToDo("Remove tagged comments after xx-xxxxxx feature stable",
          ISDate(day = 28, month = 3))

Happy to share the code for `ISDate` but it's just a more sane date builder method (3 = March cause we're not insane like some of the early Java developers).

Cheers

--------------------------------

\* - [if you can't already tell, i'm a wonderful procrastinator](http://invisiblebread.com/2014/07/procrastinator/) 
<br />
\*\* - Just kidding David, we always get the analytics done right away.