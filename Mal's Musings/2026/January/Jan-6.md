---
layout: default
title: January 6 Musings
nav_order: 3
parent: January 2026 Musings
grand_parent: Mal's 2026 Musings
---

## Musing 3 - why are LLMs so good and bad?

There's a lot to say about LLMs, from their benefits, to the acceleration of brainrot they've brought on.
And working as a software engineer, I see so much on both sides of the spectrum.

On the one hand, it can crank out test code faster than you think, and the test code is pretty good. On the 
other, I've seen even the most recent models completely collapse on basic tasks the moment your codebase has
a business domain deeper than a kiddie pool.

So why is there such a range? While many learned people can give more technical responses, I want to look at this
from the other angle and talk about what's probably an uncomfortable truth - most software engineering isn't 
that deep.

A lot of business domains ultimately boil down to some form of "take in data, store it, transform it to a more usable 
format, then return to user". And when writing these apps, most people are not breaking new ground. 

To be clear, there's nothing wrong with that. Often times, people want "some already existing app, but it does X", 
whether that X is technical (like the "rewrite this tool in Rust" craze that is currently ongoing) or 
some other real world constraint that perhaps justify rolling your own tool.  

But what's the end result - so often these tools get added to Github or some other code hosting service . .. 
and fed into AI. Which now has another example to build yet another iteration of "existing app but X". 

At least personally, this feels like a much better explanation of the range of results I've seen in AI usage
both from myself and others in the field. When I'm working on software with terminology and multiple layers
of business specific context? The AI chokes fast like it's speedrunning how hard to screw you over, with the 
'newer' models getting even better at that speedrun.

But feed in a part of that codebase where the terminology is industry standard and not too difficult to 
follow with minimal business specific rules? Well, it'll get you 80% of the way there, and you'll spend the remaining 
80% of your time fixing the edge cases the LLMs had no chance at finding.  

While I suspect that LLMs have hit their peak ability (and the last few releases leaving me feeling that's the 
case, but more because LLMs are expensive to run and corporations need $), we should expect them to stay. While
not perfect by any stretch of the imagination, they are useful tools, if you're willing to maintain vigilance
and not just blindly accept the output of LLMs. 

But the acceleration of brainrot in society is a topic for another day. . . 