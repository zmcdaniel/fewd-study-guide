# Scope and Hoisting: Not just for pirates

## What is a variable scope?

Simply put, the variable's scope is the context in which it exists. It just specifies whether or not you can access that variable. You can use it in the same way that you might use `private` or `public` in other languages like Java!

We have two levels of scope: **local** and **global**.

## Local scope, a.k.a function-level scope

Let's look at the following code snippet. We had two variables both named `variable`, but the one wrapped in the function that said `"I'm local!"` was only accessible by that function or by functions within that function.

```javascript
var variable = "I'm global!";

function showLocalVariable () {
    var variable = "I'm local!";
    return variable;
}

console.log(variable); // "I'm global!"
console.log(showLocalVariable()); // "I'm local!"
```

### Always declare local variables

Make sure you **always declare your local variables**. If you do not want them to change the global variable, make sure you have a `var` in front of them when initializing or otherwise. Or else they will bleed through and pollute the global scope. Here's an example:
```javascript
var name = "Gandalf";

function showWizardName() {
    return name;
}

function showRegularName() {
    name = "Steve";
    return name;
}

console.log("Presenting the great wizard", showWizardName()); // name = Gandalf
console.log("Presenting the normal dude", showRegularName()); // uh oh, name = Steve now
console.log("Once again, by popular demand, the great wizard", showWizardName()); // name = Steve
```

### Local variables win over global variables

Another thing to keep in mind with local and global variables is that if you declare one local variable and one global variable both with the same names, **the local/function-level variable will take precidence over the global variable in a function**. In the code example below, `showN()` will return `2`, not `1`, because the `n` variable declared inside the function takes priority over the global version of `n`
```javascript
var n = 1;

function showN(){
    var n = 2;
    return n;
}

console.log(showN()); // 2

```

## Global scope

A global-scope variable is **any variable declared outside a function**. In the browser, the global scope/context is the HTML document or window object.

Anything declared outside of a function is available to the whole application to access, modify, and use. Watch out for this as you build larger applications -- having lots of variable names floating around can be dangerous and bad for organization.

If a variable is initialized without being declared with a `var`, it is automatically added to the global context and this becomes a global variable:
```javascript
function showNum(){
    num = 1;
    return num;
}

console.log(showNum()); // 1, like we expected
console.log(num); // also gives us 1. Because we didn't iniitialze num, it's now global.
```

Here is another example of a variable that seems like it should be local, but is actually global:
```javascript
for (var i = 1; i <= 5; i++) {
	console.log (i); // outputs 1, 2, 3, 4, 5
};

// The variable i is a global variable and it is accessible in the
// following function with the last value it was assigned above ​
function aNumber () {
    console.log(i);
}

// The variable i in the aNumber function below is the global variable
// i that was changed in the for loop above. Its last value was 6, set
// just before the for loop exited:
aNumber ();  // 6
```

### setTimeout Variables are Executed in the Global Scope

Now we're getting into the idea of `this` in Javascript. Check out what the `setTimeout` function looks like in this example below:
```javascript
var greeting = "Hi!";
var myGreetObj = {
    greeting:   "Hello!",
    sayHi:      function() { 
                    setTimeout(
                        function() { 
                            console.log(this.greeting); 
                        }, 
                        2000); 
                }
} 

myGreetObj.sayHi(); // "Hi!", not "Hello!"
```
Why does `myGreetObj.sayHi()` use `"Hi!"` for `greeting` not `"Hello!"` as defined in `myGreetObj`? Because `this` when used in `setTimeout` always refers to the global scope Window object, not `myGreetObj` as we would expect. Javascript is wacky sometimes!


### Do not Pollute the Global Scope

Pollution is bad. And as mentioned above when building a large Javascript application, the last thing you want to do is wade through lots of variable names. Encapsulate them within the functions that use them and you will be much happier!

```javascript
// THE BAD DEV WAY:
var name = "Steve", age = "25";
function getName1(){
    return name, "is", age;
}

// THE GOOD WAY
function getName2() {
    var name = "Steve", age = "25";
    return name, "is", age;
}
```

## Hoisting variables

In Javascript, any variable declaration is hoisted to the top of the scope before any initialization or execution happens. This is weird to understand, but if you have `var x = 1;`, the JS runtime will grab just `var x`, hoist it straight to the top of the scope, and temporarily initialize it as `undefined` even though you technically initialized it with a number
```javascript
// this will log out undefined!
// Why?
// Because in the execution process, var name is hoisted to the top of the function scope
// var name temporarily is initialized as undefined
// Next, JS hits the console.log line and exectutes it as "firstName:" + name -- which is currently undefined
function showName(){
    console.log("firstName:", name);
    var name = "LazerThunder";
}

// Another example
// 1. First JS looks for anything with var and hoists it up, initialzing it temporarily as undefined. 
// 2. Next, it works its way down the line:
// 3. console.log("firstName:", name);
//      => "undefined", because as far as we know, that's what name is
// 4. var name = "KillaShottz";        
//      => Oh hey! Now we have some initialization! name isn't undefined, it's "KillaShottz"
// 5. console.log("lastName:", name);
//      => I know what name is -- it's "KillaShottz"! Log it out!
function showNameAgain() {
    console.log("firstName:", name);
    var name = "KillaShottz"; // JS looks for the "var" declaration first and takes this to the top
    console.log("lastName:", name);
}
```

### Function declaration overrides variable declaration in hoisting

Both function and variable declarations get hoisted, but function declarations take precidence! But, like variables, the assignment of functions has to wait for the code to execute down the line.

```javascript
var beans;
function beans() {
    return "Great Northern";
}

console.log(typeof beans); // function, because functions are more important to JS
```

Watch out though, because in this situation, the variable has an assignment and will override the function.

```javascript
var moreBeans = "Pinto";
function moreBeans() {
    return "Kidney";
}

console.log(typeof moreBeans); // string
```

Note also that a function expression (`var beans = function() { return "Great Northern"; }`) will not be hoisted.

## Sources

* [Variable Scope and Hoisting Explained - Javascript is Sexy](http://javascriptissexy.com/javascript-variable-scope-and-hoisting-explained/)
* [Understanding Hoisting in Javascript - Scotch.io](https://scotch.io/tutorials/understanding-hoisting-in-javascript)