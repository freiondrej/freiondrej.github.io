---
title: "Chasing perfect names"
date: 2024-01-27
---

> She was also coming to the conclusion that she ought to learn to read. This reading business seemed to be the key to wizard magic, which was all about words. Wizards seemed to think that names were the same as things, and that if you changed the name, you changed the thing. At least, it seemed to be something like that...
> 
> (_Equal rites_, Terry Pratchett)

When I read this paragraph, it occurred to me programmers actually *are* wizards with such powers. I already touched the topic of naming in {% link _posts/2023-10-30-chasing-perfect-names.md %}, but it does not cease to fascinate me. As I'm trying to understand more about LLMs (today's inescapable topic), I came across a powerful idea which I never fully realized before: Language is our model of the world which we can share with other people. LLMs in particular exploit this heavily - language basically becomes a shared interface between a human and the model. But even when we step away from LLMs and just focus on the premise that language is our model of the world, there are some mesmerizing takeaways.

Our programs are basically our mini worlds modeled by the language (the programming language + the natural language we use to name things in the program). Thus, in them, we indeed are the all-powerful wizards who "change the thing by changing the name". This is extremely useful to understand, because with great power comes great responsibility. The "things we change by the changing the name" are confined to the realms of the world = the program we write. But more often than not, the program is supposed to reflect something from the real world - and our program-world should thus stay aligned with the real world, reducing (or increasing) confusion of other engineers and product managers discussing the evolution of the program.

And herein also lies the biggest risk. When we use a totally different name for something (e.g. X15S for a cat), it's obvious that when we mention X15S, we're referring to the terminology of the program, not of the real world. But as we get closer, this becomes less and less obvious - if we use the term "cat" in the program as a name for any feline, we're reducing the alignment and opening the door to let confusion in.

I don't have any practical advice to wrap up the article with, and I am fully aware that the ideas stated might sound contradictory. But I believe it might be beneficial to keep both views in mind at the same time - we indeed have the powers to change how we shape our program-world, which brings all the more reason to be very careful with those powers and understand that we might easily shape our program-world into something quite different than the real-world problem we're trying to describe. I invite you to rejoice from your powers and to be mindful that we can use them to improve alignment and reduce confusion. 
