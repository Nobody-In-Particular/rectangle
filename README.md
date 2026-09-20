# Rectangle
Utility SVG rectangle element API, geared towards user selection/dragging/resizing.

There are two "levels" of API. At the higher level, the API allows the user to drag and resize a rectangle and select an area.

At the lower level, the API allows moving and resizing of the rectangle by reference to corners (`x0`/`y0`/`x1`/`y1`), with no requirement that `x0 <= x1` or `y0 <= y1`, thus allowing easy "flipping" of the rectangle, since SVG does not support negative width/height.

---

## Example
```js
const svgElement = document.createElement("svg");
const area = await Rectangle.selectArea(
	svgElement,
	0, 0,
	{fill: "none", stroke: "black", "stroke-width": 2}
);
area.set("x1", area.x1 - 2);
```

## Methods

### constructor
```js
new Rectangle(
	parentElement, bbox, style,
	{flippable = true, xBounds = null, yBounds = null, round = true, coordTransformMatrix = new DOMMatrixReadOnly()} = {}
)
```

Adds a new SVG `rect` element to `parentElement` ([`SVGElement`](https://developer.mozilla.org/en-US/docs/Web/API/SVGElement)).

`bbox` is an object with properties `x`, `y`, `width` and `height` and defines the initial position/size of the rectangle. The rectangle always starts "unflipped", so the corner coordinates will initially be as follows:

|      |   |
|---   |---|
| `x0` | `bbox.x` |
| `y0` | `bbox.y` |
| `x1` | `bbox.x + bbox.width` |
| `y1` | `bbox.y + bbox.height` |

`style` is an object of other attributes to be added to the element, e.g. `{fill: "white"}`