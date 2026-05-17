# JavaScript Clock – Full Code Explanation

# Basic Idea of the Clock

The clock consists of three hands:

- Second hand
- Minute hand
- Hour hand

Each hand is just a normal HTML element (`div`) that gets rotated using CSS.

The rotation is done with:

```js
element.style.transform = `rotate(Xdeg)`;
```

---

# HTML Structure

```html
<div class="clock">
  <div class="clock-face">
    <div class="hand hour-hand"></div>
    <div class="hand min-hand"></div>
    <div class="hand second-hand"></div>
  </div>
</div>
```

Each hand has:
- the class `hand`
- an additional specific class (`hour-hand`, `min-hand`, `second-hand`)

This allows us to select them later in JavaScript.

---

# Selecting the Hands

```js
const secondHand = document.querySelector('.second-hand');
const minuteHand = document.querySelector('.min-hand');
const hourHand = document.querySelector('.hour-hand');
```

`querySelector()` searches for the matching HTML element in the document.

---

# The setDate() Function

```js
function setDate() {

}
```

This function:
- gets the current time
- calculates the rotation angles
- rotates the hands

---

# Getting the Current Time

```js
const now = new Date();
```

`new Date()` creates a date/time object.

---

# Getting Seconds

```js
const seconds = now.getSeconds();
```

Returns values from:

```txt
0 - 59
```

---

# Getting Minutes

```js
const minutes = now.getMinutes();
```

Returns:

```txt
0 - 59
```

---

# Getting Hours

```js
const hours = now.getHours();
```

Returns:

```txt
0 - 23
```

---

# Calculating the Angles

## Second Hand

```js
const secondsDegrees =
  ((seconds / 60) * 360) + 90 + secondExtraRotation;
```

## Explanation

### Step 1

```js
seconds / 60
```

Converts the seconds into a percentage.

Example:

```txt
30 / 60 = 0.5
```

---

### Step 2

```js
(seconds / 60) * 360
```

Converts the percentage into degrees.

Example:

```txt
0.5 * 360 = 180deg
```

---

### Step 3

```js
+ 90
```

Shifts the starting position.

Without `+90`, the hand would point to the right at `0deg`.

With `+90`, the hand correctly starts at 12 o'clock.

---

### Step 4

```js
+ secondExtraRotation
```

Prevents the 360° jump when transitioning from:

```txt
59 -> 0
```

---

# Minute Hand

```js
const minutesDegrees =
  ((minutes / 60) * 360) +
  ((seconds / 60) * 6) +
  90 +
  minuteExtraRotation;
```

---

## Why include seconds?

Without seconds:
- the minute hand jumps every full minute

With seconds:
- the movement becomes smooth/fluid

---

## Why `* 6`?

A circle has:

```txt
360°
```

One minute has:

```txt
60 seconds
```

So:

```txt
360 / 60 = 6°
```

Each second moves the minute hand by 6 degrees.

---

# Hour Hand

```js
const hoursDegrees =
  ((hours / 12) * 360) +
  ((minutes / 60) * 30) +
  90 +
  hourExtraRotation;
```

---

## Why include minutes?

Without minutes:
- the hour hand jumps every full hour

With minutes:
- it moves slowly and smoothly

---

## Why `* 30`?

12 hours distributed across 360°:

```txt
360 / 12 = 30°
```

One hour therefore equals 30 degrees.

---

# The 360° Bug

## The Problem

The hand jumps from:

```txt
444deg -> 90deg
```

CSS tries to animate this large backwards movement.

As a result, the hand spins all the way around.

---

# Solution: Extra Rotation

## Variables

```js
let secondExtraRotation = 0;
let minuteExtraRotation = 0;
let hourExtraRotation = 0;
```

These variables store additional full rotations.

---

# Storing Previous Values

```js
let previousSeconds = new Date().getSeconds();
let previousMinutes = new Date().getMinutes();
let previousHours = new Date().getHours();
```

