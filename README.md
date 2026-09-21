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

## Methods

### constructor
```js
new Rectangle(
	parentElement, bbox, style,
	{flippable = true, xBounds = null, yBounds = null, round = true, coordTransformMatrix = new DOMMatrixReadOnly()} = {}
)
```

Adds a new SVG `rect` element as a child of `parentElement` (an `SVGElement`).

`bbox` is an object with properties `x`, `y`, `width` and `height` and defines the initial position/size of the rectangle. The rectangle always starts "unflipped", so the corner coordinates will initially be as follows:

|      |   |
|---   |---|
| `x0` | `bbox.x` |
| `y0` | `bbox.y` |
| `x1` | `bbox.x + bbox.width` |
| `y1` | `bbox.y + bbox.height` |

`style` is an object of other attributes to be added to the element, e.g. `{fill: "white"}`

The configuration options `flippable`, `xBounds`, `yBounds`, `round` and `coordTransformMatrix` set the initial values for the corresponding instance properties.

---

### set(prop, val)

Sets edge property `prop` (one of `x0`, `y0`, `x1`, `y1`), to `val`, obeying restrictions set by the instance properties, and rounding `val` if `rect.round` is true.

The usefulness of the "lower level" of the API is in the fact that the property names are *conserved*, e.g. if `x1` is set further left than `x0`, it remains `x1` rather than becoming `x0`.
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
### setPoint0AndShift(x, y)
Moves the rectangle such that `x0` moves to `x` and `y0` moves to `y`, obeying restrictions set by the instance properties, and rounding `x` and `y` if `rect.round` is true.
---


