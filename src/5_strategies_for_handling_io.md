# Strategies for handling I/O

Before we dive into Writing some code we'll finish off this part of the book talking a bit about different strategies of handling I/O and concurrency. Now, just note that I'm covering I/O in general here, but I use network communication as the main example. Different strategies can have different strengths depending on what type of I/O we're talking about.

## 1. Using OS threads

Now one way of accomplishing this is letting the OS take care of everything for us. We do this by simply spawning a new OS thread for each task we want to accomplish and write code like we normally would.

**Pros:**

- Simple
- Easy to code
- Reasonably performant
- You get parallelism for free

**Cons:**

- OS level threads come with a rather large stack. If you have many tasks waiting simultaneously (like you would in a web-server under heavy load) you'll run out of memory pretty soon.
- There are a lot of syscalls involved. This can be pretty costly when the number of tasks is high.
- The OS has many things it needs to handle. It might not switch back to your thread as fast as you'd wish
- The OS doesn't know which tasks to prioritize, and you might want to give some tasks a higher priority than others.


## 2. Green threads

Another common way of handling this is green threads. Languages like Go uses this to great success. In many ways this is similar to what the OS does but the runtime can be better adjusted and suited to your specific needs.

**Pros:**

- Simple to use for the user. The code will look like it does when using OS threads
- Reasonably performant
- Abundant memory usage is less of a problem
- You are in full control over how threads are scheduled and if you want you can prioritize them differently.

**Cons:**

- You need a runtime, and by having that you are duplicating part of the work the OS already does. The runtime will have a cost which in some cases can be substantial.
- Can be difficult to implement in a flexible way to handle a wide variety of tasks


## 3. Poll based event loops supported by the OS

The third way we're covering today is the one that most closely matches an ideal solution. In this solution the we register an interest in an event, and then let the OS tell us when it's ready. 

The way this works is that we tell the OS that we're interested in knowing when data is arriving for us on the network card. The network card issues an interrupt when something has happened in which the driver let's the OS know that the data is ready. 

Now, we still need a way to "suspend" many tasks while waiting, and this is where Nodes "runtime" or Rusts Futures come in to play.

**Pros:**

- Close to optimal resource utilization
- It's very efficient
- Gives us the maximum amount of flexibility to decide how to handle the events that occurs

**Cons:**

- Different operating systems have different ways of handle these kind of queues. Some of them are difficult to reconcile with each other. Some operating systems has limitations on what I/O operations support this method.
- Great flexibility comes with a good deal of complexity
- Difficult to write an abstraction layer that accounts for the differences between the operating systems without introducing unwanted costs, and at the same time provide a ergonomic API.
- Only solves part of the problem, the programmer still needs a strategy for suspending tasks that are waiting.


## Final note

The Node runtime uses a combination of both 1 and 3, but tries to force all I/O to use alternative 3. This is also part of the reason why Node is so good at handle many connections concurrently. Node uses a callback based approach to suspend
tasks.

Rusts async story is modeled around option 3, and some of the reason it has taken a long time is related to the _cons_ of this method and choosing a way to model how tasks should be suspended. Rusts Futures model a task as a [State Machine](https://en.wikipedia.org/wiki/Finite-state_machine) where a suspension point represents a `state`.
