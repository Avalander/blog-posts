---
title: Create a Snake clone with Hyperapp, part 2
published: false
description: We'll explore how to create a snake clone using Hyperapp and SVG graphics.
tags: tutorial, javascript, hyperapp, snake
cover_image: https://images.unsplash.com/photo-1508155250035-b2978b7ffbb1?ixlib=rb-0.3.5&s=490e4db15d4f4934a3eaaaa8685fdb43&auto=format&fit=crop&w=2550&q=80
---

> Cover picture by [Dominik Vanyi](https://unsplash.com/photos/YkZW4ffuDnc) on [Unsplash](https://unsplash.com/).

This is the second part of the tutorial, if you haven't already, make sure to follow [part 1](). You can checkout how the code should look like so far [here](https://github.com/Avalander/hypersnake-tutorial/tree/end-part-1/src). The demo of the final version of the game is [here](https://avalander.github.io/hypersnake-tutorial/).

# Apple

Let's start by adding a function to create apples. That function should position the apple in a random cell on the board.

```javascript
// main.js
const randInt = (from, to) =>
    Math.floor(Math.random() * (to - from) + from)

const createApple = () =>
    ({
        x: randInt(0, WIDTH/SIZE) * SIZE,
        y: randInt(0, HEIGHT/SIZE) * SIZE,
    })
```

1. `randInt` will return a random integer between `from` and `to`.
2. `createApple` will return an object with random `x` and `y` coordinates within the board.

We also need to choose some colours to render our apple, so let's add this to our `COLORS` constant.

```javascript
// main.js
const COLORS = {
    apple: {
        fill: '#ff5a5f',
        stroke: '#b23e42',
    },
}
```

Now we can add an apple in our state object.

```javascript
// main.js
const state = {
    apple: createApple(),
}
```

Easy peasy. Now let's draw our apple on the screen. We will create a new component for it, that will simply draw a rectangle with the colours we chose previously at the apple's coordinates.

```javascript
// main.js
const Apple = ({ x, y }) =>
    g({ key: 'apple' }, [
        rect({
            x, y, width: SIZE, height: SIZE,
            fill: COLORS.apple.fill,
            stroke: COLORS.apple.stroke,
            'stroke-width': 2
        })
    ])

const view = state =>
    svg({ viewBox: `0 0 ${WIDTH} ${HEIGHT}`, width: WIDTH, height: HEIGHT}, [
        Background(),
        Apple(state.apple),
        Snake(state.snake),
    ])
```

Make sure to put the `Apple` component in the `view` function before the `Snake` component, otherwise when the snake and the apple are overlapping, the apple will be drawn on top.

## Eat the apple

The snake should eat the apple when the head is in the same cell. First of all, we will create a function `collision` that will return `true` if two objects are in the same cell and `false` otherwise.

```javascript
// main.js
const collision = (a, b) =>
    a.x === b.x && a.y === b.y
```

Now we will create an action that will check if the head of the snake is in the same cell as the apple and trigger another action to eat the apple if that's the case.

```javascript
// main.js
const actions = {
    frame: () => [
        action('updateDirection'),
        action('updateSnake'),
        action('checkEatApple'),
        delay(UPDATE_INTERVAL, 'frame'),
    ],
    checkEatApple: () => state =>
        (collision(state.snake[0], state.apple)
            ? [ action('eatApple'),
                action('relocateApple'), ]
            : []
        ),
    eatApple: () => state => ({
        ...state,
        snake: growSnake(state.snake),
    }),
    relocateApple: () => state => ({
        ...state,
        apple: createApple(),
    }),
}

const growSnake = snake =>
    [ ...snake, {
        x: snake[snake.length - 1].x,
        y: snake[snake.length - 1].y,
    }]
```

1. We created the `checkEatApple` action. It will check if the snake's head and the apple are in the same cell. If that's the case, it will trigger two new actions, `eatApple` and `relocateApple`, otherwise it won't trigger any additional action.
2. We trigger the `checkEatApple` action from the `frame` action, so that it will check every frame.
3. We create the action `eatApple`. It will add a new cell at the tail of the snake.
4. We create the action `relocateApple`. It will create a new apple in a random position using the `createApple` function that we have implemented previously.

## Add score

We would like to have a score that increases every time the snake eats an apple, and that is displayed on the screen. Let's add a `score` property to the state and render it, and then we will take care of incrementing it.

```javascript
// main.js
const state = {
    score: 0,
}
```

To be able to render it, we will need an additional SVG helper to create a [tex†](https://developer.mozilla.org/en-US/docs/Web/SVG/Element/text) element. Let's add it to our `svg.js` file.

```javascript
// svg.js
export const text = (attrs, children) => h('text', attrs, children)
```

And let's create a `Score` component and render it in our `view` function.

```javascript
// main.js
import { g, rect, svg, text } from './svg'

const score_style = {
    font: 'bold 20px sans-seriff',
    fill: '#fff',
    opacity: 0.8,
}

const Score = state =>
    g({ key: 'score' }, [
        text({
            style: score_style,
            x: 5,
            y: 20,
        }, state)
    ])

const view = state =>
    svg({ viewBox: `0 0 ${WIDTH} ${HEIGHT}`, width: WIDTH, height: HEIGHT}, [
        Background(),
        Apple(state.apple),
        Snake(state.snake),
        Score(state.score),
    ])
```

1. We created some style to display the score text a bit nicer.
2. We created the `Score` component, that will recieve the score from the state and render it as a `text` element.
3. We added a call to `Score` in the `view` function.

To increase the score, we are going to create a new action, `updateScore` that will be triggered by `checkEatApple` when the snake eats the apple.

```javascript
// main.js
const actions = {
    checkEatApple: () => state =>
        (collision(state.snake[0], state.apple)
            ? [ action('eatApple'),
                action('relocateApple'),
                action('updateScore', 10) ]
            : []
        ),
    updateScore: value => state => ({
        ...state,
        score: state.score + value
    }),
```

# End game

We can control the snake, it is eating randomly located apples, and each apple consumed increases the score. The only thing missing is a way to end the game.

Traditionally, the snake game has two end conditions:
1. The head of the snake collides with one of the board's boundaries.
2. The head of the snake collides with any other cell of its body.

We are going to implement both of them.

## Out of bounds

To check if the snake has collided with a boundary, we will check if it's position is beyond any of the board borders after updating it. We will start by creating a function `isOutOfBounds` that will receive a point and return `true` if it is outside the limits of the board and `false` otherwise.

```javascript
// main.js
const isOutOfBounds = ({ x, y }) =>
    x < 0 || x >= WIDTH || y < 0 || y >= HEIGHT
```

We want to stop updating the game when it ends, so instead of triggering a new `frame` action from `frame` itself, we will create a new action and call it `continue`. This action will check whether the snake is out of bounds, if it isn't, it will trigger a new `frame`, otherwise, it won't.

```javascript
// main.js
const actions = {
    frame: () => [
        action('updateDirection'),
        action('updateSnake'),
        action('checkEatApple'),
        action('continue'),
    ],
    continue: () => state =>
        (isOutOfBounds(state.snake[0])
            ? []
            : delay(UPDATE_INTERVAL, 'frame')
        ),
}
```

Go ahead and run into all borders, you will see that the game stops running.

## Self collision

To check if the head of the snake is colliding with its tail, we will create a new function, `selfCollision`, that will iterate over every cell in the tail and return `true` if it finds a cell that is in the same position as the head, and `false` otherwise.

```javascript
// main.js
const selfCollision = ([ head, ...tail ]) =>
    tail.some(({ x, y }) =>
        x === head.x && y === head.y
    )
```

The function `Array.prototype.some` receives a predicate function and returns `true` if it evaluates to `true` for any element in the array, and `false` otherwise, exactly what we need.

To end the game when the snake steps on itself, we can add a check for `selfCollision` in the `continue` action and end the game if it returns `true`.

```javascript
// main.js
const actions = {
    continue: () => state =>
        (isOutOfBounds(state.snake[0]) || selfCollision(state.snake)
            ? []
            : delay(UPDATE_INTERVAL, 'frame')
        ),
}
```

## End game screen

Now the game stops running whenever one of the two end conditions is met, but that's not enough, we need a _game over_ screen so that the user knows that the game has ended.

We need to know whether the game is running or it has already ended to decide if we have to render the game over screen or not. We will add a `is_running` property to our state object and initialise it to `true`.

```javascript
// main.js
const state = {
    is_running: true,
}
```

When the game ends, we will set `is_running` to false. To achieve this, we will create a new action `updateIsRunning` and trigger it from the `continue` action when we end the game to set `is_running` to `false`.

```javascript
// main.js
const actions = {
    continue: () => state =>
        (isOutOfBounds(state.snake[0]) || selfCollision(state.snake)
            ? action('updateIsRunning', false)
            : delay(UPDATE_INTERVAL, 'frame')
        ),
    updateIsRunning: value => state => ({
        ...state,
        is_running: value,
    }),
}
```

Now let's create a component that will render our game over screen.

```javascript
// main.js
const game_over_style = {
    title: {
        font: 'bold 48px sans-seriff',
        fill: '#fff',
        opacity: 0.8,
        'text-anchor': 'middle',
    },
    score: {
        font: '30px sans-seriff',
        fill: '#fff',
        opacity: 0.8,
        'text-anchor': 'middle',
    }
}

const GameOver = score =>
    g({ key: 'game-over'}, [
        rect({
            x: 0, y: 0, width: WIDTH, height: HEIGHT,
            fill: '#000',
            opacity: 0.4,
        }),
        text({
            style: game_over_style.title,
            x: WIDTH/2, y: 100,
        }, 'Game Over'),
        text({
            style: game_over_style.score,
            x: WIDTH/2, y: 160,
        }, `Score: ${score}`),
    ])
```

Nothing fancy going on here, we simply create a `GameOver` function that returns a semi-transparent rectangle to darken the game, a text that says _Game Over_ and a text with the final score.

Now let's make the `view` function render it when the game is not running.

```javascript
// main.js
const view = state =>
    svg({ viewBox: `0 0 ${WIDTH} ${HEIGHT}`, width: WIDTH, height: HEIGHT}, [
        Background(),
        Apple(state.apple),
        Snake(state.snake),
        Score(state.score),
        !state.is_running ? GameOver(state.score) : null,
    ])
```

That would be enough, however, since the `GameOver` component already tells us the final score, there is no need to render also the `Score` component when the game is running, so we can render either depending on the value of `is_running`.

```javascript
// main.js
const view = state =>
    svg({ viewBox: `0 0 ${WIDTH} ${HEIGHT}`, width: WIDTH, height: HEIGHT}, [
        Background(),
        Apple(state.apple),
        Snake(state.snake),
        state.is_running
            ? Score(state.score)
            : GameOver(state.score),
    ])
```

# Improvements

The game is functional now, but there are still a few things that we can do to improve and extend it, if you want to experiment a bit more. Here is a list of possible improvements.

- Make the game run faster for every 100 score points. An easy way to achieve this is to have the update interval in the state instead of a constant, but take into account that it can never be zero or lower.
- The algorithm we use to move the snake is pretty naïve, we really don't need to calculate a new position for each cell of the body. Another approach is to pick the last cell of the tail, and move it to the beginning of the array at the new position for the head and not move any other cell.
- Add a way to restart the game (that is not reloading the window).
- Make different apples deal a different amount of score points.
- It's unlikely, but apples can appear in cells already occupied by the snake, find a way to prevent that.

# Conclusion

You can check out the final code [here](https://github.com/Avalander/hypersnake-tutorial/tree/final-code/src).

I hope this tutorial helped you understand a bit better how to model complex actions with `hyperapp` and `@hyperapp/fx` and you have a better idea of what it's capable of. Don't hesitate to write any thoughts or questions in the comments section.
