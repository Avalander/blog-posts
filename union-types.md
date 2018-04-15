---
title: Union Types in Javascript
published: false
description: 
tags: javascript, functional, union types, adt
cover_image: https://68.media.tumblr.com/6eb7fb9f4587aafdf339f99ea6966e05/tumblr_nrovyt8g2T1qd5ajyo1_1280.jpg
---

*(Cover image: __Lanterns__, by Anna Sánchez. Original picture [here](http://annasanchez.tumblr.com/post/124411134425/lanterns-2013-h%E1%BB%99i-an-vietnam-h%E1%BB%99i-an-at-night))*

Lately I've been learnig [Elm](http://elm-lang.org/) and I'm entirely fascinated by its [union types](https://guide.elm-lang.org/types/union_types.html). In this article I'll show a way to implement union types in Javascript and explain through examples how they could be useful.

# What are union types?

Union types, also known as algebraic data types (or ADTs), are a way to express complex data that can take multiple forms. I won't dive deep into union types theory, but this [Wikipedia article](https://en.wikipedia.org/wiki/Algebraic_data_type) does an excelent job at explaining them.

All you need to know for now is that a union type is a type that allows us to represent and categorise data that can take multiple forms, much like an *enum*, but more powerful.

# How to implement union types in Javascript

Before looking into why union types are useful and how to use them, let's try to implement them in Javascript. Here I've implemented a helper function that I call `union`. It receives a list of type names and returns an object describing the union type.

```javascript
const union = types =>
    types.reduce((prev, type) => ({
        ...prev,
        [type]: data => ({
            match: fns => fns[type](data),
        }),
    }), {})
```

If you're not familiar with how `reduce` works, you should watch [this video](https://www.youtube.com/watch?v=Wl98eZpkp-c), but here's a roughly equivalent version using a for loop.

```javascript
const union = types => {
    const result = {}
    for (let type of types) {
        result[type] = data => ({
            match: fns => fns[type](data),
        })
    }
    return result
}
```

This function is creating an object with a type for each name in the `types` array. Each type is a factory that can receive some data and returns an object with a method `match`. The method `match` will receive an object with a function for each available type, and then execute the function for the specific type that the object belongs to.

Now we can use the `union` helper to create union types.

Let's illustrate how this would work with a silly example. Imagine that we need to be able to process data about ponies. As everybody knows, there are three different kinds of ponies: earth ponies, pegasi and unicorns. Each type has some specific abilities particular to their kind. For instance, pegasi can fly and unicorns can use magic.

```javascript
const Ponies = union([
    'EarthPony',
    'Pegasus',
    'Unicorn',
])

const twilight = Ponies.Unicorn({
    name: 'Twilight Sparkle',
    spell: 'Levitation',
})
const rainbow = Ponies.Pegasus({
    name: 'Rainbow Dash',
    speed: 20,
})

twilight.match({
    EarthPony: ({ name }) => `${name} is a peaceful earth pony.`,
    Pegasus: ({ name, speed }) => `${name} flies at a speed of ${speed}!`,
    Unicorn: ({ name, spell }) => `${name} uses ${spell}!`,
}) // -> 'Twilight Sparkle uses Levitation!'
```

We can use the method `match` to execute specific logic depending on the kind of pony that we have. Similar to how we would use a `switch` statement on an `enum` in Java, but with the added benefit that each type can have associated a different type data.

# Usage examples

Let's look at a couple sligthly less silly examples to get an idea of how a union type could be used in a real application.

## Example 1: handle errors in node

Let's pretend we are building a REST API using node and express.js. Our API has an endpoint that returns a pony from the database by id.

Our express app looks something like this.

```javascript
const mongodb = require('mongodb')
const express = require('express')

const app = express()

mongodb.MongoClient.connect(DB_URL)
    .then(client => client.db(DB_NAME))
    .then(db => {
        app.get('/ponies/:id', /* here be our endpoint */)
        app.listen(3000, () => 'Server started.')
    })
```

If you're not familiar with express, don't worry. All you need to know is that we are going to implement a function that will receive a request object (we'll call it `req`) and a response object (we'll call it `res`) and that function will also have access to a database connection called `db`.

Our function will check that the user is authentified, because our pony database holds very sensitive information. Then, it will read the `id` parameter from the path and get the pony with that id from the database. Finally, it will send the pony data back in the response.

There are at least three things that can go wrong.

1. The user session might have expired, or the user might be trying to access the API without a valid token.
2. There might be no pony in the database with the given id.
3. We might have an unexpected failure. The database could be down, for example.

Let's create a union type that will model these three types of errors.

```javascript
const ApiError = union([
    'InvalidCredentials',
    'NotFound',
    'Other',
])
```

If the user is not properly authenticated, we'll return an `InvalidCredentials` error. If the pony doesn't exist in the database, we'll return a `NotFound`. We'll group all unexpected errors in `Other`.

Let's look at the first step. Let's assume that we have a function called `authorise` that checks a user token and returns `true` if it's valid and `false` otherwise, and that we have some middleware that reads the user token from a header or a cookie and stores it in `req.bearer`. We will wrap the call to `authorise` in a promise because we have some asynchronous operations and we want to handle all errors through the rejection branch of the promise.

```javascript
app.get('/ponies/:id', (req, res) =>
    new Promise((resolve, reject) => {
        if (authorise(req.bearer)) return resolve()
        return reject(ApiError.InvalidCredentials())
    })
)
```

So far so good. If the user is not properly authentificated, the promise will be rejected and we won't execute the rest of the chain. Otherwise, we can now read the pony from the database. We will wrap a call to the database in another promise and resolve it with the data if we find any in the database, otherwise we will reject with a `NotFound` error.

```javascript
app.get('/ponies/:id', (req, res) =>
    new Promise((resolve, reject) => {
        if (authorise(req.bearer)) return resolve()
        return reject(ApiError.InvalidCredentials())
    })
    .then(() => new Promise((resolve, reject)) =>
        db.collection('ponies').findOne({ id: req.params.id }, (err, data) => {
            if (err) {
                return reject(ApiError.Other(err))
            }
            if (data == null) {
                return reject(ApiError.NotFound(`Pony ${req.params.id} not found.`))
            }
            return resolve(data)
        })
    )
)
```

The node callback can return an error if something goes wrong, so if there is anything in the parameter `err`, we'll reject the promise with an `Other` error. If the operation was successfull, we might still get no data back if there was no record in the database, then we will reject the promise with a `NotFound` error. Otherwise, we will have some data and we can resolve the promise with it.

The next step is to send the data back in the response if everything went well, otherwise we want to send a HTTP error depending on what went wrong.

```javascript
app.get('/ponies/:id', (req, res) =>
    new Promise((resolve, reject) => {
        if (authorise(req.bearer)) return resolve()
        return reject(ApiError.InvalidCredentials())
    })
    .then(() => new Promise((resolve, reject)) =>
        db.collection('ponies').findOne({ id: req.params.id }, (err, pony) => {
            if (err) {
                return reject(ApiError.Other(err))
            }
            if (pony == null) {
                return reject(ApiError.NotFound(`Pony ${req.params.id} not found.`))
            }
            return resolve(pony)
        })
    )
    .then(pony => res.json(pony))
    .catch(err => err.match({
        InvalidCredentials: () => res.sendStatus(401),
        NotFound: message => res.status(404).send(message),
        Other: e => res.status(500).send(e)
    }))
)
```

And that's it. If we get an error in the rejection branch, we can use the method `match` to send a relevant HTTP status code and a different message.

If we're honest, this is not very impressive. We could have done exactly the same with an enum-like object. Even though I think that the type matching is rather elegant, it doesn't make a big difference compared to a goold ol' `switch` statement.

You can check the full example in this [GitHub repo](https://github.com/Avalander/union-types-example/tree/master/examples/node).

## Example 2: fetch remote data in a React component

Let's try a different example, then. Pretend that we have a React component that loads some data from a remote server. If you think about it, this component could have one of four states:

1. **Not asked**. The data has not yet been asked to the server.
2. **Pending**. The data has been asked to the server, but no response has been received yet.
3. **Success**. The data has been received from the server.
4. **Failure**. An error has happened somewhere during the communication.

Let's model this with a union type.

```javascript
const RemoteData = union([
    'NotAsked',
    'Pending',
    'Success',
    'Failure',
])
```

There we go. Now we want to create a React component that will load with the state `NotAsked` and render different stuff depending on the state.

```javascript
class Pony extends React.Component {
    constructor(props) {
        super(props)
        this.state = {
            data: RemoteData.NotAsked()
        }
    }
}
```

We create a component that will hold some data and start with the state `NotAsked`. Let's render that state. We probably want a text telling the user to load the data and a button to trigger the call to the server.

```javascript
class Pony extends React.Component {
    // previous code here...
    render() {
        return this.state.data.match({
            NotAsked: () => (
                <div>
                    <h1>Press "load"</h1>
                    <button onClick={this.fetchData}>Load!</button>
                </div>
            )
        })
    }
}
```

You might have noticed that `onClick={this.fetchData}` in the `button`. When the user presses the button, we want to trigger the request to the server, so we need to add a `fetchData` method to the component. But first, let's create a function that will simulate a call to the server, since we don't have an actual server to call.

```javascript
const fetchPony = () => new Promise((resolve, reject) =>
    setTimeout(() => {
        if (Math.random() > 0.2) {
            return resolve({
                name: 'Twilight Sparkle',
                type: 'Unicorn',
                element: 'Magic',
            })
        }
        return reject({
            message: `I just don't know what went wrong.`,
        })
    },
    500)
)
```

The function `fetchPony` returns a promise that resolves in 500 miliseconds, to simulate the round trip to the server and to give us some time to see the state changes. Also, it will return an error 20% of the time, so that we can see that state too.

Now let's implement the `fetchData` method in the `Pony` component.

```javascript
class Pony extends React.Component {
    constructor(props) {
        // previous code here...
        this.fetchData = this.fetchData.bind(this)
    }

