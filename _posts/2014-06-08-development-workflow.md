---
layout: post
title:  "Development Workflow"
date:   2014-06-08 23:56:45
description: How I make things
categories:
- blog
permalink: development-workflow
---

Making stuff is hard, right?

There are lots of ways to make things less hard, this is what works for me and
it might work for you!

Step 1 is to clearly identify the problem that needs solving. Even if you are
making something for fun or learning purposes, there is always a goal to be
considered. Part of the consideration should be breaking the *big* problem into
as many little problems as possible. And then breaking all of those problems
into smaller problems. It's problems all the way down. At some point you will
hit a bedrock of solved problems to use as a foundation.

### A (Contrived) Hypothetical

Let's imagine you want to make a module that (for some reason!) takes a
directory of files, parses all of the tarballs within it, finds any that
contains a file with a certain name, and then posts the tarball filename to a
web server.

A quick search tells us that this (surprisingly!) doesn't exist yet. If it did
and it worked, this would be a very short blog entry.

I prefer to work with streams, so for this example we will assume this app will
be created as a stream pipeline.

In order to make this module, we will need:

* A module that streams filenames from a directory
* A module that will filter a stream (so we only see tarballs)
* A module that streams objects with tarball name and filenames within it
* A module that filters those objects by filename
* A module that posts data to a web server.

Luckily for us, the first 2 and the last one already exist. So at this point,
I would begin work on a module that exposes a duplex stream reading tarball
locations and writing out objects with tarball location and entry filename,
once per entry.

{% highlight bash %}
# work begins!
mkdir tar-entry-stream; cd tar-entry-stream
# initialize a git repo
git init
# add a license, typically MIT
cp ~/license.mit LICENSE
# create a barebones readme
echo 'tar-entry-stream\n====' > README.md
# make a test dir
mkdir test
# create some placeholder files
touch index.js test/index.js
# copy in my travis config
cp ~/travis.yml .travis.yml
# create the package.json, my ~/.npmrc is filled out with my defaults
npm init
{% endhighlight %}

I then create a repo on [Github](https://github.com) to host my source code on.
I write the minimum amount of code I can in `index.js` to make my module do
what it needs to.

`npm install --save-dev tape`

Then I create a few tests to ensure that my module does what it should do.
`npm test` will do run those for me since I filled out my package.json
correctly.

Once I am satisfied that everything is working order, I edit my README to
contain:

* A super-brief description
* A code example
* Any note of gotchas or options
* An explicit callout to the included license (choose one you like
  [here!](http://choosealicense.com))

...and then...

{% highlight bash %}
# add my files
git add index.js test/index.js package.json README.md .travis.yml LICENSE
# add my github repo as a remote
git remote add origin $github-url
git push origin master
{% endhighlight %}

...wait for travis to verify build, and then: `npm publish`. Tada! Now I can
begin work on my main app.

Let's say I start working on it, but find a bug in my tar-entry-stream module.
Wuh oh.

### Patching Bugs

I inspect my code to see if I can quickly identify the cause, if I can I patch
it. Either way, I write a new test that exposes the bug so that I can verify
it is fixed whenever I *do* find the cause. I have a binding in vim that
allows me to quickly run `npm test` so that I can verify my code quickly and
constantly as I work on it.

Once my new test passes (and all of my old ones don't fail!)

{% highlight bash %}
# stage whatever files have been altered
git add index.js
# commit my changes
git commit -m 'fixes hypothetical bug'
# bump the version of my module in the package.json, commit and tag
npm version patch
# push changes to github
git push --tags origin master
# .. wait for travis build, and then ..
npm publish
{% endhighlight %}

And there you have it, from creation to publishing, to inevitable maintenance.
The benefits of this particular workflow for me are:

* I'm not tied to any specific system that gets in my way
* I can write small, focused tests
* I can write concise, pointed documentation
* I can patch bugs and make changes quickly and confidently
* Because modules are generalized, I can potentially reuse them for future
  projects. And so can other people!
* I don't have to maintain tons of configuration files or build scripts
* From start to publish goes very quick, so I feel more inclined to create
