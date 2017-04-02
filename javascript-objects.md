# Javascript objects in detail

In Javascript has one complex datatype -- objects -- and five simple/primitive datastypes -- strings, numbers, booleans, null, and undefined. Objects are **mutable**, meaning they can be changed, while the primitive datatypes are **immutable**.

### What is an object?
An object is a hashmap of key-value pairs. These key-value pairs are either **methods** (functions) or **properties** (attributes).

```javascript
var myCar = {
    make: "Ford",
    model: "Taurus",
    4WD: false,
};
```

Property names can be a string or a number, but if it's a number you must access it using bracket notation:

```javascript
console.log(myCar.make); // Will log out "Ford"
console.log(myCar.4WD); // Will throw an error
console.log(myCar["4WD"]); // Will log out "false"
```

### Reference datatypes vs primitive datatypes

Like it's name suggests, **reference** datatypes are stored as a *reference*, and not directly on the variable like a primitive datatype would do.

```javascript
// Primitive Datatypes:
var x = 1; // The primitive datatype Number is stored as a value;
var y = x; // The value of y is the value of x
x = 2;


// Even though we changed x to 2, y will still be saved as 1
console.log(x); // Will log out 2
console.log(y); // Will log out 1
```

```javascript
// Referential Datatypes
var x = {value: 1};
var y = x;
x.value = 2;

console.log(x); // Will log out 2
console.log(y); // Will log out 2, because it is saved as a reference!
```

In the reference example, value in `y` was stored as a reference and not an actual value, when we changed the `x.value` property to `2` the `y` object reflected the change because it never stored an actual copy of it’s own value of the `x`’s properties, it only had a reference to it.

### Object data properties have attributes!
 
When using HTML/CSS, when you create a new `<p>` element it has attributes like `font-size`, `color`, and `display`. Same with the properties in an object:
 
* **Configurable Attributes:** Can an object be deleted or changed?
* **Enumerable:** Can the property be returned in a for/in loop?
* **Writable:** Can the property be changed?
* New in ECMAScript 5: accessor properties! Getter and setter functions to go along with these attributes

### Creating Objects

#### Object literals

Very common and super easy. The examples above were all object literals. Here's another one:

```javascript
// An empty object initialized using object literal notation (curly brackets);
var myCat = {};

// An object literal with three properties and one method
var myDog = {
    name: "Grendel",
    age: 8,
    breed: "Bouvier",
    getToy: function(toy) {
        console.log(this.name + " fetched the " + toy);
    }
};
```

#### Object constructor

An object constructor uses the `new` keyword to call the `Object()` constructor.

```javascript
var myDog = new Object();
myDog.name = "Grendel";
myDog.age = 8;
myDog.breed = "Bouvier";
myDog.getToy = function(toy) {
    console.log(this.name + " fetched the " + toy);
};
```

Note: when naming properties in your object, you can use reserved words like `for`, but it's better to avoid it.

Also, Objects can contain whatever datatype you want -- Arrays, Booleans and even other Objects!

### Practial patterns for object creation

#### Constructor pattern

The two ways of making objects outlines above are great for making simple objects that are only used once, but say you want to create an application to store all the dogs you currently know. Because of course you do! It would be pretty tedious to write out all that every time you want to add a new dog. And if you wanted to make a change to the `getToy` function, you would have to do it for every single Dog object you made! Check out this **constructor** pattern:
  
```javascript
function Dog(pupName, pupAge, pupBreed) {
    this.name = pupName;
    this.age = pupAge;
    this.breed = pupBreed;
    
    this.getToy = function(toy) {
        console.log(this.name + " fetched the " + toy);
    }
}

var myDog = new Dog("Grendel", 8, "Bouvier");
myDog.getToy("ball"); // "Grendel fetched the ball"

var bensDog = new Dog("Pixel", 2, "Pug");
bensDog.getToy("bone"); // "Pixel fetched the bone"
```

