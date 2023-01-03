# Promise/await

Oct 2017, Kenny Yeo 

Contents

- traditional node asynchronous
- es6 promise
- es7 await
- waiting for resolving async
            
---

## traditional node asynchronous

```js
const fs = require('fs')
fs.readFile('file.md', 'utf-8', (err, content) => {
  if (err) {
    console.log('error happened during reading the file')
    return console.log(err)
  }
  console.log(content)
  fs.readFile('file2.md', 'utf-8', (err, content2) => {
    if (err) {
      console.log('error happened during reading the file')
      return console.log(err)
    }
    console.log(content2)
  }
})
```

<a href="https://caolan.github.io/async/v3/docs.html#waterfall" target="_blank">async</a>
utility can be used for handling multiple async functions

---

## es6 promise

```js
const fs = require('fs')
const promisify = require('es6-promisify')
const readFile = promisify(fs.readFile)
readFile('file.md', 'utf-8')
.then(content => {
  console.log(content);
  return readFile('file2.md', 'utf-8');
})
.then((content2) => {
  console.log(content2);
})
.catch(err => {
  console.log('error happened during reading the file')
  console.log(err)
});
```

<a target="_"
href="https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise/all">
Promise.all</a> and
<a target="_"
href="https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise/race">
race</a> can be also used for handling multiple async functions

---

## es6 promise

```js
const fs = require('fs')
const promisify = require('es6-promisify')
const readFile = promisify(fs.readFile)
Promise.all([
  readFile('file.md', 'utf-8'),
  readFile('file2.md', 'utf-8'),
])
.then(([ content, content2 ]) => {
  console.log(content);
  console.log(content2);
})
.catch(err => {
  console.log(err)
});
```

---

## es6 promise

```js
const fs = require('fs')
const promisify = require('es6-promisify')
const readFile = promisify(fs.readFile)
readFile('file.md', 'utf-8')
.then(content => {
  console.log(content);
  if (new Date().getDay() !== 5)
    throw new Error('it is not Friday');
  return readFile('file2.md', 'utf-8');
})
.then((content2) => {
  console.log(content2);
})
.catch(err => {
  console.log(err)
});
```

---

## es7 await

```js
const fs = require('fs')
const promisify = require('es6-promisify')
const readFile = promisify(fs.readFile)
const asyncReadFile = async () => {
  try {
    console.log(await readFile('file.md', 'utf-8'))
    console.log(await readFile('file2.md', 'utf-8'))
  } catch (err) {
    console.log(err)
  }
};
asyncReadFile()
```

Only sequential execution is available.

However, since the return value of async function is actually Promise,
Promise.all/race can be still used.
