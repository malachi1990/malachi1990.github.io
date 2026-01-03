---
layout: default
title: January 3, 2026 Musings
nav_order: 1
parent: January Musings
grand_parent: 2026
---

## Musing 1 (out of hopefully 300)

In 2025, I got to work on a project (for my day job) where I needed to essentially rewrite our entire 
authentication/authorization logic. Truly, a task that should strike fear into the heart of any developer
of a certain skill level (or higher). 

So, why do this?

### The situation

The application I work on is a ~~distributed monolith~~ collection of microservices that service a data 
warehouse in the healthcare space. 

And because of the nature of this data (ie, lots of PHI/PII), security is important. To access any data
you have to have an active login *and* have the appropriate role to see that data.

And since these are microservices, that means our authN/authZ mechanism gets called a LOT. To the order of 
150-200 million times a month. Each page requires on average 10 API calls to load all the data. And how 
slow, or fast, do you think the authN/authZ call was? 

On average, about 20ms, with 99% latency taking upward of 500ms (with rare spikes of >1 second!). Median call 
was 16 ms. 

But even at the median case, when you have 16 ms per call of overhead, that adds up quick, leading to a 
perception of being "slow". But more importantly, we had users on rare occasions being logged out for no 
discernible reason. They would just navigate to a page (and of course, it was different each time) 

What made these times baffling was that we were already using Redis for caching, which should be giving us
sub-1ms times. So why was there so much overhead? Unfortunately, the team that wrote this code did so under 
several constraints, which ended up making the code near unmaintainable (Redis interactivity spread across 
multiple classes, multiple similarly named methods with functionality that was mostly duplicated, overloaded methods
names used for different parts of the user token storage or secondary index storage, etc).

### Analysis of the problem

Our system is a collection of spring boot based microservices on AWS. Everything is in the same region.

Each microservice is clustered with multiple instances, and in theory we're mostly good on resources
(even though, according to AWS's Compute Optimizer we're underprovisioned, even under spike load, we never
exceed 70% capacity), and our authorization service has one of the larger cluster sizes.

On digging into the code, there were several issues discovered :

1) no connection pooling
2) Inefficient Redis usage
3) Coding error causing extra writes.


### Connection pooling not used.

On point number 1, rather than using a Spring managed Redis Configuration, the Redis connection was manually
managed, and the connection pool size was manually set to 1. 

Now on it's own, that's not necessarily a problem. Even under spike load, just a couple dozen connections
shouldn't collapse under that load, right? 

You'd probably be right, except . . .

### Inefficient Redis usage

In Redis we were using a Set to store our tokens so that we could use the data structure for efficient lookups.
There's just 1 drawback to this -- the version of Redis we were using did not support per element TTL in 
collections. So we had to manually support checking if an element had expired and making a new call to expire the 
old token and insert the new one.

Likely because of the time constraints the team found itself under, this ended up resulting in some very 
paranoid element checking of the system. Which meant that instead of a direct "get element by key" call and 
checking if the return value was null, there was a multi-step check that made calls to Redis on at least 3 out of 
4 steps, with each invoking at minimum the network latency of about 1 ms. 

But of course, this also compounds the problem in section 1, where there's only 1 connection to Redis per 
instance, so you have to wait on the connection *multiple* times, unless you're in a low traffic time.

In all, the inefficient Redis usage patterns meant that we had *millions* of calls to Redis daily support
mere thousands of calls per day. 

### Coding error

Ultimately, what drove this effort wasn't the performance concerns, it was the fact that we had users 
getting logged out seemingly randomly.

As I dug through the sprawling code it became clear that the users being logged out happened because on a 
couple different scenarios, instead of updating the secondary indexes and derived constructs, we overwrote the
user token instead. 

Which resulted in paranoia that the correct object type was being returned (and if it ever wasn't, it'd correctly
require refreshing the user's tokens to reset things).

And while refreshing the user's tokens usually worked . ..  Some of those code paths saw all the secondary indexes
get overwritten, so the user's session became completely lost, forcing the user to be logged out and be required
to log back in.

### Now what?

So how did i go from here, with all these problems, to an 8-10x improvement on the response time for the 
authorization service? Find out on my next published musing! (hopefully Monday, January 5, 2026)