    fetchData() {
        this.setState({ data: RemoteData.Pending() })
        fetchPony()
            .then(pony => this.setState({ data: RemoteData.Success(pony) }))
            .catch(err => this.setState({ data: RemoteData.Failure(err) }))
    }

    // render method here...
}
```

Our method `fetchData` will, first of all, change the state to `Pending`, and then simulate the call to the server. When the promise resolves, it will change the state to `Success` with the data received. If an error happens, it will change the state to `Failure` instead and pass on the error.

The last step is to render the three missing states.

```javascript
class Pony extends React.Component {
    // previous code here...

    render() {
        this.state.data.match({
            NotAsked: () => (
                <div>
                    <h1>Press "load"</h1>
                    <button onClick={this.fetchData}>Load!</button>
                </div>
            ),
            Pending: () => (
                <div>
                    <h1>Loading...</h1>
                </div>
            ),
            Success: ({ name, type, element }) => (
                <div>
                    <p><strong>Name:</strong> {name}</p>
                    <p><strong>Type:</strong> {type}</p>
                    <p><strong>Element of Harmony:</strong> {element}</p>
                    <button onClick={this.fetchData}>Reload</button>
                </div>
            ),
            Failure: ({ message }) => (
                <div>
                    <p>{message}</p>
                    <button onClick={this.fetchData}>Retry</button>
                </div>
            )
        })
    }
}
```

And we're done! We have a component that will inform the user about what's going on with the call to the server without using messy boolean flags all over the place.

You can check the full example in this [GitHub repo](https://github.com/Avalander/union-types-example/tree/master/examples/react).

# Limitations of this implementation

If you compare this implementation with union types in Elm, you'll find it rather defective. Elm is a strongly typed language and the complier will tell us if we've forgotten to handle a branch of the union type or if we're matching agains the wrong type of data. Also, Elm allows to match one type multiple times as long as the specificity of the data varies. With Javascript, we don't have any of this.

Truth to be told, with this implementation we won't even have any autocompletition help from our code editor. However, that could be addressed with a more verbose implementation, or using TypeScript typings.

# Conclusion

In this article I wanted to explore how union types could be implemented in Javascript and if using them could lead to code that is cleaner and easier to extend. I've got to say that I have mixed feelings about this. I like the pattern and I think it succeeds in producing code that's easy to reason about and extend. On the other hand, we miss all the safety that we would get from a statically typed language, which is half of the point. And we haven't really achieved anything that we couldn't have done with just some sensible structure in our code.

What do you think? Are union types any useful beyond appealing to an aesthetic preference for functional programming? I'd love to read your thoughts and opinions in the comments section.