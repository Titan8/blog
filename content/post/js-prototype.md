+++
draft = false
date = "2017-01-05T00:00:00+08:00"
title = "Javascript Prototype"

+++

*note of [You Don't know JS](https://github.com/getify/You-Dont-Know-JS/tree/master/this%20%26%20object%20prototypes)*
## Data types in Javascript

```javascript
// Javascript has 6 primitive data(immutable) types
typeof 3.14  // 'number';
typeof 'bla'  // 'string';
typeof true  // 'boolean'
typeof Symbol.iterator  // 'symbol'
typeof undefined  // 'undefined'
typeof null  // 'object'

// and object
typeof {a:1}  // 'object'
typeof function fn(){}  // 'function', it's in fact an object with special type tag


// object has sub type
Object.prototype.toString.call(fn)  // '[object Function]'

// primitive types have object equivalent that wrap around them.
Object.prototype.toString.call(3.14)  // '[object Number]'
Object.prototype.toString.call('bla')  // '[object String]'
Object.prototype.toString.call(true)  // '[object Boolean]'
Object.prototype.toString.call(Symbol.iterator)  // '[object Symbol]'

Object.prototype.toString.call({})  // '[object Object]'
Object.prototype.toString.call([])  // '[object Array]'
Object.prototype.toString.call(/[a-zA-Z]+/)  // '[object RegExp]'

```
`null` is not an object for it has no any property.

for why `typeof null === 'object'` see details [here] (https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/typeof#null)



## Object property

```javascript
var myObject = {
    a: 2
};

// `in` operator will check upwards the whole prototype chain
('a' in myObject) // true
('toString' in myObject) // true
// `hasOwnProperty` function check only `this` object
(myObject.hasOwnProperty('toString'))  // false

// property has characteristics decribed by sth called `descriptor`
Object.getOwnPropertyDescriptor( myObject, "a" );
// {
//    value: 2,
//    writable: true,
//    enumerable: true,
//    configurable: true
// }

// another way to define property
Object.defineProperty( myObject, "a", {
    value: 2,
    writable: true,
    configurable: true,
    enumerable: true
} );

// prevent an object from having new properties added to it
Object.preventExtensions( myObject );

Object.seal(..) 
// Object.preventExtensions(..) + marks all its existing properties as configurable:false.

Object.freeze(..)
// Object.seal(..) + marks all "data accessor" properties as writable:false

// descriptor includes data descriptor and accessor descriptor
Object.defineProperty(
    myObject,   // target
    "b",        // property name
    {           // descriptor
        // define a getter for `b`
        get: function(){ return this.a * 2 },

        // make sure `b` shows up as an object property
        enumerable: true
    }
);

myObject.b;  // 4
```


## What is `this`?

```javascript
function dcyy() {
  console.log(this.name, '到此一游 :(');
}
var name = 'global';
var zs = {
  name: '张三',
  dcyy
}
var ls = {
  name: '李四'
}

dcyy();  // global 到此一游 :(
zs.dcyy();  // '张三 到此一游 :(
dcyy.call(ls);  // 李四 到此一游 :(
new dcyy();
// undefined '到此一游 :('
// dcyy {}
```


`this` is not an author-time binding but a runtime binding.

1. Called with new? Use the newly constructed object.

2. Called with `call` or `apply` (or `bind`)? Use the specified object.

3. Called with a context object owning the call? Use that context object.

4. Default: undefined in strict mode, global object otherwise.



## Prototype Chain

```javascript
function Foo() {
    // ...
}

var a = new Foo();

Object.getPrototypeOf( a ) === Foo.prototype; // true

a.__proto__ === Foo.prototype;  // true

Foo.prototype.isPrototypeOf(a);  // true

var p = {
    sayHi: function () {
        console.log('hi!');
    }
}
var c = Object.create(p);
c.sayHi();  // 'hi!'

```

So, when u create an object use `new` operator and a function say *fn*, the prototype of this created object is `fn.prototype`

`You Dont Know JS` advocates prototype pattern which is the foundation of JS, objects OO simulation.
