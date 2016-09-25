# Chapter 1: Introduction

## A Short Introduction to Clojure
Clojure is hosted dialect of Lisp invented by Rich Hickey with it's initial release in 2007.

Its cited design goals include:

  -  A Lisp
  
  -  General purpose
  
  -  Dyanmically typed, with optional gradual typing
  
  -  Emphasis on Functional Programming
  
  -  Hosted Language
  
  -  Encourage use of immutable data structures

Clojure is maintained as an open-source project by Cognitect, who provide consulting, support, and of course an office for Rich Hickey.

#### What is hosted?
This means that Clojure is built on top of, and relies upon, the underlying type system, bytecodes, garbage collection, threads, system-services, and so forth of the hosting environment.  No reinvented wheels here.

There are three dominant implementations:

  -  **Clojure**, which is hosted on the Java Virtual Machine (JVM) - This is the focus of this material.
  
  -  **ClojureCLR**, which is hosted on the Common Language runtime (CLR) which is the execution engine of the Microsoft .Net Framework.
  
  -  **ClojureScript** which is transpiled to Javascript, which runs on platforms which include web-browsers.

The JVM Clojure is likely still the dominant hosting environment and the most mature, but ClojureScript is definitely on the rise.  

ClojureCLR has a much smaller share, and is developed and maintained outside of Cognitect (but it uses a lot of the Clojure sources and faithfully tracks Clojures releases).  The main use-case for ClojureCLR would be interoperating with an established CLR ecosystem.

In all cases, the core language is virtually the same, although they differ somewhat when it comes to matters involving host interop, since the underlying platforms are each different.

#### Why Hosted?
In a word: **Leverage**

The JVM is a mature platform than runs on oodles of different computer architectures.  It has literally man-centuries of effort invested in portability, stability, and performance.

Clojure rides the coat-tails of the Java eco-system to great benefit.  Clojure is a 1st class citizen in the JVM world.  It freely interoperates with other JVM languages, and can easily use/leverage any JVM software in existance.

At some level, Clojure doesn't *use* java, it *is* Java.  Strings are java.lang.Strings, integers are java.lang.Longs, and so forth.

Clojure code *compiles* into JVM bytecode, just like Java does.  It can be compiled ahead-of-time and put into .jar files, or it can be compiled dynamically - replacing it's bytecode on the fly. 

It has all the development leverage of acting interpretive, while still having the performance characteristics of a compiled language.

There is also the "sneak-in" factor - which is to say that it an easy sell into existing IT infrastructures as it likely directly interoperates with platforms that are already in their shop.  You can usally "sneak it in" without having to indulge in platform wars and concerns.

#### What is a Lisp?

> Any sufficiently complicated C or Fortran program contains an ad hoc, informally-specified, bug-ridden, slow implementation of half of Common Lisp. 
-- Philip Greenspun

Lisp is the second-oldest high-level programming language (surpassed only by FORTRAN), it's roots go back to 1958.

Many of the "advanced" or "upcoming" features in popular programming languages have their origins in Lisp.  First class functions, functional programming, recursion, dynamic typing, garbage collection, and polymorphism are ideas that all originated in Lisp.

We'll get into the details of syntax a little later, but for the moment, you can think about Lisp as having these main aspects:

  - there are only functions, no statements
  
  - the syntax is specified as a set of nested literals and forms.  The nesting is where all the parentheses come from.
  
  - "code is data", meaning all of your code is defined by building a data-structure that happens to conform to the conventions of "code".


#### What is functional? 
Functional programming is a paradigm or style of building computer programs that treats computation as the evaluation of (mainly pure) functions rather than a series of statements which mutate data and internal state, as is the case with imperative languages.

It is a more mathematical view of the world that sees functions that take values and produce others without side-effects.
   

#### What is immutable?
Immutability goes hand-in-hand with functional programming.  There are no notions of things like:

   - incrementing a variable
   
   - changing the value of an array element
   
   - adding a key,value to a map
   
   - objects that can change their internal state (maybe in another thread!)
   
Values can never change, you can only make new ones from old ones.  This applies to entire data-structures just as it does to simple scalars.

This makes certain things much, much simpler - for example, concurrency is a complete non-issue in an immutable world.  You don't have to manually coordinate "updates" when there are no updates possible in the first place.  Threads can't step on each other when there is no stepping to begin with.

But, you say, all of this immutability must come at a price.  If I have a huge map, doing something to it makes a completely new huge map out of the old one - that must be mighty expensive, no?

That, right there, might be the primary reason imperative languages got their foothold back in the day.  Making copies of value-based things is quite expensive, and people found that out hence the following historical quip:

> LISP programmers know the value of everything and the cost of nothing -- Alan Perlis

Clojure has an elegent solution to the problem of making immutable data-structures practical.

For each of the basic data-types: lists, vectors, maps, sets, and so forth, Clojure implements these persistent data structures using a technique known as **structural sharing**.  This makes the operation of transforming an old value to a new one very efficient - since the new value will share most of it's structure with the old one.

The result is very performant.  Many "copies" of a shared data-structure can exist, and previous versions that are no longer interesting get garbage collected, which in effect is only getting rid of the parts which are not shared.

Of course, managing this is no small feat, but that is all handled by the core of Clojure, and is not something that requires concern at the application level.  From the outside, lovely functional programming with values that are very cheap to create and use.

This is an abstraction that makes immutable functional programming practical in Clojure.

## Getting Started
Getting Clojure up and running on your computer is delightfully simple.

There are two things you need to get there.

   - first, you need Java, since that that is what all things Clojure are made from.  You will likey want to opt for the entire JDK (developer kit) rather than just the JRE (runtime only).
   
   - second, you need a program called Leiningen, which will bootstrap the rest of the process.
   

## Installing Java

Make sure you have Java 1.8 installed.
```
▶ java -version
java version "1.8.0_40"
Java(TM) SE Runtime Environment (build 1.8.0_40-b27)
Java HotSpot(TM) 64-Bit Server VM (build 25.40-b25, mixed mode)
```
if not follow this [Install Java](https://www.java.com/en/download/help/download_options.xml)


## Installing Leiningen

Multiple paths depending on your system type.

But before we go there, a brief overview of what leiningen is up to:

  - it (maybe) bootstraps itself
  - it does this by going off and downloading packages from package repositories on the internet
  - it maintains these packages in a maven repository on your local machine (typically find this in your home directory in a directory called '.m2')
  - eventually you might use lein to manage your own projects, at which point it will get packages you need in addition to what it needs to bootstrap
  
So, during bootstrapping, you will likely see it doing this sort of stuff.  Relax, that's what it's supposed to do.

For Mac people, you can install this with `brew` on a mac.

Linux distros may also have Leiningen included in it's package manager.  If so, that is a very easy place to get it as well.

Failing that visit the following site [Leiningen](http://leiningen.org/). Follow the instruction on the page and it is super easy!

So whether via brew, package manager, shell script or .bat file from the leiningen site you are properly setup with with leiningen if you can do the following from the command line:

```
▶ lein version
Leiningen 2.7.0 on Java 1.8.0_102 OpenJDK 64-Bit Server VM
```
or something similar

At this point you are now Clojure Enabled.


Next [Chapter 2: Basics](/Pages/2-basics.md)
