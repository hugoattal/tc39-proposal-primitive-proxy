# TC39 proposal: Primitive proxy

## Synopsis

The proposed primitive proxy is aimed at providing a mechanism for creating primitive variables (such as strings and numbers) with user-defined set and get functions. This allows developers to add custom functionality to these primitive types without having to wrap them in objects.

## Motivation

The proposed primitive proxy would simplify properties of reactive frameworks such as React or Vue. These frameworks rely heavily on the ability to create reactive properties that can update automatically in response to changes in the underlying data.

There are two solutions:
- React-like reactive properties use a getter to get the variable, and a function to set it.
- Vue-like reactive properties wrap the variable into a [Proxy](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Proxy), where the value can be set and get using the `value` property.

A primite proxy would allow to set and get a reactive variable seemlessly, here's an example:

```ts
// React
const [myReactVar, setMyReactVar] = useState("");
setMyReactVar("Hello world");
console.log(myReactVar);

// Vue
const myVueVar = ref("");
myVueVar.value = "Hello world";
console.log(myVueVar.value);

// Primitive proxy
const myPrimitiveVar = ref("");
myPrimitiveVar = "Hello world";
console.log(myPrimitiveVar);
```

## Possible API

Here is a possible implementation inspired by the [Proxy object](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Proxy), but without the prop argument of the handler `get` and `set`.

```ts
const myVar = new PrimitiveProxy({ storedValue: "" }, {
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
