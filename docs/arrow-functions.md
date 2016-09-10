* [Arrow Functions](#arrow-functions)
* [Tip: Arrow Function Need](#tip-arrow-function-need)
* [Tip: Arrow Function Danger](#tip-arrow-function-danger)
* [Tip: Libraries that use `this`](#tip-arrow-functions-with-libraries-that-use-this)
* [Tip: Arrow Function inheritance](#tip-arrow-functions-and-inheritance)

### Arrow Functions

Lovingly called the *fat arrow* (because `->` is a thin arrow and `=>` is a fat arrow) and also called a *lambda function* (because of other languages). Another commonly used feature is the fat arrow function `()=>something`. The motivation for a *fat arrow* is:
1. You don't need to keep typing `function`
2. It lexically captures the meaning of `this`
2. It lexically captures the meaning of `arguments`

For a language that claims to be functional, in JavaScript you tend to be typing `function` quite a lot. The fat arrow makes it simple for you to create a function
```ts
var inc = (x)=>x+1;
```
`this` has traditionally been a pain point in JavaScript. As a wise man once said "I hate JavaScript as it tends to lose the meaning of `this` all too easily". Fat arrows fix it by capturing the meaning of `this` from the surrounding context. Consider this pure JavaScript class:

```ts
function Person(age) {
    this.age = age
    this.growOld = function() {
        this.age++;
    }
}
var person = new Person(1);
setTimeout(person.growOld,1000);

setTimeout(function() { console.log(person.age); },2000); // 1, should have been 2
```
If you run this code in the browser `this` within the function is going to point to `window` because `window` is going to be what executes the `growOld` function. Fix is to use an arrow function:
```ts
function Person(age) {
    this.age = age
    this.growOld = () => {
        this.age++;
    }
}
var person = new Person(1);
setTimeout(person.growOld,1000);

setTimeout(function() { console.log(person.age); },2000); // 2
```
The reason why this works is the reference to `this` is captured by the arrow function from outside the function body. This is equivalent to the following JavaScript code (which is what you would write yourself if you didn't have TypeScript):
```ts
function Person(age) {
    this.age = age
    var _this = this;  // capture this
    this.growOld = function() {
        _this.age++;   // use the captured this
    }
}
var person = new Person(1);
setTimeout(person.growOld,1000);

setTimeout(function() { console.log(person.age); },2000); // 2
```
Note that since you are using TypeScript you can be even sweeter in syntax and combine arrows with classes:
```ts
class Person {
    constructor(public age:number) {}
    growOld = () => {
        this.age++;
    }
}
var person = new Person(1);
setTimeout(person.growOld,1000);

setTimeout(function() { console.log(person.age); },2000); // 2
```

#### Tip: Arrow Function Need
Beyond the terse syntax, you only *need* to use the fat arrow if you are going to give the function to someone else to call. Effectively:
```ts
var growOld = person.growOld;
// Then later someone else calls it:
growOld();
```
If you are going to call it yourself, i.e.
```ts
person.growOld();
```
then `this` is going to be the correct calling context (in this example `person`).

#### Tip: Arrow Function Danger

In fact if you want `this` *to be the calling context* you should *not use the arrow function*. This is the case with callbacks used by libraries like jquery, underscore, mocha and others. If the documentation mentions functions on `this` then you should probably just use a `function` instead of a fat arrow. Similarly if you plan to use `arguments` don't use an arrow function.

#### Tip: Arrow functions with libraries that use `this`
Many libraries do this e.g `jQuery` iterables (one example http://api.jquery.com/jquery.each/) will use `this` to pass you the object that it is currently iterating over. In this case if you want to access the library passed `this` as well as the surrounding context just use a temp variable like `_self` like you would in the absence of arrow functions.

```ts
let _self = this;
something.each(function() {
    console.log(_self); // the lexically scoped value
    console.log(this); // the library passed value
});
```

#### Tip: Arrow functions and inheritance

If you have an instance method as an arrow function then its goes on `this`. Since there is only one `this` such functions cannot participate in a call to `super` (`super` only works on prototype members). You can easily get around it by creating a copy of the method before overriding it in the child.

```ts
class Adder {
    // This function is now safe to pass around
    add = (b: string): string => {
        return this.a + b;
    }
}

class ExtendedAdder extends Adder {
    // Create a copy of parent before creating our own
    private superAdd = this.add;
    // Now create our override
    add = (b: string): string => {
        return this.superAdd(b);
    }
}
```
