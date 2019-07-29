---
title: Introduction to unit testing with tape, mocks
published: true
description: 
tags: tutorial, beginners, unit testing, javascript
cover_image: https://thepracticaldev.s3.amazonaws.com/i/1frz4aaqmylap0fux6q1.png
series: Introduction to unit testing with tape
---

In this part we are going to learn how to write unit tests for functions that have side effects and/or are asynchronous.

We will continue working with [Minuette](https://github.com/Avalander/Minuette), so go ahead and clone the project if you haven't done so already. We will write test for the module `store`, so take a look at the files under the folder `src/store`.

# What does the store module do?

The module exports two functions, `loadData` and `saveData`, that are used to get the to-do list from a json file and to save the to-do list to that same file, respectively.

The problem with these functions is that they interact with the file system. If we want to test that `saveData` behaves as we expect, we would have to invoke the function and then look for the file in our file system. This is inconvenient and makes our tests run slower.

But we can avoid all that overhead if we _mock_ the interaction with the file system. After all, we should trust the developers of node to have properly tested `fs.writeFile` and `fs.readFile`, so no need for us to test them again and again.

A _mock_ is a function that is used in a unit test instead of a dependency that adds complexity. In our case, we are going to use mocks to replace all the functions that interact with the file system.

The function `saveData` is declared in `src/store/index.factory.js` as follows:

```javascript
module.exports.saveData = ({ writeFile, store_path }) => data => {...}
```

This means that we export a function that receives an object with `writeFile` and `store_path` and returns another function that receives a `data` argument.



, which the function will eventually use to write the data into the file, and `store_path`, which is the file where the function is going to save the data.
