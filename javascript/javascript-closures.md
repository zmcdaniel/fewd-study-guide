# Time for some Closures 

## What is a closure?

Put simply as closure is a function that exists within another function and has access to the encapsulating function's variables. Make sure you understand the idea of scoping beforehand!

Closures are any function that keeps reference to variables from its parent’s scope *even after the parent has returned*.

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

### Closures can refer to variables outside the current function

In the example below, `printLocation()` refers to the `state` variable and the `city` parameter of the enclosing/parent `setLocation()` function. As a result, when `setLocation()` is called, `printLocation()` uses the variables and parameters of its parent to print "You are in Seattle, Washington".

```javascript
function setLocation(city) {
    var state = "Washington";
    
    function printLocation() {
        return "You are in " + city + ", " + state;
    }
    
    return printLocation();
}

console.log(setLocation("Seattle")); // "You are in Seattle, Washington"
```


### Closures have access to the outer function's variable*even after*the outer function returns 

The inner function still has access to the outer function’s variables even after the outer function has returned. 

Yep, you read that correctly. 

When functions in JavaScript execute, they use the same scope chain that was in effect when they were created. This means that even after the outer function has returned, the inner function still has access to the outer function’s variables. 

Therefore, you can call the inner function later in your program. 

```javascript
function setNewLocation(city){
    var state = "Washington"
    
     // this inner function has access to the outer function's variables, including the parameter​
    function printLocation() {
        return "You are in " + city + ", " + state;
    }
    
    return printLocation;
};

// At this juncture, the setLocation outer function has returned

var myNewLocation = setNewLocation("Seattle");

// The closure (printLocation) is called here after the outer function has returned above​
// Yet, the closure still has access to the outer function's variables and parameter​

myNewLocation(); // "You are in Seattle, Washington"
```

You see, `printLocation()` is *returned* inside the outer `setLocation()` function instead of being immediately called. The value of `currentLocation` is in the inner `printLocation()` function.

The inner `printLocation()` gets executed outside of its lexical scope and still "remembers" its variable (`state`) and parameter (`city`) even though `setLocation()` has completed.

Closures remember their surrounding scope (the outer function) even when it's executed outside its lexical scope! You can call it at any time later in your program!

### Closures store*references*to the outer function's variables

Not the actual value, but the reference! If the outer function's variable changes before the closure is called, things start to get really interesting

```javascript
function cityLocation() {
    var city = "Seattle";
    
    // Returning an object with inner functions
    // The inner functions are closures and have access to the outer vars
    return {
        getCity: function() {
            // This will return whatever the current value of city is.
            return city;
        },
        setCity: function(newCity) {
            // This can change the outer variable
            city = newCity;
        }
    }
}

var zoeCity = cityLocation(); // myId() is called

zoeCity.getCity(); // "Seattle"
zoeCity.setCity("Bainbridge");
zoeCity.getCity(); // "Bainbridge"
```

Closures can both read and update their stored variables. The updates made are also visible to the closures, as seen in the `getCity()` method.

One more time: closures store *references* to varibles, *not values*!

## Closures gone wild

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
            // instead of returning a function.
            } ()
            
        // immediately invoke the function passing the i variable as a parameter
        } (i);
    }

    return theCelebrities;
};

var actionCelebs = [{name:"Stallone", id:0}, {name:"Cruise", id:0}, {name:"Willis", id:0}];

var createIdForActionCelebs = celebrityIDCreator (actionCelebs);

var stalloneID = createIdForActionCelebs [0];
console.log(stalloneID.id); // 100​

var cruiseID = createIdForActionCelebs [1]; 
console.log(cruiseID.id); // 101
```

## Sources

* [Understranding Javascript Closures with Ease - Javascript is Sexy](http://javascriptissexy.com/understand-javascript-closures-with-ease/)
* [Javascript Closures - W3Schools](https://www.w3schools.com/js/js_function_closures.asp)
* [Demystifying Closures, Callbacks, and IIFEs - Sitepoint](https://www.sitepoint.com/demystifying-javascript-closures-callbacks-iifes/)