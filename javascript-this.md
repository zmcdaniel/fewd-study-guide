# *this* in Javascript

First and foremost, if you know about `this` in other languages, immediately forget it. Javascript is quirky and naturally, so this `this`.

I find it best to imagine that `this` is similar to the way that we use pronouns in English. For example, we can say "Michael is running because *he* likes it," or "Michael is running because Michael likes it".
 
It's much easier to say the sentence with *he* instead of repeating *Michael* a secondary time. Like using pronouns, Javascript gives us the nifty shorthand of `this` to refer to the subject in context. Or rather, the subject of the executing code.

```javascript
var person = {
    firstName: "Tyrion",
    lastName: "Lannister",
    // We can use 'this'
    fullNameWithThis: function() {
        return this.firstName + " " + this.lastName;
    },
    // ...or we can use 'person'
    fullNameWithoutThis: function() {
        return person.firstName + " " + person.lastName;
    }
}
```
The problem with `person.firstName` vs. `this.firstName` is that it's pretty ambiguous. What is there is another global variable with the name `person`? References to `person.firstName` could then attempt to access the variable and not the Object instance. But with `this.firstName`, we know that it is referring to *this* person. 

## The basics of *this*

All objects in Javascript have properties, and so by extensions so do all functions. When a function executes it get the `this` property -- a variable with the **value of *the object* that invokes the function where `this` is used**.

The `this` reference ALWAYS refers to and hold the value of a singular object. Usually it is used inside the scope of a function/method, but can be used outside a function in the global scope (like `setTimeout()`).

```javascript
var doggo = {
    name: "Pooch",
    getName: function() {
        return this.name;
    }
}

doggo.getName(); // "Pooch"
```

In the case of `getName()`, the `this` refers to the `doggo` object that is invoking `getName()`. 

