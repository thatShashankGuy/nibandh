# NodeJS Design Patterns and Core Concepts 

## Async IO programming with Node js 

### Introduction 
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

## Callback and Related Design Patterns 

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

### Callback hell 
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
This could further be complicated if we define callback in arguments 
