#### HTML复习

1. **HTML** stands for **H**yper**T**ext **M**arkup **L**anguage and is used to create the structure and content of a webpage.
2. Most HTML elements contain opening and closing tags with raw text or other HTML tags between them.
3. Single-closing tags cannot enclose raw text or other elements.
4. Comments are written in HTML using the following syntax: `<!-- comment -->`.
5. HTML elements can be nested inside other elements. The enclosed element is the child of the enclosing parent element.
6. Whitespace between HTML elements helps make code easier to read while not changing how elements appear in the browser.
7. Indentation also helps make code easier to read. It makes parent-child relationships visible.
8. The `<!DOCTYPE html>` declaration should always be the first line of code in your HTML files.
9. The `<html>` element will contain all of your HTML code.
10. Information about the web page, like the title, belongs within the `<head>`of the page.
11. You can add a title to your web page by using the `<title>` element, inside of the head.
12. A webpage's title appears in a browser's tab.
13. Code for visible HTML content is placed inside of the `<body>` element.

#### CSS选择器

- CSS can change the look of HTML elements. In order to do this, CSS must select HTML elements, then apply styles to them.
- CSS can select HTML elements by tag, class, or ID.
- Multiple CSS classes can be applied to one HTML element.
- Classes can be reusable, while IDs can only be used once.
- IDs are more specific than classes, and classes are more specific than tags. That means IDs will override any styles from a class, and classes will override any styles from a tag selector.
- Multiple selectors can be chained together to select an element. This raises the specificity, but can be necessary.
- Nested elements can be selected by separating selectors with a space.
- The `!important` flag will override any style, however it should almost never be used, as it is extremely difficult to override.
- Multiple unrelated selectors can receive the same styles by separating the selector names with commas.

`color` 颜色

- Foreground color:Foreground color is the color that an element appears in. 
- Background color

In CSS, these two design aspects can be styled with the following two properties:

- `color`: this property styles an element's foreground color
- `background-color`: this property styles an element's background color

```css
p{
  color: red;
  background-color: blue;
}
```
`opacity`透明度，从0到1

```css
.overlay {
  opacity: 0.5;
}
```

`background-image`背景图片

```css
.main-banner {
  background-image: url("https://www.example.com/image.jpg");
}
```

#### Box Model

![box_model.png](../img/html_css_js/box_model.png)

 The model includes the content area's size (*width* and *height*) and the element's *padding*, *border*, and *margin*. The properties include:

1. Width and height — specifies the width and height of the content area.
2. Padding — specifies the amount of space between the content area and the border.
3. Border — specifies the thickness and style of the border surrounding the content area and padding.
4. Margin — specifies the amount of space between the border and the outside edge of the element.

An element's content has two dimensions: a height and a width. 

```css
p{
  width:30px;
  height:30px;
}
```

**Border**

A *border* is a line that surrounds an element, like a frame around a painting. Borders can be set with a specific `width`, `style`, and `color`.

