# Async IO programming with Node js 

## Introduction 
Asynchronous programming is a technique that enables your program to start a potentially long-running task and still be able to be responsive to other events while that task runs, rather than having to wait until that task has finished.

Consider Node/ Browser Architecture , When program starts it start a main thread on which wil run your javascript program . While one set of instructions ( code ) is running on this thread the thread is `busy`

Now say a set of instructions called  **Program1** reading a large file  from a disk .Another set of instruction called **Program2** need to execute another function . **Program2** cannot start till **Program1** is finished. Our code also need to run another **Program3** with the result set of first program . 

Instead of waiting for **Program1** to run **Program2** NodeJS runs **Program1** asyncronously and let **Program2** start execution while **Program1** finishes its program to start **Program3**.

![Alt text](/assets/nodeJSAdvance_img1.png)

Javascript have three techniques to execute programs asyncronously 

1. Callbacks functions - Original Manner Node used to handle async programs 
2. Promises - Newer more robust 
3. Async/Await - Newest and most readable.

We will discuss all three 

## Callbacks

A callback is a function passed as an argument to another function, which will be executed later when a specific event or condition occurs.

Here we are writing a function called `clac`
which provides square of a given number asyncronously Once the numbers are caculated , instead of returning number our async function executes callback function which then returnsan object with result We have defined our call back function as `callbackforcalc` and called it as `cb` in the arguments of `clac`

```javascript 



/**
 * @param {*} arg1 
 * @param {*} cb 
 */
function calc(number, cb) {
    let res = number * number
    cb(res);
}

function callbackforcalc(res){
    console.log({
        result : res
    })
}

//lets execute our function 
calc(56,callbackforcalc) // { result: 3136 }

```
Often times callback for a specific function have no reusablity so they can be directly 
defined as anonymous functions.
  
As resul of operation of first argument is passed to callback for result we can directly pass the full operation to the call back
 
``` Javascript
calc(56,()=>{
    let res = 56*56
    console.log({
        result : res
    })
})

```

### Error handling in callback
Async operation may pass or fail hence , we also have to handle error, lest handle erorr in ourcalc function for string values 

```Javascript 
calc("A", ()=>{
    res = "A" * "A";
    if(isNaN(res)){
        console.log({
            "error" : "invalid input"
        })
    }else{
        console.log({
            "error" : "invalid input"
        })
    }
})//{ error: 'invalid input' }

/**
 * We can also write it as 
 */

function calc(arg1, cb){
    const res = arg1 * arg1
    cb(res)
}   
function callbackforcalc(res) {
    if(isNaN(res)){
    console.log({error :"invalid input"})
    } else{
    console.log({
        result: res
    });
    }
}

calc("A",callbackforcalc)

```

#### Callback hell 
your call back can itself be a async process meaning it may also have callback . 
It can get out of pretty quick if your callbacks' callback also have its own callback. 
This is called `callback hell or pyramid of doom` as this makes your program hard to read and debug . 

```javascript
function operation1(data, callback) {
    setTimeout(() => {
        console.log("Operation 1 completed");
        callback(data + 10);
    }, 1000);
}

function operation2(data, callback) {
    setTimeout(() => {
        console.log("Operation 2 completed");
        callback(data * 2);
    }, 1000);
}

function operation3(data, callback) {
    setTimeout(() => {
        console.log("Operation 3 completed");
        callback(data - 5);
    }, 1000);
}

operation1(5, (result1) => {
    operation2(result1, (result2) => {
        operation3(result2, (result3) => {
            console.log("Final result:", result3);
        });
    });
});


```
This could further be complicated if we define callback in arguments .

#### Callback Design Patterns 

- **Sequential Callback Execution**
Task in callback will execute one after other. This is especially useful if output of first callback is input of next. Straightforward for small tasks or when operations need to be performed in a specific order.But can end up in callback hell due to nature of specific order. 

Consider a program where you read from a file asyncronously and find address of second file and start reading this. In this case you must only start reading second file after first operation yeilds result successfully. 

