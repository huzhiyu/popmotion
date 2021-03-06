---
title: Dynamic props
description: Set props as dynamic functions
category: basics
next: dynamic-props
---

# Dynamic pose props

Each pose property can be set as a function that is resolved when the pose is entered:

```javascript
const config = {
  visible: {
    x: 0,
    y: (props) => 100, // Resolved on `visible` enter
    transition: {
      x: { type: 'tween' },
      y: (props) => ({ type: 'spring' }) // Resolved on `visible` enter
    }
  }
}
```

By using the provided `props` argument, this allows us to create dynamic properties that will react to changes in your app.

- [Props](#props)
- [Transition props](#transition-props)
- [](#)
- [Dynamic transitions](#dynamic-transitions)
  - [Transition props](#transition-props)
- [`transition` props](#transition-props)
  - [Pose-generated `props`](#pose-generated-props)

## Props

`props` can be set on the initial configuration of a poser:

```javascript
const config = {
  visible: {
    opacity: 1,
    transition: ({ i }) => ({ delay: i * 50 })
  },
  props: { i: 0 }
};
```

In vanilla Pose, we can update these props later using `setProps`:

```javascript
poser.setProps({ i: 1 })
```

In React Pose, all props set on the posed component are set as the poser's `props`:

```javascript
({ i, isVisible }) => <Item pose={isVisible ? 'hidden' : 'visible'} i={i} />
```

For instance, we could use this `i` index property to animate something from a different starting position based on its position within a list:

```javascript
closed: {
  y: ({ i }) => i * 50
}
```

<CodePen id="jzXzdz" height="400" />

## Transition props

`transition` works a little differently than other pose props.

If it's set as a function, that function is run **once each for every property being animated**.

That function is provided a few extra props, automatically generated by Pose:

- `from`: The current state of this value
- `to`: The target state defined in the pose
- `velocity`: If a numerical value, the current velocity of the value
- `key`: The name of the current value
- `prevPoseKey`: The name of the pose this value was previously set to

These props can be used to return a different transition definition based on the state of the value:

```javascript
const config = {
  open: {
    x: '-100%',
    transition: ({ velocity, to }) => velocity < 0
      ? { to: 0 }
      : { to }
  }
};
```

If `transition` is a named map, **some or all** of these can be defined as functions:

```javascript
const config = {
  open: {
    x: 0,
    opacity: 1,
    transition: {
      x: ({ velocity, to }) => velocity < 0 ? { to: -300 } : { to },
      opacity: { type: 'spring' }
    }
  }
};
```









Every pose property can be set as a function that is only executed when the pose is entered.

This allows you to dynamically react to changes in your app, for instance to changes in viewport.

For instance, we could pass through an `i` property to animate something differently based on its position in a list.

That property would look like this:

```javascript
closed: {
  y: ({ i }) => i * 50
}
```

In vanilla Pose, we could set `i`

## 



<CodePen id="jzXzdz" height="400" />



## Dynamic transitions

`transition` is handled a little differently. If it's run as a 

### Transition props

Some properties are only provided to the `transition` property.

- `from`: The current state of this value
- `to`: The target state defined in the pose
- `velocity`: If a numerical value, the current velocity of the value
- `key`: The name of the current value
- `prevPoseKey`: The name of the pose this value was previously set to









Just like CSS, every pose has a `transition` property. This is a function you can optionally provide that creates a [Popmotion animation](/api) (or `false`, for no animation).

If you haven't installed Popmotion, you can do so locally with `npm install popmotion`. Pose uses Popmotion as its engine, so this won't add anything to your filesize.

If you're following along with the CodePen, `popmotion` is already available as a global variable.

Replace the `config` you added to either the [Pose](https://codepen.io/popmotion/pen/bvqJbV?editors=0010) or [React Pose](https://codepen.io/popmotion/pen/mxmrPZ?editors=0010) playgrounds with the following:

```javascript
const { tween } = popmotion

const config = {
  visible: { opacity: 1 },
  hidden: {
    opacity: 0,
    transition: (props) => tween({ ...props, duration: 1000 })
  }
}
```

As you can see, the animation now runs at `1000`ms: 

<CodePen id="qoVXKj" />

## `transition` props

This function is run **once, for every value being animated in the current pose**. This allows you to return a different animation for each value.

It's provided a single object, a `props` object. The object contains properties that you can use to decide which animation is being returned for each value.

### Pose-generated `props`

The following props are generated by Pose:

- `from`: The current state of this value
- `to`: The target state defined in the pose
- `velocity`: If a numerical value, the current velocity of the value
- `key`: The name of the current value
- `prevPoseKey`: The name of the pose this value was previously set to

For instance, we can use `key` to return a different animation based on the value being animated. Let's add a `scaleY` property to our poses, and use `key` to change return a different [easing](/api/easing) for it:

```javascript
const { easing, tween } = popmotion

const config = {
  visible: { scaleY: 1, opacity: 1 },
  hidden: {
    scaleY: 0,
    opacity: 0,
    transition: (props) => tween({
      ...props,
      duration: 1000,
      ease: props.key === 'opacity' ? easing.linear : easing.anticipate
    })
  }
}
```

<CodePen id="wmPrKV" />

You're also under no obligation to end the animation on the `to` property. You can do anything with the returned animation.

Set the `.box`'s `opacity` to `1` in the CSS. Now replace our `config` with a new set of poses:

```javascript
const config = {
  initialPose: 'rest',
  rest: { scale: 1 },
  alert: { scale: 1.2 }
}
```

Set the poser or posed component to "alert" and the box will transition to the `scale: 1.2` defined in that pose.

Now we can add a custom `transition` function that will return a `spring` animation with `damping` set to `0`. This will make the animating oscillate around the `to` value indefinitely:

```javascript
const config = {
  rest: { scale: 1 },
  alert: {
    scale: 1.2,
    transition: (props) => spring({
      ...props,
      stiffness: 20,
      damping: 0
    })
  }
}
```

<CodePen id="mxqBOo" />

We can add a `setTimeout` function that will set the `boxPoser` to `rest` after any amount of time:

```javascript
setTimeout(() => boxPoser.set('rest'), 2000)
```

Try changing the timeout duration. The box will animate smoothly from any point, even though the "alert" animation didn't end on its `to` value. And because `scale` uses a spring animation by default, it also maintains some of the property's velocity.