# You Don't Know JS Yet: Types & Grammar - 2nd Edition
# Chapter 3: Object Values

| NOTE: |
| :--- |
| Work in progress |

Now that we're comfortable with the built-in primitive types, we turn our attention to the `object` types in JS.

I could write a whole book talking about objects in-depth; in fact, I already did! The "Objects & Classes" title of this series covers objects in-depth already, so make sure you've read that before continuing with this chapter.

Rather than repeat that book's content, here we'll focus our attention on how the `object` value-type behaves and interacts with other values in JS.

## Types of Objects

The `object` value-type comprises several sub-types, each with specialized behaviors, including:

* plain objects
* fundamental objects (boxed primitives)
* built-in objects
* arrays
* regular expressions
* functions (aka, "callable objects")

Beyond the specialized behaviors, one shared characteristic is that all objects can act as collections (of properties) holding values (including functions/methods).

## Plain Objects

The general object value-type is sometimes referred to as *plain ol' javascript objects* (POJOs).

Plain objects have a literal form:

```js
address = {
    street: "12345 Market St",
    city: "San Francisco",
    state: "CA",
    zip: "94114"
};
```

This plain object (POJO), as defined with the `{ .. }` curly braces, is a collection of named properties (`street`, `city`, `state`, and `zip`). Properties can hold any values, primitives or other objects (including arrays, functions, etc).

The same object could also have been defined imperatively using the `new Object()` constructor:

```js
address = new Object();
address.street = "12345 Market St";
address.city = "San Francisco";
address.state = "CA";
address.zip = "94114";
```

Plain objects are by default `[[Prototype]]` linked to `Object.prototype`, giving them delegated access to several general object methods, such as:

* `toString()` / `toLocaleString()`
* `valueOf()`
* `isPrototypeOf(..)`
* `hasOwnProperty(..)` (recently deprecated -- alternative: static `Object.hasOwn(..)` utility)
* `propertyIsEnumerable(..)`
* `__proto__` (getter function)

```js
address.isPrototypeOf(Object.prototype);    // true
address.isPrototypeOf({});                  // false
```

## Fundamental Objects

JS defines several *fundamental* object types, which are instances of various built-in constructors, including:

* `new String()`
* `new Number()`
* `new Boolean()`

Note that these constructors must be used with the `new` keyword to construct instances of the fundamental objects. Otherwise, these functions actually perform type coercion (see Chapter 4).

These fundamental object constructors create object value-types instead of a primitives:

```js
myName = "Kyle";
typeof myName;                      // "string"

myNickname = new String("getify");
typeof myNickname;                  // "object"
```

In other words, an instance of a fundamental object constructor can actually be seen as a wrapper around the corresponding underlying primitive value.

| WARNING: |
| :--- |
| It's nearly universally regarded as *bad practice* to ever directly instantiate these fundamental objects. The primitive counterparts are generally more predictable, more performant, and offer *auto-boxing* (see "Automatic Objects" section below) whenever the underlying object-wrapper form is needed for property/method access. |

The `Symbol(..)` and `BigInt(..)` functions are referred to in the specification as "constructors", though they're not used with the `new` keyword, and the values they produce in a JS program are indeed primitives.

How, there are internal *fundamental objects* for these two types, used for prototype delegation and *auto-boxing*.

By contrast, for `null` and `undefined` primitive values, there aren't `Null()` or `Undefined()` "constructors", nor corresponding fundamental objects or prototypes.

### Prototypes

Instances of the fundamental object constructors are `[[Prototype]]` linked to their constructors' `prototype` objects:

* `String.prototype`: defines `length` property, as well as string-specific methods, like `toUpperCase()`, etc.

* `Number.prototype`: defines number-specific methods, like `toPrecision(..)`, `toFixed(..)`, etc.

* `Boolean.prototype`: defines default `toString()` and `valueOf()` methods.

* `Symbol.prototype`: defines `description` (getter), as well as default `toString()` and `valueOf()` methods.

* `BigInt.prototype`: defines default `toString()`, `toLocaleString()`, and `valueOf()` methods.

Any direct instance of the built-in constructors have `[[Prototype]]` delegated access to its respective `prototype` properties/methods. Moreover, corresponding primitive values also have such delegated access, by way of *auto-boxing*.

### Automatic Objects

I've mentioned *auto-boxing* several times (including Chapters 1 and 2, and a few times so far in this chapter). It's finally time for us to explain that concept.

Accessing a property or method on a value requires that the value be an object. As we've already seen in Chapter 1, primitives *are not* objects, so JS needs to then temporarily convert/wrap such a primitive to its fundamental object counterpart[^AutoBoxing] to perform that access.

For example:

```js
myName = "Kyle";

myName.length;              // 4

myName.toUpperCase();       // "KYLE"
```

Accessing the `length` property or the `toUpperCase()` method, is only allowed on a primitive string value because JS *auto-boxes* the primitive `string` into a wrapper fundamental object, an instance of `new String(..)`. Otherwise, all such accesses would have to fail, since primitives do not have any properties.

