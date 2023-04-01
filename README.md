# TC39 proposal: Primitive proxy

## Synopsis

The proposed primitive proxy is aimed at providing a mechanism for creating primitive variables (such as strings and numbers) with user-defined set and get functions. This allows developers to add custom functionality to these primitive types without having to wrap them in objects.

## Motivation

The proposed primitive proxy would simplify properties of reactive frameworks such as React or Vue. These frameworks rely heavily on the ability to create reactive properties that can update automatically in response to changes in the underlying data.

There are two solutions:
- React-like reactive properties use a getter to get the variable, and a function to set it.
- Vue-like reactive properties wrap the variable into a [Proxy](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Proxy), where the value can be set and get using the `value` property.

A primitive proxy would allow setting and getting a reactive variable seamlessly, here's an example:

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
console.log(myVar); // "Hello world !"
```

Here, `storedValue` is just for example purpose. The first argument of `PrimitiveProxy` is an object where you can store values that can be accessed with the `target` argument of `set` and `get`.

## API details

The `PrimitiveProxy` constructor takes two arguments: `target` and `handler`. `target` is an object to store values, and `handler` is an object containing the methods to intercept the operations performed on the target. The handler object can have `get` and `set` methods, which will be called when the target is accessed or modified.

```ts
PrimitiveProxy<TTarget, TPrimitive>(
    target: TTarget,
    handler: {
        set: (target: TTarget, value: TPrimitive) => void
        get: (target: TTarget) => TPrimitive
    }
);
```

## Questions

### Isn't it confusing not to be able to reassign a variable?

It's the same thing as standard Proxy. Setting a value to `myObject.thing` does not necessary mean that the `thing` property of `myObject` contains the set value, if the object is behind a Proxy.

### Why using a `let` instead of a `const`?

For consistency, `const` should not be updatable. So a `const` Primitive Proxy should disable the `set` handler.

### How to unproxify a primitive proxy?

Simply use the [`valueOf`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/valueOf) function it like this:
```js
let a = new PrimitiveProxy(...);
let b = a.valueOf(); // b is a standard primitive
```

### What happens when passing a Primitive Proxy to a function?

When passed to a function, the Primitive Proxy reference is copied just like an object.

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
console.log(a); // "primitive:Hello"

((value) => {
    console.log(value); // "primitive:Hello"
    value = "test";
    console.log(value); // "primitive:test"
})(a);

console.log(a); // "primitive:test"
```

### So, when is getter called?

The getter is called when the primitive value of the Primitive Proxy is needed.

Here are some examples:
```js
let a = new PrimitiveProxy({storedValue: ""}, {
    set: (target, value) => {
        console.log("set");
        target.storedValue = value;
    },
    get: (target) => {
        console.log("get");
        return target.storedValue;
    }
});

a = 5 // set

console.log(a) // get

const b = a; // copy Primitive Proxy
const c = a + 5 // get
const d = 5 + a // get
const e = "Hello" + a // get
const f = (a === 5) // get
```

## Additional examples

Seamless date formatting

```js
const date = new PrimitiveProxy(new Date(), {
  set: (target, value) => {
    target.setTime(value.getTime());
  },
  get: (target) => {
    const year = target.getFullYear();
    const month = target.getMonth() + 1;
    const day = target.getDate();
    return `${year}-${month}-${day}`;
  }
});

date = new Date("2023-04-01");
console.log(date); // "2023-04-01"
```

Seamless data validation

```js
const input = new PrimitiveProxy("", {
  set: (target, value) => {
    if (value > 10) {
      throw new Error("Input must be 10 characters or less.");
    }
    target = value;
  },
  get: (target) => {
    return target;
  }
})

try {
  input = "12345678901";
} catch (error) {
  console.error(error); // "Input must be 10 characters or less."
}

console.log(input); // ""
```

Seamless number rounding

```js
const number = new PrimitiveProxy(0, {
  set: (target, value) => {
    target = Math.round(value);
  },
  get: (target) => {
    return target;
  }
});

number = 3.14159;
console.log(number); // 3
```

## Limitations and Drawbacks

One potential limitation is that it may have performance implications. Proxies can be slower than direct property access, so using Primitive Proxies extensively in performance-critical code could lead to slower execution times.

Additionally, the proposal may introduce confusion or unexpected behavior when developers are not aware of the differences between Primitive Proxies and standard primitive types.
