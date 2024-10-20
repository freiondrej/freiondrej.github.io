---
title: "Learning in public: Redwood.js pt2"
date: 2024-10-20
---

# Preface

As documented in [my last post]({% link _posts/2024-09-28-learning-redwood-js-part-1.md %}) (which I don't really suggest you to read, as today's should hopefully be much better), I attempted the [learning in public approach](https://www.swyx.io/learn-in-public) for the first time. Since then, I reflected on my experience and realized that I got sidetracked quite a bit: while my original intention with RedwoodJS was to evaluate its capabilities as a framework offering many things out-of-the-box, I digressed towards the RSC concept instead. So today, I don't mean to follow up on my last attempt, but instead start anew fully focused on the original goal. This should hopefully also mean the post would bring more value to you, the reader. Thank you for your patience!

# Intro

When evaluating a framework, the "classic" things that come to my mind are e.g.: routing, request/response controllers and validating their input, authentication, hooks / middlewares, modularity (or building multiple services within a [SoA](https://cs.wikipedia.org/wiki/Service_Oriented_Architecture)), event emitters + listeners (or message publishers + consumers), transforming data for api output, cron, background jobs, websockets. How the framework encourages / enforces consistency and structure. Built-in testability - test beds, fixture generators, methods for asserting controllers, whether the framework encourages test cohesiveness and maintainability (which can be key to rapid iteration).

This might sound a bit like a buzzword bingo, but if a framework offers clear guidelines for implementing these "application aspects" or even ready-made prebuilt functionality we can leverage, to me that is a huge deal. So, I'd like to assess at least some subset of those as part of my evaluation / playing around.

# Goal

What was less than ideal within my first post, was that it lacked a clear goal. Having one could help me build something "real-ish", instead of just fragmented experimentation, disconnected with how I would use it at work.

So let's build a web app! Or, more specifically, a small part of it, as I don't have much time to write and I'd love to have something functional, albeit super limited.

A project I wanted to create some time ago but never got to it was a mini audiobook remixing app. When listening to some audiobooks, I sometimes think of silly ways I could rearrange the sentences in order to get some funny outcomes, while still keeping the original recognizable. Silly enough, right? 😁

So today, I'd love to implement a super-simple mp3 file upload, which will then create a background task to communicate with an audio-parsing SaaS like [Deepgram](https://deepgram.com), and then notifying the end user about the progress via websockets. For now, we won't save the result of the API. Also, maybe we could leverage some TDD when writing the parts of the solution.

# Implementation plan

Normally, I would draw a diagram at this point, but I'll try without it for now and start by noting some particular details which are clear enough. **Also, I think I won't use the RSC alpha of Redwood, because the GraphQL approach is something I'm more familiar with as a backend engineer.** Moreover, I'll be able to use the documentation, which the RSC version lacks at the moment, AFAIK.

## Building blocks

- the controller
   - it should handle a mp3 file, presumably as a multipart/form-data body, or maybe as pure binary data would be enough too.
   - upon a non-mp3 file, it should return HTTP 400
   - upon success, it should return HTTP 202 with an id of the background job it would spawn for the uploaded file (this will be abstracted through some application service)
- the service
   - will offer a method which will accept some Buffer or plain string with the raw file data
     - this method will spawn a background job for the actual file upload, and return some identifier of this job to the caller (here, the controller)
- the job
   - will perform the upload to Deepgram (possibly through a service call, as I see the job as more or less an equivalent to a controller, not to a service)
   - it would be great to have some kind of abstraction for Deepgram to not be coupled to it too much, but we'll see if I have time for that
   - it's possible that Deepgram's API will also return some kind of 202, which then we'd need to spawn another job to fetch, or respond to Deepgram's webhook which it might call when it's done processing. Or, maybe their API will be so fast that the response will already contain the parsed data, let's see. This is an unknown at this point.
      - anyway, after we get the result one way another, I think we can emit an event about that
- some listener reacting to the "done" event from the job - at this point, the listener will just emit a websocket event / message that would show up in the user's browser

That should be about it for now. All the listed building blocks should be nicely testable, so this might be a great opportunity for a TDD approach (by which we'll also evaluate how the testing in Redwood is, out of the box).

However, I have like 15 mins remaining of my time for this blogpost, so I might need to continue some time in the evening 😅

# Scaffolding

I'll follow https://docs.redwoodjs.com/docs/quick-start. `yarn create redwood-app audiobook-remix-app` it is! It prompted me for JS vs TS, whether to initialize git and we're ready. It prompts for a `yarn install`, `yarn rw dev` and ... a welcome page is shown in the browser! We're halfway there ... err almost. The default page suggests to run `yarn redwood generate page my-page`. I'll try it, hopefully it will give me some idea of how to build the controller.

OK so this created a page in the `web` directory. It will be the user-facing one, while what I'm interested in is in the `api` directory. Let's explore how to work with that one! About that, Redwood itself is not giving me any concrete clues, it only did about the "frontend" ones. I guess the Redwood CLI will contain some answers. Maybe `rw generate service` could be it? Not sure if they mean a service class or a whole service module. Let's call it `upload`, that would work for either.

`"upload" model not found, check if it exists in "./api/db/schema.prisma"`

Okay, so this needs to work with the DB it seems. For the plan I outlined above, we don't need DB yet. Let's get back to the docu then. The [Reference](https://docs.redwoodjs.com/docs/index) is neatly structured, I would expect to find my answer under [Services](https://docs.redwoodjs.com/docs/services).

An interesting tidbit there: [Service Validations](https://docs.redwoodjs.com/docs/services#service-validations). This seems to be something akin to zod, albeit less declarative and more procedural.

A lot of what I'm seeing is quite DB-oriented. Does that mean I won't be able to use all the ready-made goodies when I don't need DB in my today's usecase? 🤔

I'm getting stuck a bit. To avoid that, I'll move on to the background job, hopefully that will unblock me.

# Background job

My experience with BullMQ is that jobs are stored in Redis. Surprise though - Redwood stores them in the DB by default! (There are some adapters, so I suppose Redis is an option, too. But let's try the default settings.) 

I'll follow [In-depth start](https://docs.redwoodjs.com/docs/background-jobs#in-depth-start). Basic setup done, let's create our actual job: `yarn rw g job UploadAudioFileForParsing`.

```ts
import { jobs } from 'src/lib/jobs'

export const UploadAudioFileForParsingJob = jobs.createJob({
  queue: 'default',
  perform: async () => {
    jobs.logger.info('UploadAudioFileForParsingJob is performing...')
  },
})
```

Uuu, that's pretty elegant! I was wondering how they will handle the "handler" / "performer" / "processor", and this looks pretty cool. The only little downside I see that I'd probably not call the exported const a "*Job", but rather some "*JobFactory". ... I take that back, I read about the usage in the docs: `await later(SendWelcomeEmailJob, [user.id])`. So there's actually really just one instance of the Job, it's not that the job is instantiating anything. All good, then.

> Within Bull, my impression was that the "job" is just the data while we define the behavior on a "worker". Here, the worker logic seems to be hidden away from us for the most part. I suppose the Redwood way is more straightforward for building a mental model of. But also Bull overall will probably offer more advanced capabilities, like flows of nested jobs.
> I might be completely off as I don't have hands-on experience with serverless, but maybe Bull's variant is not serverless-ready while Redwood's is, since the jobs seem to be implicitly relying on state less? Take this with a boulder of salt though. 

Let's now explore the generated test.

```ts
describe('UploadAudioFileForParsingJob', () => {
  it('should not throw any errors', async () => {
    await expect(UploadAudioFileForParsingJob.perform()).resolves.not.toThrow()
  })
})
```

Wow nice! So for exercising the functionality, we don't seem to need to actually create anything in the DB, consume it or nothing like that - we're testing just the actual `perform()` function itself. Me likey! So far, I've seen bull jobs abstracted by some class which all processors extend from, but this seems like much less code so far, and with a much leaner test. The potential downside of testing like this is that we don't really exercise that the whole app itself is able to run the job properly, but I assume that's the part of the "contract" with Redwood here - that it will do its job and we can sorta blindly trust it with that. Another benefit of being able to exercise only the actual `perform` function is that the test is very fast, since no actual app needs to start.

Which makes me think ... Oh wow, there doesn't actually *seem* to be any "app"! How does the stuff in the `api` folder actually run, then? Without verifying it, I was expecting something like a fastify app / express app being started somewhere explicitly, but it seems all of this is abstracted away from us ... ?? This is something I cannot really wrap my head around at the moment, I'll keep it in mind and hope I'll be able to clarify as I explore around.

OK so the docs mention that `rw studio`, the amazing observability utility I bumped into at the end of my last post, does not support jobs yet... meh. But on the other hand, with the prisma adapter it should be fairly easy to observe what's happening, so it's not *as* bad. But... `deleteSuccessfulJobs` is only a boolean, doesn't accept a number of jobs to retain ... sob.

Also, my implementation plan above mentions storing the raw audio data inside the job arguments. Which does not sound like such a great idea now as I see the DB adapter for jobs... So, I might want to update this a bit: the controller will create an AudioFile record in the DB, containing ~~the data as "blob"~~ Prisma [discourages that](https://github.com/prisma/prisma/discussions/21689), but I don't like the idea of storing the raw file data in some Blob Storage like AWS S3 upon upload, just to download it immediately within the Job and use it to pass to Deepgram. I'll need to think what options I have here.

To do that, I think I need to see how can I upload the file in the first place. So let's switch to the audio upload again.

# Form

`yarn rw g form AudioFile` - gotcha, `form` generator doesn't exist. Aaah, according to https://docs.redwoodjs.com/docs/forms, I need to write it pretty much manually, using `@redwoodjs/forms`. 

Adding a `<FileField name="file"/>` does render a file upload, but to understand how I should upload the file to the `api` side, I'll need to make a web search. Aaaah okay! https://docs.redwoodjs.com/docs/how-to/file-uploads/ "As you've probably heard, Redwood thinks the future is serverless. This concept introduces some interesting problems you might not have had to worry about in the past."

OK so this will be much more elaborate than I thought. Meh, but also ... at least it will be an interesting learning opportunity!

# Sadly, wrap up

Although I already feel some benefits of the "learning in public" approach, such as much more focus and structured thinking, the downside is ... the writing takes up a huge portion of time! Which means that much sooner than I hoped for initially, my time today is up. **I didn't reach any of my goals from the implementation plan yet.** In retrospect, I assumed that too many things will work the same way I am used to, just with a lot of stuff done by the framework, so essentially I would write the code the same way, but less of it. And I could reuse the same mental models, e.g. from bull and Fastify. Which turned out to not be the case, and the file upload will be much different than I assumed (which was only a simple POST endpoint, hehe).

But this is very exciting! See you next time, hopefully with a part 3 we'll have some actual code running 🤞🏻
