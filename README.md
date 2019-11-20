# asyncresult-js

Small set of utils for working with promises.

![version](https://img.shields.io/github/package-json/v/taburetkin/asyncresult.svg)
[![Coverage Status](https://coveralls.io/repos/github/taburetkin/asyncresult/badge.svg?branch=master)](https://coveralls.io/github/taburetkin/asyncresult?branch=master)
![Build status](https://secure.travis-ci.org/taburetkin/asyncresult.svg?branch=master)


## examples:

usual promise approach:

```js
fetchSomething()
  .then(() => {
    fetchSomethingElse()
      .then(() => {
        fetchLastThing()
          .then(data => {
            showTheData(data);
          })
          .catch(err => {
            throw err;
          });
      })
      .catch(err => {
        throw err;
      });
  })
  .catch(err => {
    throw err;
  });
```

usual async/await approach

```js
// suppose you are in some async context
async () => {
  try {
    await fetchSomething();
  } catch (err) {
    throw err;
  }

  try {
    await fetchSomethingElse();
  } catch (err) {
    throw err;
  }

  try {
    let data = await fetchLastThing();
    showTheData(data);
  } catch (err) {
    throw err;
  }
};
```

and how it can be done with asyncresult-js

```js
import { wrapMethod } from 'asyncresult-js';
// wrapping methods with special util
const fetchSomethingAsync = wrapMethod(fetchSomething);
const fetchSomethingElseAsync = wrapMethod(fetchSomethingElse);
const fetchLastThingAsync = wrapMethod(fetchLastThing);

// assume we are in some async context
async () => {
  let res = await fetchSomethingAsync();
  if (res.isError()) {
    throw res.err();
  }

  res = await fetchSomethingElseAsync();
  if (res.isError()) {
    throw res.err();
  }

  res = await fetchLastThingAsync();
  if (res.isError()) {
    throw res.err();
  } else {
    showTheData(res.val());
  }
};
```

---

# install

```
npm i asyncresult-js
```

or

```
yarn add asyncresult-js
```

# content

## class AsyncResult - constructor(error, value)

> every wrapped and awaited method returns instance of AsyncResult

### instance methods

- **isError()**  
  returns `true` if there is error, empty string treated as error.
- **isOk()**  
  returns `true` if there is no error
- **isEmpty()**  
  returns `true` if there is no error and any value.
- **hasValue()**  
  returns `true` if there is any value. empty string treated as value.

- **err()**  
  returns error.
- **val()**  
  returns value.
- **errOrVal()**  
  returns error or value in such order.

- **setValue(value)**  
  sets given value.
- **setError(err)**  
  sets given error.

### static methods

- **AsyncResult.success(value)**  
  shorthand for `new AsyncResult(null, value)`
- **AsyncResult.fail(erro)**  
  shorthand for `new AsyncResult(error)`

example

```js
import { AsyncResult } from 'asyncresult-js';
let r1 = new AsyncResult(null, "foo"); // same as AsyncResult.success('foo')
console.log(r1.isError()); // false
console.log(r1.val()); // 'foo'

let r2 = new AsyncResult("foo"); // same as AsyncResult.fail('foo')
console.log(r1.isError()); // true
console.log(r1.err()); // 'foo'
```

## util toAsyncResult(arg, AsyncResultClass)

> converts given argument to a promise which will be resolved or rejected with AsyncResult instance.  
> If `arg` is instanceof `Error` then it will goes to the error value

**arg** - any, required  
**AsyncResultClass** - optional, pass if you need your own extended AsyncResult

examples

```js
import { toAsyncResult } from 'asyncresult-js';

// assume we are in some async context
async () => {
  let mypromise = Promise.resolve("foo");
  let value = await mypromise; // 'foo';
  let mypromiseAsync = toAsyncResult(mypromise);
  value = await mypromiseAsync; // AsyncResult { value: 'foo' }
};
```

## util wrapMethod(method, options)

> returns new method which always return promise which will resolve with AsyncResult

**method** - function, required.  
**options** - object, optional.

### options

**context** - binds returned new function to this context.  
**AsyncResult** - AsyncResult constructor, provide it if you need extended version of AsyncResult

examples

```js
import { wrapMethod } from 'asyncresult-js';

const myMethod = function() {
  return "foo";
};

const myAsyncMethod = wrapMethod(myMethod);

let result = myAsyncMethod(); // returns promise;

// assume we are in some async context
async () => {
  result = await myAsyncMethod(); // returns AsyncResult with value 'foo';
};
```

## util addAsync(context, methodNames, options)

> adds async version of exist methods, founded by given names

example:

```js
import { addAsync } from 'asyncresult-js'

const Something = {
	foo() { ... },
	bar() { ... },
	baz() { ... },
}

addAsync(Something, ['foo', 'bar']); // adds to Something fooAsync, and barAsync methods

addAsync(Something, 'baz'); // adds to Something bazAsync method

```

in general its a sugar for this operation

```js
const Something = {
	foo() { ... },
}
Something.fooAsync = wrapMethod(Something.foo, { context: Something })
```

Also, its possible to extend prototype of some class  
You can do this in two ways
1. patch prototype
```js
addAsync(SomeClass.prototype, [...], { context: null });
```
Note, that there should be null context specified in options.  

2. pass class itself as argument
```js
addAsync(SomeClass, [...]);
```
actually its just a suggar for case 1. 

**Warning**: Note, that both modifies prototype so you should know what are you doing.

In case you want to add class static methods you have to use option `static`

```javascript
// assume that SomeClass.staticMethod is a function.
addAsync(SomeClass, 'staticMethod', { static: true });
await SomeClass.staticMethodAsync();
```

### arguments

**context** - object, required. Context for methods lookup and for adding asynced methods.  
**methodNames** - string | array of strings, required. Array of methods names or single method name to be wraped.  
**options** - object, optional.

### options

**context** - forced context, will be used internally in `wrapMethod` as context.  
**AsyncResult** - AsyncResult class, in case you need extend version of AsyncResult.

## config

> keeps default AsyncResult class, you can replace it with your own.  
> Used by default if there is no AsyncResult option provided.

example

```js
import { config, wrapMethod } from 'asyncresult-js';
config.AsyncResult = MyOwnAsyncResult;

const test = () => true;
const testAsync = wrapMethod(test);

// assume we are in some async context
async () => {
  const result = await testAsync();
  console.log(result instanceof MyOwnAsyncResult); // true
};
```
