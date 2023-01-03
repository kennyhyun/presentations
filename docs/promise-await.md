<!--
theme: gaia
paginate: true
html: true
style: |
  section pre code {
    --marpit-root-font-size: 17px;
  }
  section {
    font-size: 27px;
  }
-->

# Promises and await

### Oct 2017, Kenny Yeo
### Aug 20202

---

## Contents

- Using callback
- ES6 promise
- ES7 await
- More about promises
- Running from array

---

## using callback

```js [3-100]
const fs = require('fs')

fs.readFile('file.md', 'utf-8', (err, content) => {
  if (err) return console.error(err)
  try {
    console.log(JSON.parse(content))
  } catch (err) {
    console.error(err)
  }
  fs.readFile('file2.md', 'utf-8', (err, content2) => {
    if (err) return console.error(err)
    try {
      console.log(JSON.parse(content2))
    } catch (err) {
      console.error(err)
    }
  });
});
```

Note the nested structure.
[async waterfall](https://caolan.github.io/async/v3/docs.html#waterfall) utility can be used for handling multiple async functions sequencially

---

## es6 promise
                
```js [3-010]
const { promises: fsp } = require('fs')

fsp.readFile('file.md', 'utf-8')
  .then(content => JSON.parse(content))
  .then(json => console.log(json))
  .then(() => readFile('file2.md', 'utf-8'))
  .then(content => JSON.parse(content))
  .then(json => console.log(json))
  .catch(err => console.error(err))
```

- any exception will be caught in the last catch block
- console.log returns void, any return value in then/catch block become promise
- [Promise.all](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise/all) and [race](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise/race) can be also used for resolving multiple async functions

---

## es6 promise (concurrency)

```js [3-010]
const { promises: fsp } = require('fs')

Promise.all([
  fsp.readFile('file.md', 'utf-8')
    .then(content => JSON.parse(content))
    .then(json => console.log(json)),
  fsp.readFile('file2.md', 'utf-8')
    .then(content => JSON.parse(content))
    .then(json => console.log(json)),
]).catch(err => console.error(err));
```

---


## es7 await

```js [3-010]
const { promises: fsp } = require('fs')

(async () => {
  const content = await fsp.readFile('file.md', 'utf-8')
  console.log(JSON.parse(content))
  const content2 = await fsp.readFile('file2.md', 'utf-8')
  console.log(JSON.parse(content2))
})()
  .catch(err => console.error(err))
```

- Only sequential execution is available.
- However, since the return value of async function is actually Promise
  - Promise.all/race can be still used.


---

## es7 await (concurrency)

```js [3-010]
const { promises: fsp } = require('fs')

Promise.all([
  (async () => {
    const content = await fsp.readFile('file.md', 'utf-8')
    console.log(JSON.parse(content))
  })(),
  (async () => {
    const content2 = await fsp.readFile('file2.md', 'utf-8')
    console.log(JSON.parse(content2))
  })(),
]).catch(err => console.error(err))
```


---


## es7 await

### Async returns promise

```js [6-010]
const { promises: fsp } = require('fs')

const printJsonFile = async (file, enc = 'utf-8') =>
  console.log(await fsp.readFile(file, enc));
  
Promise.all([
  printJsonFile('file.md'),
  printJsonFile('file2.md')
])
  .catch(err => console.error(err))

printJsonFile('file.md')
  .then(() => printJsonFile('file2.md'))
  .catch(err => console.error(err))
```


---


## More about promises

```js
Promise.resolve()
// -> Promise {<fulfilled>: undefined}

Promise.resolve().then(() => 5)
// -> Promise {<fulfilled>: 5}

(async() => console.log(await 5))()
// -> 5
// -> Promise {<fulfilled>: undefined}

(async () => JSON.parse('asdasf'))()
  .catch(e => console.error(e.message))
// -> Unexpected token a in JSON at position 0
// -> Promise {<fulfilled>: undefined}
```

---

## Running from array

### Fetch one by one and stop if failed

```js
[0,1,2,3,4,5,6,7,8,9]
  .reduce(
    async (p, i) => {
      await p; // wait the previous fetch response
      return fetch(`http://server/doc/${i}`)
        .then(resp => resp.json())
        .then(json => console.log(`${i}:`, json)
    },
    Promise.resolve()
  )
  .catch(e => console.error(e))
```

---

## Running from array

### Fetch all without awaiting even if failed

```js
[0,1,2,3,4,5,6,7,8,9]
  .map(i =>
    // request without waiting previous response
    fetch(`http://server/doc/${i}`).then(resp => resp.json())
  )
  .reduce(
    async (p, pJson, i) => {
      await p
      const json = await pJson
        .catch(e => `Error: ${e.message}`);
      console.log(`${i}:`, json);
    },
    Promise.resolve()
  )
  .catch(e => console.error(e))
```

---


## Learn more

- [ES5 async library](https://caolan.github.io/async/v3/docs.html#waterfall)
- [Promise.all](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise/all)
- [Promise.race](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise/race)
- [Awaiting in C++ (coroutines)](https://en.cppreference.com/w/cpp/language/coroutines)
- [C# await(ver 5~)](https://docs.microsoft.com/en-us/dotnet/csharp/language-reference/operators/await)
