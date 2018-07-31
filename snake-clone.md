---
title: Create a Snake clone with Hyperapp, part 1
published: false
description: We'll explore how to create a snake clone using Hyperapp and SVG graphics.
tags: tutorial, javascript, hyperapp, snake
cover_image: https://images.unsplash.com/photo-1508155250035-b2978b7ffbb1?ixlib=rb-0.3.5&s=490e4db15d4f4934a3eaaaa8685fdb43&auto=format&fit=crop&w=2550&q=80
---

> Cover picture by [Dominik Vanyi](https://unsplash.com/photos/YkZW4ffuDnc) on [Unsplash](https://unsplash.com/).

[Here is a demo](https://avalander.github.io/hypersnake-tutorial/) of what we are going to build.

In this tutorial I'm going to cover how to create a snake clone using [hyperapp](https://github.com/hyperapp/hyperapp). There are no big requirements, but you should at least have read the [getting started guide](https://github.com/hyperapp/hyperapp#getting-started) for hyperapp and be familiar with ES6 syntax.

In particular, these are the ES6 features you should be familiar with to understand the code.
- [Import statements](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/import).
- [Arrow functions](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Functions/Arrow_functions).
- [Destructuring assignments](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Destructuring_assignment).
- [Spread syntax](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Spread_syntax).
- [Ternary operator](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Conditional_Operator). Not actually an ES6 feature, but I use it abundantly, so you should be able to read it.

# Create project and install dependencies

To create the project, simply create a new project in an empty folder using `npm init` and install the following dependencies.

```bash
$ npm i --save hyperapp @hyperapp/fx
```

- **hyperapp**: [hyperapp](https://github.com/hyperapp/hyperapp) is a minimalistic javascript framework for creating web applications, heavily inspired by Elm.
- **hyperapp/fx**: [hyperapp/fx](https://github.com/hyperapp/fx) provides functions that we can use to set up time intervals and other side effects easily.

I'm using webpack to build this project, but I won't get into how to set it up here. If you are feeling lazy, you can download the set up [from this repo](https://github.com/Avalander/hypersnake-tutorial/tree/setup-build).

Now we should be ready to start coding.

# Set up hyperapp

Hyperapp exposes a function called `app` that receives an initial state, the actions available for our app, a function to render the view from the state, and a DOM element to mount the app. Since we are using `hyperapp/fx`, we need to wrap our `app` with the `withFx` method. Let's start with our `main.js` file.

```javascript
// main.js
import { app } from 'hyperapp'
import { withFx } from '@hyperapp/fx'


const state = {}

const actions = {}

const view = state => {}

const game = withFx(app) (state, actions, view, document.body)
```

## Create SVG helpers

We are going to use SVG to render our game. We could easily use the canvas API instead, but I find SVG to be a great choice when you only need to draw basic shapes.

The package `@hyperapp/html` provides some handy functions to create virtual dom nodes. Unfortunately, there is no official package with similar helper functios to create SVG elements. We can still create SVG elements with `hyperapp`'s `h` function, but wouldn't it be nice if we could write our code like this?

```javascript
svg({ viewBox: '0 0 600 400' }, [
    g({}, [
        rect({ x: 0, y: 0, width: 50, height: 50, fill: '#a4b398' })
    ])
])
```

We can easily write such helpers ourselves, so let's go ahead and create a `svg.js` file and import it in our `main.js`.

```javascript
// svg.js
import { h } from 'hyperapp'


export const svg = (attrs, children) => h('svg', attrs, children)
export const g = (attrs, children) => h('g', attrs, children)
export const rect = (attrs, children) => h('rect', attrs, children)
```

```javascript
// main.js
import { svg, g, rect } from './svg'
```

> Note: why not jsx?
> ---
> If you look around for resources on hyperapp, you will notice that most of them use jsx instead of the html helpers. While we could do the same here, I favour the html helpers because I find easier to reason about plain Javascript code, I don't have to write a closing tag for my elements, and most of the time we need to mix plain Javascript code in our jsx anyway.
>
> Nevertheless, if you would rather write jsx, you should be able to follow this tutorial anyway and you don't need the `svg.js` file with the svg helper functions. Just make sure that you are importing the function `h` from `hyperapp` in your `main.js` if you use jsx.

Now we are all set up and it's time to start actually building our game.

# Create the background

The background is going to be a green rectangle covering the entire playable area. Let's start defining some constants.

```javascript
// main.js
const SIZE = 15
const WIDTH = SIZE * 40
const HEIGHT = SIZE * 27

const COLORS = {
    background: '#088c64',
}
```

`SIZE` is how big the cells will be. `WIDTH` and `HEIGHT` are the sizes of the playing area. Instead of defining them with absolute values, we do it in proportion to `SIZE` so that the board has always the same relative size independently of scale.

`COLORS.background` is the colour we are going to use to fill our background.

We need to add a `svg` element where the game is going to be rendered, so let's modify our `view` function.

```javascript
// main.js
const view = state =>
    svg({ viewBox: `0 0 ${WIDTH} ${HEIGHT}`, width: WIDTH, height: HEIGHT}, [

    ])
```

We could nest some more SVG elements there to create our background, but the `view` function could get huge if we had a lot of elements to draw, so let's create a component for the background instead.

```javascript
// main.js
const view = state =>
    svg({ viewBox: `0 0 ${WIDTH} ${HEIGHT}`, width: WIDTH, height: HEIGHT}, [
        Background(),
    ])

const Background = () =>
    g({ key: 'background' }, [
        rect({ x: 0, y: 0, width: WIDTH, height: HEIGHT, fill: COLORS.background }),
    ])
```

with that we should see a big, green rectangle on the screen.

# Create the snake

Let's add the main character of our game, the snake. We will store the position of the snake as an array of points in our `state` object.

```javascript
// main.js
const state = {
    snake: [
        { x: 3 * SIZE, y: 3 * SIZE},
        { x: 2 * SIZE, y: 3 * SIZE},
        { x: 1 * SIZE, y: 3 * SIZE},
    ]
}
```

Let's add a couple of colours to render our snake with.

```javascript
//main.js
const COLORS = {
    snake: {
        fill: '#bcaba0',
        stroke: '#706660',
    },
}
```

And let's create another component to render the snake.

```javascript
// main.js
const Snake = state =>
    g({ key: 'snake' },
        state.map(({ x, y }) => rect({
            x, y, width: SIZE, height: SIZE,
            fill: COLORS.snake.fill,
            stroke: COLORS.snake.stroke,
            'stroke-width': 2
        }))
    )

const view = state =>
    svg({ viewBox: `0 0 ${WIDTH} ${HEIGHT}`, width: WIDTH, height: HEIGHT}, [
        Background(),
        Snake(state.snake),
    ])
```

The `Snake` component is receiving the array of points from our `state` object and converting each point to a `rect` element in the point's coordinates.

## Make the snake move

Now we should see our snake on screen, but it's not moving yet. It's time to fix that.

We are going to need a way to update our state regularly. We can use `hyperapp/fx`'s `delay` function. `delay` works much like `setTimeout`, but it receives the name of an action to call after the given delay instead of a function. Let's see how we can use `delay` to create our game loop.

```javascript
// main.js
import { withFx, delay } from '@hyperapp/fx'

const UPDATE_INTERVAL = 150

const actions = {
    frame: () => [
        delay(UPDATE_INTERVAL, 'frame')
    ]
}
```

1. We import the function `delay` from `hyperapp/fx`.
2. We create the constant `UPDATE_INTERVAL`, which is the amount of milliseconds that will elapse between each frame.
3. We create an action called `frame` that will spawn another frame every `UPDATE_INTERVAL` milliseconds.

That's handy, but nothing is happening yet. We need to trigger the first frame, so the chain of updates will start rolling. Fortunately, `hyperapp`'s `app` function returns an object with all the actions wired, so we can just call `frame` for the first time from there.

```javascript
// main.js
const game = withFx(app) (state, actions, view, document.body) // This line is there already, don't write it again.
game.frame()
```

This should get the ball rolling. However, nothing is happening yet, we only have `frame` actions spawning more `frame` actions every 150 milliseconds, but they are not doing anything else. Let's just create an action that will print to the console every time a frame is spawned, to check that it's working.

```javascript
// main.js
const actions = {
    sayHi: () => console.log('Hello, there!'),
}
```

Now we need a way to trigger that action every time we enter a new frame. That is easy enough with `hyperapp/fx`. With `hyperapp/fx`, an action can return an array of effects (one of such effects is `delay`, we are already acquainted with it). There is another effect called `action` that triggers an action from our action object. So let's import `action` from `hyperapp/fx` and trigger `sayHi` from `frame`.

```javascript
// main.js
import { withFx, delay, action } from '@hyperapp/fx'

const actions = {
    frame: () => [
        action('sayHi'),
        delay(UPDATE_INTERVAL, 'frame'),
    ],
    sayHi: () => console.log('Hello, there!'),
}
```

If you check the console now, you will see a bunch of `Hello, there!` texts piling up.

As we have seen, `action` receives the name of an action in our `actions` object and triggers it. Optionaly, it receives a second parameter with an argument that will be sent to the triggered action. We will use this later.

Printing text on the console is fun, but we're here to see the snake move, so let's get to it.

The first thing we need is the direction where the snake is moving towards. We will add a `direction` property in the `state` object with the value `'right'`.

```javascript
// main.js
const state = {
    direction: 'right',
}
```

Now we will remove the `sayHi` action and create an action to update the snake instead.

```javascript
// main.js
const actions = {
    frame: () => [
        action('updateSnake'),
        delay(UPDATE_INTERVAL, 'frame'),
    ],
    updateSnake: () => state => ({
        ...state,
        snake: updateSnake(state.snake, state.direction),
    }),
}
```

There we go, we have created the action `updateSnake`, that will return a shallow copy of the current state with an updated version of the snake, and we trigger that action in our `frame`.

We still need to implement the function `updateSnake`. There are many ways to make the snake move. The naïve approach would be to go through the array starting at the tail and move each cell to the position of the cell before it, then move the head towards the current direction.

```javascript
// main.js
const updateSnake = (snake, direction) => {
    for (let i = snake.length - 1; i > 0; i--) {
        snake[i].x = snake[i - 1].x
        snake[i].y = snake[i - 1].y
    }
    if (direction === 'right') {
        snake[0].x += SIZE
    }
    if (direction === 'left') {
        snake[0].x -= SIZE
    }
    if (direction === 'down') {
        snake[0].y += SIZE
    }
    if (direction === 'up') {
        snake[0].y -= SIZE
    }
    return snake
}
```

1. We loop through the snake, starting at the last cell and ending at the second. We move each cell to the position of the cell before it.
2. We move the head one position towards the current direction.

Now we should see the snake moving to the right. Even though this works, we can do something neater to move the head instead of having a bunch of `if` statements. The approach I suggest is to have a dictionary with the possible directions as keys and a vector with `x` and `y` components that will be applied to the speed to calculate movement.

This is easier than it sounds. Let's start by creating the directions dictionary.

```javascript
// main.js
const DIRECTIONS = {
    left: { x: -1, y: 0 },
    right: { x: 1, y: 0 },
    up: { x: 0, y: -1 },
    down: { x: 0, y: 1 },
}
```

And now we remove that bunch of `if` statements from our `updateSnake` function and instead transform the coordinates `x` and `y` of the head by adding the cell size multiplied by the relevant coordinate of the current direction.

```javascript
// main.js
const updateSnake = (snake, direction) => {
    for (let i = snake.length - 1; i > 0; i--) {
        snake[i].x = snake[i - 1].x
        snake[i].y = snake[i - 1].y
    }
    
    snake[0].x += SIZE * DIRECTIONS[direction].x
    snake[0].y += SIZE * DIRECTIONS[direction].y

    return snake
}
```

## Control direction

Our snake is now moving. The next step is to be able to change the direction with the arrow keys.

To achieve that, we are going to use an effect to trigger an action when a key is pressed. As you might suspect by now, `hyperapp/fx` exposes a function for that, called `keydown`, so let's import it and use it.

```javascript
// main.js
import { withFx, delay, action, keydown } from '@hyperapp/fx'
```

`keydown`, much like `action` and `delay` receives the name of an action to trigger when a key is pressed as a parameter. We only need to trigger that effect once, so we have to find a place for it. The easiest is to create a `start` action that will trigger the `keydown` effect and the first `frame` action and call that action instead of `frame` to start the game loop.

```javascript
// main.js
const actions = {
    start: () => [
        keydown('keyPressed'),
        action('frame'),
    ],
}

// Replace 'game.frame()' with this.
game.start()
```

And now we have to implement the `keyPressed` action. Basically, we want to ignore all keys that are not `ArrowUp`, `ArrowDown`, `ArrowLeft` or `ArrowRight`, and we want to translate these four to the equivalent direction. Let's first create a new dictionary with the translation between keys and directions.

```javascript
// main.js
const KEY_TO_DIRECTION = {
    ArrowUp: 'up',
    ArrowDown: 'down',
    ArrowLeft: 'left',
    ArrowRight: 'right',
}
```

This may look like a bit of repetition, but it will make our life easier in a minute.

Now for the `keyPressed` action. It is going to receive a regular `keydown` event, of which we are only interested in knowing the property `key` (the property key have be one of those four `Arrow[Something]` values if we're interested in it or another string otherwise). The `keyPressed` action should update the direction in the state if an arrow key is pressed and do nothing otherwise.

```javascript
// main.js
const actions = {
    keyPressed: ({ key }) => state => ({
        ...state,
        direction: Object.keys(KEY_TO_DIRECTION).includes(key)
            ? KEY_TO_DIRECTION[key]
            : state.direction
    })
}
```

While this works, it is semantically inaccurate. We called our action `keyPressed`, but it's actually changing the direction. We can be more accurate if `keyPressed` only checks if another action needs to be triggered according to the pressed key and we create a new action that takes care of changing the direction.

```javascript
// main.js
const actions = {
    keyPressed: ({ key }) =>
        (Object.keys(KEY_TO_DIRECTION).includes(key)
            ? [ action('changeDirection', KEY_TO_DIRECTION[key]) ]
            : []
        ),
    changeDirection: direction => state => ({
        ...state,
        direction,
    }),
}
```

There we go. Now `keyPressed` will check if the `key` property of the event is a key in our `KEY_TO_DIRECTION` dictionary. If that is the case, it will trigger a `changeDirection` with the appropriate direction, otherwise it won't trigger any additional action.

`changeDirection` simply receives a direction and updates the state with that direction.

There is yet one thing we need to take care of. In the current state, our snake can switch to the opposite direction. If it is moving to the right and the player presses the left arrow, it will change direction to the left and walk over itself. We would like to prevent that.

To achieve that, we will sophisticate our `changeDirection` action a bit more. Instead of blindly updating the direction, it will update it *only* if the new direction is not opposite to the current direction. To easily know if the current and new directions are opposite, we will create a new dictionary with each direction's opposite (this is the last directions dictionary we create, I promise).

```javascript
// main.js
const OPPOSITE_DIRECTION = {
    up: 'down',
    down: 'up',
    left: 'right',
    right: 'left',
}

const actions = {
    changeDirection: direction => state => ({
        ...state,
        direction: (direction === OPPOSITE_DIRECTION[state.direction]
            ? state.direction
            : direction
        )
    }),
}
```

Now `changeDirection` will only switch to the new direction if it is not opposite to the previous direction.

However, there is a bug in that code. `changeDirection` can be triggered multiple times between frames, while the snake will only move once. Therefore, if the snake is moving to the left and the player presses the up arrow, the `direction` while change to `'up'`. Now, if the player presses the right arrow before the next frame, `direction` will change to `'right'` before the snake has moved up. Effectively, the snake will switch directions from left to right in the next frame.

Go ahead, change `UPDATE_INTERVAL` to a larger value, like `500`, and see it for yourself.

One way to avoid that is to add a new property in the state, `next_direction`, and have `changeDirection` update that property instead. Then, we always have the current direction in `direction` and we can check that we are not setting the opposite direction.

Then, we will create a new action, `updateDirection`, that will update the direction only once per frame.

```javascript
// main.js
const state = {
    direction: 'right',
    next_direction: 'right',
}

const actions = {
    frame: () => [
        action('updateDirection'),
        action('updateSnake'),
        delay(UPDATE_INTERVAL, 'frame'),
    ],
    updateDirection: () => state => ({
        ...state,
        direction: state.next_direction,
    }),
    changeDirection: direction => state => ({
        ...state,
        next_direction: (direction === OPPOSITE_DIRECTION[state.direction]
            ? state.next_direction
            : direction
        )
    }),
}
```

There we go.

1. We added a new property `next_direction` to `state`.
2. `changeDirection` will place the direction for the next frame in `next_direction` instead of `direction`, checking that the new value is not the opposite direction to what is in `direction`.
3. We created a new action, `updateDirection`, that will be triggered once per frame and will take the most recent value in `next_direction` and place it in `direction` before the snake is updated.

# Conclusion

That was a lot of text, congratulations on making it so far! In the second part of the tutorial we will explore how to add apples and score, make the snake grow, and end the game when the snake collides with a border or with itself.

You can find the code we've written so far [here](https://github.com/Avalander/hypersnake-tutorial/tree/end-part-1/src).
