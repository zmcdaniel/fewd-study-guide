# Javascript prototypes in plain English

In most languages you use inheritance by having one class inherit another parent class to automatically have access to all of the parents' methods and variables. At least until ES6, Javascript didn't really have a **class** but instead used **prototypes**. 

In a nutshell, **prototypal inheritance** looks like this:

1. An object has a number of properties, including properties/attributes (like `this.name = name` or `name: Fred`) and methods/functions.
2. An object has a special parent property, also called the **prototype attribute** or **prototype object**, also in certain browsers `__proto__` as a shorthand. This points to the object’s “parent”—the object it inherited its properties from. For example:
    ```javascript
    function Dog() {};

    Dog.prototype.bark = function() {
      if (this.canBark == true) {
        console.log("Woof!");
      } else {
        console.log("...");
      }
    }

    function Labrador() {
      this.canBark = true;
    };

    function Basenji() {
      this.canBark = false;
    };

    // The prototype for a Labrador is now Dog, so we have access to the bark function
    Labrador.prototype = new Dog();
    Basenji.prototype = new Dog();

    var bear = new Labrador();
    var wiley = new Basenji();

    // our Labrador instance will bark!
    bear.bark();
    // ...but our Basenji is silent!
    wiley.bark();
 
    console.log(bear.__proto__ == Labrador); // true
    ```
 3. An object can also override a property of its parent by setting the property on itself:
    ```javascript
    // Let's say Wiley is a special Basenji who can actually bark!
    wiley.canBark = true;
    wiley.bark(); // "Woof!"
    ```
4. A **constructor** creates/instantiates objects. Each constructor has an associated **prototype object** or, in code, `Object.prototype`. Use the `new` keyword to call a constructor
    ```javascript
    function Pony(){};
    
    // This is using the Pony constructor to create my faithful mount, Kelly
    var kelly = new Pony();
    
    //This returns the Pony constructor function on line 1 (Pony())
    console.log(kelly.constructor);
 
    //This goes a little deeper -- the constructor for Pony in the constructor for Function objects
    console.log(Pony.constructor);
 
    // This lets use know that the Prototype of Pony is the Object prototype
    console.log(Pony.prototype);
 
    // Is Kelly's prototype a child of the Pony prototype? Yes! True!
    console.log(kelly.__proto__ == Pony.prototype);
    ```
5. When an object is instantiated, the parent is set to the prototype object associated with the constructor that created it.

### Prototype Property and Prototype Attribute

#### Prototype property determines what is inherited

Every Javascript function has a **prototype property**, which is empty by default. Whenever you want to make it so a method or property can be inherited, you attach them to the prototype property. It's important to note the the prototype property is NOT enumerable, and so will not be accessable for a `for`/`in` loop.

* `prototype` is a property of a Function object -- the prototype of objects constructed by that function. It should've really been called something like, `prototypeToInstall`, since that's what it is. You use it to add new properties to an existing prototype.

Prototype property is used mostly for inheritance -- anything you want to pass down to children. Take this for an example:

```javascript
// Let's make a Person prototype a sayHi method to the prototype property
function Person(){};
Person.prototype.sayHi = function() {
    console.log('Hello ' + this.name);
}

// Now let's make a Guy prototype that inherits from Person's prototype but also has its own Name property
function Guy(name){ this.name = name};
Guy.prototype = new Person();

// Instantiating a Guy object named 'michael', using the Guy constructor
var michael = new Guy("Michael");

// michael inherited all the properties and methods, including sayHi. Now michael can call sayHi directly, even though we never created a sayHi method on him
michael.sayHi();
```

Cool! The way  that this prototype chain works looks like this:

`michael` < `Guy.prototype` < `Person.prototype` < `Object.prototype`

You can add properties to any of `michael`'s parents and `michael` will automatically gain those properties too, even *after* `michael` has already been created!

Also, note that this will be `true` for reasons mentioned above: `michael.__proto__ == Guy.prototype`

#### Prototype attribute tells us who the object's parent is

The attribute is a characteristic of an object that points to who their parent is. This is also often called the **prototype object**. It's automatically set as a part of any new object you instantiate on creation. The `__proto__` pseudo property contains the object's prototype object aka the prototype attribute -- the parent object it inherited its method and properties from. 

To get a peek and access prototype properties in most modern browsers, use `__proto__`. When console.log-ed, you can look at any object's inherited properties and methods.
 
* `__proto__` is an internal property of an object and *points* to the parent prototype. It's that "installed prototype" on an object (that was created/installed upon the object from said `constructor()` function). 

##### Prototype attribute of objects created via Object Literal or 'new Object()'

Anything made with an object literal or the native Object constructor inherit from `Object.prototype`, so `Object.prototype` is the prototype attribute/prototype object. Note that `Object.protoype` inherits from no one -- it is the top of the hierarchy.

