# Callbacks and how to use them

Functions in Javascript are first class citizens, or more accurately, *first class objects*. One of the results of this behavior is that functions can be passed as arguments to other functions. Also, functions can be returned by other functions. 

Functions are objects in Javascript. Other objects are String, Array, Number. If those can be passed in as arguments, then functions can be passed in too. This is called a **callback function**.

## Why use callbacks?

* They keep your code DRY
* Better abstraction so you can have generic functions that are versitile
* Better maintainability
* More readable code
* Specialize your functions

It's easy to write a function that turns into an `if`/`else` soup. Not to mention that function would be limited and not reusable. 

Callbacks appear all the time in modern web app development, libraries, and frameworks:

* Asynchronous execution like reading files and making HTTP requests
* Event listeners and handlers
* In setTimeout and setInterval methods
* General code conciseness

## What is a callback function?

Also called a higher-order function because of it's first-class status, it's a function passed into another function as a parameter. 

Here is a simple jQuery example. Notice how instead of a variable, we are passing a function into the `click()` method. That's a callback!
```javascript
$(#button).click(function() {
   console.log("Button clicked!"); 
});
```

Check out this one, utilizing the `forEach()` method. Again, we pass an anonymous (unnamed) function as a parameter into the function. 
```javascript
var countDown = [1, 2, 3, 4];

countDown.forEach(function(number) {
    console.log(number);
});
```

An additional built-in example, this time using the `setTimeout` method of the browser's `window` object.
```javascript
function showSecret(mySecret) {
    setTimeout(function(){
        console.log(mySecret)    
    }, 2000);
}

showSecret("I eat worms");
```

When we pass the callback function as an argument to another function, we are ONLY giving it the function definition. We are NOT exectuting the function in the parameter. It is not executed immediately, it is "called back" at some point in the containing function's body. Look at this for an example:
```javascript
function doSomeAddition(firstNum, secondNum, callbackFunction) {
    console.log("I want to add " + firstNum + " & " + secondNum);
    callbackFunction(firstNum, secondNum);
}

function addition (n1, n2) {
    console.log(n1 + n2);
}

doSomeAddition(1, 1, addition); 
// Output:
// "I want to add 1 & 1"
// 2
```

Check out how in `doSomeAddition()`, we are passing it a parameter called `callbackFunction`. This just contains the blueprint, or function definition for our callback. Nothing is executed just yet. Also, the same concept is illustrated in `doSomeAddtion(1, 1, addition);` Note that there are no parentheses -- because we do not want it to be executed right away.

Callback functions are in effect **closures**. All the same rules apply. They have access to the containing function's scope, their own variables, and the global scope's variables.

## Principles when implementing callbacks

### Use named OR anonymous functions as callbacks

The example `doSomeAddition()` above is using an existing named function, but like with the jQuery, `forEach`, and `setTimeout` examples, we can also use anonymous functions:

```javascript
function doSomeAddition(firstNum, secondNum, callbackFunction) {
    console.log("I want to add " + firstNum + " & " + secondNum);
    callbackFunction(firstNum, secondNum);
}

doSomeAddition(1, 1, function(n1, n2){ console.log(n1 + n2); });
```

### Pass parameters to callbacks

The callback function is just a normal function when it is executed in context, so we can pass it parameters just like usual. This includes function scope variables or properties or global scope variables.

Let's make a callback that uses both.

```javascript
var input = { "name":"John", "age":31, "city":"New York" };
var array = [];

function parseIt(data) {
    if (typeof data === "string") {
        console.log("String detected! Time to parse!");
        data = JSON.parse(data);
    } else if (typeof data === "object") {
        console.log("Object detected. Time to stringify!");
        data = JSON.stringify(data);
    }
    array.push(data);
}

function getInput(data, callbackFunction) {
    callbackFunction(data);
}

getInput(input, parseIt);
console.log(array);
```

In this example, we fed out callback function `parseIt()` the parameter `data`, which was then called as `parseIt(input)`.

### Make sure a callback is a function before executing it

