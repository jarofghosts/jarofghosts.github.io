---
layout: post
title:  "Front-end Streams: Architecture"
date:   2015-04-19 17:56:45
description: Coordinating Streams
categories:
- blog
permalink: front-end-streams-2
---

## What

When we approach our problem we first consider how to effectively model the data
involved. A perfect implementation of a flawed data model is not of any value to
anyone.

## Why

* Easy to reason about
* Everything is very explicit
* Recursion (pipelines of streams of pipelines of streams)
* Code re-use (npm is a big deal)

## How

We model all of our components as duplex streams. They are each in charge of a
particular "chunk" of data. They can be written to at any time with that data
and their "view" updates to reflect it. As interactions occur, if they affect
the data model, the updated data is streamed.

This is great model because it allows us to think about everything in terms of
simple I/O. It also means that we don't have to consider anything outside of
what each component is itself responsible for.

ObjectState gets us observability and flow control. It is impossible to change
something about the object that it is in charge of without it emitting the
change. By the same token, duplicate emissions will be swallowed by ObjectState
and prevent unnecessary emissions.

In cases where we don't want to use ObjectState, we can use
object-cursor-stream, which provides us with the de-duping and key-pathing
capabilities of ObjectState, without the persistence.

```javascript
var sectionStream = sectionWidget(el)

state.listen(sectionStream, 'section')
state.pipe(objectCusor('section', {})).pipe(sectionStream)
```

### But, what about ... ?

In some situations there are interactions that need to be communicated but that
aren't necessarily part of the data that the component is in charge of. In these
cases we emit custom events. If information needs to be communicated to the
component externally, we provide a custom method exposed to do so.
