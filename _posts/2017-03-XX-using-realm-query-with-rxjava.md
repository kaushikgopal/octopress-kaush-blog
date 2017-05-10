---
layout: post
title: Using Realm in RxJava Presenter
date: 2017-03-02 23:26
comments: false
published: false
external-url: 
---


https://paper.dropbox.com/doc/Learning-Realm-Moving-from-a-flat-json-file-structure-Database-gmnsgZwWycWg1zYuX0AHL
https://realm.io/docs/java/latest/#rxjava

gotcha 1:

.doOnTerminate(realm::close);

i was doing the try resources but the observable was returning RelamResults or List only within a scoped context. 

Realm is reference counted, which is awesome

RxJava 2 support · Issue #3497 · realm/realm-java
https://github.com/realm/realm-java/issues/3497
https://www.reddit.com/r/androiddev/comments/5obss2/fragmented_podcast_70_an_honest_discussion_about/dcjc8dr/
also see Disqus comment on the same for Fragmented episode
zeyad.gasser@gmail.com on email

“@FragmentedCast In episode 70 @kaushikgopal mentioned that he wrote a DBService that has live updates and closes realms. code example please”

http://fragmentedpodcast.com/episodes/70/#comment-3111466784