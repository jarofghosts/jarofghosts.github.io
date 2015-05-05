---
layout: post
title:  "Front-end Streams: Architecture"
date:   2015-05-05 17:56:45
description: Coordinating Streams
categories:
- blog
permalink: front-end-streams-2
---

When approaching a problem on the front-end, I prefer to start by thinking
about how I would best represent it in terms of a data structure. If it helps,
consider what you would return if asked to provide a JSON endpoint representing
the entire state of your page.

The result typically manifests as a nested structure with discrete chunks
representing areas of concern for the application at hand. Once that has been
determined, I can start thinking about the data in bite-sized chunks,
considering only the immediate problem at hand.

Each of these "chunks" end up being controlled by a module that presents a
Duplex Stream interface. Each module is in control of rendering its view and
managing its "chunk". That means both updating its representation when written
to, as well as emitting changes in the data as they occur.

This approach can then be applied recursively. At every level, the system is
just pipelines of streams of data. This makes conceiving of the system as a
whole very simple, everything essentially works in the exact same way.

## Why

This is great model because it allows us to think about everything in terms of
simple I/O. It also means that we don't have to consider anything outside of
what each component is itself responsible for.

ObjectState gains us observability and flow control. It is impossible to change
something about the object that it is in charge of without it emitting the
change. The other side of that coin is that ObjectState will *not* emit when you
perform an operation on its state object that does not actually change it.

In cases where we don't want to use ObjectState, we can use
[object-cursor-stream](http://npm.im/object-cursor-stream), which provides us
with the de-duping and key-pathing capabilities of ObjectState, without the
statefulness.

{% highlight javascript %}
var sectionStream = sectionWidget(el)
  , state = objectState()

state.listen(sectionStream, 'section')
state.pipe(objectCusor('section', {})).pipe(sectionStream)
{% endhighlight %}

### But, what about ... ?

In some situations there are interactions that need to be communicated but that
aren't necessarily part of the data that the component is in charge of, or that
should not be directly written externally. In these cases we emit custom events.
If information needs to be communicated *to* the component externally, we
provide a custom method exposed to do so.

## The View Layer

As mentioned in my previous post, [Ractive](http://www.ractivejs.org) makes for
an excellent view layer. Like React it is DOM-aware, so you can throw data at it
without having to worry about it completely re-rendering everything and
ruining your life like is common with standard string-based templating
solutions.

In order to side-step some aspects of Ractive that are problematic for the way
I prefer to structure things, I have adopted a couple of somewhat uncommon
practices.

1. I namespace all writable data under `state` in the template context. This
  allows me to provide static context (like expressions or constants) without
  fear of stomping on them on write.
2. I use [Ractive decorators](http://docs.ractivejs.org/latest/decorators) as
  a system for dynamic components. Essentially using them as a hook into the
  lifecycle of DOM nodes.