More importantly, when the primitive value is *auto-boxed* to its fundamental object counterpart, those internally created objects have access to predefined properties/methods (like `length` and `toUpperCase()`) via a `[[Prototype]]` link to their respective fundamental object's prototype.

So an *auto-boxed* `string` is an instance of `new String()`, and is thus linked to `String.prototype`. Further, the same is true of `number` (wrapped as an instance of `new Number()`) and `boolean` (wrapped as an instance of `new Boolean()`).

Even though the `Symbol(..)` and `BigInt(..)` "constructors" (used without `new`produce primitive values, these primitive values can also be *auto-boxed* to their internal fundamental object wrapper forms, for the purposes of delegated access to properties/methods.

| NOTE: |
| :--- |
| See the "Objects & Classes" book of this series for more on `[[Prototype]]` linkages and delegated/inherited access to the fundamental object constructors' prototype objects. |

Since `null` and `undefined` have no corresponding fundamental objects, there is no *auto-boxing* of these values.

A subjective question to consider: is *auto-boxing* a form of coercion? I say it is, though some disagree. Internally, a primitive is converted to an object, meaning a change in value-type has occurred. Yes, it's temporary, but plenty of coercions are temporary. Moreover, the conversion is rather *implicit* (implied by the property/method access, but only happens internally). We'll revisit the nature of coercion in Chapter 4.

## Other Built-in Objects

In addition to fundamental object constructors, JS defines a number of other built-in constructors that create further specialized object sub-types:

* `new Date(..)`
* `new Error(..)`
* `new Map(..)`, `new Set(..)`, `new WeakMap(..)`, `new WeakSet(..)` -- keyed collections
* `new Int8Array(..)`, `new Uint32Array(..)`, etc -- indexed, typed-array collections
* `new ArrayBuffer(..)`, `new SharedArrayBuffer(..)`, etc -- structured data collections

## Arrays

Arrays are objects that are specialized to behave as numerically indexed collections of values, as opposed to holding values at named properties like plain objects do.

Arrays have a literal form:

```js
favoriteNumbers = [ 3, 12, 42 ];

favoriteNumbers[2];                 // 42
```

The same array could also have been defined imperatively using the `new Array()` constructor:

```js
favoriteNumbers = new Array();
favoriteNumbers[0] = 3;
favoriteNumbers[1] = 12;
favoriteNumbers[2] = 42;
```

Arrays are `[[Prototype]]` linked to `Array.prototype`, giving them delegated access to a variety of array-oriented methods, such as `map(..)`, `includes(..)`, etc:

```js
favoriteNumbers.map(v => v * 2);
// [ 6, 24, 84 ]

favoriteNumbers.includes(42);       // true
```

Some of the methods defined on `Array.prototype` -- for example, `push(..)`, `pop(..)`, `sort(..)`, etc -- behave by modifying the array value in place. Other methods -- for example, `concat(..)`, `map(..)`, `slice(..)` -- behave by creating a new array to return, leaving the original array intact. A third category of array functions -- for example, `indexOf(..)`, `includes(..)`, etc -- merely computes and returns a (non-array) result.

## Regular Expressions

A regular expression (or "regex") is an object that represents a pattern for matching character combinations in strings. Regular expressions are a powerful tool for searching, replacing, and validating text.

Creating Regular Expressions

There are two ways to create a RegExp object:

```js
// Literal form (preferred for static patterns)
let pattern1 = /hello/i;

// Constructor form (useful for dynamic patterns)
let pattern2 = new RegExp("hello", "i");
```

The literal form is more performant and readable when the pattern is known at write-time. The constructor form is necessary when the pattern is built dynamically from strings.

Flags

Flags are optional modifiers that change how matching behaves:

| Flag | Name | Description |
|------|------|-------------|
| `g` | Global | Find all matches, not just the first |
| `i` | Case-insensitive | Case-insensitive matching |
| `m` | Multiline | Treats `^` and `$` as matching line boundaries |
| `u` | Unicode | Treat pattern as Unicode code points |
| `y` | Sticky | Matches only from `lastIndex` property |

Key Methods

The .test() method returns true if a match exists:

```js
let pattern = /world/;
pattern.test("hello world");  // true
pattern.test("hello there");  // false
```

The .exec() method returns match details or null:

```js
let pattern = /(\d+)-(\d+)/;
let result = pattern.exec("My number is 123-4567");

console.log(result[0]);   // "123-4567" (full match)
console.log(result[1]);   // "123" (first capture group)
console.log(result[2]);   // "4567" (second capture group)
console.log(result.index); // 13 (match position)
```

Properties

Every RegExp instance has these properties:

```js
let pattern = /hello/gi;
console.log(pattern.source);    // "hello" (pattern text)
console.log(pattern.flags);     // "gi" (flags as string)
console.log(pattern.global);    // true (has g flag?)
console.log(pattern.ignoreCase); // true (has i flag?)
console.log(pattern.lastIndex);  // 0 (position to start next search)
```

The lastIndex property is especially important with the g flag:

```js
let pattern = /a/g;
let str = "abracadabra";

console.log(pattern.test(str));  // true (matches at index 0)
console.log(pattern.lastIndex);  // 1 (starts next search after index 0)
console.log(pattern.test(str));  // true (matches at index 3)
console.log(pattern.lastIndex);  // 4
```

When to Use Literal vs. Constructor

Use literal form Use constructor form
Pattern is known at development time Pattern comes from user input
Better performance Need to interpolate variables
Cleaner syntax / characters need escaping anyway

```js
// Dynamic pattern example
let userInput = "foo";
let dynamicPattern = new RegExp(userInput, "i");
dynamicPattern.test("FOO");  // true
```

Regular expressions are a deep topic. This section covers the object nature of RegExp instances; for advanced pattern syntax, refer to MDN's Regular Expression documentation.

## Functions

Functions in JavaScript are a special type of object. They are callable objects — meaning they have all the capabilities of objects (properties, methods, etc.) but can also be invoked as functions.

Functions as Objects

Because functions are objects, you can attach properties to them:

```js
function greet(name) {
    return `Hello, ${name}!`;
}

greet.language = "English";
greet.description = "A simple greeting function";

console.log(greet.language);      // "English"
console.log(greet.description);   // "A simple greeting function"
```

This is a powerful feature that enables techniques like memoization and function configuration.

Function Declarations vs. Expressions

There are two primary ways to create functions:

```js
// Function declaration (hoisted)
function add(a, b) {
    return a + b;
}

// Function expression (not hoisted)
const subtract = function(a, b) {
    return a - b;
};
```

Function declarations are hoisted, meaning they can be called before they appear in the code. Function expressions behave like any other variable assignment.

The function Keyword as a Native

The function keyword is a built-in native constructor:

```js
// Using the Function constructor (rarely needed)
const multiply = new Function("a", "b", "return a * b");
console.log(multiply(3, 4));  // 12
```

You almost never need the Function constructor. It has security implications (similar to eval()) and is significantly slower than function declarations or expressions.

Methods vs. Standalone Functions

When a function is a property of an object, it's called a method:

```js
const calculator = {
    value: 0,
    // Method (attached to calculator)
    add(n) {
        this.value += n;
        return this;
    },
    // Standalone function assigned to a property
    multiply: function(a, b) {
        return a * b;
    }
};

calculator.add(5);        // 'this' refers to calculator
calculator.multiply(3, 4); // Works like a normal function
```

The key difference is how this behaves. Methods called as object.method() have this bound to the object. Standalone functions have this determined by how they're called (strict mode vs. non-strict mode).

Arrow Functions

Arrow functions are a shorter syntax introduced in ES6, but more importantly, they behave differently with this:

```js
const traditional = function() {
    console.log(this);  // Depends on caller
};

const arrow = () => {
    console.log(this);  // Inherits from surrounding scope
};

const obj = {
    name: "My Object",
    traditional: function() {
        console.log(this.name);  // "My Object"
    },
    arrow: () => {
        console.log(this.name);  // undefined (or outer 'this')
    }
};

obj.traditional();  // "My Object"
obj.arrow();        // undefined
```

Arrow functions do not have their own this binding, cannot be used as constructors (no new), and do not have a prototype property.

Key Takeaways

· All functions are objects, but not all objects are functions
· Functions can have their own properties and methods
· The Function constructor exists but should almost never be used
· Methods are functions attached to objects with a this context
· Arrow functions provide lexical this binding and a concise syntax

---

## Proposed: Records/Tuples

At the time of this writing, a (stage-2) proposal[^RecordsTuplesProposal] exists to add a new set of features to JS, which correspond closely to plain objects and arrays, but with some notable differences.

Records are similar to plain objects, but are immutable (sealed, read-only), and (unlike objects) are treated as primitive values, for the purposes of value assignment and equality comparison. The syntax difference is a `#` before the `{ }` delimiter. Records can only contain primitive values (including records and tuples).

Tuples have exactly the same relationship, but to arrays, including the `#` before the `[ ]` delimiters.

It's important to note that while these look and seem like objects/arrays, they are indeed primitive (non-object) values.

[^FundamentalObjects]: "20 Fundamental Objects", EcamScript 2022 Language Specification; https://262.ecma-international.org/13.0/#sec-fundamental-objects ; Accessed August 2022

[^AutoBoxing]: "6.2.4.6 PutValue(V,W)", Step 5.a, ECMAScript 2022 Language Specification; https://262.ecma-international.org/13.0/#sec-putvalue ; Accessed August 2022

[^RecordsTuplesProposal]: "JavaScript Records & Tuples Proposal"; Robin Ricard, Rick Button, Nicolò Ribaudo;
https://github.com/tc39/proposal-record-tuple ; Accessed August 2022
