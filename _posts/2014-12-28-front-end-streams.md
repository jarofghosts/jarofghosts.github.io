---
layout: post
title:  "Front-end Streams"
date:   2014-12-28 17:56:45
description: Streams all up in your browser window
categories:
- blog
permalink: front-end-streams
---
[Streams](http://nodejs.org/api/stream.html) are infectious. They are, to date,
my favorite abstraction for dealing with large amounts of data over time.

They force you to think about big problems in small pieces, and present a handy
composable interface to allow communication with one another via `.pipe()`.

A problem I deal with every day is user input changing over time in the form of
DOM events. Streams are designed to deal with data changes over time, so it
seems like a logical fit.

Thinking about your application in terms of streams and pipelines of data
provides you with a lot of benefits.

1. The pieces that your system consists of (the streaming modules) tend to be
much smaller and simpler. These properties tend to make them more testable,
more reusable, easier to reason about, and less prone to bugs.
2. The code that glues the pieces together as pipelines tends to be very
expressive. `a.pipe(b).pipe(c)` is a very powerful statement. Not only is it
very easy to write, it is also very easy to read: you are just chaining data
transformations.
3. Your system is talking [POJOs](http://odetocode.com/blogs/scott/archive/2012/02/27/plain-old-javascript.aspx)
at its deepest levels. This means that stepping through code and introspecting
data is much easier and more intuitive. It also means that you are free to
introduce *any* libraries or modules into your "stack".

For a more in-depth analysis, see
[The Stream Handbook](https://github.com/substack/stream-handbook)

## An Ideal World

Imagine you want to take what a user types into an input element and display it
in an output element. All we have to do is think of these things in terms of
readable and writable streams and suddenly it's very trivial.

Let's imagine the simplest way to write that:

{% highlight javascript %}
// pseudo-code incoming!

inputElement.pipe(outputElement)
{% endhighlight %}

The good news is that this ideal world is not much different from the world
that you can very easily be living within.

## How

Browserify enables us to have [node streams](http://nodejs.org/api/stream.html)
in the browser. This means that not only do we get stream primitives for cheap,
but we also get the *wealth* of streaming modules on
[npm](https://www.npmjs.org) that Just Work(TM), because they deal simply with
isolated forms of data.

Generally, I use [through](http://npm.im/through) for stream construction. It
presents a nice, functional interface that is very intuitive.

For more specific DOM tasks, we can use speciality modules, such as:

* [dom-delegation-stream](http://npm.im/dom-delegation-stream)
* [dom-value-stream](http://npm.im/dom-value-stream)
* [dom-replace-html-stream](http://npm.im/dom-replace-html-stream)

If I want to use something in a pipeline that doesn't provide a streaming
interface, I simply wrap it with one.

## Examples

{% highlight javascript %}
var write = require('dom-replace-html-stream')
  , events = require('dom-delegation-stream')
  , values = require('dom-value-stream')

// `.pipe()` always returns the destination stream, so we set up a pipeline and
// assign `input` to the stream of values
var input = events(document.getElementById('input'), 'input').pipe(values())
var output = write(document.getElementById('output'))

input.pipe(output)
{% endhighlight %}

Let's take a look at what is happening here. 

1. We require all the modules that we are going to use.
2. We set up a first pipeline consisting of:
    - 'input' events triggered by the element with ID 'input'
    - the value of the element that triggered the original event
3. Because `.pipe()` always returns the destination stream, `input` is assigned
   to the `values()` stream that is emitting the element values.
4. We create a writable stream to the element with ID 'output'
5. We create a new pipeline from the "input" to the "output"
6. Magic

### Introducing "ObjectState"

Now, let's imagine that you want to actually collect this data somewhere to be
used later, perhaps to send off to an API. This is where
[ObjectState](http://npm.im/objectstate) comes in. Check this out:

{% highlight javascript %}
var objectState = require('objectstate')

var state = objectState()

// ...the previous example happens...

state.listen(input, 'userInput')
{% endhighlight %}

Now, as you enter text into the input, ObjectState is collecting the emissions
into a state object like:

{% highlight javascript %}
{
    userInput: 'whatever you typed!'
}
{% endhighlight %}

The best part is that ObjectState is itself a stream. Which means that you can
pipe it anywhere and it will happily emit its "state objects" anywhere you
desire. This gives you a very simple and powerful way to compose streams of
data into meaningful "chunks" and act upon them accordingly.

### Putting it all together

Let's make a slightly more advanced (if still somewhat contrived) system. Say
you want to create a module that lets the user input an ID number into an input
and have it rendered as it might appear on an ID card.

For this example, we will want to use a template engine. At Urban Airship, we
use [Ractive](http://www.ractivejs.org/). As mentioned earlier, because it does
not expose a streaming interface, we wrap it in one. That wrapper looks very
much like:

{% highlight javascript %}
var Ractive = require('ractive')
  , through = require('through')

module.exports = ractiveStream

function ractiveStream(el, template, _options) {
  var options = _options || {}
  var stream = through(write)

  options.el = el
  options.template = template

  var ractive = new Ractive(options)

  stream.view = ractive

  return stream

  function write(data) {
    ractive.reset(data)
  }
}
{% endhighlight %}

Pretty straight-forward, but let's take a quick look.

* We expose a simple API that meets our needs (given an element and a template,
  optionally a
  [Ractive options object](http://docs.ractivejs.org/latest/options), return a
  stream).
* We provide access to the Ractive instance via the `.view` property of the
  returned stream.
* Whenever our stream is written to, it resets the context of the Ractive
  instance.

Now, let's build our theoretical widget! We can assume that it will be handed a
DOM element to render itself into.

Let's start with the template

{% highlight html %}
<div class="card-input">
  <input type="text" name="full-name"/>
  <input type="text" name="card-number"/>
</div>
<div class="card-output">
  <div class="card-template">
    <span class="card-number">{{ cardNumber }}</span>
    <span class="card-name">{{ fullName }}</span>
  </div>
</div>
{% endhighlight %}

Simple enough, now for the JavaScript:

{% highlight javascript %}
var events = require('dom-delegation-stream')
  , values = require('dom-value-stream')
  , objectState = require('objectstate')

var ractiveStream = require('./ractive-stream')
  , cardTemplate = require('./template.ract') // check out the ractify transform
  , stripNonDigits = require('./strip-non-digits')
  , addDashes = require('./add-dashes')

module.exports = cardWidget

function cardWidget(el) {
  var ractive = ractiveStream(el, cardTemplate)
    , state = objectState()

  // DOM events -> values, assign `name` to the values stream
  // we select on the `name` attribute, because it is not typically subject to
  // change as frequently as class or ID.
  var name = events(el, 'input', '[name=full-name]')
    .pipe(values())

  // DOM events -> values -> strip non-digits -> add dashes, assign `cardNumber`
  // to the dashes stream
  // " 123 45678,91011a12" -> "123456789101112" -> "1234-5678-9101-112"
  var cardNumber = events(el, 'input', '[name=card-number]')
    .pipe(values())
    .pipe(stripNonDigits())
    .pipe(addDashes())

  state.listen(name, 'fullName')
    .listen(cardNumber, 'cardNumber')

  state.pipe(ractive)
}
{% endhighlight %}

Let's break it down:

* We require all of the modules we will be using, including a couple of
  theoretical but trivially-implemented modules that export a function that
  returns a stream:
  - "strip-non-digits" just removes all non-digit characters from a string.
  - "add-dashes" inserts dashes into a string in a pattern like you might
    expect on an ID card.
* We declare our export, which is the function that constructs this "widget"
* Our function takes an element, and immediately creates a ractiveStream with
  that element and our template.
* We initialize an ObjectState to serve as our source of context.
* We create two pipelines:
  - They both start as events -> values, effectively turning input keystrokes
    into the value of the element they were triggered from.
  - The `cardNumber` stream is further piped into our "add-dashes" module for
    formatting.
* Our objectState is told to listen to these streams, and consider them as keys
  "fullName" and "cardNumber" respectively.
* We pipe our objectState directly into our ractiveStream to be used as context.

## In Conclusion

Streams are an undeniably powerful concept. They enforce a separation of
concerns and provide handy mechanisms for composition. Hopefully, I have made
clear how you can leverage them for making your front-end code simpler and
more portable. In future blog posts, I plan to explore options for creating
large systems using these concepts.

## Further Reading

* [Streams Documentation](http://nodejs.org/api/stream.html)
* [The Stream Handbook](https://github.com/substack/stream-handbook)
* [Ractive](http://www.ractivejs.org/)
* [Browserify](http://browserify.org/)
* [Ractify Transform](http://npm.im/ractify)
