# RWD: Responsive Web Design

## Intro:

### Reference:

- [Responsive Web Design Introduction (w3schools.com)](https://www.w3schools.com/css/css_rwd_intro.asp)

- [Viewport - 术语表 | MDN (mozilla.org)](https://developer.mozilla.org/zh-CN/docs/Glossary/Viewport)

## Viewport:

### What is The Viewport?

The viewport is the user's visible area of a web page.

The viewport varies with the device, and will be smaller on a mobile phone than on a computer screen.

Before tablets and mobile phones, web pages were designed only for computer screens, and it was common for web pages to have a static design and a fixed size.

Then, when we started surfing the internet using tablets and mobile phones, fixed size web pages were too large to fit the viewport. To fix this, browsers on those devices scaled down the entire web page to fit the screen.

This was not perfect!! But a quick fix.

### Setting The Viewport

HTML5 introduced a method to let web designers take control over the viewport, through the `<meta>` tag.

You should include the following `<meta>` viewport element in all your web pages:

```html
<meta name="viewport" content="width=device-width, initial-scale=1.0">
```

This gives the browser instructions on how to control the page's dimensions and scaling.

The `width=device-width` part sets the width of the page to follow the screen-width of the device (which will vary depending on the device).

The `initial-scale=1.0` part sets the initial zoom level when the page is first loaded by the browser.

### Two Examples:

实例1, 没有添加 viewport：[点击查看](https://www.runoob.com/try/demo_source/example_withoutviewport.htm)

实例2, 添加 viewport：[点击查看](https://www.runoob.com/try/demo_source/example_withviewport.htm)

### More Attributes

- width：控制 viewport 的大小，可以指定的一个值，如 600，或者特殊的值，如 device-width 为设备的宽度（单位为缩放为 100% 时的 CSS 的像素）。
- height：和 width 相对应，指定高度。
- initial-scale：初始缩放比例，也即是当页面第一次 load 的时候缩放比例。
- maximum-scale：允许用户缩放到的最大比例。
- minimum-scale：允许用户缩放到的最小比例。
- user-scalable：用户是否可以手动缩放。

### Reference:

- [响应式 Web 设计 – Viewport | 菜鸟教程 (runoob.com)](https://www.runoob.com/css/css-rwd-viewport.html)
- [Responsive Web Design Viewport (w3schools.com)](https://www.w3schools.com/css/css_rwd_viewport.asp)

## Grid-View

### 概述

对于当前页面，进行网格划分并设计

### 示例

```css
.col-1 {width: 8.33%;}
.col-2 {width: 16.66%;}
.col-3 {width: 25%;}
.col-4 {width: 33.33%;}
.col-5 {width: 41.66%;}
.col-6 {width: 50%;}
.col-7 {width: 58.33%;}
.col-8 {width: 66.66%;}
.col-9 {width: 75%;}
.col-10 {width: 83.33%;}
.col-11 {width: 91.66%;}
.col-12 {width: 100%;}
```

### Reference:

- [Responsive Web Design Grid (w3schools.com)](https://www.w3schools.com/css/css_rwd_grid.asp)
- [响应式 Web 设计 – 网格视图 | 菜鸟教程 (runoob.com)](https://www.runoob.com/css/css-rwd-grid.html)

## <u>***IMPORTANT! Media Queries:***</u>

### Syntax

```css
@media not|only mediatype and (mediafeature and|or|not mediafeature) {
  CSS-Code;
}
```

#### **not**, **only** and **and** keywords:

**not:** The not keyword inverts the meaning of an entire media query.

**only:** The only keyword prevents older browsers that do not support media queries with media features from applying the specified styles. **It has no effect on modern browsers.**

**and:** The and keyword combines a media feature with a media type or other media features.

They are all optional. However, if you use **not** or **only**, you must also specify a media type.

>The media queries specification also provides the keyword `only`, which is intended to hide media queries from older browsers. Like `not`, the keyword must come at the beginning of the declaration. For example:

#### Media Type:

| Value  | Description                                           |
| :----- | :---------------------------------------------------- |
| all    | Default. Used for all media type devices              |
| print  | Used for printers                                     |
| screen | Used for computer screens, tablets, smart-phones etc. |
| speech | Used for screenreaders that "reads" the page out loud |

### Example:

```css
@media only screen and (max-width: 600px) {
    body {
      background-color: lightblue;
    }
}
```

### Description

We can add a breakpoint where certain parts of the design will behave differently on each side of the breakpoint.

For the Grids above:

```css
@media only screen and (max-width: 768px) {
    /* For mobile phones: */ /* why mobile here? */
    [class*="col-"] {
      width: 100%;
    }
}
```

### Typical Device Breakpoints

There are tons of screens and devices with different heights and widths, so it is hard to create an exact breakpoint for each device. To keep things simple you could target five groups:

```css
/* Extra small devices (phones, 600px and down) */
@media only screen and (max-width: 600px) {...}

/* Small devices (portrait tablets and large phones, 600px and up) */
@media only screen and (min-width: 600px) {...}

/* Medium devices (landscape tablets, 768px and up) */
@media only screen and (min-width: 768px) {...}

/* Large devices (laptops/desktops, 992px and up) */
@media only screen and (min-width: 992px) {...}

/* Extra large devices (large laptops and desktops, 1200px and up) */
@media only screen and (min-width: 1200px) {...}
```

### Another media feature: {Orientation: Portrait / Landscape}

Media queries can also be used to change layout of a page depending on the orientation of the browser.

You can have a set of CSS properties that will only apply when the browser window is wider than its height, a so called "Landscape" orientation:

```css
@media only screen and (orientation: landscape) {
    body {
        background-color: lightblue;
    }
}
```

### Reference: 

- [CSS @media Rule (w3schools.com)](https://www.w3schools.com/cssref/css3_pr_mediaquery.asp)
- [Responsive Web Design Media Queries (w3schools.com)](https://www.w3schools.com/css/css_rwd_mediaqueries.asp)
- [css - What is the difference between "screen" and "only screen" in media queries? - Stack Overflow](https://stackoverflow.com/questions/8549529/what-is-the-difference-between-screen-and-only-screen-in-media-queries)
- [使用媒体查询 - CSS（层叠样式表） | MDN (mozilla.org)](https://developer.mozilla.org/zh-CN/docs/Web/CSS/Media_Queries/Using_media_queries)

## And More

[Responsive Web Design Frameworks (w3schools.com)](https://www.w3schools.com/css/css_rwd_frameworks.asp)

[Responsive Web Design Templates (w3schools.com)](https://www.w3schools.com/css/css_rwd_templates.asp)

[浅谈响应式Web设计与实现思路 - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/32690777)
