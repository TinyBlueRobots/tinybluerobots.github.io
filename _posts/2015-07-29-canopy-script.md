---
layout: post
title:  Scripting with F# and Canopy
date:   2015-07-29
categories: fsharp
---
Sometimes when you're developing a website you can get caught in a slow feedback loop of making a change, starting the project, navigating to the page, checking it, rinse, repeat. You may start to wish you'd automated it.

[Canopy](http://lefthandedgoat.github.io/canopy/) is your friend here. It's a great demonstration of the simplicity and power of F# DSLs, and a really easy way to get started with the language and introduce it into your work flow. Automated tests is where it really shines, but using it from a script can help you write those tests, and gives you a quick and flexible way of doing some adhoc automation. And it works on mono too, but make sure you set [this](http://www.mono-project.com/docs/advanced/iomap) environment variable.

I've saved you the job of downloading and setting everything up, you just need to have F# installed, create a directory for your scripts, then download and run this using fsi:

https://github.com/TinyBlueRobots/CanopyScript/blob/master/install.fsx

This will install Canopy, and create a couple of files: ```refs.fsx``` and ```demo.fsx```. This is ```demo.fsx```:

	#load "refs.fsx"
	open canopy
	start firefox
	url "http://google.com"
	"input" << "canopy fsharp"
	press enter
	sleep 5000
	quit()

The first line loads a script which contains the necessary references for canopy, and the rest speaks for itself. You can change the browser to be firefox, chrome, ie, aurora, or phantomJS.

One of the biggest pains of creating automated tests is getting the selector right, and here is where interactive scripting can save you. Open fsi, run the first three lines of this script, and now you can control the browser. This lets you easily experiment with interactions and selectors, so have a look at the rest of the supported [actions](http://lefthandedgoat.github.io/canopy/actions.html) and start automating.
