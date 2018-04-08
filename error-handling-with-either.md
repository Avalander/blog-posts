# Error handling with Either

An `Either` is basically a container for a value that might be an error. With an `Either` we can apply transformations to the contained value without having to worry whether it is an error or not until we reach a point in our code where we want to handle the error, should it have happened. It's a bit like a [Schrödinger's box](https://en.wikipedia.org/wiki/Schr%C3%B6dinger%27s_cat): the value might or might not be an error, we won't know until we open it.

## How does Either work?

To illustrate the `Either` construct, let's build it in Javascript.

First of all, an `Either` can hold a value or an error. We'll call them `Right` and `Left` respectively. In a sense, it's like having two branches, and you go either to the left if you get an error, or to the right if you get a valid value.

Also, we need to be able to apply transformations to the value that is in the `Either`. Otherwise it's not really useful. We want a `map` function to do that. And we are going to apply the transformation only if we are on the `Right` branch, and ignore it if we have a `Left`.

```javascript
const Left = x => ({
    map: fn => Left(x),
})

const Right x => ({
    map: fn => Right(fn(x)),
})
```

Note that `Left.map` returns a `Left` holding the same value, without applying the transformation `fn`, while `Right.map` returns a `Right` containing the result of applying `fn` to the value. The reason for that is that we only want to apply the transformation on a valid value, not on an error.

```javascript
Right(3).map(x => x * x) // -> Right(9)
Left(3).map(x => x * x) // -> Left(3)
```

Now imagine that we want to apply a transformation to a value contained in an `Either`, but that transformation can return an error. Since we are handling error branches with `Either`, we might as well return a new `Either`.

```javascript
const result = Right(3)
    .map(x => x % 2 == 0
        ? Right(x)
        : Left('Odd'))
```

We have a number contained in an `Either` and we only want to accept even numbers. If it's odd, we return a `Left` saying that the number is odd.

The problem is that now we have a `Left` contained inside a `Right`. If we would inspect the variable `result` it would hold `Right(Left('Odd'))`. If we want to apply another transformation, should we apply it to the outer `Right` or to the inner `Left`? What happens when the next transformation returns another `Either`?

To solve this issue, we can implement the method `chain`. `chain` is much like `map`, but it expects the transformation to return an `Either`, so it doesn't wrap the result of applying the transformation in a new `Either`.

```javascript
const Left = x => ({
    map: fn => Left(x),
    chain: fn => Left(x),
})

const Right x => ({
    map: fn => Right(fn(x)),
    chain: fn => fn(x),
})
```

`Left.chain` still doesn't apply the transformation, and it returns a `Left` holding the error, so we're sure we are not going to operate on an error should it have happened.

`Right.chain` will apply the transformation `fn` to the contained value and return the result, without wrapping it in another `Right`, because it expects the function `fn` to return an `Either`. If we were implementing this in a real project, we would probably want to check that `fn` returns an `Either` and throw an error if it doesn't.

We can use `chain` in the previous example to make sure that we don't end up with an `Either` inside another `Either`.

```javascript
const result = Right(3)
    .chain(x => x % 2 == 0
        ? Right(x)
        : Left('Odd'))

result // -> Left('Odd')
```

Now we only have a `Left`, and we would have a `Right` if our value had been even.

And that's it. We can use `map` to apply transformations to our contained value and keep it inside the same `Either`, or `chain` if we want to apply a transformation that returns another `Either` because it might fail.

Even though it's nice to be able to operate over a value without caring whether it's an error or not, it's not really that useful if we can't access the value. Right now the value is contained forever in an `Either`, and we will never know if the operation succeeded and the transformations were applied to the value, or if we have an error waiting to be handled.

We can implement one last method to solve this issue: `fold`. `fold` takes two callbacks, the first one (or *left*) will be called if the `Either` contains an error and the second one (or *right*) will be called if the `Either` contains a valid value.

```javascript
const Left = x => ({
    map: fn => Left(x),
    chain: fn => Left(x),
    fold: (fnLeft, fnRight) => fnLeft(x),
})

const Right x => ({
    map: fn => Right(fn(x)),
    chain: fn => fn(x),
    fold: (fnLeft, fnRight) => fnRight(x),
})
```

If we have a `Left`, `fnLeft` will be invoked, so we can handle the error in that function. If we have a `Right`, `fnRight` will be invoked and we can use it to send the value in an HTTP response, or store it in a database or whatever we need that value for.

```javascript
Right(3)
    .chain(x => x % 2 == 0
        ? Right(`${x} is even.`)
        : Left('Odd'))
    .fold(
        console.error,
        console.log
    )
```

This simple example handles errors by printing them in `console.error`, and prints valid values in `console.log`, but we could handle errors and successes in any other way we need.

## Handy Either factories

There are a few common factories for `Either` that we can implement easily.

### Maybe

Maybe is a well known data structure, called **Optional** in some languages, that might or might not contain a value. We could model it with an `Either` that will be a `Right` if it has a value and an empty `Left` if it doesn't. Let's see how to build it.

```javascript
const maybe = value =>
    (value != null
        ? Right(value)
        : Left())
```

### TryCatch

Sometimes we might want to call a function that can throw an exception and treat the exception as an error with an `Either`. That might come in handy if we are using `Either` to handle errors in our code and need to interface with a library that handles errors by throwing exceptions (and expecting the user to catch them).

```javascript
const tryCatch = (fn, ...args) => {
    try {
        const result = fn.apply(null, args)
        return Right(result)
    } catch (e) {
        return Left(e)
    }
}
```

### Conditional

We might want to check if a value fulfills a certain condition and return an error if it doesn't. We can define a factory that will take a predicate (i.e., a function that checks a condition on the value an returns either `true` or `false`) and a value, and return a `Right` if the condition holds true for the given value and a `Left` otherwise. We can get a bit fancier and allow an extra argument with an error value (usually a message explaining why the value wasn't accepted) if the value doesn't fulfill the condition.

```javascript
const condition = (pred, value, reason) =>
    (pred(value)
        ? Right(value)
        : Left(reason))
```

Or, if you don't like ternary operators that much,

```javascript
const condition = (pred, value, reason) => {
    if (pred(value)) {
        return Right(value)
    }
    return Left(reason)
}
```

Remember the `maybe` factory that we implemented a bit earlier? Turns out that it's only a specific case of `condition`.

```javascript
const maybe = value =>
    condition(x => x != null, value)
```

## When to use Either

My personal opinion is that `Either` is simply a strategy to handle application errors, and choosing this or another strategy is more a matter of preference that anything else.

Some languages, like Python or Java, offer a well-thought exception system that can be used to handle any application errors that might happen. In these languages it's usually a good idea to keep things idiomatic.

Other languages don't have an exception system and expect the programmer to return an error value if a function call might return an invalid value (I'm looking at you, Go). Then I think it's better to use an `Either` than returning `(err, result)` and having to check for `err` every time we call a function, especially if we need to pass the error one layer up where it can be handled.

And then there is Javascript. It has an exception system. Sort of. The problem is that catching specific errors while letting others propagate with Javascript's exception system is not a trivial task. Therefore it might be worth to use `Either` for application errors and leave exceptions for programming errors, instead of catching exceptions and trying to figure out if it's an error that should be handled here, elsewhere or make the application crash.