It is always wise to check that the callback function passed in the parameter is indeed a function before calling it. Also, it is good practice to make the callback function optional.

Let’s refactor the `getInput` function from the previous example to ensure these checks are in place.

```javascript
function getInput(data, callbackFunction) {
    if (typeof callbackFunction === "function") {
        callbackFunction(data);
    }
}
```
Without the check in place, if we accidentally passed `getInput` something that wasn't a function, our code would result in a runtime error.

### Preserving the *this* context in callbacks

One of the unique and interesting parts about `this` in Javascript is how it is entirely contextual. When utilizing callbacks that use the `this` object, we have to watch out how we execute the callback or risk the `this` object pointing to the global `window` object in the browser or the object of the containing method.
 
```javascript
var character = {
    type: "Paladin",
    fullName: "Not Set",
    setFullName: function(name, title) {
        this.fullName = this.type + " " + name + " the " + title;
    }
}

function applyCharacterName(name, title, callbackFunction) {
    callbackFunction(name, title);
}

applyCharacterName("Horvald", "Strong", character.setFullName);

console.log(character.fullName); // "Not Set"
console.log(window.fullName); // "undefined Horvald the Strong"
```

Oh no! This is not what we wanted. `this.fullName` is setting the `fullName` property on the `window` object. The `window` object doesn't even know what to do with `type`.

#### Use the *call* or *apply* function to preserve *this*

Every function in Javascript has two methods: `.call()` and `.apply()`. 
* `call`:  The first parameter takes the value to be used as teh `this` object. The remaining arguments are passed in individually and seperated by commas.
* `apply`: The first parameter is also the value to be used as the `this` object. The last parameter is an array of values (or the *arguments* object) to pass into the function.

To fix the problem in our `applyCharacterName`, we will use `apply` and modify the code to look like this:

```javascript
var character = {
    type: "Paladin",
    fullName: "Not Set",
    setFullName: function(name, title) {
        this.fullName = this.type + " " + name + " the " + title;
    }
}

function applyCharacterName(name, title, callbackFunction, callbackObject) {
    callbackFunction.apply(callbackObject, [name, title]);
}

applyCharacterName("Horvald", "Strong", character.setFullName, character);

console.log(character.fullName); // "Paladin Horvald the Strong"
```
You could have used `call` here, but `apply` worked just as well.

### Use as many callbacks as you want

Feel free to chain together as many callbacks as you want. Here is a classic pseudocode example using jQuery's AJAX function.

```javascript
function successCallback() {
    // Do stuff if success message recieved
}

function completeCallback() {
    // Do stuff on completion
}

function errorCallback() {
    // Do stuff if an error is returned
}

$.ajax({
    url: "http://fiddle.jshell.net/favicon.png",
    success: successCallback,
    complete: completeCallback,
    error: errorCallback
});
```

## "Callback hell" and how to solve it

In asynchronous code execution, which is simply execution of code in any order, sometimes it is common to have numerous levels of callback functions to the extent that you have code that looks like the following. The messy code below is called callback hell because of the difficulty of following the code due to the many callbacks. I took this example from the node-mongodb-native, a MongoDB driver for Node.js.
```javascipt
var p_client = new Db('integration_tests_20', new Server("127.0.0.1", 27017, {}), {'pk':CustomPKFactory});
p_client.open(function(err, p_client) {
    p_client.dropDatabase(function(err, done) {
        p_client.createCollection('test_custom_key', function(err, collection) {
            collection.insert({'a':1}, function(err, docs) {
                collection.find({'_id':new ObjectID("aaaaaaaaaaaa")}, function(err, cursor) {
                    cursor.toArray(function(err, items) {
                        test.assertEquals(1, items.length);
​
                        // Let's close the db​
                        p_client.close();
                    });
                });
            });
        });
    });
});
```
Solutions for this:
1. Name your functions and declare them. Then, pass the name of the function as the callback instead of building out all these anonymous functions within the parameter of the main function.
2. Use *modularity*. Seperate your code into modules so you can export a section of code that does a specific, particular job. Import the module as needed. 