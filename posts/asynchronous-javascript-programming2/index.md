
In the last article, we introduced what is asynchronous programming and the concurrency model in JavaScript runtime. In this article, we will talk about the evolution of asynchronous JavaScript: from **async callback** to **Promise**, **async await**

## Asynchronous Callback
The earliest solution to asynchronous programming might be **asynchronous callback**(like setTimeout) which is a function that's treated as a parameter when calling another **higher order function** which will start to execute in the background. When that higher order function finishes running, it calls the callback function to let you know the job is done. But sometimes it has some problems.

### Callback hell
When you have a series of tasks where each step depends on the results of the previous step. If it is synchronous code, then it will be pretty straightforward:

```JavaScript
var text = readFile(fileName),
    tokens = tokenize(text),
    parseTree = parse(tokens),
    optimizedTree = optimize(parse),
    output = evaluate(optimizedTree),
console.log(output);
```

But if you try to do these tasks in asynchronous code, which happens very often in Front-end with lots of Ajax requests and end-up looking like this:

```JavaScript
readFile(fileName, (text) => {
  tokenize(text, (tokens) => {
    parse(tokens, (parseTree) => {
      optimize(parseTree, (optimizedTree) => {
        evaluate(optimizedTree, (output) => {
          console.log(output);
        });
      });
    });
  });
});
```

It's called **callback hell**, this may drive most people's brain crazy just by a quick glimpse. Having a project end up with hundreds of similar code blocks will make it extremely hard to maintain the code even for the person who wrote it, and the **DRY** principle apparently has no value here as well.

That's where **Promise** come in.

## Promise
**Promise** is a new modern style of solution to async programming. Simply put, a Promise is a **container** which stores an event(an asynchronous action) that may complete at some point in the future and produce a value, and that future value indicate a **success** or **failure**. You can imagine you order some food in a restaurant, and you'll either get the food eventually(success), or you'll get the bad news of food shortage(failure).

When you initiate a **Promise**, you can use **.then** to pass two different callback handlers to **fulfillment**(success) and **rejection**(failure).

```JavaScript
promise.then(function(value) {
  // success
}, function(error) {
  // failure
});
```

or you can use `Promise.prototype.catch(onRejected)` to deals with rejected error cases, which actually behaves the same as calling `Promise.prototype.then(undefined, onRejected)`

Here are some features of **Promise**
* A Promise may be in one of 3 possible states: **fulfilled**, **rejected**, or **pending**, 
* Promise doesn't expose states outside, instead, you're expected to treat the promise as a black box
* Because **.then()** will always return a new **Promise**, you can chain multiple async operations together using multiple **.then()** operations, passing the result of one into the next one as an input. 
* Once a **Promise** is **resolved**, it becomes an immutable value, and it will trigger the callback function binding with **.then()**. Therefore, it's safe to pass that result because of the fact that it cannot be modified. 
* It will start doing whatever task you give it as soon as the promise constructor is initiated immediately.

Why promise? As mentioned previously, if you have a series of asynchronous tasks where each step depends on the results of previous step, which often easily ends up with **callback hell**. If we use **Promise**, then we can synchronizing these asynchronous tasks, rather than writing nested callback functions. Even though this method does not remove the use of callbacks, it makes the chaining functions straightforward, making it much easier to read and write. 

Another good reason to use **Promise** is that **Promise** can be see as a **trustable** and **repeatable mechanism** for encapsulating and composing **future values**, meanwhile, you don't need to worry about the time aspect of it.

If we refactor our callback hell codes above by using **Promise**:

```JavaScript
readFile(fileName)
  .then((text) => {
    return tokenize(text);
    })
  .then((tokens) => {
    return parse(tokens);
    })
  .then((parseTree) => {
    return optimize(parseTree);
    })
  .then((optimizedTree) => {
    return evaluate(optimizedTree);
    })
  .then((output) => {
    console.log(output);
    });
```

Doesn't it look much better to you as an sequence?

## Generator
However, we still have some drawbacks to use callbacks:
* Async Callback doesn’t fit how our brain plans out steps of a task.
* Callback is not trustable or composable because of inversion of control.

If we want to express async flow in a sequential, synchronous-looking style, we need some other ways. That' where **Generator** comes in.
**Generator** functions are JavaScript functions that may yield multiple values before finally returning.
But we won't discuss much about **Generator** here, because we have a more modern solution: Async/Await.

## Async/Await
**Async/Await** is basically a **Syntactic Sugar** for **Generator** function
If we refactor our html processing codes above to Async/Await, then it turns out like this:

```JavaScript
const asyncProcessor = async function(fileName) {
  const text = await readFile(fileName);
  const tokens = await tokenize(text);
  const parseTree = await parse(tokens);
  const optimzedTree = await optimize(parseTree);
  const output = await evaluate(optimzedTree);
  console.log(output);
}
```

You can see **async function** as a **Promise** with multiple asynchronous operations, and see **await** as a syntactic
sugar for **.then()**. In fact, you can also chain **.then()** right from the end of **async** function:

```JavaScript
asyncProcessor('./filename.html').then((result) => {
  console.log(result)
}
```
And sometimes it's also a good practice to put a data process function into the Promise.then after it resolved and you can get the data from the asynchronous data query, otherwise, sometimes the web page will render the UI components or execute other UI actions before getting the real data that should be rendered because of the non-block event driven programming model in JavaScript.   

So now you can write async codes as if it were synchronous, but without blocking the main thread.

## Single threaded Brain
So why the async codes evolve this way? Why we have figured out all kinds of solutions to write asynchronous codes, but in the end it seems that we come back to the original style like a roundabout.

I think a good answer to this is:
Humans are not as good at **multitasking** as we think we are. That's may just not really how our brains appear to be set up. We'd like to plan things in a synchronous order, rather then let the tasks to hang randomly and pop up like async events.

