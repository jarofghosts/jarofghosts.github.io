---
layout: post
title:  "Freud and Uncanny"
date:   2013-05-13 14:56:45
description: Words about things
categories:
- blog
permalink: freud-and-uncanny
---

I had an idea a while ago that I might start blogging. Of course, what good what an idea be without a proper barrier to entry? Thus: [Uncanny](https://github.com/jarofghosts/uncanny). (and [Freud](https://github.com/jarofghosts/freud).)

The idea was pretty simple. I wanted to be able to do all of my edits from the comfort of [my text editor](http://www.sublimetext.com/) and not have to worry about pushing it somewhere or dragging or dropping or any of those other things that people that aren't exceedingly lazy wouldn't even think about as an issue.

The idea was, then, I would make a piece of software that watched whatever directory I wanted to and intelligently parsed the files I dumped in there, eventually spitting them out somewhere remotely. Initially, the plan was to watch the directory and parse the files with [node.js](http://nodejs.org/), then trigger some fancy [git](http://git-scm.com/) magic to deploy it to [the dubs](http://en.wikipedia.org/wiki/Internet).

After some finagling, it became moderately clear that what I was making was actually (at least) two modules. I needed something to watch and transform files, and something to intelligently amass all of them and dump them together in a semi-coherent manner. First things first: I made Freud.

Freud can be set to watch any directory for changes in files and subdirectories *Note: it does not recurse into the subdirectories*
It also has the pleasant distinction of being my first [npm](https://npmjs.org/) module. And the first instance of me writing automated tests! Either way, it makes it very easy for me to tell it to listen for files with certain extensions and react accordingly whenever they are changed. Writing it really helped me to understand and deepen my love for node. Although, it's [fs.watch API](http://nodejs.org/api/fs.html#fs_fs_watch_filename_options_listener) left a bit to be desired.

[Uncanny](https://npmjs.org/package/uncanny) is currently in alpha, so there's a still a lot of work to be done. It *is*, however, being used to power this site-- so it's not totally worthless. I hope to chronicle a bit about what I learn during the continual development of Freud and Uncanny (as well as the eventual development of [Battlecats](https://github.com/jarofghosts/battlecats), my browser-based [Battleship](http://en.wikipedia.org/wiki/Battleship_\(game\)) clone).

So stay tuned!
