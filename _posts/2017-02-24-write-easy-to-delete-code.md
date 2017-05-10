---
layout: post
title: Writing easy to delete code and what it means for AndroidDevs
date: 2017-02-24 07:46
comments: false
published: false
external-url: 
---

first came across this here. i agreed with it, but didn't pause to really think about it in great detail and how big an impact it could have on my developement
http://programmingisterrible.com/post/139222674273/write-code-that-is-easy-to-delete-not-easy-to


A big theme and meditation subject for my programming goals this year has been the ability to move fast... rapidly fast. In service of that goal I've been exploring multiple techniques and approaches. Improving build times, improving my tooling etc.

But a buddy (and ex-boss) sent me a link over slack the other day saying,"this is something that has changed how I write software a little bit recently". This is one of the smartest software engineers I know, so despite having come across this concept before and agreeing with it, I decided to take a more closer look.

Write 

* move faster: cause you're more willing to try more things
* just better code
* easier for others to grok what's happening


# How do you write code thats easier to delete

* Liberally copy paste

DRY your code, and here DRY stands for Do Repeat Yourself
obviously there's a balance as is the case with almost anything. but i would default to repeating myself. i've been a strong proponent of this, and it's worked out great in my experience

* it's important to not confuse this as throw away or hacky code.
 
 For e.g. populating line items. which is easier to remove? i like to think the modular one is. thus encourages modularity
populateOrderCardWithBatchInfo(batch);



* Functional code is inherently easier to dispose off

e.g. from TrueTime

* Less Abstraction

write more layers of abstraction makes code more complex
base activity classes. you want to do something in the base class, change means changing in a bunch of places

* Composition vs Inheritance

here's an example from recent memory : ItemDecorations for RecyclerViews

original version did multiple things?

i imagine there's obviously a performance implication. but you know what. this is faster, and if i see an implication, i'll go back and change it later


# Resources

http://programmingisterrible.com/post/139222674273/write-code-that-is-easy-to-delete-not-easy-to
http://www.codedtested.com/easy-to-delete.html
