I just spent a good two hours writing a node module that will never see the light of day.

I wanted to write a good router that didn't have a ton of useless dependencies and was essentially just an [EventEmitter](http://nodejs.org/api/events.html#events_class_events_eventemitter) with a route-friendly interface.

Turns out, [such a router already exists](https://github.com/wookiehangover/node-ramrod). And it's pretty sweet.

The reason I'm pretty pumped and not kind of pissed that I just wasted a decent chunk of my weekend is that, internally, it works (basically) just like mine did. It's nice because not only has someone **already** done all the work and written all the tests to get me what I want, turns out I was on the right path to get there myself.

Not to mention, in the process I learned a little more about stuff and that's always a good thing.