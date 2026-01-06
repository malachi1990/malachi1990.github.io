---
layout: default
title: January 5 Musings
nav_order: 2
parent: January 2026 Musings
grand_parent: Mal's 2026 Musings
---

## Musing 2

So, continuing from musing # 1, today i want to focus on how I fixed the problems plaguing the system's 
authorization service performance.

To recap, I'm working on a system that was slow and would occasionally kick you out for no reason.

On digging into the problem, there were 3 primary contributors :

1) Connection pooling not being used.
2) Inefficient Redis access patterns
3) Coding errors

Points 1 and 3 are pretty straightforward. When you're forcing millions of connections to go through 1
connection per instance in a cluster, you're going to get contention, no matter how fast Redis is. And 
if you wrongly overwrite the user's session token stored in redis (effectively logging them out), it's no 
wonder people got kicked out.

one correction i want to call out (there's a few others I probably should make, but that would get 
into revealing technical details I'm not sure I can) --

1) I said we were using a Redis set for the collection, it was actually a redis hash. 

### The task

While the primary business goal was to fix the issue about users being randomly logged out, this 
code was . . . not well liked for a lot of reasons. The code was sprawled out across about a dozen 
classes, 4 of them either bordering on God classes or actually being God classes. 

As mentioned last time, it was at least once a year that I and at least 1-2 other senior engineers (or
even our architect) who'd have to go into a war room to understand edge cases and fix them with increasingly
convoluted patches.

So, my task priority was 

1) fix as many of the longstanding bugs in our authentication/authorization logic as I could (while the random logouts were the biggest issue, there were a few other lingering issues to fix)
2) make the code maintainable for as many devs as possible.
3) improve the system performance so the authorization service wouldn't be as much as a bottleneck.

And do this while 

1) maintaining backwards compatibility as much as possible.
2) not being able to upgrade the underlying redis instance.

### Where to begin?

In the initial profiling of the system, I quickly noticed how many hash commands were being run in our day to day operations. 
I mentioned last time that we were running millions of commands per day despite only reaching into the millions 
of API calls (and thus triggerings of our authorization service) over the course of months.

One of the biggest places we could improve this was simply asking "well, can Redis manage the TTL for me and 
automatically expire cached elements?" .  . . . And yes, it can. If you use String objects, but not collections
like sets or hashes (note that Redis has since implemented this in a more recent version!).

Since upgrading Redis wasn't permissible at the time (since that would allow me to use the per-entry TTL on redis collections),
it left me with needing to move away from using a redis hash as the storage object for the user's session 
tokens. And because of the structure of the code, that meant a full rewrite of the storage/retrieval process. 

Which I was not opposed to, as it would allow me to simplify our code by converting that code to 
use spring-data-redis as a wrapper around our redis interactivity. 

And so, I began the rewrite. Immediately, the biggest issue was "how to maintain backwards compatibility" as 
the original user token object exposed TTL information to the UI (which in turn used that TTL to calculate 
session refresh calls). Tldr - if you mark a field as containing the TTL, redis autoincrements this for you.
As long as you track the time units correctly, you can turn that TTL into the actual time that is read to see if 
the token is about to expire.

### First steps

First, the obvious steps were to update the connection configuration to use Spring's injected configuration
to manage the Redis connection and setup the connection pool and RedisTemplate instances.

Second, I created and initial implementation that relied on the Redis Repository pattern. This allowed me to
fairly quickly write an implementation that handled storing the user's session and refresh tokens, the related
secondary indexes and other derived objects.

Initial coding went quick, and while I tested locally, I did not it felt a bit slow, but at the time, attributed
it to the network latency I had with the data center where the application was running.

### First round of testing

As is often the case, local testing was nothing like what happened in our dev environment. What seemed
to be just latency for me was me finding out that the Spring Redis Repositories truly aren't fast. My attempt
at speeding up the app had instead, amplified system slowness a fair bit.

Digging around online, it turned out that even the Spring team recommended using RedisTemplate directly 
for high performance scenarios. Something I had missed prior to this.

Thankfully, the rewrite to use RedisTemplate didn't take too much effort.

### Second round of testing

For the most part, the second round of testing didn't go bad. Aside from a few weird edge cases (such as 
users with only specific user roles getting stuck in a login/logout loop), there weren't any big surprises 
in terms of the perceived speedup. 

### production time!

After lots and LOTS of testing (after all, if you're session tokens don't work, ain't nothing gonna work!), 
we finally go to prod. 

And . . . it's smooth as silk. We go to prod during downtime window after hours, and everything just works.

After months of work and testing, we went from 3k lines of code to manage user session tokens to 1.5k lines. 
Instead of 12 classes (across 4 God classes that handled various pieces of the management logic), we now had 3 
classes that stored the core logic. 

And now, instead of running nearly 100 million calls to Redis per day, we now ran about 5 million, meaning we 
were running on average 20 Redis commands per API call, replaced with just 1-2 calls to Redis per API call.

Other notable results :

1) our authorization service now ran in an average of 2 ms per call, with a p95 of 18 ms, down from an average of 16 ms with a p95 of 70ms.
2) simple endpoints saw a 30-50% decrease in time per call (in one example, we went from 40 ms-> 20 ms per call)
3) overall system performance on page navigation had significant perceived speedup. With an average of 10 calls per page load, that's at least 100ms gained back (often more), and was more than enough to go from feeling sluggish to feeling like pages loaded near instantly (though I'm clearly biased here).


### Further improvements

There's a few more improvements I'd like to do. There were some objects heavily stringified that shouldn't have been that I 
didn't get to fix due to time constraints, and between the constant type conversions between String and Long, and the equals
checks, there's probably at least another half millisecond of time to save.

There's also probably more optimizations to be found in better query tuning and replacing Java lambda/streams with for loops. 

### Conclusions

At some point, I'd like to do a proper writeup on this effort, rather than keeping things relatively vague like I have in this
musing, but that'll be up to those who can tell me how much I can share.

But still, there's a few key lessons here :

1) letting Redis manage TTL probably saves you a lot of overhead
2) USE CONNECTION POOLS
3) Don't use Redis Repositories (indeed, the more I've read about them, the more I think the Spring team needs to just re-engineer them in their entirety)/
   4) RedisTemplate is very easy to learn. IMO - the extra bit of convenience of Redis Repositories just isn't worth it. 

And that's all for now.