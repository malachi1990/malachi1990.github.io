---
layout: default
title: January 12 Musings
nav_order: 8
parent: January 2026 Musings
grand_parent: Mal's 2026 Musings
---

## Musing 8

Continuing on from last  time, some other performance problems I've hit more than once are :

- using '*' in queries that use (multiple) join statements.

It should go without saying why this is dangerous. But too often when you do have to get all (or almost all) columns from
one of the tables, it is tempting to just shortcut it with a wildcard. 

But wrongly placed, or if the filtering is wrong, and suddenly, instead of pulling back a few KB of data, you're now 
pulling back 100 MB or more in a single query.

- large IN clauses

On older versions of Postgres (specifically version 13), I've found that the Aurora version just does *not* perform well at all with  large IN clauses.

Why, I have no idea. But even when you index everything correctly, large IN clauses (and "large" can be as small as 50-100 in 
some cases) can just absolutely kill your database, even when non-Aurora Postgress handles the *exact* same queries and
data (EVEN WHEN HOSTED IN THE SAME AWS REGION) like a champ. 

I'm getting ready to upgrade postgres . . . . so hopefully the new version will handle IN clauses better. . .

- unnecessary stringifying

One of the most common signs a junior developer was a leading developer is the unnecessary stringifying of the objects, 
including in many cases, unnecessarily stringifying numbers or making a single string represent relatively complex 
objects.

While I haven't frequently encountered the latter (though I have seen it a few times when dealing with older fixed width
text based programs), the former becomes annoying when you have to convert the String to some numeric type just to run
a value check (ie equality or greater/less than) to do some action.

To be clear, there are some occasions when you have to use a string to represent a number (ie - you have a leading 0 that 
matters). But the impact of converting a string -> int/long (or wrapper types) adds up really quick. Both in the actual 
computation being done, then in the extra GC that needs to be done in Java based programs to cleanup the temp object.

JUST USE THE CORRECT TYPING WHEN MAKING YOUR OBJECTS.

- unnecessary threading

I get it - multithreading is *cool*, it can make your program faster. And it always looks great on the resume. But in web
based applications (at least in the Spring/Java EE world) you need to be *really* sure that the benefits of multithreading 
outweighs the overhead threading brings. 

I've consistently found that removing multithreading and fixing any of the other problems I've mentioned (including inefficient queries)
to be a FAR higher ROI than using multithreading. 

To be clear, are there places where web applications can benefit from "normal" multithreading processes? Of course. But those
are far fewer in number than you think, especially when you consider the thread per request model that Java app servers
have normally used. 

That being said, with the rise of virtual threads, this point could change, especially as virtual threads are tailored to
help with I/O bound scenarios. 

And while I'll caution against threading unnecessarily, one should know when the best practice *is* to use extra threads.
A good example is how if you're writing a Swing based app, you should have at least 1 thread for rendering the Swing 
components and 1 separate thread for running the business logic. I generally just work with Java based web apps, so my frustration
is tailored to that world (though I've also previously maintained swing apps, and yes, my heart breaks for my brothers still
in that world).

- improper use of java lambdas

Java lambdas are awesome. I love them, they make my life a lot easier. But when you need the fastest performance possible, 
they often get in the way.

While simply calling .parallelStream() is nice for helping my hotspots, it also results in the best cases being a slight bit slower
than simply using for loops. 

In my first two musings, I wrote on a project I recently implemented at work where I rewrote most of the role based authorization
stack we were using, resulting in response times going from 16-20ms average to 2-3 ms average response times. Just a massive
improvement. 

One of the things that made it fairly easy to rewrite was wide use of lambdas. Which is great. But because this is performance
critical work, the next step I've been taking is to remove those lambdas in the performance critical paths. 

Some back of napkin math *already* shows a ~10%-20% improvement in response time (ie going from 2ms to 1.8 ms) with no other changes.
It sounds small, but what makes this almost magical is that the JVM is better able to use the JIT over time to *further*
optimize over time. Even during testing of a decently sized job (throughput of roughly 1k requests/second), you could see how
every few minutes, the response times of the authorization service improved. In contrast, during the same scenario testing
with the lambda implementation . . . and the performance just remained flat. It had less variability than the for loop implementation, 
but it also didn't seem to benefit from the JIT optimizing things.

## conclusion

These last 2 musings cover a few places where I've personally encountered the same performance problems over and over.
There's a few other performance problems that were once offs  that would be fun to talk about (for example, where a client's VPN gave dialup levels of performance, 
causing testing to almost always bottleneck on pulling back data), but that's for another time.

These things I've mentioned the last couple days *primarily* apply to the world of Java web apps. But I'm sure there are 
corollary things (especially since many languages have to have some database interactivity) in other languages.

