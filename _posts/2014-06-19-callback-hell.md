"[Callback hell](http://callbackhell.com/)" is a real place, and it is a place
that you create for yourself.

It's pretty easy to fall into the trap of nesting virtually infinitely. And
just like most things in life, it's easy to attribute the trouble that you
have encountered to other people as opposed to mistakes that you have made.

## Step 1: Name your functions.

Are you naming all of your functions? NO?! Are you kidding me? Consider the
following:

```js
thing.on('event', function() {
  return 0
}
```

vs

```js
thing.on('event', eventThinger)

function eventThinger() {
  return 0
}
```

Which makes more sense immediately? Which seems scalable?

## Step 2: Use modules.

Are you putting all of your JS in one file? Oh god. No wonder you are having
trouble. You need to choose a module system. Be it
[browserify](http://browserify.org) or.. actually, let's just use browserify.

This will allow you to `require` modules and write your code in a sane way.

## Step 3: Consider state.

If you can save some indentation without causing confusion by simply dictating
shared state, it just might be worth it.

Example:

```js
function lol() {
  var state = 'a'

  parse()

  function parse() {
    console.log(state)
  }
}
```

vs

```js
var state = 'a'

function lol() {
  parse()
}

function parse() {
  console.log(state)
}
```

## Step 4: Return early

```js
function test(str) {
  if(str.length) {
    return str.split('').reverse().join('')
  } else {
    return 'No string found!'
  }
}
```

vs.

```js
function test(str) {
  if(!str.length) return 'No string found!'

  return str.split('').reverse().join('')
}
```
