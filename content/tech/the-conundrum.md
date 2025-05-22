+++
date = '2025-05-22T21:43:16+05:30'
draft = false
title = 'The Conundrum: Sync Or Async'
+++
Suppose I migrate my code from being synchronous to asynchronous what does change ? Let's try to think through.

Synchronous code execution is very predictable and linear. If you make some network call, it will relax there till the data comes. On the contrary, asynchronous code is quite busy. The poor guy keeps on running in (event) loop juggling multiple tasks concurrently. If it makes any network call it will register an event for that and pause execution there. Meanwhile it (eventloop) will pick and process some other event which is ready. 

Now there are few important things to consider. 

What happens to the function that was paused ? Does it remain in the memory to be processed again (once the event completes) or is it removed from memory ? If it remains in memory, will this not result in more memory consumption as there will be lot of pending events to be processed and they will be held in memory. And suppose it is removed from memory how does it `resume` from the same location ? A function can't be invoked from in-between, right ? To complicate it, let's say a function had multiple `await`s then how would it know from which await to resume ? 

As you might know, the paused (or suspended) function (execution context to be precise) is held in memory. That's how it is able to resume quickly. Another important thing to note here is if there are long waiting concurrent tasks they can really clog memory. Or if there are functions that hold large data near await boundary they can also lead to memory issues quickly. 

---
NodeJs is non-blocking by default. So, if you make a network call it would automatically be pushed to event-loop and function execution would continue without getting blocked. Python, on the contrary is sync by default. So, if you make a network call it will be blocked there unless you explicitly use async libraries to make the network call. Ok, suppose you make a (network) db call to fetch some data. Your next code will obviously be dependent on this data so you made the call. So, you will have to `await` on this call. Python has a restriction that `await` can be called only inside a coroutine. So, the function where db call is being made should be made coroutine with `async` keyword. Now, where this function is being called that should also `await`. So, that function would also need to be coroutine. And the chain continues. Thus, the async-chain quickly escalates till source. All the way from source till last call it would become async. It is also fair. How would the interpreter know whether something is to be executed in event loops ? Also, the flagging with `async` keyword informs about non-blocking nature of function and reduces chances of unknowingly adding blocking functions inside it.

---
Let's take another scenario. Suppose an API was making 100 DB calls. In synchronous mode, all these calls would go one after the other. If we make the code asynchronous, it would make these 100 DB calls and sit for their responses. I guess DBs handle such things efficiently with connection pools. The point is what would have been a distributed load over a second would become a spike in few milliseconds. Systems should be robust to handle such cases in production. 

---
Suppose that you made a db call. It takes 10ms to process that call. Till the call is processed few more (say 100) ready events are added to event loop. Now, even if your data is received it will be processed after events in line are clear. Thus, even though the overall throughput of the system increased, it can happen that P99 response time has increased. If event loop has some priority management mechanism (like in WebFlux) then it can be controlled. 

---
Also, can it happen that events are lost ? Say, event handler throws an uncaught exception, the process will be terminated and all pending events might be lost. Such cases can obviously happen even in synchronous flow but the point is with increased concurrency the impact of such events can be larger if not handled properly. 

---
In the end, with the evolving tools the complexity and unpredictability of async code is reducing. In the bid to get that extra performance out of your application it would no longer be a conundrum. Async would be an obvious winner.