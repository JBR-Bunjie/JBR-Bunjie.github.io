## basic animation: transition

[CSS Transitions (w3schools.com)](https://www.w3schools.com/css/css3_transitions.asp)

```html
<!--example code-->
<style> 
    div {
      width: 100px;
      height: 100px;
      background: red;
      transition: width 2s, height 4s;
    }

    div:hover {
      width: 300px;
      height: 300px;
    }
</style>
```

more examples:

[Hover underline animation - 30 seconds of code](https://www.30secondsofcode.org/css/s/hover-underline-animation)

Creates an animated underline effect when the user hovers over the text.

- Use `display: inline-block` to make the underline span just the width of the text content.
- Use the `:after` pseudo-element with `width: 100%` and `position: absolute` to place it below the content.
- Use `transform: scaleX(0)` to initially hide the pseudo-element.
- Use the `:hover` pseudo-class selector to apply `transform: scaleX(1)` and display the pseudo-element on hover.
- Animate `transform` using `transform-origin: left` and an appropriate `transition`.
- Remove the `transform-origin` property to make the transform originate from the center of the element.

```html
<!--html part-->
<p class="hover-underline-animation">Hover this text to see the effect!</p>

<!--css part-->
<style>
.hover-underline-animation {
  display: inline-block;
  position: relative;
  color: #0087ca;
}

.hover-underline-animation:after {
  content: '';
  position: absolute;
  width: 100%;
  transform: scaleX(0);
  height: 2px;
  bottom: 0;
  left: 0;
  background-color: #0087ca;
  transform-origin: bottom right;
  transition: transform 0.25s ease-out;
}

.hover-underline-animation:hover:after {
  transform: scaleX(1);
  transform-origin: bottom left;
}
</style>
```

