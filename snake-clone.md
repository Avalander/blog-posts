---
title: Create a Snake clone with Hyperapp
published: false
description:
tags: tutorial, javascript, hyperapp, snake
---

In this tutorial I'm going to cover how to create a snake clone using [hyperapp](https://github.com/hyperapp/hyperapp). There are no big requirements, but you should at least have read the [getting started guide](https://github.com/hyperapp/hyperapp#getting-started) and be familiar with ES6 syntax.

# Create project and install dependencies

To create the project, simply create a new project in an empty folder using `npm init` and install the following dependencies.

```bash
$ npm i --save hyperapp @hyperapp/html @hyperapp/fx
```

- **hyperapp**: [hyperapp](https://github.com/hyperapp/hyperapp) is a minimalistic javascript framework for creating web applications, heavily inspired by Elm.
- **hyperapp/html**: [hyperapp/html](https://github.com/hyperapp/html) provides helper functions to create DOM elements.
- **hyperapp/fx**: [hyperapp/fx](https://github.com/hyperapp/fx) provides functions that we can use to set up time intervals and other side effects easily.

I'm using webpack to build this project, but I won't get into how to set it up here. If you are feeling lazy, you can download the set up [from this repo](https://github.com/Avalander/hypersnake-tutorial/tree/setup-build).

Now we should be ready to start coding.

# Set up hyperapp

Hyperapp exposes a function called `app` that receives an initial state, the actions available for our app, a function to render the view from the state, and a DOM element to mount the app. Since we are using `hyperapp/fx`, we need to wrap our `app` with the `withFx` method. Let's start with our `main.js` file.

```javascript
// main.js
import { app } from 'hyperapp'
import { withFx } from '@hyperapp/fx'
import { div } from '@hyperapp/html'


const state = {

}

const actions = {

}

const view = state =>
    div()

const game = withFx(app) (state, actions, view, document.body)
```

## Create SVG helpers

We are going to use SVG to render our game. We could easily use the canvas API instead, but I find SVG to be a great choice when you only need to draw basic shapes.

The package `hyperapp/html` provides some handy functions to create virtual dom nodes. Unfortunately, there is no official package with similar helper functios to create SVG elements. We can still create SVG elements with `hyperapp`'s `h` function, but wouldn't it be nice if we could write our code like this?

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

> **Note: why not jsx?**
>
> If you look around for resources on hyperapp, you will notice that most of them use jsx instead of the html helpers. While we could do the same here, I favour the html helpers because I find easier to reason about plain Javascript code, I don't have to write a closing tag for my elements, and most of the time we need to mix plain Javascript code in our jsx anyway.

Now we are all set up to start actually building our game.

#Background

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

`SIZE` is how big the cells will be. `WIDTH` and `HEIGHT` are the sizes of the playing area. Instead of defining them with absolute values, we do it in proportions to `SIZE` so that the board has always the same relative size independently of scale.

`COLORS.background` is the colour we are going to use to fill our background.

We need to add a `svg` element where the game is going to be rendered, so let's modify our `view` function.

```javascript
// main.js
const view = state =>
    div([
        svg({ viewBox: `0 0 ${WIDTH} ${HEIGHT}`, width: WIDTH, height: HEIGHT}, [

        ]),
    ])
```

We could nest some more SVG elements there to create our background, but the `view` function could get huge if we had a lot of elements to draw, so let's create a component for the background instead.

```javascript
// main.js
const view = state =>
    div([
        svg({ viewBox: `0 0 ${WIDTH} ${HEIGHT}`, width: WIDTH, height: HEIGHT}, [
            Background(),
        ]),
    ])

const Background = () =>
    g({ key: 'background' }, [
        rect({ x: 0, y: 0, width: WIDTH, height: HEIGHT, fill: COLORS.background }),
    ])
```

with that we should see a big, green rectangle on the screen.

# Snake

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
    div([
        svg({ viewBox: `0 0 ${WIDTH} ${HEIGHT}`, width: WIDTH, height: HEIGHT}, [
            Background(),
            Snake(state.snake),
        ]),
    ])
```

The `Snake` component is receiving the array of points from our `state` object and converting each point to a `rect` element in the point's coordinates.

## Add time

## Control direction

# Add apple

# Eat apple

# End game

## Out of bounds

## Self collision

## End game screen