1. `width` — The thickness of the border. A border's thickness can be set in pixels or with one of the following keywords: `thin`, `medium`, or `thick`.
2. `style` — The design of the border. Web browsers can render any of [10 different styles](https://developer.mozilla.org/en-US/docs/Web/CSS/border-style#Values). Some of these styles include: `none`, `dotted`, and `solid`.
3. `color` — The color of the border. Web browsers can render colors using a few different formats, including [140 built-in color keywords](https://developer.mozilla.org/en-US/docs/Web/CSS/color_value).

```css
p {
  border: 3px solid coral;
}
```

The default border is `medium none color`, where `color` is the current color of the element.

You can modify the corners of an element's border box with the `border-radius` property.

```css
div.container {
  height: 60px;
  width: 60px;
  border: 3px solid rgb(22, 77, 100);
  border-radius: 100%;
}
```

**padding**

The space between the contents of a box and the borders of a box is known as *padding*. 

1. `padding-top`
2. `padding-right`
3. `padding-bottom`
4. `padding-left`

```css
p.content-header {
  border: 3px solid grey;
  padding: 6px 11px 4px 9px;
}
```

In the example above, the four values `6px 11px 4px 9px` correspond to the amount of padding in a clockwise rotation.

```css
p.content-header {
  padding: 5px 10px;
}
```

The first value, `5px`, sets the padding value for the top and bottom sides of the content. The second value, `10px`, sets the padding value for the left and right sides of the content.

**margin**

Margin refers to the space directly outside of the box. 

1. `margin-top`
2. `margin-right`
3. `margin-bottom`
4. `margin-left`

```css
p {
  margin: 6px 10px 5px 12px;
}
```

In the example above, the four values `6px 10px 5px 12px` refer to the amount of margin around the box in a clockwise rotation.

```css
p {
  margin: 6px 12px;
}
```

The first value, `6px`, sets a margin value for the top and bottom of the box. The second value, `12px`, sets a margin value for the left and right sides of the box.

```css
div.headline {
  width: 400px;
  margin: 0 auto;
}
```

The `auto` value instructs the browser to adjust the left and right margins until the element is centered within its containing element.

In order to center an element, a width must be set for that element. Otherwise, the width of the div will be automatically set to the full width of its containing element, like the `<body>`, for example. It's not possible to center an element that takes up the full width of the page.

**Margin Collapse**

One additional difference is that top and bottom margins, also called vertical margins, *collapse*, while top and bottom padding does not.

Horizontal margins (left and right), like padding, are always displayed and added together.

**Minimum and Maximum Height and Width**

1. `min-width` — this property ensures a minimum width of an element's box.
2. `max-width` — this property ensures a maximum width of an element's box.

```css
p {
  min-width: 300px;
  max-width: 600px;
}
```

1. `min-height` — this property ensures a minimum height for an element's box.
2. `max-height` — this property ensures a maximum height of an element's box.

```css
p {
  min-height: 150px;
  max-height: 300px;
}
```

**Overflow**

The `overflow` property controls what happens to content that spills, or overflows, outside its box. It can be set to one of the following values:

- `hidden` - when set to this value, any content that overflows will be hidden from view.
- `scroll` - when set to this value, a scrollbar will be added to the element's box so that the rest of the content can be viewed by scrolling.
- `visible` - when set to this value, the overflow content will be displayed outside of the containing element. Note, this is the default value.

```css
p {
  overflow: scroll; 
}
```

**Resetting Defaults**

Many developers choose to reset these default values so that they can truly work with a clean slate.

```css
* {
  margin: 0;
  padding: 0;
}
```

**Visibility**

Elements can be hidden from view with the `visibility` property.

The `visibility` property can be set to one of the following values:

1. `hidden` — hides an element.
2. `visible` — displays an element.

```css
.future {
  visibility: hidden;
}
```

**Note:** What's the difference between `display: none` and `visibility: hidden`? An element with `display: none` will be completely removed from the web page. An element with `visibility: hidden`, however, will not be visible on the web page, but the space reserved for it will.

**Review**

1. The box model comprises a set of properties used to create space around and between HTML elements.
2. The height and width of a content area can be set in pixels or percentage.
3. Borders surround the content area and padding of an element. The color, style, and thickness of a border can be set with CSS properties.
4. Padding is the space between the content area and the border. It can be set in pixels or percent.
5. Margin is the amount of spacing outside of an element's border.
6. Horizontal margins add, so the total space between the borders of adjacent elements is equal to the sum of the right margin of one element and the left margin of the adjacent element.
7. Vertical margins collapse, so the space between vertically adjacent elements is equal to the larger margin.
8. `margin: 0 auto` horizontally centers an element inside of its parent content area, if it has a width.
9. The `overflow` property can be set to `display`, `hide`, or `scroll`, and dictates how HTML will render content that overflows its parent's content area.
10. The `visibility` property can hide or show elements.

#### JavaScript复习

- Variables hold reusable data in a program.
- JavaScript will throw an error if you try to reassign `const` variables.
- You can reassign variables that you create with the `let` keyword.
- Unset variables store the primitive data type `undefined`.
- Mathematical assignment operators make it easy to calculate a new value and assign it to the same variable.
- The `+` operator is used to interpolate (combine) multiple strings.
- In JavaScript ES6, backticks (`) and `${}` are used to interpolate values into a string.

All variables that have been created and set are truthy (and will evaluate to true if they are the condition of a control flow statement) unless they contain one of the seven values listed below:

- `false`
- `0` and `-0`
- `""` and `''` (empty strings)
- `null`
- `undefined`
- `NaN` (Not a Number)
- `document.all` (something you will rarely encounter)