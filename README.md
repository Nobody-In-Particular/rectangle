# Rectangle
Utility SVG rectangle element API, geared towards user selection/dragging/resizing.

There are two "levels" of API. At the higher level, the API allows the user to drag and resize a rectangle and select an area.

At the lower level, the API allows moving and resizing of the rectangle by reference to edges (`x0`/`y0`/`x1`/`y1`), with no requirement that `x0 <= x1` or `y0 <= y1`, thus allowing easy "flipping" of the rectangle, since SVG does not support negative width/height.

---

## Example
```js
const svgElement = document.createElement("svg");
const rect = await Rectangle.selectArea(
	svgElement,
	0, 0,
	{fill: "none", stroke: "black", "stroke-width": 2}
);
rect.set("x1", area.x1 - 2);
rect.allowResizeAndDrag(() => { console.log(rect.bbox) }, "#handle-template");
```

## Constructor
```js
new Rectangle(
	parentElement, bbox, style,
	{flippable = true, xBounds = null, yBounds = null, round = true, coordTransformMatrix = new DOMMatrixReadOnly()} = {}
)
```

Adds a new SVG `rect` element as a child of `parentElement` (an `SVGElement`).

`bbox` is an object with properties `x`, `y`, `width` and `height` and defines the initial position/size of the rectangle. The rectangle always starts "unflipped", so the edge values will initially be as follows:

|      |   |
|---   |---|
| `x0` | `bbox.x` |
| `y0` | `bbox.y` |
| `x1` | `bbox.x + bbox.width` |
| `y1` | `bbox.y + bbox.height` |

`style` is an object of other attributes to be added to the element, e.g. `{fill: "white"}`

The configuration options `flippable`, `xBounds`, `yBounds`, `round` and `coordTransformMatrix` set the initial values for the corresponding settable instance properties.

---

## Instance methods

### set(prop, val)

Sets edge property `prop` (one of `'x0'`, `'y0'`, `'x1'`, `'y1'`), to `val`, obeying restrictions set by the instance properties, and rounding `val` if `rect.round` is true.

The usefulness of this "lower level" of the API is in the fact that the property names are *conserved*, e.g. if `x1` is set further left than `x0`, it remains `x1` rather than becoming `x0`.
```js
rect.set("x0", 4);
rect.set("x1", 2);
// The rectangle now spans [2, 4] on the x-axis

rect.set("x1", 5);
// The rectangle now spans [4, 5] on the x-axis
```

---
### set x0, set x1, set y0, set y1
Setters for the edge properties
```js
// These two statements are equivalent
rect.x0 = 5
rect.set("x0", 5)
```

---
### get x0, get x1, get y0, get y1
Getters for the edge properties.
```js
const currentX0 = rect.x0
```

---
### get bbox
Returns an object with properties `x`, `y`, `width` and `height`, where (`x`, `y`) is the current top-left corner of the rectangle. The returned object is a copy and mutations to it have no effect on the rectangle.
```js
rect.x0 = 4;
rect.x1 = 7;

rect.y0 = 5;
rect.y1 = 1;

rect.bbox // {x: 4, y: 1, width: 3, height: 4}
```

---
### resetFlip()
Doesn't change the rectangle, but "relabels" the edges so that `x0 <= x1` and `y0 <= y1`. Equivalent to the end result of:
```js
const {x, y, width, height} = rect.bbox;
rect.x0 = x
rect.x1 = x + width;
rect.y0 = y;
rect.y1 = y + height;
```

---
### setPoint0AndShift(x, y)
Translates the rectangle such that `x0` moves to `x` and `y0` moves to `y`, obeying restrictions set by the instance properties, and rounding `x` and `y` if `rect.round` is true.

---

### draw() 
Edits the SVG rectangle's properties to match the `Rectangle`. Currently, the only use for this function is if `coordTransformMatrix` has been changed, as it does not change the rectangle according to changes to any other constraining instance properties (e.g. `xBounds`), and changes to the edge properties automatically edit the SVG element.

---

### remove() 

Removes the SVG rectangle (and its resizing handles, if it has them) from its parent.

---
### contains(x, y) 

Returns a boolean indicating if the point (`x`, `y`) is within or on the rectangle.

---
### asBounds()

Returns an object with keys `xBounds` and `yBounds` which can be used to keep another `Rectangle` inside this one.
```js
const bounds = rect.asBounds()
// is equivalent to
const bounds = {
	xBounds: [rect.bbox.x, rect.bbox.x + rect.bbox.width],
	yBounds: [rect.bbox.y, rect.bbox.y + rect.bbox.height]
}
```

