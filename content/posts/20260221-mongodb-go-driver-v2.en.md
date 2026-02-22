---
title: "MongoDB Go Driver v2: A Migration That Didn't Have to Hurt This Much"
slug: mongodb-go-driver-v2
date: 2026-02-21T12:50:07-06:00
draft: false
tags: ["geek stuff"]
---

![](/images/posts/mongodb-go-driver-v2.png)

I recently migrated several Go projects from the MongoDB Go driver v1 to v2. The v1.x line was formally deprecated in early 2026 (though critical security patches are still being released), so there was no choice but to move. What I found on the other side was a migration far more painful than it needs to be, a handful of genuinely baffling design decisions, and a major version bump that somehow managed to miss the biggest opportunity Go has offered library authors in the last decade.

<!--more-->

This isn't a migration guide. MongoDB has [one of those](https://github.com/mongodb/mongo-go-driver/blob/master/docs/migration-2.0.md), and there's a tool (`marwan-at-work/mod`) that handles the import path change from `go.mongodb.org/mongo-driver` to `go.mongodb.org/mongo-driver/v2`. This is about the decisions that made the migration unnecessarily painful, and the opportunities that were left on the table.

## A Brief History of Go and MongoDB: It's the Second Forced Migration

Some context on what came before. The Go community didn't always have an official MongoDB driver. For years, it had something better: a community-built one.

