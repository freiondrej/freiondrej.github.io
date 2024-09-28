---
title: "Learning in public: Redwood.js pt1"
date: 2024-09-28
---

_Disclaimer: as I only found out at the end of the blogpost, I got stuck in a situation which I didn't figure out in the short time I had for this. So feel free to skip this post for now, I'll update it once I get unstuck and (hopefully) write a Part 2 as well._

_Original text follows:_

Recently, I've ran across a very nice [post](https://www.swyx.io/learn-in-public) about "learning in public", i.e. extending and empowering the learning experience by sharing it, e.g. by writing. I've wanted to look into Redwood.js for a while now, so let's begin.

> I don't have a specific goal for the "part 1", I'll just give it a go and see how far I'll get, or if there's ever going to be a part 2 (or more).

## Intro

When I first got into the node.js world, what struck me as a PHP developer was that the various "frameworks" did not seem to be providing as much out of the box as e.g. my beloved Symfony framework does to a PHP engineer. A lot of patterns, approaches, libraries etc popular in the node.js world seemed quite disparate to me, unlike the rather cohesive experience of Symfony and the likes I had been used to. Then, I came across Redwood, which from a glimpse seemed to me to be promising a much more united experience. The project caught my attention at the time (among others because the guy in the tutorial video had a pipe! how cool is that?!) but I did not look into it further.

Some months ago, I remembered about it and went through an intro of the documentation. I cannot find the exact quote now but the gist I took away was that one of the selling points was how it's an UI plus an API, so the API is then reusable for a mobile app, shall you find out you need one.

Then I visited it yesterday and found out that there's a new "epoch" upcoming (Redwood's versioning would have to be a standalone chapter, but imagine an epoch to be a major-major version) which will be built upon React Server Components, as the GraphQL the API was based upon had a learning curve and Redwood wants to be even more accessible than that. That made me curious - how will that go together in having a reusable API for a mobile app? Is that no longer something it wants to offer?

I have a (super limited) idea of React on the FE, have never tried it on the BE yet, so this seems like a nice opportunity to try out both Redwood as a framework and React Server Components as a technology.

I'm really curious at this point!


## Creating a project

At this point, I'm still not sure what app I'd like to build throughout my learning, so maybe I'll go with the blog project mentioned in Redwood's Quickstart 😁 we'll see.

When reading [the announcement blogpost](https://redwoodjs.com/blog/rsc-now-in-redwoodjs), even though it seems it's from March already, the RCS features within the new epoch (which is called Bighorn btw, nice touch to be able to refer to something else than version numbers) are rather non-prominent. I'd expect it would be much easier to start a project built upon the features of this new epoch, but it seems they're still making it stable ... ? Don't really understand what happened since March neither when the epoch is expected to become stable, the website is not really clear about that. But I dug out some instructions about a canary release: `npx -y create-redwood-app@canary -y ~/rsc_app`, so I'll try that.

I'm installing [nvm](https://github.com/nvm-sh/nvm) first as my node version is apparently too old and we're ready to rock.

Hm, ran into problems already - next step would be to `yarn install`, but the command is complaining that `Couldn't find package "//verdaccio.tobbe.dev/" on the "npm" registry.` Did I do something wrong? 🧐 By any chance, am I using some prehistoric yarn installation maybe? Ah that might indeed be the case, I'm using some 1.x whereas the newest one is 4.x. Let's [upgrade](https://yarnpkg.com/getting-started/install) it. It helped, yay!

Time for enabling the RCS features which even in the canary are not enabled by default. As per the [blogpost](https://redwoodjs.com/blog/rsc-now-in-redwoodjs),
```sh
yarn rw experimental setup-streaming-ssr -f
yarn rw experimental setup-rsc
```
All good. Let's proceed to run and serve. There's some default content on localhost, hooray! As suggested, I'm opening `web/src/Routes.ts`. From the getgo, there's a lot in it that the IDE doesn't like, everything is red. Possibly my IDE is not working with the right node version? Fixed that, fixed tsc also, but still a lot of red. I'm not going to pay attention to it now, might be an IDE thing.

One interesting thing I'm noticing: a comment saying `In this file, all Page components from 'src/pages` are auto-imported.` How does that work? Will the linters be okay with that...? But everything compiles, so probably yes. I won't dive deep into that now.

"**Cambium** is a simple light-table/photo-editing app", says the blogpost. Sure, let's try that. 

"We’re still deciding where to put things like `functions`, `services` and your `prisma.schema` file in the final Bighorn release." Interesting, it seems to me they really decided to go big on RCS and not let those questions stand in the way of it. Or did I misunderstand and it's changing because of other stuff than RCS, too?

Back to Cambium. They're cloning [this repo](https://github.com/cannikin/cambium-rsc.git), but I'll rather view it only and copy the functionality piece by piece into the default app I cloned, to better see how everything works together.

BTW interesting tidbit, I peeked into the commits of that repo and [one of them](https://github.com/cannikin/cambium-rsc/commit/839590532bdccc7f74d19b339d5f5202f8894a18) is "Remove the api-side completely". Is it just because this specific app doesn't really need it, or is the answer to my above question "how will that go together in having a reusable API for a mobile app? Is that no longer something it wants to offer?" a simple "correct, api will no longer be a focus by default"?

Quite a lot of stuff in `pages`. All react components. Some of them indeed contain the directive `'use client'`, which according to the blogpost means they are rendered in the client, not in the server. I wonder how step debugging will look like!

A surprise - after putting everything in place (possibly I missed something?), restarting `serve` and visiting localhost renders the previous components. Caveat - I deleted them in the code. Is this cached by the browser? Aaah, seems there's a build step necessary. I was hoping for a hot reload, but maybe there is one somewhere.

I just noticed that there are three `package.json`s - one in root, another one in `api` (which the `cambium` app deleted) and yet another one in `web`. I wonder how they function together. `yarn install` in the root level does install the dependencies in the other ones, too. Aah, possibly because of the
```
  "workspaces": {
    "packages": [
      "api",
      "web"
    ]
  }
```
config in the root package.json? Might be.

Another tidbit that caught my attention: the march-built Cambium repo depends on package `@apollo/experimental-nextjs-app-support`, while the new one I just created instead depends on @apollo/client-react-streaming`. AFAIK Next.js has been React Server Components-enabled for quite some time, so is Redwood taking inspiration from them ... while Apollo created something more vendor-agnostic since March?

OK, copied all changes (hopefully), build, serve and ... empty page. How can I find out what's (not) happening? No output in the console running `serve`, nothing in the DevTools console... I assume I missed some detail when copying from the Cambium repo. In my experience, situations like this can be a great learning opportunity. Let's see how to proceed from here.

It would be awesome if I could use a step debugger, but I have no idea where to place the breakpoints. Tried placing them in `web/src/App.tsx`, no luck. Is this because there's the build step and possibly source maps are missing? I'd try adding a breakpoint to something in `web/dist`, but TBH I have no idea yet how the built files correspond to the source ones. The browser seems it's importing something with `rwjs_client_entry` in it, but placing a breakpoint there did not seem to do anything. I disabled cache to make sure we're reaching the server (and with it, the debugger), but did not make a difference. Could my IDE be the culprit? 

---

I need to stop this now and be with my family again, so sadly this Part 1 ends without a clear success. But I'm intrigued! Hopefully I'll get to make a Part 2 soon.
