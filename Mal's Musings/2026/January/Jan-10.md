---
layout: default
title: January 10 Musings
nav_order: 7
parent: January 2026 Musings
grand_parent: Mal's 2026 Musings
---

## Musing 7

As I finish a week where I've spent a LOT of time dealing with software performance issues, I'm glad that for once, 
it's not the cleanup of basic issues that were the root cause.

Most of the time when dealing with slow software, it's usually some variation of N+1 querying, a mis-annotated Hibernate
entity, or easily spotted inefficient query. Which . ..  is nice when it's a clear "hey, I can fix this and move on" once 
in a while, but I've more often enjoyed the "hey, here's a problem where we're having trouble hitting our performance requirements
because of XYZ." 

But in a moment of nostalgia, I just want to look at the common problems I've encountered.

- N+1 querying.

Starting with the big one because the pattern is so pervasive and happens in a bunch of ways, even when using native SQL
queries.

What makes this problem more pernicious is that in microservice architectures, you can have N+1 *Service calls*, compounding
this problem further.

Especially at the service level, we should ensure that we can effectively handle returning the results for 1 request or multiple
(ie - if looking up resources by id, make sure you can handle either a single id being passed in or a list of ids).

Probably the most memorable time I encountered this problem was when I was relatively new at a job where we were converting
old green screen applications to spring boot/angular applications. In developing the UI for one such screen, another developer
approached me about why his app was so slow. On digging into it, it turned out that he had setup the UI to request all the 
relevant ids for resources, then *individually* looked up each resource, and was convinced such an approach would be faster
because "each query was faster" . . .. completely forgetting that there was a network latency to each request made.

But I've pretty much had to clear out some form of N+1 querying in every team I've worked with. The less obvious version 
of this is when there are queries to be made on each element of a collection and only writing the query so as to operate 
on one entry at a time, rather than structuring the query to operate on multiple entries at once (or at least batching the 
queries or updates to minimize network latency).


- bad configuration

Early on in my career, I was babysitting a small application that was the mediator between 2 major systems for a health 
insurance company. Every 5 minutes, it would check the external system and start importing it into and internal system to 
start the next step of procesing.

Well, one of the LoB deployments started having 20+ minute JVM pauses for garbage collection. Which . ..  was unusual 
considering how small this app was. And of course, it only happened in our production and prod-preview environment, But 
we naturally had to investigate. We cleaned up a fair number of things, like string concatentation in methods 
(and even worse, these were fairly static queries, so they could've easily been instance variables),  implemented 
come appropriate caching, and because these were EJBs, we even had to manually null out some fields that were potentially
being held onto too long. Deploy those fixes, and we saw improvement to 8 minute JVM pauses in our prod-preview environment.

Great, but no where near good enough. And just as start prepping for round 2, I get a call from the Ops team, asking if they
could properly size the servers. . ..  Somehow this app had been deployed to a machine that only had 2 GB of ram. . . 
on a machine where because of the large files that had to be processed, the JVM alone was setup with 6 GB of ram. . . .
Surprise surprise, when the server was properly sized to (however big it ended up being, I want to say 20GB, but I don't remember)
the garbage collection pauses, even without the improvements made, now are sub-second. We still deployed those improvements
since we also addressed other performance problems. But this still drove home the need to do basic research.

Since then, I've seen multiple occasions when bad configuration caused bad performance. Check your setup people!

- unnecessary queries/querying for static data

Probably my favorite bad pattern to cleanup is that of unnecessary service calls (usually because of duplication) or 
repeated querying of cacheable data.

In addition to improvements in performance removing unnecessary calls is such an easy maintainability win since there's now
less code to support and that code is easier to follow. 

What's always been more interesting to me is how people miss that static data can be cached. Several times I found versions
of N+1 querying that were just "making database calls in a for loop" because the static data needed for each record being
processed might be different.

Beyond this, there's a few other performance things I've hit I should bring up in another musing.