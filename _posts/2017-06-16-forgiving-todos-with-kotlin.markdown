---
layout: post
title: Forgiving Todos With Kotlin
date: 2017-06-16T19:38:16-07:00
published: false
---


- todos in kotlin blow up and are awesome

- code is the truest representation of the product
- love todos as it documents what needs to be done

- prefer not to have these "crash" for your users
- but for our developers we want to get as aggressive as we can


so my first thoughts were, what if we could blow up on a specific date.
after all, todos should have "due dates" right

why this will work

if you're good about keeping a CI and having automated tests, these will blow up for you and your devs will have to fix it


brownie points - crate a nice builder like function in kotlin that makes it obvious when the todo will blow up


https://kotlinlang.org/api/latest/jvm/stdlib/kotlin/-t-o-d-o.html
throws NotImplementedError stating the reason

bonus points:

* add date extension functions for before/after
* add nice date builder constructor. Kotlin's named parameters make this nice and concise with sensible defaults