```javascript
var objectLiteral = {type: 'Object Literal'};;
var newObject = new Object();

console.log(objectLiteral.__proto__); // Object {...}
console.log(newObject.__proto__); // Object {...}
```

##### Prototype attribute of objects created with constructor functions

If you are making your objects using the `new` keyword or constructor other than the `Object()` constructor (`var blah = new Object()`), the prototype attribute will come from the constructor function. 

```javascript
function User(){};
function Admin(){};

Admin.prototype = new User();

var steveJobs = new Admin();
var unassignedTestAccount = new User();

console.log(steveJobs.__proto__); // User {}
console.log(unassignedTestAccount.__proto__); // Object {constructor: function}
```

Essentially, we are looking one up on the hierarchy to see who the prototype parent is. Steve Jobs is an Admin -- where does Admin inherit from? unassignedTestAccount is a User -- where does User inherit from?

##### In a nutshell...

1. If an object is created with an object literal (`var obj = {};`) or uses the Object constructor (`var obj = new Object();`), the prototype attribute / prototype object is `Object.prototype` and it inherits all properties and methods from it.
2. If an object is created with a different constructor, like `new Dog()`, `new Pony()`, or `new Guy()`, it inherits from `Dog.prototype`, `Pony.prototype`, or `Guy.prototype` -- namely the constructor for all of the above. 

### Summary of prototypes and their importance to Javascript

There are two important ways the prototype is used in Javascript. Let's go over them one more time, just to drive it home.

1. **Prototype property**: great for inheritance!
    - Since Javascript does not have *classical inheritance* like most object oriented languages, inheritance is made possible through the prototype property.
    - What is inheritance exactly?
        - A programming paradigm where objects (or Classes in some languages) can inherit properties and methods from other objects (or Classes).
        - For example, you can create a `Reptile` function (which is an object, because all functions in JS are objects) and add properties and methods that will apply to all instances created from the `Reptile` prototype.
    ```javascript
    
    // Let's create a Reptile 'class' with properties that most Reptiles have
    function Reptile(){
       this.reproduction = 'eggs';
       this.blood = 'cold-blooded';
    };
 
    // Adding a method to the Reptile prototype to show the characteristics in the console
    Reptile.prototype.showCharacteristics = function() {
       console.log("I am a " + this.name + ". I am " + this.blood);
       if (this.reproduction == 'eggs') {
           console.log("I reproduce by laying " + this.reproduction);
        } else {
           // Some reptiles have alternative ways besides eggs! Just in case, let's add this conditional
           console.log("I actually reproduce via " + this.reproduction); 
        }
    };
 
    // Now we are creating a constructor for a new Snake prototype
    function Snake(snakeName) {
       this.name = snakeName;
    }
 
    // Snake will inherit its prototype attributes / prototype object from the Reptile constructor in addition to its own
    Snake.prototype = new Reptile();
 
    var rattlesnake = new Snake("rattlesnake");
    var cobra = new Snake("cobra");
    
    rattlesnake.showCharacteristics();
    cobra.showCharacteristics();
 
    // We can change the prototype property for reproduction on this cobra instance but it will still be the same for other instances of the Snake / Reptile class
    rattlesnake.reproduction = 'live birth';
    
    rattlesnake.showCharacteristics();
    cobra.showCharacteristics();
    ```
    * Note that `showCharacteristics` was inherited by `cobra` and `rattlesnake` even though they were defined way up in Reptile. The prototype properties still got passed down the parent-child hierarchy!
2. **Prototype attribute**: great for accessing the properties and methods on objects
    * Also called the **prototype object**. This is the "parent" object from where the inherited properties were originally defined.
        * In the example above, the prototype attribute / object of `Snake` is `Reptile` and all its associated prototype properties.
    * If you want to access the property of an object, start on the object itself. Not there? The Javascript runtime will move one up the chain and look on the object's prototype. It will continue to move up the prototype hierarchy chain, from the 'father' to the 'grandfather' to the 'great-grandfather'. If it's not found, then the property does not exist and `undefined` is returned.
    * The ancestor to all Objects is `Object.prototype`. For example: `hasOwnProperty()`, `isPrototypeOf()`, `propertyIsEnumerable()`, `toLocaleString()`, `toString()`, and `valueOf()`
        * All built-in constructors (`Array ()`, `Number ()`, `String ()`, etc.) were created from the Object constructor, and as such their prototype is Object.prototype.


### Sources

* [Javascript Prototype in plain Language - Javascript is Sexy](http://javascriptissexy.com/javascript-prototype-in-plain-detailed-language/)
* [Javascript Constructors and Prototypes - Toby Ho](http://tobyho.com/2010/11/22/javascript-constructors-and/)
* [Prototypes - General Assembly WDI](https://wdi_sea.gitbooks.io/notes/content/02-js-jquery/js-prototypes/03prototypes.html)
* [__Proto__ vs Prototype - StackOverflow](http://stackoverflow.com/questions/9959727/proto-vs-prototype-in-javascript)