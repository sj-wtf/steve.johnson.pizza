---
layout: post
title: Getting Jython and Groovy to play nice together
description:
headline: JVM language integration
modified: 2016-03-09
category: tech
tags: [update, python, jython, groovy, java, jvm]
image:
---
UPDATE: I've abandoned this.  It's not easy to switch between Groovy and Python on the fly, and it's easier just to port the library I was trying to integrate to Python rather than do something that's this janky, especially using something as unmaintained as Jython appears to be.

Please note: this article is a work in progress.  I haven’t actually gotten joint compilation of Jython and Groovy sources set up correctly yet, but I’m actively working on it, and based on what I’ve read it’s totally possible.

One of my most recent projects at work involves adopting some legacy Groovy code for use in a scripting DSL I am working on.  The legacy code wraps most of the HTTP APIs that our application ecosystem is composed of, and I’ve written extensive Python code for accessing the various datastores (Mongo, Cassandra, MySQL, memcached, SQLite, and others) the applications use directly.  I don’t want to re-write every single function of the applications, so of course I want to have easy programmatic access to the APIs of the applications I’m supporting, as well as direct access to the underlying data, to do things that the apps can’t.  So, rather than re-write everything from scratch, I’m going to start running my Python code in Jython, and dropping in the existing Groovy code as-is.


One of the added benefits I can get from moving from Python to Jython is concurrency.  I no longer have to be concerned about CPython’s Global Interpreter Lock (GIL), which prevents multiple native threads on whatever system you’re on from operating on Python bytecode at any given time.  This becomes a problem with multithreaded code, where you would want to run the same code at the same time many times over.  Native threading in a scripting DSL means there’s no need for asynchronous coding techniques to defend against GIL problems.  Because this is client-side code and it’s much easier from a coding perspective to do concurrency with a thread or actor pool, it makes our code much simpler.  With the GIL, we’d have to use asynchronous programming defensively to get better performance, so not having to do that ends up making my code cleaner, and prevents me from ending up in callback hell.

So, brass tacks, how can you make it work?  I write up all the Python code I need in a development environment with Groovy and Jython set up side-by-side, testing code separately, but writing unit tests along the way that test the interoperability of the two types of objects.  The scripts are written in Python, and import Groovy code.  Calling Groovy code in these scripts is pretty much the same as calling Java code in Jython.  You import the objects, and treat them the same as Python objects  One problem we run into is that we loose all the flexibility and ease of quick tests that languages with a REPL provide.  Instead, we need to build .jar files for every script we want to run.  On top of that, the .jar files are enormous, because they have to contain all the parts of both Groovy and Jython that we want to use.  Long story short, I wouldn’t recommend it.  Long-term, I want to port all the Groovy interfaces I’m using to Python, so I can make my jars smaller, and make my development process easier.  Maybe I’ll clean up the interfaces while I’m at it so they look more like my database objects so I don’t have to remember as much.  Concurrency is implemented at the script level, so I don’t plan to move back to CPython.  I’d have to re-work all my methods to have callbacks to get similar performance down the line.

When I have more working examples, I’ll either add them here or throw them in a separate post.
