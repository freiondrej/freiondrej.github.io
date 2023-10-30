---
title: "Chasing perfect names"
date: 2023-10-30
---
 
As a perfectionist, I've encountered a lot of obstacles along my programming path. Over time, I learned that perfectionism can often stand in the way of actually delivering great work, but there's one area which I didn't change my view about so far: naming things.

> There are only two hard things in Computer Science: cache invalidation and naming things.
-- Phil Karlton

Human brain performs an awful lot of "connect the dots" operations - probably much more than we realize, let alone admit. This provides an incredible benefit to programmers: if we choose a great name for a variable / method / class, we're rewarded by the ease with which our coworkers will be able to grasp the concepts, relationships and mechanisms described by our code. We're actually able to transplant our thoughts to another person if we succeed. It can be a magical feeling.

Of course, there is a dark side to this: if we don't succeed, we transp. Let me rephrase: not only we can easily fail to transplant the context we wanted to, we can even easily transplant a totally different context which we didn't even think of. Hint: during a call about your code / a PR review, focus on how often you find yourself correcting the other person, or in worse case scenario, you feel you are on the same page, only to find out later that you most definitely were not. Some pieces of meaning got lost along the way, possible in those particular moments where your colleague was giving you an encouraging nod. Not that anyone would do that on purpose - every party simply *assumes* that the other party's *assupmtions* are the same. Ever tried playing the broken telephone game?

I recall an advice that "If you find yourself not sure what to name a class, it might not be clear what the class is actually supposed to be doing". This alone is already a valuable principle, but if we think about it a bit longer, we can sense the underlying big idea: The words actually guide your brain how to think about a problem. This fact can be a big help when we're aware of it and we leverage it - or it can easily become a curse.

I'm only starting to unveil all the principles of using ubiquitous language within a product team, but I suppose the above lends the idea some more credibility: the terms we use to discuss concepts are actually important. Very important. And even minute differences, which a lot of people can dismiss as negligible, can easily anchor the wrong assumptions.

When coming up with a variable / method / class name, I've recently read a piece of advice from John Ousterhout's great book "A Philosophy of Software Design". It goes along the lines of "Don't only focus on what the thing *is*; also think about what it is *not*." And this is actually extremely important - by trying to focus on what it is not, we can make sure our name does not accidentally bring some unintended connotations. If you've ever played [Codenames](https://en.wikipedia.org/wiki/Codenames_(board_game)), it is a bit similar to the assasin card - you must think twice to make sure your clue most definitely does not refer to it, otherwise the game is lost.

If you're like me, you encounter a ton of mini friction points throughout a regular workday. Do yourself and your team a favor and make the names in your project something that actually reduces this friction, not increases it. The next time you find yourself discussing a complex user flow in your product, I wish you can say to yourself "At least I know we're discussing the same thing".
