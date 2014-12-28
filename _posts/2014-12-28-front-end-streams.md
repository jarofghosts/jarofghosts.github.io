## What

[Streams](http://nodejs.org/api/stream.html) are infectious. They are, to date,
my favorite abstraction for dealing with large amounts of data over time.

They force you to think about big problems in small pieces, and present a handy
composable interface to allow communication with one another via `.pipe()`.

A big problem that I deal with every day is that of working with a large amount
of dynamic user input over time in the form of DOM events. That, to me,
indicates that streams are a great fit for modeling web applications!

## How

Browserify enables us to have [node streams](http://nodejs.org/api/stream.html)
in the browser. This means that not only do we get stream primitives for cheap,
but we also get the *wealth* of streaming modules on
[npm](https://www.npmjs.org) that Just Work(TM), because they deal simply with
isolated forms of data.

For more specific DOM tasks, we use speciality modules, such as:

* [dom-delegation-stream](http://npm.im/dom-delegation-stream)
* [dom-value-stream](http://npm.im/dom-value-stream)
* [dom-replace-html-stream](http://npm.im/dom-replace-html-stream)

Another module we lean heavily upon for aggregating and contextualizing streams
(and event emitters in general) is [ObjectState](http://npm.im/objectstate).

If we want to use something in a pipeline that doesn't provide a streaming
interface, we simply wrap it with one!

## Why

Thinking about your application in terms of pipelines of data is likely a
novel concept, but it provides you with a lot of benefits.

1. The pieces that your system consists of (the streaming modules) tend to be
much smaller and simpler. These properties tend to make them more testable,
more reusable, easier to reason about, and less prone to bugs.
2. The code that glues the pieces together as pipelines tends to be very
expressive. `a.pipe(b).pipe(c)` is a very powerful statement! Not only is it
very easy to write, it is also very easy to read.
3. Your system is talking [POJOs](http://odetocode.com/blogs/scott/archive/2012/02/27/plain-old-javascript.aspx)
at its deepest levels. This means that stepping through code and introspecting
data is much easier and more intuitive. It also means that you are free to
introduce *any* dependency at your endpoints, because there aren't any prior
opinions baked into your data-- only your own.

## Examples

Imagine you want to implement a binding between an input element and an output.
All we have to do is think of these things in terms of readable and writable
streams and suddenly it's very trivial!

Let's imagine an ideal world:

```javascript
// pseudo-code incoming!

inputElement.pipe(outputElement)
```

The good news is that this ideal world is not much different from the world
that you can very easily be living within!

```javascript
var write = require('dom-replace-html-stream')
  , events = require('dom-event-stream')
  , values = require('dom-value-stream')

var input = events(document.getElementById('input'), 'input').pipe(values())
var output = write(document.getElementById('output'))

input.pipe(output)
```

Let's take a look at what is happening here. 

1. We require all the modules that we are going to use.
2. We set up a first pipeline consisting of:
    - 'input' events triggered by the element with ID 'input'
    - the value of the element that triggered the original event
3. We create a writable stream to the element with ID 'output'
4. We create a new pipeline from the "input" to the "output"
5. Magic

### Introducing "ObjectState"

Now, let's imagine that you want to actually collect this data somewhere to be
used later, perhaps to send off to an API. This is where
[ObjectState](http://npm.im/objectstate) comes in! Check this out:

```javascript
var objectState = require('objectstate')

var state = objectState()

// ...the previous example happens...

state.listen(input, 'userInput')
```

Now, as you enter text into the input, ObjectState is collecting the emissions
into a state object like:

```javascript
{
    userInput: 'whatever you typed!'
}
```

The best part is that ObjectState is itself a stream! Which means that you can
pipe it anywhere and it will happily emit its "state objects" anywhere you
desire. This gives you a very simple and powerful way to compose streams of
data into meaningful chunks of data and act upon them accordingly.

### Putting it all together

Let's make a slightly more advanced (if still somewhat contrived) system. Say
you want to create a module that lets the user input their credit card number
into an input and have it rendered as it might appear on a card itself.

For this example, we will want to use a template engine. At Urban Airship, we
use [Ractive](http://www.ractivejs.org/). As mentioned earlier, because it does
not expose a streaming interface, we wrap it in one. That wrapper looks very
much like:

```javascript
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
```

Pretty straight-forward, but let's take a quick look.

* We expose a simple API that meets our needs (give an element and a template,
  optionally direct Ractive options, return a stream).
* We provide access to the Ractive instance via the `.view` property of the
  returned stream.
* Whenever our stream is written to, it resets the context of the Ractive
  instance.

Now, let's build our theoretical widget! We can assume that it will be handed a
DOM element to render itself into.

Let's start with the template

```html
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
```

Simple enough, now for the actual code:

```javascript
var events = require('dom-delegation-stream')
  , values = require('dom-value-stream')
  , objectState = require('objectstate')

var ractiveStream = require('./ractive-stream')
  , cardTemplate = require('./template.ract') // check out the ractify transform
  , stripNonDigits = require('./strip-non-digits')
  , addSpaces = require('./add-spaces')

module.exports = creditCard

function creditCard(el) {
  var ractive = ractiveStream(el, cardTemplate)
    , state = objectState()

  var name = events(el, 'input', '[name=full-name]')
    .pipe(values())

  var cardNumber = events(el, 'input', '[name=card-number]')
    .pipe(values())
    .pipe(stripNonDigits())
    .pipe(addSpaces())

  state.listen(name, 'fullName')
    .listen(cardNumber, 'cardNumber')

  state.pipe(ractive)
}
```

This might look scary at first, but let's break it down

* We require all of the modules we will be using, including a couple of
  theoretical but trivially-implemented modules.
  - "strip-non-digits" just removes all non-digit characters from a string.
  - "add-spaces" inserts spaces into a string in a pattern like you would
    expect on a credit card.
* We declare our export, which is the function that constructs this "widget"
* Our function takes an element, and immediately creates a ractiveStream with
  that element and our template.
* We initialize an ObjectState to serve as our source of context.
* We create two pipelines.
  - They both start as events -> values, effectively turning input keystrokes
    into the value of the element they were triggered from.
  - The `cardNumber` stream is further piped into our "add-spaces" module for
    formatting.
* Our objectState is told to listen to these streams, and consider them as keys
  "fullName" and "cardNumber" respectively.
* We pipe our objectState directly into our ractiveStream to be used as context.

## In Conclusion

Streams are an undeniably powerful concept. They enforce a separation of
concerns and provide handy mechanisms for composition. Hopefully, I have made
clear how you can leverage them for making your front-end code simpler and
more portable!