[mgo](https://github.com/go-mgo/mgo) was created by Gustavo Niemeyer in late 2010 and publicly announced in March 2011. It wasn't perfect (its own quirks and rough edges), but it worked, the API was reasonably straightforward, and the entire Go+MongoDB community adopted it. For seven years it was *the* way to use MongoDB from Go. MongoDB Inc. itself used mgo internally, their own `mongo-tools` were built on top of it, and they even sponsored the project starting in 2011.

Then in early 2018, Gustavo stopped maintaining mgo. He'd moved on from MongoDB for his own projects, and the unpaid maintenance burden wasn't sustainable. A community fork ([globalsign/mgo](https://github.com/globalsign/mgo)) kept it alive for a while, but it too eventually went dormant.

MongoDB's response? They'd actually started an official Go driver in early 2018, releasing the first alpha in February. It went through a long beta period and finally reached [v1.0 GA in March 2019](https://github.com/mongodb/mongo-go-driver/releases/tag/v1.0.0) — a full year after mgo's author stepped away, and eight years after mgo was created.

Here's the part that stings: the official driver's API was *similar enough* to mgo to feel familiar, but *different enough* to require rewriting everything. Not a drop-in replacement, not a fork-and-extend, but a ground-up reimplementation with its own conventions. The Go community had to rewrite all their MongoDB code once to migrate from mgo to the official v1 driver. Now, with v2, they're being asked to do it again.

Three drivers, two forced migrations, zero backward compatibility.

## The Migration Tax

The sheer volume of breaking changes is impressive — and not in a good way. Entire packages were deleted: `primitive`, `description`, `gridfs`, `bsonrw`, `bsonoptions`. Types were renamed, method signatures changed, error types removed, option builders reworked, event constants renamed. The `Client.Connect()` method disappeared. `SessionContext` was replaced with plain `context.Context`. `Collection.Clone()` no longer returns an error. `Distinct()` returns a different type. `InsertMany()` takes a different parameter.

For a small project, this is annoying but manageable. For a large codebase with hundreds of MongoDB touchpoints — and especially for shared libraries that other services depend on — this is a multi-day, all-or-nothing change. There's no compatibility layer, no v1 shim that bridges to v2 internals, no incremental migration path. Everything must change atomically. Even small details like `mongo.Connect()` no longer accepting a context (you pass it to `Ping` instead) require touching every connection setup in your codebase.

## The Primitive Package: Created Just to Be Killed

In v1, MongoDB created `bson/primitive` as a separate package containing `ObjectID`, `Timestamp`, `DateTime`, and other fundamental types. Every Go project using MongoDB imported this package everywhere: models, handlers, services, utility functions. It was all over any non-trivial codebase.

In v2, they merged it all back into `bson`. A massive find-and-replace across every file that touches a MongoDB type. The question that keeps coming up is obvious: why was it a separate package in the first place? If the answer is "it was a mistake," that's fine — everyone makes design mistakes. But it's a mistake that every user now pays for, and there's no acknowledgment of that cost.

## How a 25% Regression Shipped as GA

v2.0 shipped as GA in January 2025, ready for production, with approximately 20-25% slower BSON unmarshaling and significantly higher memory allocations (up to 3-4x in benchmarks). The most basic operation a database driver performs is reading data, and they shipped it slower. The regression was caused by v2's introduction of streaming support in the BSON `valueReader`, and it took until [v2.3 in August 2025](https://github.com/mongodb/mongo-go-driver/releases/tag/v2.3.0) to fix — seven months during which MongoDB's own release notes acknowledged "the regression in v2.0."

How does a performance regression in the core read path make it through testing for a major release? Were there no benchmarks? Were they run and ignored? This is the kind of thing that erodes trust in the driver team's QA process.

## InsertMany and Five Years of Boilerplate

v1's `InsertMany` required `[]any`, which meant every Go developer using it had to write this:

```go
docs := make([]any, len(myDocs))
for i, d := range myDocs { docs[i] = d }
```

A well-known Go antipattern. Everyone who wrote Go code with MongoDB wrote this loop hundreds of times. Go generics landed in March 2022 with Go 1.18. The v2 driver came out in January 2025. The fix (accepting `any` instead of `[]any`) is better, but it took a major version bump to address something that was a known pain point for five years.

## SessionContext: A Custom Context That Should Never Have Existed

`mongo.SessionContext` in v1 was a custom type embedding both `context.Context` and `mongo.Session`. It violated a basic Go convention: `context.Context` is passed as-is, and you store values in it using `context.WithValue`. A custom context wrapper forcing type assertions throughout your code? That's the kind of thing you'd flag in a code review from a junior developer.

v2 fixed this by using plain `context.Context` with `SessionFromContext()`. The right direction, but the execution is awkward. Look at the official migration guide's own example:

```go
client.UseSession(context.TODO(), func(ctx context.Context) error {
    sess := mongo.SessionFromContext(ctx)
    if err := sess.StartTransaction(options.Transaction()); err != nil {
        return err
    }
    _, err = coll.InsertOne(ctx, bson.D{{"x", 1}})
    return err
})
```

`UseSession` stuffs the session into the context, and the first thing you do is pull it back out with `SessionFromContext`. Pack, unpack. The callback could just give you both the context and the session as separate arguments, but instead you play this round-trip game. The ceremony didn't decrease, it just changed shape.

## The Options Pattern: Not Quite Idiomatic

v1 used method chaining: `options.Find().SetLimit(10).SetSort(...)`. Not how most Go libraries do options, but at least it was familiar. v2 keeps the same chaining surface but changed the internals to a setter-function-slice pattern: options are now immutable after construction, stored options must use `options.Lister[T]` instead of concrete types, and all `Merge*Options` functions (except `MergeClientOptions`) were removed. The rest of the Go ecosystem has largely converged on two patterns: functional options (`WithLimit(10)`) or plain structs. MongoDB's approach remains its own invention.

I keep coming back to one question: is the Go driver team writing Go daily, or are they primarily working in other languages and porting patterns? The `SessionContext` design, the `primitive` package structure, the options pattern... none of these feel like decisions made by people who live in the Go ecosystem.

## Timeout Handling: Simplified but Less Flexible

Gone are `SocketTimeout`, `MaxTime` per operation, and other granular timeout controls. v2 replaced all of them with two mechanisms: a client-level `SetTimeout()` (the CSOT model) and per-operation `context.WithTimeout()`. The philosophy is sound, context-based timeouts are the Go way, and CSOT provides a sensible default for the entire client. But the per-operation granularity is gone. If your client timeout is 5 seconds but one specific aggregation pipeline legitimately needs 30, you have to wrap that call with its own context. Not terrible, but it's more boilerplate at every call site that deviates from the default.

## The Sneakiest Breaking Change: Silent Decoding Behavior Switch

Pay close attention to this one, because nothing will fail to compile. You just get wrong runtime behavior.

In v1, when you decoded a BSON document into an `any` value, nested documents came back as `bson.M` — a map you could access with `result["field"]`. In v2, they come back as `bson.D` — an ordered slice of `bson.E` elements. Your code compiles fine. It passes code review. And then it panics at runtime:

```go
raw := bson.M{}
coll.FindOne(ctx, filter).Decode(&raw)

// v1: raw["address"] is bson.M → raw["address"].(bson.M)["city"] works
// v2: raw["address"] is bson.D → raw["address"].(bson.M)["city"] PANICS
```

The reasoning is that `bson.D` preserves field order, which matters for some MongoDB command documents. Fair enough. But 99% of application code doesn't care about field order — it's reading user data, not constructing custom MongoDB commands. The people who need ordered documents are a tiny minority and already knew to use `bson.D`.

So the default behavior changed silently, no compile error, no warning, nothing. And it was changed to serve a use case almost nobody has. The opt-out exists, but you have to know about it and set it globally. In practice, every wrapper library that migrated to v2 and wants backward compatibility ends up with something like this in its connection setup:

```go
opts.SetBSONOptions(&options.BSONOptions{DefaultDocumentM: true}) // use bson.M for backward compatibility with v1
```

That comment — "for backward compatibility with v1" — tells the whole story. And if somewhere in your codebase you use `map[string]any` instead of `bson.M`, you need a separate escape hatch (`DefaultDocumentMap`). Two separate options to undo one default change that nobody asked for.

## When a Personal GitHub Namespace Broke the Driver

Not about the driver API per se, but it shows the fragility of the dependency chain. David Golden, a MongoDB engineer, moved personal Go libraries (`xdg/scram`, `xdg/stringprep`) that the driver depended on from `github.com/xdg` to `github.com/xdg-go`. A personal namespace change broke `go get -u` for everyone, and the Go module proxy cached the broken state. Production database driver availability, dependent on a personal GitHub namespace decision. Not great.

## Where Are the Generics?

v2.0 shipped in January 2025, nearly three years after Go generics became available. The driver was already breaking everything: new import path, renamed types, reworked APIs. If you're going to force every user to rewrite their MongoDB code anyway, this was the moment to do it right. v2 does use generics in one place (`options.Lister[T]` for the options plumbing), but where it actually matters, the document types and query results, everything is still `any`.

What generics could have given us:

**Typed collections.** Instead of decoding into `any` and hoping:

```go
// current v2 — manual decode, no type safety
coll := db.Collection("users")
var user User
err := coll.FindOne(ctx, filter).Decode(&user)

// what could have been
coll := mongo.TypedCollection[User](db, "users")
user, err := coll.FindOne(ctx, filter)  // returns User directly
```

**Typed queries.** The `bson.D{{"age", bson.D{{"$gt", 25}}}}` syntax is one of the most common complaints about the Go driver since its inception. Generics could enable:

```go
filter := query.Field[User]("age").Gt(25)
```

Field name typos caught at compile time. The C# MongoDB driver has had this for years.

**Typed inserts.** `InsertMany` now takes `any`, better than `[]any`, but still untyped. With generics, inserting a `Post` into a `User` collection would be a compile error, not a runtime surprise.

**Typed aggregation results.** Currently, aggregation pipelines return `[]bson.M`, completely untyped, requiring manual type assertions for every field. With generics, you define a result struct and get it back directly.

In practice, every team I know that migrated to v2 ended up building their own generic wrappers (`Reader[T]`, `BufferedWriter[T]`, typed cursor helpers) because the driver doesn't provide them. Code that the driver team should have written once, thousands of teams are writing independently. The demand is clearly there. The driver team chose not to meet it when they had the perfect opportunity: a major version that was breaking everything anyway.

Is v3 planned? Because if so, that's three rounds of "rewrite all your MongoDB code" for the Go community.

## What This Says

The Go community once had a driver that, despite its flaws, was universally adopted and got the job done. MongoDB let that driver die, took eight years to ship an official replacement, made it different enough to force a full rewrite, and then six years later shipped v2, which forces another full rewrite.

Most of v2's breaking changes share a common root: the v1 driver was designed without following Go conventions, and v2 is paying the price of fixing those original sins while introducing a few new ones. `SessionContext` shouldn't have existed. `primitive` shouldn't have been a separate package. The options pattern shouldn't have been method chaining. These were all things the Go community would have flagged on day one — and probably did, if anyone was listening.

The v2 migration was the chance to adopt generics and provide typed APIs, to bring the developer experience up to the level of other modern Go libraries. Instead, it's a large tax to fix past mistakes, with the future still on hold.

If you're planning the migration, budget more time than you think. The import path tool handles the easy part. The API changes, the silent decoding behavior change, and the general API surface rework — that's where the real work lives. Test thoroughly at runtime, not just compilation. And set `DefaultDocumentM` globally unless you have a specific reason not to.

_Everything above is based on my experience migrating real production codebases. Your mileage may vary, but I suspect not by much._
