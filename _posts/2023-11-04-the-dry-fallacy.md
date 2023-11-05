---
title: "The DRY fallacy"
date: 2023-11-04
---

One of the very first principles I've heard of when I started with programming was the famous Don't Repeat Yourself principle. From the very first moments, I believed this is a cornerstone of being a real programmer - where some of my programming fellows used copy-paste, I took great pride in doing it "the programmer's way": reuse as much code as I could by trying to never ever write the same code twice.

Quite a few years further down in my career, I casually suggested to my colleague in a code review to remove a minor duplicity, to which my colleague reacted: "I don't think it's necessary - sometimes it's completely fine not to follow DRY". I was shocked. How could anyone question the importance of following DRY? I didn't have any arguments up my sleeve at that time, just accepted the pull request and moved on.

But the idea didn't leave me. I always try doing the best I can, so this shook my confidence; if following DRY is not always the best, am I actually doing the best? And for the first time, I thought a bit more of what is actually so important about DRY.

What I concluded back then was pretty close to what I still think about it today: **When making a change in your code, you shouldn't need to start searching your codebase for similar usecases to also be changed manually.** In the words of Sandi Metz from her wonderful book _Practical Object-Oriented Design_:

> DRY code tolerates change because any change in behavior can be made by changing code in just one place.

This is eye-opening: we don't try to avoid duplication just because; we're doing it to prevent a maintenance nightmare.

But to really make the most of this principle, we must focus on something that can often slip away from the equation when thinking about what DRY means. It can be almost too subtle for junior programmers to grasp: **When two pieces of code look the same, it does not necessarily mean they are duplicated**. In other words, if we're overly fixated on the "looks" of the code and not the real-world business logic the code represents, we might let the code take the center stage, letting it obfuscate the underlying behavior - all of that in good faith that we're actually helping the app become more maintainable.

Another important lesson I learned: Abstraction, most often probably introducing a parent class, is not a tool for code deduplication. This also sounds counterintuitive at first: "But that's exactly what happens when you introduce it, no?" Yes, but it is a byproduct, not the goal. The abstraction in its core is primarily about describing one particular concept, not multiple concepts that look the same on the outside but are in fact different. This is well described by the AHA principle - Avoid Hasty Abstractions. The principle argues that introducing abstraction too early can actually bring more bad than good, because what might seem to be the same concept when we're starting with the project, can later turn out to be two similar-looking but unrelated concepts once we get to understand our problem domain more. Its advice: wait until you're repeating the code for the third time, by then it can already get clear enough if it's really the same thing.

I'll conclude with a bit unrelated tidbit which was also interesting for me in regards to my understanding of abstractions. I introduced a decent abstract class with three child classes and asked my colleague how to best write the test for the resulting code, while benefitting from having the code deduplicated in the parent class. The colleague explained to me that I should write three separate tests, one for each of the child classes, and ignore the fact that some parent class exists. The reason: the abstraction was an implementation detail, the parent class itself did not represent some separate entity in the real world. If in the future I'd need to rewrite those classes using different abstractions, the tests would fall apart even if the behavior they tested stayed the same. So this turned out to be my overzealousness to have everything as DRY as possible.

My takeaway: DRY should be a side effect of writing good code, but never the goal.