These values remember:
- what the hand value was during the previous update

---

# Detecting Transitions

## Seconds

```js
if (seconds === 0 && previousSeconds === 59) {
  secondExtraRotation += 360;
}
```

This means:

```txt
59 -> 0
```

The hand completed a full rotation.

Therefore:

```js
+= 360
```

---

# Why does this work?

Without extra rotation:

```txt
444deg -> 90deg
```

With extra rotation:

```txt
444deg -> 450deg
```

Because:

```txt
450deg = 90deg + 360deg
```

The hand visually stays in the same position,
but CSS now sees a small forward movement.

This keeps the animation smooth.

---

# Why do we store previousSeconds = seconds?

At the end:

```js
previousSeconds = seconds;
previousMinutes = minutes;
previousHours = hours;
```

This turns:
- the current value
- into the previous value
- for the next function call

Example:

## Current State

```txt
seconds = 37
previousSeconds = 36
```

At the end:

```js
previousSeconds = seconds;
```

Then:

```txt
previousSeconds = 37
```

During the next update, JavaScript can compare:

```txt
37 -> 38
```

or:

```txt
59 -> 0
```

---

# Rotating the Hands

```js
secondHand.style.transform =
  `rotate(${secondsDegrees}deg)`;
```

This is what actually moves the hand.

The same thing happens for:
- minutes
- hours

---

# Why call setDate() manually?

```js
setDate();
setInterval(setDate, 1000);
```

---

## Problem without setDate()

`setInterval()` only starts after 1 second.

The clock would briefly display the wrong position.

---

## Solution

```js
setDate();
```

runs the function immediately once.

After that:

```js
setInterval()
```

handles the regular updates.

---

# Why did changing the hand width break the clock?

Originally:

```css
.hand {
  width: 50%;
}
```

This worked because the hand happened to end exactly in the center of the clock.

When changing:

```css
width: 30%;
```

The hand no longer ended in the center.

Since:

```css
transform-origin: 100%;
```

means:

> rotate around the right edge

The hand started rotating around the wrong point.

---

# Fixing the Rotation Point

```css
.hand {
  right: 50%;
}
```

This means:

> place the right side of the hand exactly in the middle of the clock

Now the rotation point always stays centered,
regardless of the hand length.

---

# Final Version of the Clock

```js
const secondHand = document.querySelector('.second-hand');
const minuteHand = document.querySelector('.min-hand');
const hourHand = document.querySelector('.hour-hand');

let previousSeconds = new Date().getSeconds();
let previousMinutes = new Date().getMinutes();
let previousHours = new Date().getHours();

let secondExtraRotation = 0;
let minuteExtraRotation = 0;
let hourExtraRotation = 0;

function setDate() {
  const now = new Date();

  const seconds = now.getSeconds();
  const minutes = now.getMinutes();
  const hours = now.getHours();

  if (seconds === 0 && previousSeconds === 59) {
    secondExtraRotation += 360;
  }

  if (minutes === 0 && previousMinutes === 59) {
    minuteExtraRotation += 360;
  }

  if (hours === 0 && previousHours === 23) {
    hourExtraRotation += 360;
  }

  const secondsDegrees =
    ((seconds / 60) * 360) +
    90 +
    secondExtraRotation;

  const minutesDegrees =
    ((minutes / 60) * 360) +
    ((seconds / 60) * 6) +
    90 +
    minuteExtraRotation;

  const hoursDegrees =
    ((hours / 12) * 360) +
    ((minutes / 60) * 30) +
    90 +
    hourExtraRotation;

  secondHand.style.transform =
    `rotate(${secondsDegrees}deg)`;

  minuteHand.style.transform =
    `rotate(${minutesDegrees}deg)`;

  hourHand.style.transform =
    `rotate(${hoursDegrees}deg)`;

  previousSeconds = seconds;
  previousMinutes = minutes;
  previousHours = hours;
}

setDate();
setInterval(setDate, 1000);
```

