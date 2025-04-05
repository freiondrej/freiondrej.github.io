---
title: "My serverless Remix adventure - part II"
date: 2025-04-05
---

Hello, dear reader! As this is a second post on the Remix topic, please read [the previous part first]({% link _posts/2025-02-15-remix-adventure-part-1.md %}) to have full context.

In today's episode, I want to talk more about [Gel](https://www.geldata.com) ([formerly EdgeDB](https://www.geldata.com/blog/edgedb-is-now-gel-and-postgres-is-the-future)). In the last post, I mentioned how Gel felt as a DB for the 21st century. There were several aspects to it:

Instead of thinking in tables as I've always had, Gel has [object types](https://docs.geldata.com/learn/schema#object-types). Those are very helpful for an OOP-bred programmer, as they feel much more natural. But they also support inheritance! I know I know, today's common view is that inheritance is evil and composition is the right way, but reading Sandi Metz's Practical object-oriented design helped me understand better where each of those approaches shine. And in the blogpost project, I had a perfect usage for inheritance! The blog contains either Posts or Links. In the previous Mysql schema, I addressed this with nullable fields and then clumsy `if` statements in the code. Not anymore! I created an abstract type Entry with all the common properties and made Posts and Links [extend](https://docs.geldata.com/reference/datamodel/objects#ref-datamodel-objects-inheritance) it. It worked just as expected, really nice!

Another useful tidbit is type safety - when using any create / update queries, the params need to be explicitly [typed](https://docs.geldata.com/learn/schema#scalar-types), e.g. to a `<uuid>`. It's a bit verbose but the additional safety is definitely worth it.

A similarly nice surprise was querying a `single` result by a non-unique column. For example, I have a table with only one result which I simply want to return - possibly there would be nicer ways to represent this conceptually, but that's another topic. As far as I remember, all the databases I've previously worked with allow a simple select in such case - and only would throw a runtime error that the result is not unique once a new row would get inserted into that table. Not so Gel! Even though I had only one row in the table (or rather in proper terms, another Object), once I tried calling `querySingle` on the client library, I was presented with an error that the selection has a potential non-unique cardinality. So I added a very plain `limit 1` and all was well! No error at some point in the future to fear. Neat.

Also, while nested queries have always felt as something I could have solved better by a harder-to-read-but-correct join statement, not so in Gel. As far as I know, Gel does not have the concept of joins at all. An example from Gel's [documentation](https://docs.geldata.com/learn/edgeql#select-objects) how easy it is to select a related ("nested") object:

```
select Movie {
  id,
  title,
  actors: {
    name
  }
};
```

I had another usecase where I wanted the related results to be sorted. This is the way to do it in Gel (maybe there's even more idiomatic approach to this, but it worked for me):

```
select Author {
   entries := (
      select Entry { * } filter .author = Author order by .listRank
   )
}
```

Pretty readable IMO!

Gel boasts right on its homepage that it solves the N+1 problem. I haven't tried any more complex stuff like paginating a super high amount of results, but I'm definitely intrigued. Allegedly, Gel is a SQL's take on NoSQL databases like Mongo by mixing the best of both worlds. I'm not very experienced with Mongo so I cannot really tell, but I think there is some truth to this.

Out of the box, Gel also provides [natural language search](https://docs.geldata.com/reference/stdlib/fts) 🤯 Sadly, Czech is currently not supported so I wasn't able to try this out. Maybe at some point in the future.

One thing I didn't like as much were migrations. Sadly, I didn't note down what didn't suit me in particular, possibly it was the inability to rollback (in a prod DB, this makes a lot of sense as only forward migrations might be possible, but in a dev env this would be really good).

But there are some very good DX enablers, like [branching](https://docs.geldata.com/learn/branches)! Since my project is super small, I didn't have to use it, but such feature might come in very handy.

Another one I didn't even try but looks very interesting is the [auth](https://docs.geldata.com/reference/auth#enabling-authentication-providers). Some providers particularly stand out - Magic Link and WebAuthn (passkeys) for example. If that means the developer effectively does not need to implement those in their app and can get up and running with those built-in ones, that could be a signficant speed up for certain projects.

As for alternatives, I came across [Nile](https://www.thenile.dev), [Turso](https://turso.tech), [TiDB](https://www.pingcap.com) and [Supabase](https://supabase.com). Each of them had some interesting aspects! But Gel stood out the most to me.

Alright, this post turned out to be pretty fragmentary, but I hope it was interesting to you nevertheless.

I'm not quite sure if there will be a Part III blog post or I'll explore another topic, please stay tuned!
