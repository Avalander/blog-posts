---
title: Create a Snake clone with Hyperapp, part 2
published: false
description: We'll explore how to create a snake clone using Hyperapp and SVG graphics.
tags: tutorial, javascript, hyperapp, snake
---

```javascript
// main.js

```

# Apple

Let's start by adding a function to create apples. That function should position the apple in a random cell on the board.

```javascript
// main.js
const randInt = (from, to) =>
    Math.floor(Math.random() * (to - from) + from)

const createApple = () =>
    ({
        x: randInt(0, WIDTH/SIZE),
        y: randInt(0, HEIGHT/SIZE),
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

Easy peasy. Now let's draw our apple on screen. We will create a new component for it, that will simply draw a rectangle with the colours we chose previously at the apple's coordinates.

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

## Eat apple

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

## Out of bounds

## Self collision

## End game screen

# Improvements