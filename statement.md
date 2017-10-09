## Introduction

I wrote about <a href='https://dev.to/damcosset/i-promise-i-wont-callback-anymore-cp3' target='_blank'>promises</a> and <a href='https://dev.to/damcosset/introduction-to-generators-in-es6-5h1' target='_blank'>generators</a> being introduced in ES6. Another way to make asynchronous code look synchronous almost made it in ES6, but not quite: **async/await**. This functionality is built on top of promises. Let's take a look at it.

## Syntax

The syntax is as follow: you must declare a function to be `async`:

```javascript
const asyncFunction = async () => {
  // This is a good start
}

// or

const asyncFunction = async function(){
  // Old school function keyword? I like it!
}
```

Then, inside this `async` function, you can use the `await` keyword to tell the function it should wait for something:

```javascript
const asyncFunction = async () => {
  const step1 = await fetchingData() // Wait for this

  const step2 = await savingData() // Then wait for that

  // Do something else
}
```

## You can still keep your promises

I mentioned that **async/await** is build on top of promises. An `async` function returns a promise. This means you can call *.then()* and *.catch()* on them:

```javascript runnable
const fs = require('fs')

// promisify is a neat tool in the util module that transforms a callback function into a promise one
const { promisify } = require('util')
const writeFile = promisify(fs.writeFile)
const readFile = promisify(fs.readFile)

const writeAndRead = async () => {
  await writeFile('./test.txt', 'Hello World')
  const content = await readFile('./test.txt', 'utf-8')

  return content
}

writeAndRead()
  .then(content => console.log(content)) 
```

Ok, what is happening here?

- We create an `async` function called *writeAndRead*. 
- The function has two `await` keywords: first, we wait for the function to write to the file *test.txt*
- Second, we wait for the function to read the *test.txt* file we just wrote to.
- We store that in a variable and return it
- Because *async* functions return promises, I can use *.then()* after calling the *writeAndRead()* function.

Pretty sweet huh? We don't even need to specify a resolve() and reject() method anymore. Which brings me to the next point.

## You are all the same errors to me <3

Let's assume a scenario where you have promises-based logic and non-promises-based logic ( synchronous and asynchronous ) in your code. You would probably handle errors this way:

```javascript
const someComplicatedOperation = () => {
  try {
    // Blablabla do something
    db.getUsers()     //promise
    .then( users => {
      const data = JSON.parse( users )    // ===> What if this fail bro?
      console.log(data)
    })
    .catch( err => {
      console.log('You broke your promise!!!')
    })
  }
  catch( err ){
    console.log('I caught a error. But I only catch synchronous stuff here :(')
  }
}

```

That's right. The try/catch won't catch the JSON.parse error because it is happening inside a promise. A rejected promise triggers the *.catch()* method, but **NOT** the other catch. That's annoying, we have to duplicate code for catching errors. Well, that time is now over with *async/await*!

```javascript
const allErrorsAreDeclaredEqualInTheEyesOfAsyncAwait = async () => {
  try {
    const users = await db.getUsers
    const data = JSON.parse( users )
    console.log(data)
  }
  catch( err ){
    console.log('All errors are welcomed here! From promises or not, this catch is your catch.')
  }
}
```

Clean, concise and clean, while being concise. Good old try/catch can handle all the errors we can throw.

## How high can you stack them errors?

As developers, if there is one thing we love, it is an infinite amount of functions in an error stack. It is probably not a huge deal, but more like a nice thing to know when you work with async/await. Check it out:

```javascript runnable

const usefulPromise = () => {
    return new Promise( ( resolve, reject ) => { resolve() })
}

const stackingAllTheWayToTheSky = () => {
  return usefulPromise()
    .then(() => usefulPromise())
    .then(() => usefulPromise())
    .then(() => usefulPromise())
    .then(() => usefulPromise())
    .then(() => usefulPromise())
    .then(() => {
      throw new Error('I can see my house from here!!')
    })
}

stackingAllTheWayToTheSky()
  .then(() => {
    console.log("You won't reach me.")
  })
  .catch(err => {
    console.log(err) // FEEL THE PAIN!
  })

```

Now with async/await:

```javascript runnable
const usefulPromise = () => {
    return new Promise( ( resolve, reject ) => { resolve() })
}

const debuggingMadeFun = async () => {
  await usefulPromise()
  await usefulPromise()
  await usefulPromise()
  await usefulPromise()
  await usefulPromise()
  throw new Error('I will not stack.')
}

debuggingMadeFun()
  .then(() => {
    console.log('Not here')
  })
  .catch(err => {
    console.log(err)
  })

```

Ain't that much much more cleaner and easy to read?

## Values in the middle

You probably wrote some code where you executed one operation and used that to execute a second one. Finally, you need those two values for the third and final operation. So, you may write something like that:

```javascript runnable
const firstPromise = () => Promise.resolve( 43 )
const secondPromise = value => Promise.resolve( value + 100 ) 
const thirdPromise = ( value1, value2 ) => Promise.resolve( value1 + value2 + 100 ) 

const withPromises = () => {
  return firstPromise()
    .then( firstValue => {
      return secondPromise( firstValue )
    })
    .then( secondValue => {
      return thirdPromise( firstValue, secondValue )
    })
    .then( result => console.log( result ) )
    .catch( err => console.log(err))
}
// Or using Promise.all. It's a bit ugly, but the job is done

const withPromiseAll = () => {
  return firstPromise() 
    .then(firstValue => {
      return Promise.all([ firstValue, secondPromise(firstValue) ])
    })
    .then(([firstValue, secondValue]) => {
      return thirdPromise(firstValue, secondValue)
    })
    .then( result => console.log( result ) )
    .catch( err => console.log(err))
}

withPromises()
withPromiseAll()
```

Let's look at how much better it is with async/await:

```javascript
const withAsyncAwait = async () => {
  const firstValue = await firstPromise()
  const secondValue = await secondPromise()
  return thirdPromise( firstValue, secondValue )
}
```

Do I need to say more?

## Conclusion

Well, async/await is a very cool way to write asynchronous code in Javascript. You can try it out in Node.js because it is natively supported since version 7.6. Have fun!!
