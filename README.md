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
```

---
### get element

The `SVGRectElement` that the `Rectangle` created/modifies.

---
### allowDrag(callback, inside = true)

Allow a user to drag the rectangle with the [pointer](https://developer.mozilla.org/en-US/docs/Web/API/Pointer_events). If `inside` is true, the css attribute `pointer-events` is set to `visibleFill`, meaning that an unfilled rectangle can still be dragged by clicking inside it.

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
rect.allowResize(() => {}, "handle-template");
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

---
### round

---
### xBounds

---
### yBounds

---
### coordTransformMatrix

---
## Static methods

---
### selectArea

---
## License


