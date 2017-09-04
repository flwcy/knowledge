## CSS Start

### 认识CSS

`CSS`全称为“层叠样式表 (Cascading Style Sheets)”，它主要是用于定义HTML内容在浏览器内的显示样式，如文字大小、颜色、字体加粗等。

`CSS`规则由两个主要的部分构成：选择器，以及一条或多条声明。

每条声明由一个属性和一个值组成。

![ct_css_selector](../img/html_css_js/css_syntax_declaration.png)

[属性](https://developer.mozilla.org/en-US/docs/Glossary/property/CSS)（property）是您希望设置的样式属性（style attribute）。每个属性有一个值。属性和值被`冒号`分开。

```
selector {property: value}
```

![ct_css_selector](../img/html_css_js/ct_css_selector.png)

+ 使用花括号来包围声明。
+ 如果值为若干单词，则要给值加引号。

一个CSS快速示例：

```html
<!DOCTYPE HTML>
<html>
<head>
<meta http-equiv="Content-Type" content="text/html; charset=utf-8">
<title>认识CSS样式</title>
<style type="text/css">
p{
   font-size:20px;/*设置文字字号*/
   color:red;/*设置文字颜色*/
   font-weight:bold;/*设置字体加粗*/
}
</style>
</head>
<body>
    <p>This is my first CSS example</p>
</body>
</html>
```

### CSS是怎样工作的？

当浏览器显示文档时，它必须将文档的内容与其样式信息结合。它分两个阶段处理文档：

1. 浏览器将[HTML](https://developer.mozilla.org/en-US/docs/Glossary/HTML) 和[CSS](https://developer.mozilla.org/en-US/docs/Glossary/CSS) 转化成[DOM](https://developer.mozilla.org/en-US/docs/Glossary/DOM) （*文档对象模型*）。DOM在计算机内存中表示文档。它把文档内容和其样式结合在一起。
2. 浏览器显示 DOM 的内容。

![rendering.png](../img/html_css_js/rendering.png)