With this constructor pattern, it's easy to create lots of Dogs and changed the `getToy` method so that it applies accross all Dog objects. All Dogs inherit from the Dog function, and so they will also inherit the changes made!

#### Prototype pattern for creating objects

```javascript
function Dog(){};

Dog.prototype.isCanine = true;
Dog.prototype.noise = "Woof!";
Dog.prototype.makeNoise = function() {
    console.log(this.noise);
};

var myDog = new Dog();

myDog.makeNoise(); // "Woof!"
```
#### The difference between using Constructor and Prototype

With methods on construtor-style / class-like objects, every time a new Dog object is created, a new `getToy` will be created too. With prototypes, only one `makeNoise` is created and shared among all other dogs -- because `Dog.prototype` is their parent. Declaring methods on the Prototype is much more memory efficient.   

### Properties

#### Own and inherited

Objects' properties are either inherited or owned. Owned means that the property was defined on the object itself, where inherted properties are inherited from the parent Prototype object.

To find out if a property exists on an object, either as inherited or own, use the `in` operator.

 `hasOwnProperty` is a built in method to check if a property is owned or inherited
 
 Anything inherited from the `Object` prototype (`toString` for example), will not be enumerable using a `for`/`in` loop. 
 ```javascript
function Mammal(){};
Mammal.prototype.hasHair = true;

function Dog(){};
Dog.prototype = new Mammal(); // Dog inherits properties and methods from Mammal prototype
Dog.prototype.dietType = "omnivore";

var pupper = new Dog();

// This is an "own" property
pupper.name = "Frank";
pupper.breed = "Jack Russell";
pupper.color = "White";

// Examples of the 'in' operator
console.log("name" in pupper); // Prints true, because there is a "name" property in the pupper object
console.log("age" in pupper); // Prints false -- there is no age property definied
console.log("hasHair" in pupper); // Prints true. Dog objects inherit this from the Mammal prototype

// Examples of the 'hasOwnProperty' method
console.log(pupper.hasOwnProperty("dietType")); // false, because this is inherited from the Dog prototype
console.log(pupper.hasOwnProperty("color")); // true, because 'color' is directly on the pupper object

// Accessing/Enumerating inherited and own properties
for(var eachProperty in pupper) {
     console.log(eachProperty); // Will log out "name" "breed" "color" "dietType" "hasHair"
 }
```

#### Deleting properties of an object

Use the `delete` operator! Keep in mind when using this operator that you cannot delete properties that were inherited or properties with their attributes set to 'configurable'. 
```javascript
var myList = {first: '1', second: '2', third: '3'}
delete myList.third;

for (var item in myList) {
    console.log(item); // Will print 'first' 'second', but not 'third'
}

delete myList.toString(); // Will return true, but since toString is inherited from the Object prototype, it won't actually be gone
```

### Serializing and deserializing objects

JSON is Javascript Object Notation. If you ever want to transfer your objects over HTTP or just convert them to a string, you use the `JSON.stringify` function to serialize your objects. This was standardized in ES5. To deserialize (convert from a string to an object), use the `JSON.parse` function.

```javascript
var request = {url: "reddit.com", q: "cats"};
stringifyRequest = JSON.stringify(request, null, 4);
console.log(stringifyRequest);
// Prints this string:
// {
//      "url": "reddit.com",
//      "q": "cats"
// }
```
```javascript
// Since this is a JSON string, we cannot access the properties with dot notation (like request.url) 
var request = {"url": "weather.com", "q": "98104"};
parsedRequest = JSON.parse(request);
console.log(parsedRequest);
// Prints this:
// Object {
//      q: "98104",
//      url: "weather.com"
// }
```

### Useful sources

[Javascript Constructors and Prototypes](http://tobyho.com/2010/11/22/javascript-constructors-and/)
[Javascript Objects in Detail](http://javascriptissexy.com/javascript-objects-in-detail/)
[OOP with Constructors - General Assembly WDI](https://wdi_sea.gitbooks.io/notes/content/02-js-jquery/js-prototypes/02constructors.html)