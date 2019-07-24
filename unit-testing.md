---
title: Introduction to unit testing with tape, part 1
published: false
description: 
tags: tutorial, beginners, unit testing, javascript
cover_image: https://thepracticaldev.s3.amazonaws.com/i/1frz4aaqmylap0fux6q1.png
series: Introduction to unit testing with tape
---

If you have never heard about unit testing, or have just heard about it and have no idea where to start, this post is for you. Here I will introduce the basic concepts of unit testing and I'll show a practical example with [tape](https://github.com/substack/tape) to get you started.

# What is unit testing?

Unit testing could be roughly summarized as writing code that test code units. A _code unit_ is basically an individual component, more often than not, a function. The purpose of a unit test is to validate that the code unit performs as expected by executing it with crafted inputs and validating the output.

Unit testing is a desirable practice for multiple reasons. For starters, the behaviour of the component can be validated quickly and automatically, which is useful especially after changing the component to accomodate for new functionality. Also, the behaviour of the component is documented in the tests, so they can be used as a reference by any developer using the component in question.

It's worth mentioning that unit testing is a lot easier with [pure functions](https://en.wikipedia.org/wiki/Pure_function). Therefore, it is a good practice to try and keep most of the components in a codebase as pure as possible.

# Let's get started

First of all, you need to have node installed. You can download it from the [official site](https://nodejs.org/en/download/) or use [nvm](https://github.com/nvm-sh/nvm/blob/master/README.md) to manage multiple versions of node.

Secondly, we are going to use a toy project of mine, [Minuette](https://github.com/Avalander/Minuette.git). It is a very simple shell to-do application. Go ahead, clone the project and try it to see how it works.

```bash
git clone https://github.com/Avalander/Minuette.git
cd Minuette
npm install
npm start
```

You can run the different commands with `node start add 'text'`, `node start complete 0` and `node start list`.

Lastly, we need a test runner. We are going to use [tape](https://github.com/substack/tape) because it's simple and straightforward. We will also use a pretty reporter. I like tap-dot, but you can check [this list](https://github.com/substack/tape#things-that-go-well-with-tape) and try a different one.

```bash
# Run in the project's root folder
npm install -D tape tap-dot
```

# Our first unit test

Now we are ready to go. The first function we will test is `sortItems`, which is located in `src/sort.js`.

Let's create a folder named `test` and place a file called `sort.test.js` inside. Then we will write our tests in that file.

We will start by importing the `tape` module.

```javascript
const test = require('tape')
```

The `tape` module returns a function that receives two arguments: a string describing the test case, and a function to execute the text case.

```javascript
test('This is my first test #yolo', t => {
  t.plan(3)
  t.equal(3, 3)
  t.deepEqual([ 1, 2, 3 ], [ 1, 2, 3 ])
  t.pass('We good')
  t.end()
})
```

The argument passed on to the test function, `t`, is an object with several assertions that we can use to perform our test. These are some of the assertions we can use, check [the docs](https://github.com/substack/tape#methods) for a complete list.

- `plan` receives an integer, it makes the test fail if more or fewer assertions than the number set are executed.
- `equal` checks that two values are equal. It does not work well with arrays and objects, for those you need
- `deepEqual` is like `equal` but it works on arrays and objects.
- `pass` always passes.
- `end` signals the end of the test.

It's important to notice that a test function must use either `plan` or `end`.

# How about we write that test?

Of course, let's test the function `sortItems`. `sortItems` receives an array of objects with the structure `{ status, text, timestamp }` and sorts them according to the following criteria:
1. Items with `status` `'done'` are sent to the end of the array.
2. Items with the same `status` are sorted according to `timestamp` in ascending order.

So we can write a test case to check the first criteria.

```javascript
const test = require('tape')

const { sortItems } = require('../src/sort')

test('sortItems should place items with status done at the back', t => {
  const result = sortItems([
    { status: 'done' },
    { status: 'todo' },
  ])

  t.deepEqual(result, [
    { status: 'todo' },
    { status: 'done' },
  ])
  t.end()
})
```

There we go. This test will call `sortItems` with an array that contains two items and check that they are sorted according to the first criteria with `t.deepEqual`. Then we call `t.end` to signal that we are done.

To run the test, simply type the following command in the console and check the output.

```bash
npx tape test/**/*.test.js | npx tap-dot
```

To simplify further executions, you can update the `"test"` script in the file `package.json` to `"tape test/**/*.test.js | tap-dot"` and then run your tests simply typing `npm test`.

Let's wirte a test to check the second sorting criteria. Given two items with the same status, they should be sorted according to their timestamp, in ascending order.

```javascript
test('sortItems should order items with the same status according to timestamp', t => {
  const result = sortItems([
    { status: 'todo', timestamp: 800 },
    { status: 'todo', timestamp: 500 },
  ])

  t.deepEqual(result, [
    { status: 'todo', timestamp: 500 },
    { status: 'todo', timestamp: 800 },
  ])
  t.end()
})
```

# More tests

We could be satisfied with our tests for `sortItems`, but we have only tested it with two arrays with two items. These hardly cover all thinkable inputs that this function will have to operate over. Let's try something else.

First, we will create an array with a few more items, let's say ten.

```javascript
const items = [
  { status: 'todo', text: 'Rainbow Dash thinks Fluttershy is a tree.', timestamp: 1000 },
  { status: 'todo', text: 'I simply cannot let such a crime against fabulousity go uncorrected.', timestamp: 1100 },
  { status: 'todo', text: `Huh? I'm pancake...I mean awake!`, timestamp: 1200 },
  { status: 'todo', text: `Don't you use your fancy mathematics to muddy the issue!`, timestamp: 1300 },
  { status: 'todo', text: `Reading's for eggheads like you, Twilight. Heh, no offense, but I am *not* reading. It's undeniably, unquestionably, uncool.`, timestamp: 1400 },
  { status: 'done', text: 'Too old for free candy? Never!', timestamp: 1000 },
  { status: 'done', text: 'Trixie is the highest level unicorn!', timestamp: 1100 },
  { status: 'done', text: `I'd like to be a tree.`, timestamp: 1200 },
  { status: 'done', text: 'Ha! Once again, the Great and Powerful Trixie has proven herself to be the most amazing unicorn in all of Equestria. Was there ever any doubt?', timestamp: 1300 },
  { status: 'done', text: 'What the hay is that supposed to mean?', timestamp: 1400 },
]
```

Notice that the array is sorted according to the criteria we have defined. Next we can shuffle it randomly a few times and check that the output of `sortItems` always equals the sorted array.

Sadly, node doesn't have any `shuffle` function, so we will have to implement our own.

```javascript
const shuffle = ([ ...items ]) => items.sort(() => Math.random() - 0.5)
```

Notice how we use destructuring and the spread operator in `([ ...items ])`. This will make a shallow copy of the array. We need to do it this way because `Array.sort` sorts the array in place, so if we wouldn't make a copy, it would shuffle our reference array and it would be useless to test against the output of `sortItems`.

Then we use `items.sort`, which receives a function that receives two arguments, let's call them `a` and `b`, and should return a number greater than `0` if `a` precedes `b`, lesser than `0` if `b` precedes `a` and `0` if both have the same precedence.

In our case, we want to shuffle the array, not sort it, so we don't care about the input arguments and just return `Math.random() - 0.5`, which will generate a random number between `-0.5` and `0.5`. It is perhaps not the best algorithm to generate very shuffled results, but for demonstration purposes it will suffice.

Now to the test case.

```javascript
test('sortItems sorts a randomly shuffled array', t => {
  const input = shuffle(items) // Remember how we declared items a few lines above?
  const result = sortItems(input)

  t.deepEqual(result, items)
  t.end()
})
```

And _voilà_, we have a test that verifies that a randomly shuffled list of ten items is always sorted properly.

We can even go a step further and test several permutations of the `items` array.

```javascript
for (let i = 0; i < 20; i++) {
  test('sortItems sorts a randomly shuffled array', t => {
    const input = shuffle(items)
    const result = sortItems(input)

    t.deepEqual(result, items)
    t.end()
  })
}
```

# Summary

In this tutorial we have learned the most basic functionality of the test runner [tape](https://github.com/substack/tape) to write and execute unit tests, and we have created unit tests for the function `sortItems`, which happens to be a pure function. In the next part of this series we will see how to test functions that produce side effects like printing to the console or reading files.

# Challenges

- Try out different test reporters from [this list](https://github.com/substack/tape#things-that-go-well-with-tape) and pick the one you like the best.
- Instead of shuffling the `items` array, generate all possible permutations for that array and run the test case for each one of them.
