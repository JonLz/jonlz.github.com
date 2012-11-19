---
layout: post
title: "Guard, LiveReload, and Spork"
date: 2012-10-22 11:31
comments: true
categories: [guard, livereload, spork, rails, gems, ide, testing]
---

I sat down last night and started getting back into Rails finally.  I feel like I'm at a point with ruby now where I'm pretty comfortable with basic syntax and program flow.  I probably spent a little more time than I would have liked to getting up to speed with ruby since my main aim is to build web apps but damn if ruby programming isn't just *fun*.

As part of my rails project I made sure to set it up so I could do it in a TDD way.  I kept hearing great things about Guard (a ruby gem) on the rogues podcast and a couple of other places (the rails tutorial by Michael Hartl also talks about it) so I gave it a try.  Long story short, this thing is amazing.  
<!--more-->

Guard is a gem that watches for file changes in your working folders (i.e. when you save a file you're working on) and automatically performs actions when that file changes.  It's most commonly used to automatically run tests on files when you save.  The only problem is this can get slow very fast since tests have to generate and run your entire rails server every time.  This is where spork comes in.  Spork basically speeds up your tests by running a server in the background that stays up instead of being regenerated every time your tests are run.  

Live Reload lets you do the same guard-esque features with your actual browser by reloading the browser automatically any time you make changes to files that alter the source files (i.e. HTML, CSS, JS).  It seems simplistic but it's positively addicting and makes it easy to see exactly how the code you're writing impacts the front-end views.

These are all great in their own rights but truly amazing when paired together as they are synergistic tools.  You can automate all of these tasks by linking them into your guard gem.  This way you just run guard once and it will run your spork server, livereload server, and then watch your files for automatic testing (both for tests and for livereload).  Now your tests are blazingly fast, and you can get instant feedback on your code changes directly in your browser!

I didn't have too much difficulty setting any of this up thanks to the RailsCasts videos.  There it's covered in great depth and you can see some of the nuances:  
[RailsCasts - Guard](http://railscasts.com/episodes/264-guard)  
[RailsCasts - Spork](http://railscasts.com/episodes/285-spork)

Here are the gems if you're curious:
```  
gem 'guard'
gem 'guard-livereload' 
gem 'guard-spork'       #these gems automatically include the dependency gems 'livereload' and 'spork'
``` 

Time to get cracking on some Rails now that the testing hell has been alleviated.