---
title: Destructors are a terrible place to implement join()
layout: post
date: 2017-09-19 09:10 PDT

---

Destructors are required to block the thread. They are not allowed to suspend execution (coroutines). They should not start new tasks without a `join()` to block. Exception unwinders are particularly sensitive to this requirement.

There is passionate desire to make async operations follow all the precepts of a _Regular_ value as currently expressed in C++. This has resulted in pressure to make destructors `join()` any pending async operations. 

## attempts

I have been involved in discussions on the stlabs and cpplang slacks where we try to bind cancellation and completion of an async producer to the destructor of a consumer.

The whole community has experienced the result of one attempt at this binding through the current implementation of the return value of `std::async()`.

I have tried to implement this binding as a part of my demo of the promises that can be built on the _Single_ Concepts.

All of these share one outcome. Each destructor becomes `join()`, adding serialization and blocking to the end of every async operation. This directly contravenes the very intent of adding support for non-blocking async to the language and libraries.

## synthesis

When I got bit by the destructor's unfitness to be used as a completion signal, in the _Single_ concepts implementation, I thought it was specific to that code. I let the issue sit in the back of my head for a couple of weeks and suddenly saw a pattern emerge from the variety of efforts to incorporate async operations into the C++ language and libraries that I mentioned above.

## conclusion 

My conclusion is that the cancellation and completion of an async operation is itself an asynchronous operation that happens before destruction. This is either represented as an explicit signal or perhaps as a new phase of scope exit where all coroutines in the scope are completed via co_await, inverse to declaration, prior to starting the unwind. This phase would also have to precede the unwind phase during an exception.

## be explicit 

I prefer the explicit signal. In nearly every app I have seen, there is a global scope that owns shutdown of a variety of ongoing operations. the pattern almost always involves adding the top-level operations into a bag that is then effectively joined before exiting main.

Even an explicit signal is usually sent implicitly. producers implement a contract that includes sending the signal when they complete. The bag is just to tell the few perpetual producers, that each app has, that it is time to stop and propagate the signal. This means that there is not a huge risk of forgetting to signal when it is time to stop.

## discuss

This is a stream-of-consciousness recording of this moment of synthesis. Hopefully it adds to the effort. My only goal is to improve the representation of async operations in C++.
