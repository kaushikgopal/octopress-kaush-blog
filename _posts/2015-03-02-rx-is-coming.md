---
layout: post
title: "Rx Is Coming"
date: 2015-03-02T18:24:28-08:00
---


i spent a huge part of my 2014 on "[Rx](https://rx.codeplex.com/)" (or reactive extensions) which is essentially a library that helps with a development pattern called "reactive programming". i think Rx is going to be huge in the app development world. it's already picked up a lot of steam, but i think it's going to become a **staple** for professional app developers.

_consider this extremely common scenario_:

you're in [Burundi](http://en.wikipedia.org/wiki/Burundi) surfing facebook. you want to get the latest updates from facebook, so  your smart phone (from a network in Burundi) shoots a request to the facebook servers in Menlo Park. that server reconciles your request and sends back 5 epic selfies from your dearest friends. in our universe, this takes time. maybe not a whole lot according to you (a second perhaps) but for that mini super-computer you hold in your hand, that's 1000000 microseconds. that's a long time that you're asking it to twiddle its thumbs (given that it usually processes stuff in a couple of microseconds). what's it supposed to do until it hears back from the facebook servers? after it's done showing you the updates for your first screen, when would it know to trigger another update? what if the data that came back from the servers indicated that you need some more information? what if the third request in that chain failed for some reason? welcome to the world of asynchronous programming. Rx (or FRP as it is mistakingly assumed to be synonymous with) was built to deal with a lot of this pain.

a natural question is: don't app developers do this anyway today? yes we do. but it's painful and error prone. it also results in bad UX (spinners, progress indicators, alert dialogs for failed network issues and other technical problems that users really  shouldn't have to deal with).

you can get by very far without having to deal with many of the aforementioned problems (like keeping all your information saved on your device instead of say a big server machine located elsewhere that would require a network connection) but as more services move to storing data online, streaming data from servers, sharing data across multiple platforms and devices, dealing with the complexity of asynchronous programming has become common place.

how is it then, that every developer you know, isn't already talking about this Rx stuff? the reception to Rx ([see what i did there?](http://en.wikipedia.org/wiki/Rx)) in the developer community has been ...warm. the few serious developers who have started to use it, swear by it and say it's the most amazing thing ever. the ones that are yet to go through this feat, currently see it as another fancy hipster developer pattern. it also has a very steep learning curve. this doesn't help its cause (did i mention it happens to be a pattern [pioneered by Microsoft](https://msdn.microsoft.com/en-us/data/gg577609.aspx)?).

but make no mistake, Rx is coming. eventually everyone is going to suck it up and use it in its current form, or the dev community will fashion a variant that's easier for newer developers to pick up.