---
### get element

The `SVGRectElement` that the `Rectangle` created/modifies.

---
### allowDrag(callback, inside = true)

Allow a user to drag the rectangle with the [pointer](https://developer.mozilla.org/en-US/docs/Web/API/Pointer_events). If `inside` is true, the CSS attribute `pointer-events` is set to `visibleFill`, meaning that an unfilled rectangle can still be dragged by clicking inside it.

`callback` is called, with no arguments, when the rectangle is moved by the user.

---
### allowResize(callback, useRef)

Allow a user to resize the rectangle with the [pointer](https://developer.mozilla.org/en-US/docs/Web/API/Pointer_events). For this, resizing handles are added to each edge and corner. The handles are SVG [`use` elements](https://developer.mozilla.org/en-US/docs/Web/SVG/Reference/Element/use), using `useRef` as their `href` attribute. For example:

```html
<svg width="1000" height="1000">
	<defs>
		<circle id="handle-template" r="4" fill="blue"/>
	</defs>
</svg>
```

```js
const rect = new Rectangle(
	document.querySelector("svg"),
	{x: 10, y: 10, width: 20, height: 20},
	{stroke: "black", fill: "none", "stroke-width": 2}
);
rect.allowResize(() => {}, "#handle-template");
```

`callback` is called, with no arguments, when the rectangle is resized by the user.

---
### allowResizeAndDrag(callback, useRef, inside = true)

Shorthand for `allowResize` and `allowDrag` when using the same callback.
```js
rect.allowResizeAndDrag(callback, useRef, inside)
// is equivalent to
rect.allowResize(callback, useRef)
rect.allowDrag(callback, inside)
```

---
## Instance properties

---
### flippable

`boolean`

Can the rectangle be "flipped", i.e. can `x1` be set or moved by the user to be less than `x0`, and `y1` than `y0`.
N/B: usually, it is desired to call `resetFlip()`  before setting `flippable = true`.

---
### round
`boolean`

When setting edge properties with `set()` or `setPoint0AndShift`, or when a user sets them after `allowResize()` or `allowDrag()` have been called or during `selectArea()`, the given value(s) will be rounded to the nearest integer if `round = true`.

---
### xBounds
`Array` - `[number, number]`

Constrains the span of the x-axis that the rectangle can be moved in. No part of the rectangle can go further left than `xBounds[0]` or further right than `xBounds[1]` (the interval is inclusive, e.g. `x0` or `x1` *can* be equal to `xBounds[0]`).

---
### yBounds
`Array` - `[number, number]`

Constrains the span of the y-axis that the rectangle can be moved in. No part of the rectangle can go further up than `yBounds[0]` or further down than `yBounds[1]` (the interval is inclusive, e.g. `y0` or `y1` *can* be equal to `yBounds[0]`).

---
### coordTransformMatrix

`DOMMatrixReadOnly`

Allows the rectangle to exist in a "virtual" coordinate space other than the SVG coordinate space of its parent element.

This property is a matrix that transforms "virtual" coordinates into SVG coordinates. It is used to transform the `Rectangle`'s properties before drawing/editing the SVG rectangle, and its inverse is used when the user is dragging/resizing/selecting, to transform the SVG coordinates of the user's pointer to "virtual" coordinates for the `Rectangle`.

N/B: user pointer coordinates are already transformed from screen space to SVG space before any transformation by `coordTransformMatrix`.

---
## Static methods

---
### selectArea(parentElement, x, y, style, options)

Create a `Rectangle` with the corner (`x0`, `y0`) at (`x`, `y`). The other corner (`x1`, `y1`) follows the users's pointer until a `pointerup` event is fired on the document.

`parentElement`, `style` and `options` are passed to the `Rectangle`'s constructor (`options` is the object of initial values for the instance properties such as `flippable`).

For example (using the `getSVGCoords` function of [svg_utils](https://github.com/Nobody-In-Particular/svg-utils), which is a dependency of this module):

```js
const svgElement = document.querySelector("svg");
var selectedArea;

svgElement.addEventListener("pointerdown", async function(event) {
	const {x, y} = getSVGCoords(svgElement, event.x, event.y);
	selectedArea = await Rectangle.selectArea(svgElement, x, y, {fill: "none", stroke: "black"}, { round: true });
});

```





