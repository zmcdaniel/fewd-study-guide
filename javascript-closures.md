# Time for some Closures 

## What is a closure?

Put simply as closure is a function that exists within another function and has access to the encapsulating function's variables. Make sure you understand the idea of scoping beforehand!

Closures have three scope chain:
1. It's own scope. (variables defined within its own curly brackets)
2. The outer function's scope (variables within the function surrounding it)
3. The global scope (variables available to the whole application)

It's easier to see than explain sometimes:

```javascript
var hey = "Hey ";

function sayFullGreeting() {
    var weather = "pretty nice out!";
    function alice() {
        name = "Alice ";
        return hey + name + "sure is " + weather;
    }
    return alice();
}

console.log(sayFullGreeting()); // "Hey Alice sure is pretty nice out!
```

Closures are used extensively in Node.js; they are workhorses in Node.js’ asynchronous, non-blocking architecture. Closures are also frequently used in jQuery and just about every piece of JavaScript code you read.

Here's a jQuery closure:

```javascript
$(function() {
    var selections = [];
    
    // The closure below has access to the 'selections' array
    $(".button-class").click(function() {
        // on button click, push the button-name property to the selections array
        selections.push(this.prop("button-name"));
    });
});
```

## Closure rules

### Closures have access to the outer function's variable*even after*the outer function returns 

The inner function still has access to the outer function’s variables even after the outer function has returned. 

Yep, you read that correctly. 

When functions in JavaScript execute, they use the same scope chain that was in effect when they were created. This means that even after the outer function has returned, the inner function still has access to the outer function’s variables. 

Therefore, you can call the inner function later in your program. 

```javascript
function showFormalGreeting(firstName){
    var greeting = "Good day ";
    
     // this inner function has access to the outer function's variables, including the parameter​
    function showLastName(lastName) {
        return greeting + firstName + " " + lastName;
    }
    
    return showLastName;
}

// At this juncture, the showFormalGreeting outer function has returned
var myName = showFormalGreeting("Zoe");


​// The closure (lastName) is called here after the outer function has returned above​
​// Yet, the closure still has access to the outer function's variables and parameter​
myName("McDaniel"); // "Good day Zoe McDaniel"
```

### Closures store*references*to the outer function's variables

Not the actual value, but the reference! If the outer function's variable changes before the closure is called, things start to get really interesting

```javascript
function myId() {
    var myId = 1;
    
    // Returning an object with inner functions
    // The inner functions are closures and have access to the outer vars
    return {
        getId: function() {
            // This will return whatever the current value of myId is.
            return myId;
        },
        setId: function(newId) {
            // This can change the outer variable
            myId = newId;
        }
    }
}

var zoeId = myId(); // myId() is called

zoeId.getId();  // 1
zoeId.setId(2); // Changes myId in outer function to 2
zoeId.getId();  // 2
```

### Closures gone wild

Because closures have access to the update values of the outer function's variables, strange things can happen when you use `for` loops in the outer function.

Check this out:

```javascript
function celebrityIDCreator(theCelebrities) {
    var i;
    var uniqueID = 100;
    for (i = 0; i < theCelebrities.length; i++) {
      theCelebrities[i]["id"] = function ()  {
        return uniqueID + i;
      }
    }
    
    return theCelebrities;
};
var actionCelebs = [
    {name:"Stallone", id:0}, 
    {name:"Cruise", id:0}, 
    {name:"Willis", id:0}
];
var createIdForActionCelebs = celebrityIDCreator(actionCelebs);
var stalloneID = createIdForActionCelebs [0];
console.log(stalloneID.id());
```
In the preceding example, by the time the anonymous functions are called, the value of `i` is 3 (the length of the array and then it increments). The number 3 was added to the `uniqueID` to create 103 for ALL the `celebritiesID`. So every position in the returned array get `id = 103`, instead of the intended 100, 101, 102.

The reason this happened was because, as we have discussed in the previous example, the closure (the anonymous function in this example) has access to the outer function’s variables by reference, not by value. So just as the previous example showed that we can access the updated variable with the closure, this example similarly accessed the `i` variable when it was changed, since the outer function runs the entire `for` loop and returns the last value of `i`, which is 103.

To fix this side effect (bug) in closures, you can use an **Immediately Invoked Function Expression (IIFE)**, such as the following:

```javascript
function celebrityIDCreator (theCelebrities) {
    var i;
    var uniqueID = 100;
    for (i = 0; i < theCelebrities.length; i++) {
        // the j parametric variable is the i passed in on invocation of this IIFE​
        theCelebrities[i]["id"] = function (j)  { 
            return function () {
                
                // each iteration of the for loop passes the current value of i 
                // into this IIFE and it saves the correct value to the array​
            
                return uniqueID + j;
            // BY adding () at the end of this function, we are executing it 
            // immediately and returning just the value of uniqueID + j, 
            // instead of returning a function.​    
            } ()
            
        // immediately invoke the function passing the i variable as a parameter​    
        } (i);
    }
​
    return theCelebrities;
}
​
​var actionCelebs = [{name:"Stallone", id:0}, {name:"Cruise", id:0}, {name:"Willis", id:0}];
​
​var createIdForActionCelebs = celebrityIDCreator (actionCelebs);
​
​var stalloneID = createIdForActionCelebs [0];
 console.log(stalloneID.id); // 100​
​
​var cruiseID = createIdForActionCelebs [1]; 
console.log(cruiseID.id); // 101
```

## Sources

* [Understranding Javascript Closures with Ease - Javascript is Sexy](http://javascriptissexy.com/understand-javascript-closures-with-ease/)
* [Javascript Closures - W3Schools](https://www.w3schools.com/js/js_function_closures.asp)