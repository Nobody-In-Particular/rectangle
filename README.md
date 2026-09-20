# Rectangle
Utility SVG rectangle element API, geared towards user selection/dragging/resizing.

There are two "levels" of API. At the higher level, the API allows the user to drag and resize a rectangle and select an area.

At the lower level, the methods `set` and `setPoint0AndShift` simply allow moving and resizing of the rectangle by reference to corners (`x0`/`y0`/`x1`/`y1`), with no requirement that `x0 <= x1` or `y0 <= y1`, thus allowing easy "flipping" of the rectangle since SVG does not support negative width/height.

## Example
```js
svgElement = document.createElement("svg");
area = await Rectangle.selectArea(
	svgElement,
	0, 0,
	{fill: "none", stroke: "black", "stroke-width": 2}
)
```