``` javascript
const fs = require('fs');

// Reading the first file
fs.readFile('/path/to/first/file', 'utf8', (err, data1) => {
    if (err) {
        console.error("Error reading the first file:", err);
        return;
    }
    console.log('First file read:', data1);

    const secondFilePath = data1

    // Reading the second file using the path from the first file
    fs.readFile(secondFilePath, 'utf8', (err, data2) => {
        if (err) {
            console.error("Error reading the second file:", err);
            return;
        }
        console.log('Second file read:', data2);

        // Further processing can be done here with data2
    });
});

```

- **Parallel Callback Execution**
If your next operation do not depend on output of last operation , to avoid deep nesting and callback hell you can execute callbacks parallely. 

```javascript 
const fs = require('fs');

// Function to read a file and invoke a callback
function readFile(path, callback) {
    fs.readFile(path, 'utf8', (err, data) => {
        if (err) {
            callback(err, null);
            return;
        }
        callback(null, data);
    });
}

// Reading multiple files in parallel
let count = 0;
const totalFiles = 2;
let data1, data2;

readFile('/path/to/first/file', (err, data) => {
    if (err) throw err;
    data1 = data;
    count++;
    if (count === totalFiles) {
        console.log('All files read');
        // Further processing with data1 and data2
    }
});

readFile('/path/to/second/file', (err, data) => {
    if (err) throw err;
    data2 = data;
    count++;
    if (count === totalFiles) {
        console.log('All files read');
        // Further processing
    }
});
```

While it is better to use parallel handling for more readable code and better performance, but it comes with its own challenges. Parallel execution is resource intensive and can consume more memory than sequential execution at a time. 

Another issue of parallel execution is if not limited, launching an excessive number of parallel operations that could overwhelm the system and cause memory leaks . 

Based on operation type and resulting executions, one of either can be choose. Though Node have evovled and callbacks having serious disadvantages have moved to better handling of async calls using `promises`. let discuss them. 

## Promises 

Callbacks is a messy way to handle async code . To deal with it , ES6 in general standarized promises async desgin pattern also called promise/A+ at that time and it is supported natively in Node since v4. Below is a messy callback hell and its promise counterpart. 

```javascript

//Messy callback

function fetchData(callback) {
    // Simulate an asynchronous operation like a database call
    setTimeout(() => {
        callback(null, 'Data fetched');
    }, 1000);
}

function processData(data, callback) {
    // Simulate data processing
    setTimeout(() => {
        callback(null, data + ' and processed');
    }, 1000);
}

// Nested callbacks (callback hell)
fetchData((error, data) => {
    if (error) {
        console.error('Error fetching data:', error);
    } else {
        processData(data, (error, processedData) => {
            if (error) {
                console.error('Error processing data:', error);
            } else {
                console.log('Processed Data:', processedData);
            }
        });
    }
});

//Cleaner Promise

fetchData()
    .then(processData)
    .then((processedData) => {
        console.log('Processed Data:', processedData);
    })
    .catch((error) => {
        console.error('Error:', error);
    });
```

#### Promisification 
Promises are built internally on idea of callbacks , and as result a callback can be converted to promise . This is called `Promisification of callback`

Lets promisify our fetchdata and process data callback to make what we saw as cleaner promise chaining. 

```javascript
function fetchData() {
    // Return a new Promise
    return new Promise((resolve, reject) => {
        setTimeout(() => {
            resolve('Data fetched');
        }, 1000);
    });
}

function processData(data) {
    return new Promise((resolve, reject) => {
        setTimeout(() => {
            resolve(data + ' and processed');
        }, 1000);
    });
}

// using both promises one after other is called promise chaining
fetchData()
    .then(processData)
    .then((processedData) => {
        console.log('Processed Data:', processedData);
    })
    .catch((error) => {
        console.error('Error:', error);
    });

```
### Fundamentals of Promise 
- In above code you create function and return a `new Promise` wrapping around the callback.
- A new Promise take two arguments `resolve` and `reject`. we `resolve`  or `fulfill`  a promise when program execution was successful and `reject` a promise if error occured. 
- This function can be uses as promise now with a `thenable` using .`then` execute the code after promise is `fulfilled ` or `rejected` .   
- The `catch` is special `then` which can be used instead of then when promise is not `resolved` but `rejected`  and returns error arg. It is similar in concept to `error first callbacks`

#### The Promise API 
The Promise api exposes several method to handle execution of several async operations either sequentially or parallely. I have mentioned the important ones below. 

