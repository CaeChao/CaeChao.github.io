
## Introduction
What is asynchronous programming?
Well, imagine you are sitting in front of your computer screen, putting your hands on your beloved keyboard and doing your coding and thinking, and suddenly you realize that there is a piece of necessary information you need to ask your friend to finish your work, so you grab your phone nearby and send an asking message to your friend, and this might take him some time to respond.

Now there are different ways for you to proceed:
* You stay on your phone and wait for his respond (**Synchronous**)
* You tell your friend to send back that important information once he has that information. Meanwhile you can continue what tasks are you focusing on. (**Asynchronous**)

The same principle applies to Computer:
The central part of the computer is the processor, which just like your brain, but it works all the time. 
The programs are nothing but tasks that will keep the processor busy. But many programs not just interact with processors, they may also communicate with other computers over a network. (like sending messages to your friends)
So when a single program need to make progress while it is waiting for some data back from a network request, asynchronous behavior happens.

## How does it work in JavaScript?
There are some basic components in JavaScript runtime to construct the JavaScript concurrency model.

### Stack and Execution context
Because JavaScript is a Single-threaded language, JavaScript has a single execution stack which execute synchronously
* Stacks are fundamental to function calls, each time a function is called it forms a new *stack frame*
* Each function call creates a new execution context, even a call to itself.
* 1 Global context as the base of stack
...

Because of these facts about stack, JavaScript will execute functions synchronously and a synchronous function blocks until it completes its operations, and behaves as LIFO (last in first out).

```JavaScript
...
taskA();
taskB();
taskC();

# The single thread process:
Task A --> Task B --> Task C
```
In JavaScript, the code inside taskA()(and taskB(), taskC()) is atomic, which means that once taskA() starts running, it cannot be interrupted and will finish before any code in taskB() can run.

### Another Worker
As I said before, JavaScript is generally a **single-threaded synchronous** language, so how can JavaScript performs **asynchronous actions**? Actually the asynchronous behavior is not part of the JavaScript language, rather they are built on host Environment like Browser or Nodejs, act like another worker or friend.
If you replace asynchronous actions with synchronous actions, the webpage will be unresponsive while the script is running, the user will not be able to do anything, because the UI is locked by that time delay task.
Here is how they work together:

```JavaScript
function taskA() {
    console.log('a');
    setTimeout(taskB, 500);
}
function taskB() {
    console.log('b');
}
function taskC() {
    console.log('c');
}
taskA();
taskC(); 

# JS Main Thread: Task A --> Task C
# Worker Thread: Task B

# output: a c b
```

Have you noticed the taskB inside setTimeout function above? That is an asynchronous **callback** function which is a function that performs a slow action. Just like a friend sends you a message back.

## Message Queue
JavaScript runtime uses a message queue to receive incoming events, which is a list of messages to be processed. Queue behaves as FIFO(First in First out)

## Event Loop
The event loop runs in the same thread your JavaScript code runs in

```JavaScript
const queue = [];
while(!queue.isEmpty()) {
    const event = queue.shift();
    event();
}
```

Event loop always waits for a message to arrive, and will pop message(event) from the message queue to stack after JS main thread finishes the tasks.
Here is how event loop and message queue work together:
![Js-concurrency-model](/images/javascript/js-concurrency-model.png)

Remember the setTimeout function above? Actually that function doesn't put that callback function taskB on the event loop queue. What it really does is set up a timer; when the timer expires, the host environment will place your callback into message queue. But if you have already had a lot of items inside your event loop at that moments, say like 20...then your new callback function must wait. That's why sometimes your setTimeout timers may not triggers your function punctually.

Because of this event loop model, it allows JavaScript runtime behaves concurrently and never blocking I/O operations

## Summary

So why asynchronous programming?
Most of time JavaScript is synchronous, blocking, single-threaded, in which only one operation can be in progress at a time.
But if we want to load some data from server side database like performing an Ajax querying and displaying the result, then it is better to push this off main JS thread and perform this task asynchronously.


So this is my introduction to asynchronous JavaScript Programming, I'll talk about some common problems involved in this model, and the evolution of asynchronous JavaSript from **Callback** to **Promise** to **async await** in the next article.


