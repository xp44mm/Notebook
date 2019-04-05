# bootstrap v4

### Responsive breakpoints for devices

`xs`: Extra small

`sm`: Small

`md`: Medium

`lg`: Large

`xl`: Extra large

## grid

`container`

`container-fluid`

`row`

`col-`

`no-gutters`





## utilities


color

* primary

* secondry

* success

* danger

* warning

* info

* lght

* dark

* white

`rounded`: border-radius

`bg-`: background



`d-`: display

`justify`: 水平对齐

`align`: 垂直对齐

`sr-`：Screenreaders

`w-`: width

`h-`: height

`mw-`: max-width

`mh-`: max-height

### Spacing

The classes are named using the format `{property}{sides}-{size}` for `xs` and `{property}{sides}-{breakpoint}-{size}` for `sm`, `md`, `lg`, and `xl`.

Where *property* is one of:

* `m` - for classes that set `margin`
* `p` - for classes that set `padding`

Where *sides* is one of:

* `t` - for classes that set `top` or `padding-top`
* `b` - for classes that set `margin-bottom` or `padding-bottom`
* `l` - for classes that set `margin-left` or `padding-left`
* `r` - for classes that set `margin-right` or `padding-right`
* `x` - for classes that set both `*-left` and `*-right`
* `y` - for classes that set both `*-top` and `*-bottom`
* blank - for classes that set a `margin` or `padding` on all 4 sides of the element

Where *size* is one of:

* `0` - for classes that eliminate the `margin` or `padding` by setting it to `0`
* `1` - (by default) for classes that set the `margin` or `padding` to `$spacer * .25`
* `2` - (by default) for classes that set the `margin` or `padding` to `$spacer * .5`
* `3` - (by default) for classes that set the `margin` or `padding` to `$spacer`
* `4` - (by default) for classes that set the `margin` or `padding` to `$spacer * 1.5`
* `5` - (by default) for classes that set the `margin` or `padding` to `$spacer * 3`
* `auto` - for classes that set the `margin` to auto：最后取值，计算值

`.mx-auto` class for horizontally centering fixed-width block level content

"font-awesome": "4.7.0"

```html
<html>
<head>
   <meta charset="utf-8">
   <title>font-awesome</title>
   <link href="https://cdn.bootcss.com/bootstrap/4.1.1/css/bootstrap.min.css" rel="stylesheet">
   <link href="https://cdn.bootcss.com/font-awesome/4.7.0/css/font-awesome.min.css" rel="stylesheet">
</head>
<body>
   <button class="btn btn-danger">
      <i class="fa fa-trash">test</i>
   </button>
</body>
</html>
```



