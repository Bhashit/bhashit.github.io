---
layout: post
title: Understanding Concurrency Models
date: 2014-11-08
comments: true
---


## The free lunch is over

Do you ever get the questions that why we don't have 10 GHz CPU yet?
It should be available considering that the current highest available
clock speeds of 3.x GHz started appearing around 2003. More than 10
years ago. Even with the transistors getting smaller and smaller,
increasing clock speeds beyond a limit hasn't really happened for a
few basic reasons.

1. Too much heat, which is hard to dissipate
2. Too high power consumption

For general-purpose processors, much of the motivation for multi-core
processors comes from greatly diminished gains in processor
performance from increasing the operating frequency. This is due to
three primary factors:

1. The memory wall; the increasing gap between processor and memory
   speeds. This, in effect, pushes for cache sizes to be larger in
   order to mask the latency of memory. This helps only to the extent
   that memory bandwidth is not the bottleneck in performance.

2. The ILP wall; the increasing difficulty of finding enough
   parallelism in a single instruction stream to keep a
   high-performance single-core processor busy. 

3. [The power wall][power-wall]; the trend of consuming exponentially increasing
   power with each factorial increase of operating frequency. This
   increase can be mitigated by "shrinking" the processor by using
   smaller traces for the same logic. The power wall poses
   manufacturing, system design and deployment problems that have not
   been justified in the face of the diminished gains in performance
   due to the memory wall and ILP wall. The amount of power directly
   corresponds to the [amount of heat generated][cpu-heating] and dissipating that
   heat is getting prohibitively expensive.


More comprehensive coverage at. Really great stuff:

1. [The future of computers - Part 1: Multicore and the Memory Wall][future-of-computers-1]
2. [Future of computers - Part 2: The Power Wall][future-of-computers-2]
3. [Future of computing - Part 3: The ILP Wall and pipelines][future-of-computers-3]


## Model 1: Threads and Locking

First of all, if anyone asks you to use the java's Thread class
directly for any new code, slap them, hard, to bring them back to,
erm... the present. Basically, several Thread related methods have
been deprecated back in, wait for it... 2005. Yeah. And I have heard
of a few examples of people asking to write threaded code using
methods like [Thread.suspend()][thread-suspend] etc in
interviews. Don't take that job.


If you need some very basic concurrency, your language will provide
several primitives that you can use. The most common is of course the
threads and locks model. However, concurrency with the threads and
locks model is hellishly difficult to get right. Here is a quote from
Bruce Eckel, the author of [Thinking in Java][thinking-in-java]:

> And most important: you can never let yourself become too confident
> about your programming abilities when it comes to shared-memory
> concurrency. I would not be surprised if, sometime in the future,
> someone comes up with a proof to show that shared-memory concurrency
> programming is only possible in theory, but not in practice. It's
> the position I've adopted.

See example: Thread-Locks/Counting-Broken and Thread-Locks/Memory Visibility Puzzle

Regarding Memory Visibility:

1. The compiler is allowed to statically optimize your code by
   reordering things.
2. The JVM is allowed to dynamically optimize your code by reordering
   things.
3. The hardware you're running on is allowed to optimize performance
   by reordering things.

In the example that shows memory visibility, see the commented out
section in the second thread. If we write something like that,
something that looks completely innocent, it's theoretically possible
for it to be stuck forever.

### Memory Models and Visibility

Has anyone heard of memory models in programming languages? In Java,
the [memory model][memory model] didn't always exist. It became part
of the official spec as a part of [JSR 133][jsr-133] in 2004. However,
it was important only because of the memory visibility issue with
multiple threads. Several other languages still do not have
well-defined memory models. In C++, a well-defined memory model became
part of the standard with C++ 11 in 2011.

Take a look at [this article][happens-before] to understand the
various guarantees provided by the java memory model. You need this
kind of guarantees from the memory model of your language to get
concurrency right. All the relevant classes in the
[java.util.concurrent] package specify the memory consistency effects
explicitly. For ex. [ConcurrentLinkedQueue][doc-example]

### Deadlocks

The memory visibility problem can be solved with careful use of locks
and synchronization (or platform specific APIs in some other languages
if the memory model is not well defined).

However, as soon as you have multiple locks, the issue of deadlocks
arises.

Not going to explain this in too much detail. Simple example: the
dining philosophers problem. Deadlock happens if all the philosophers
pick up their left hand fork simultaneously.

