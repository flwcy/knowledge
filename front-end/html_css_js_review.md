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