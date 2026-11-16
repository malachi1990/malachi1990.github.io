---
layout: default
title: November 15 Musings
nav_order: 260
parent: November 2026 Musings
grand_parent: Mal's 2026 Musings
---

## Musing 260

As my Bible study tool hits feature complete status, I just wanted to jot down some thoughts about
vibe coding something I've literally been wanting for 15 years. 

1) It's *really* nice to have a tool I thought I'd never have.

Because of life and how it takes time to do stuff, I legit thought that I'd just have to use the freely 
available tools. Which, to be clear, I *love* Xiphos and BibleTime (not to mention that the little exposure I've had to Logos left be blown away). But I know that my way of organizing things is unique 
and I never really clicked with the workflow (though I loved other parts of the workflow).

Even just at a base level, it's a desire fulfilled that's brought a certain lightheartedness since I got 
it done. Proverbs 13:12 alone tells us the truth of this.

2) it's a terrifying testimony to what AI can do.

Setting aside the legal/ethical questions of developing this tool, it's genuinely impressive that AI can
fairly quickly (I've been using my AI credits for the last ~6 weeks on this) generate this level of feature work in a relatively short period of time. 

Is the code perfect? No. Does the functionality work as intended? It's about 95% there. 

3) It's also a witness to AI's limitations.

This is where there is a difference between the layman and a professional comes out. It's *obvious* this project is essentially in a feature complete alpha state if evaluated on a professional level. In large measure, it's the little things that get it. Places where assumptions are made where it shouldn't be. Subtle errors in the import functionality that only show up when you get out of the "happy path" scenarios of the simplest entries. 

While AI can probably be used to fix these issues, a person has to catch them, and then be able to know 
how to express it to the AI. For something like accessibility, a person may not have the knowledge needed to guide the AI into doing accessibility work correctly. Same with security. 

While I chose kotlin/kmp because I could ultimately support the software without much issue, I can see 
others choosing a trendy language that doesn't have good support for the relevant edge cases and getting
into a rough situation because of it. 

At least as of right now, AI doesn't "automagically" do a lot of the boring background stuff not immediately relevant to your prompt. Things like accessibility and security are ignored, because in
the vast majority of cases, it will be ok to. But for someone who doesn't have the knowledge or experience to take care of that explicitly . . . well, that's why there are headlines about AI slop.