Deadlock is a danger whenever a thread tries to hold more than one
lock.

#### How can deadlocks be avoided:

1. Synchronize all access to shared variables. Both the writing and
   the reading threads need to use synchronization.
2. Acquire multiple locks in a fixed, global order.
3. Don't call alien methods while holding a lock. They might acquire a
   lock without you knowing about it.
4. Hold locks for the shortest possible amount of time.
5. Lock timeouts


### Spurious Wake-ups

Spurious wake ups have been a part of concurrency API since the posix
thread days. They still exist. Here is a small explanation. 

> The pthread_cond_wait() function in Linux is implemented using the
> futex system call. Each blocking system call on Linux returns
> abruptly with EINTR when the process receives a
> signal. ... pthread_cond_wait() can't restart the waiting because it
> may miss a real wakeup in the little time it was outside the futex
> system call. This race condition can only be avoided by the caller
> checking for an invariant. A POSIX signal will therefore generate a
> spurious wakeup.

The [java documentation][https://docs.oracle.com/javase/8/docs/api/java/lang/Object.html#wait-long-]
on spurious wake-ups. The gist of which is:

> A thread can also wake up without being notified, interrupted, or
> timing out, a so-called spurious wakeup. While this will rarely
> occur in practice, applications must guard against it by testing for
> the condition that should have caused the thread to be awakened, and
> continuing to wait if the condition is not satisfied.


**The Point is, you need to be very careful if you are writing any
  non-trivial concurrent code**



## Concurrency with Functional Programming

Two main concepts:

1. Immutability
2. Pure Functions

The problems with mutability:

1. Hidden mutable state. "Immutability/DateFormatBug"
2. Escaped state: "Immutability/EscapingState"

There are several other. The whole problem of synchronization comes
from having to share mutable state between multiple threads.

Executions can be partitioned and trivially parallelized. Java
parallel streams. Similar facilities already available in other languages.


## Model 2: Functional Reactive Programming

### The old Java Futures

Java Futures are straightforward to use for single level of
asynchronous execution. For ex. See "OldFutures/FutureA.java"

However, real life scenarios are always more complicated than
that. When the Futures are nested, they add non-trivial complexity
with harder-to-read code. See "OldFutures/FutureB.java"

Callbacks solve the issue somewhat, but they have their own
problems. That problems is sometimes called Callback-Hell, and it is a
very common issue in non-trivial javascript applications especially on
the server-side with nods.js where the entire architecture is based on
callbacks. See "OldFutures/Callbacks.java". The results are
synchronized using countdown-latches. This works, but is too
complex. Any more levels of asynchronicity and things will begin to
become unwieldy.

### The Composable, Reactive Version

Available in Java 8. See "NewWay/ComposableFutures"

These kind of facilities have been available in other languages and
frameworks like Akka for quite some time.

### Something even better: FRP

#### What are Observables in Rx

Observable are asynchronous/push mirror-image of iterables.

1. [Observables as duals of Iterables](https://github.com/ReactiveX/RxJava/wiki#observables-are-flexible)
2. [Sample of how the code with Observable looks](https://github.com/ReactiveX/RxJava/wiki#reactive-programming)

With observables, the values are pushed to the consumer by the
producer. This is very flexible because values may arrive
synchronously or asynchronously.

The Observable type adds two missing semantics to the Gang of Four's
Observer pattern, to match those that are available in the Iterable
type: `onCompleted()` and `onError()`

With these additions, the only difference between Iterable and
Observable is the direction in which the data flows. This is very
important because now any operation you can perform on an Iterable,
you can also perform on an Observable.

We sometimes call this approach Functional Reactive Programming
because it applies functions (lambdas/closures) in a reactive
(asynchronous/push) manner to asynchronous sequences of data.

#### FRP, because Hacking should be fun

!["Programming Motherfucker, do you speak it?"][progmo]

FRP is programming with asynchronous data streams.

**And everything is a stream**

Anything can be a data-stream. Arrays, database rows, twitter feeds,
network streams, click events, log entries, satellite imagery,
facebook updates, your heartbeats, every frickin' thing; even the
single element entities.  You can listen to those streams and react
accordingly.

On top of that, you are given the entire toolbox of functions to
combine, process, create and filter any of those streams. That's where
the "functional" magic kicks in. A stream can be used as an input to
another one. Even multiple streams can be used as inputs to another
stream. You can merge two streams. You can filter a stream to get
another one that has only those events you are interested in. You can
map data values from one stream to another new one.

To show the real power of FRP, let's just say that you want to have a
stream of "double click" events in JS. To make it even more
interesting, let's say we want the new stream to consider triple
clicks as double clicks, or in general, multiple clicks (two or
more). Take a deep breath and contemplate how you would do that in a
traditional imperative manner. How bad would that look, with timers
and stuff?

With FRP, it's just [4 lines of code](http://jsfiddle.net/staltz/4gGgs/27/). 
Example taken from [this place](https://gist.github.com/staltz/868e7e9bc2a7b8c1f754)

!["What did I just read"][what-did-i-read]

Here is [another JS example](http://jsfiddle.net/staltz/8jFJH/48/)
from that same site.


## Model 3: Actors

Actors are kind of like objects, with their own internal state and
behavior. They communicate with each other via message-passing. But
unlike normal objects, actors run concurrently, and they actually pass
messages instead of calling methods.

[Elixir Documentation](http://elixir-lang.org/docs/stable/elixir/)

### In Elixir, actors are called processes.

Elixir processes are not the normal OS processes; rather, they are
very lightweight. Even lighter than most thread implementations. It is
pretty common for erlang/elixir programs to have thousands of such
running processes without running out of memory.

Simple example of an actor: "Actors/actors-basic.exs"

### Messages are queued in a per-actor mail-box

Actors are decoupled from each other and never block while sending
messages. Messages in the mail-box are always processed sequentially.

### Infinite recursion is handled via tail-call optimization.

Actors generally involve an infinite loop that calls itself at the end
of the function body. But we don't get stack-overflow because of TCO.

Also, actors maintain state by using recursion.

For ex. See "Actors/named-actor-counter.ex"

### Understanding links for fault-tolerance

For ex. see "Actors/links.ex". Load that file using `iex links.ex`

and then, execute the following commands.

{% highlight elixir %}

# Spawn one actor
pid1 = spawn(&LinkTest.loop/0)

# Spawn second actor
pid1 = spawn(&LinkTest.loop/0)

# Link pid1 to pid2
send(pid1, {:link_to, pid2})

# terminate pid2 to see what happens
send(pid2, {:exit_because, :bad_thing_happened})

## Even though we have linked the processes, the exit signal is not
## trapped by pid1. However, if we query the status, we can see that
## both are terminated
Process.info(pid2, :status)
Process.info(pid1, :status)

{% endhighlight %}

## Some good reading

1. [The java memory model FAQ][memory-model]: Explains the concepts
   really well. Applicable to other languages.
2. [Memory Consistency Properies (happens-before)][happens-before]


[cpu-heating]: http://research.ac.upc.edu/HPCseminar/SEM9900/Pollack1.pdf
[power-wall]: http://cs.nyu.edu/courses/spring12/CSCI-GA.3033-012/lecture12.pdf
[future-of-computers-1]: http://www.edn.com/design/systems-design/4368705/The-future-of-computers--Part-1-Multicore-and-the-Memory-Wall
[future-of-computers-2]: http://www.edn.com/design/systems-design/4368858/Future-of-computers--Part-2-The-Power-Wall
[future-of-computers-3]: http://www.edn.com/design/systems-design/4368983/Future-of-computing--Part-3-The-ILP-Wall-and-pipelines
[thread-suspend]: https://docs.oracle.com/javase/8/docs/api/java/lang/Thread.html#suspend
[thinking-in-java]: http://www.amazon.com/Thinking-Java-4th-Bruce-Eckel/dp/0131872486/
[jsr-133]: https://jcp.org/en/jsr/detail?id=133
[memory-model]: http://www.cs.umd.edu/~pugh/java/memoryModel/jsr-133-faq.html
[happens-before]: https://docs.oracle.com/javase/8/docs/api/java/util/concurrent/package-summary.html#MemoryVisibility
[doc-example]: https://docs.oracle.com/javase/8/docs/api/java/util/concurrent/ConcurrentLinkedQueue.html
[progmo]: /img/Programming-Motherfucker.png "Programming Motherfucker, do you speak it?"
[what-did-i-read]: /img/what-did-i-read.gif "What did I just read"