- `Promise.resolve` is  used to create a Promise that is resolved with a given value. If the value is a Promise, it returns the Promise itself.
- `Promise.reject` is used to create a Promise that is rejected with a given reason.
- `Promise.all` is used when you have multiple asynchronous operations that you wish to run concurrently, and you need to wait for all of them to complete. It takes an array of Promises as an input.If all the Promises resolve, `Promise.all` resolves with an array of the results of the input Promises, in the same order. If any of the Promises are rejected, Promise.all immediately rejects with the reason of the first Promise that rejects.
- `Promise.allSettled` is similar to Promise.all except it waits for all promises ( both resloved and rejected). It also return array the results with resolved , rejected alike instead of rejecting on first reject.
- `Promise.race` is used when you have multiple asynchronous operations and you want to take action as soon as the first one resolves or rejects. It resolves or rejects as soon as the first Promise in the input array resolves or rejects, with the value or reason from that Promise.
- `Promise.any` is somewhat the opposite of `Promise.all `regarding how it handles rejections.t resolves as soon as any of the input Promises resolve, with the value of the first resolved Promise. If all input Promises reject, `Promise.any` rejects with an `AggregateError`.

Certainly! Here's a revised version of your article with corrected spelling mistakes and some suggested improvements:

### Promise Chaining

Promise chaining allows you to create a sequence of asynchronous operations in a more structured and readable manner compared to callback hell. Let's take an example from [javascript.info](https://javascript.info/promise-chaining):

```javascript
new Promise(function(resolve, reject) {
  setTimeout(() => resolve(1), 1000);
}).then(function(result) { // (**)
  alert(result);
  return result * 2;
}).then(function(result) {
  alert(result);
  return result * 2;
}).then(function(result) {
  alert(result); // 4
  return result * 2;
});
```

In this example, promises are chained together using `.then()`. Each `.then()` block handles the result of the previous promise and performs further operations. This approach improves code readability but can still become verbose and challenging to debug in more complex scenarios.

## Async/Await

Async/await is a syntactical sugar built on top of promises, enabling the writing of asynchronous code in a more synchronous style. An `async function` is a special type of function that can use the `await` keyword to pause execution until an asynchronous operation is complete, making code appear more linear. Here's how we can rewrite the previous example using async/await:

```javascript
async function exampleAsyncFunction() {
  try {
    const result1 = await new Promise((resolve, reject) => {
      setTimeout(() => resolve(1), 1000);
    });

    alert(result1);

    const result2 = result1 * 2;
    alert(result2);

    const result3 = result2 * 2;
    alert(result3);

    const result4 = result3 * 2;
    alert(result4);
  } catch (error) {
    console.error(error);
  }
}

exampleAsyncFunction();
```

In this version, we define an `async function`, and within it, we use `await` to wait for promises to resolve. This makes the code cleaner and easier to follow, especially for sequential execution patterns where the output of one function feeds into the next.

### Promises in Async/Await

It's worth noting that `await` is designed to work with promises. Without using `await`, you won't get the resolved value of the promise; instead, you'll receive an unresolved promise.

```javascript
// Using async/await without await
async function exampleAsyncFunction() {
  const a = somePromiseResolution(); // Returns an unresolved promise
  console.log(a);
}
```

#### Comparing Sequential Execution Patterns with Async /Await and Promises 

Async/await is particularly favorable for sequential execution patterns, where the output of one function serves as the input for the next. 

```javascript
// Sequential execution using async/await
async function multipleAsync(x) {
  const a = await func1(x);
  const b = await func2(a);
  const c = await func3(b);

  return c;
}

// Sequential execution using promises
multiplePromises(x)
  .then(a => func1(a))
  .then(b => func2(b))
  .then(c => func3(c))
  .then(result => {
    // Use the final result here
  })
  .catch(error => {
    console.error(error);
  });
```

In this comparison, we can see that async/await simplifies the code by allowing us to write asynchronous operations in a more sequential and synchronous looking manner.


Certainly, here's the revised explanation with the suggested corrections:

---

### Parallel Execution Patterns and the Need for Promises

