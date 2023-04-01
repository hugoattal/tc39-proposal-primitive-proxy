# TC39 proposal: Primitive proxy

## Synopsis

The proposed primitive proxy is aimed at providing a mechanism for creating primitive variables (such as strings and numbers) with user-defined set and get functions. This allows developers to add custom functionality to these primitive types without having to wrap them in objects.

## Motivation

The proposed primitive proxy would simplify properties of reactive frameworks such as React or Vue. These frameworks rely heavily on the ability to create reactive properties that can update automatically in response to changes in the underlying data.

There are two solutions:
- React-like reactive properties use a getter to get the variable, and a function to set it.
- Vue-like reactive properties wrap the variable into a [Proxy](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Proxy), where the value can be set and get using the `value` property.

A primitive proxy would allow to set and get a reactive variable seemlessly, here's an example:

```js
// React
const [myReactVar, setMyReactVar] = useState("");
setMyReactVar("Hello world");
console.log(myReactVar);

// Vue
const myVueVar = ref("");
myVueVar.value = "Hello world";
console.log(myVueVar.value);

// Primitive proxy
let myPrimitiveVar = ref("");
myPrimitiveVar = "Hello world";
console.log(myPrimitiveVar);
```

## Possible API

Here is a possible implementation inspired by the [Proxy object](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Proxy), but without the prop argument of the handler `get` and `set`.

```js
let myVar = new PrimitiveProxy({ storedValue: "" }, {
    set: (target, value) => {
        target.storedValue = value + " world";
    },
    get: (target) => {
        return target.storedValue + " !";
    }
});

myVar = "Hello";
console.log(myVar); // Hello world !
```

Here, `storedValue` is just for example purpose. The first argument of `PrimitiveProxy` is an object where you can store values that can be accessed with the `target` argument of `set` and `get`.

### `PrimitiveProxy.prototype.clone`

Cloning a Primitive Proxy into another variable is not as simple as just assigning it to another variable, because it would just return a standard primitive using the getter of the Primitive Proxy.

So you would need to wrap it into another primitive proxy like this:
```js
let a = new PrimitiveProxy(...);
let b = new PrimitiveProxy(null, {
    set: (_target, value) => {
        a = value;
    },
    get: (_target) => {
        return a;
    }
})
```

This could be simplified with a `PrimitiveProxy.protoype.proxyOf` function:
```js
let a = new PrimitiveProxy(...);
let b = a.proxyOf();
```

## Questions

### Isn't it confusing not to be able to reassign a variable?

It's the same thing as standard Proxy. Setting a value to `myObject.thing` doesn't not necesarry mean that the `thing` property of `myObject` contains the set value, if the object is behind a Proxy.

### Why using a `let` instead of a `const`?

For consistency, `const` should not be updatable. So a `const` Primitive Proxy should disable the `set` handler.

### How to unproxify a primitive proxy?

Simply reassign it like this:
```js
let a = new PrimitiveProxy(...);
let b = a; // b is a standard primitive
```

### What happens when passing a Primitive Proxy to a function?

When passed to a function, the value of the Primitive Proxy getter is copied. If you want to pass the raw Primitive Proxy to a function, you must wrap it into an object using `proxyOf`.

```js
let a = new PrimitiveProxy({storedValue: ""}, {
    set: (target, value) => {
        target.storedValue = value;
    },
    get: (target) => {
        return "primitive:" + target.storedValue;
    }
});

a = "Hello";
console.log(a); // primitive:Hello

((value) => { // here, value is a standard primitive
    console.log(value); // primitive:Hello
    value = "test";
    console.log(value); // test
})(a);

console.log(a); // primitive:Hello

(({ value }) => { // here, value is a proxy primitive
    console.log(value); // primitive:Hello
    value = "test";
    console.log(value); // primitive:test
})({ value: a.proxyOf });

console.log(a); // primitive:test
```