Async/await is the most recent and recommended way to write asynchronous operations today, but that doesn't mean we won't encounter promises in our codebases. While `async/await` excels in sequential execution, `Promise APIs` are preferred (along with `async/await`) for parallel execution where you don't need to pause the execution of each async call.

Let's first try running multiple executions of an async function using just `async/await`:

```javascript
async function fetchDataFromAPI(apiName, delay) {
  return new Promise((resolve) => {
    setTimeout(() => {
      resolve(`Data from ${apiName}`);
    }, delay);
  });
}

async function execute() {
  const apiRequests = [
    { name: "API 1", delay: 2000 },
    { name: "API 2", delay: 1500 },
    { name: "API 3", delay: 1000 },
  ];

  const results = [];

  for (const request of apiRequests) {
    try {
      const data = await fetchDataFromAPI(request.name, request.delay);
      results.push(data);
    } catch (error) {
      console.error(`An error occurred for ${request.name}:`, error);
    }
  }

  for (const data of results) {
    console.log(data);
  }
}

```

In the above code, we run multiple executions of an async function. Note that none of the instances are dependent on the output from any of the previous API calls. We can run multiple async calls using a `for...of` loop.

However, there is an obvious challenge here. Even though all executions are independent and can be triggered in parallel, `async/await` is designed to pause the execution of the current function until it is resolved. Hence, the next function must wait in the queue of the `for...of` loop, and if any execution fails, the rest of the execution will only run if the rejection is handled.

### Anti-Pattern: `forEach`

Before we see the resolution to this issue, let's discuss a common anti-pattern when iterating through async functions in JavaScript, which is using a `for` loop or `forEach` to execute async code.

When you use `forEach` with asynchronous code, it can lead to unexpected behavior and may not work as intended. This is because `forEach` is designed for synchronous operations and does not wait for asynchronous tasks to complete.

```javascript
const asyncTasks = [1, 2, 3];

asyncTasks.forEach(async (task) => {
  try {
    const result = await someAsyncFunction(task);
    console.log(`Task ${task} completed with result: ${result}`);
  } catch (error) {
    console.error(`Error in task ${task}:`, error);
  }
});

console.log('All tasks started...');
```

In this example, even though you're using `await` within the `forEach` callback, it won't work as expected. The `console.log('All tasks started...')` line will execute before any of the asynchronous tasks have completed because `forEach` doesn't wait for them.

### Resolving Parallel Execution Issues with `Promise.all`

The challenge of using a `for...of` loop and sequential execution of independent calls of asynchronous code can be addressed using the async/await syntax along with `Promise.all()`. `Promise.all()` allows you to run multiple asynchronous tasks concurrently and wait for all of them to complete.

Let's use `Promise.all()` to execute the previous example.

```javascript
async function execute() {
  const apiRequests = [
    { name: "API 1", delay: 2000 },
    { name: "API 2", delay: 1500 },
    { name: "API 3", delay: 1000 },
  ];

  try {
    // Use Promise.all to run all API requests concurrently
    const results = await Promise.all(
      apiRequests.map((request) => fetchDataFromAPI(request.name, request.delay))
    );

    for (const data of results) {
      console.log(data);
    }

    // You can continue working with the data here
  } catch (error) {
    console.error("An error occurred:", error);
  }
}

execute();
```

This approach is more efficient and concise compared to the `for...of` loop when you want to run multiple asynchronous tasks concurrently.

`Promise.all()` allows all asynchronous tasks to run in parallel, whereas a `for...of` loop processes them sequentially. When dealing with multiple independent asynchronous operations, parallel execution can significantly improve performance.

## Conclusion

This concludes my attempt at a comprehensive guide on asynchronous programming in JavaScript and Node.js. We learned about callbacks and the event loop, which are the building blocks of asynchronous programming in Node.js. We saw how we have evolved using promises and async/await.

Even though today `async/await` along with promise APIs are the go-to ways to handle async code, understanding callbacks is very important, as both newer methods are essentially abstractions over original callbacks. We also discussed a few anti-patterns like "callback hell," "promise chaining," and "iterating over async functions with `forEach`."

Async programming is key to building large-scale apps in JavaScript and Node.js, and it's precisely what allows Node.js to be fast and non-blocking, even though it runs on a single thread. Understanding these concepts gives us a deep understanding of the inner workings of Node.js and makes us better Node.js programmers.