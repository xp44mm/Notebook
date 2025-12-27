# Wild-Card Characters Reference

Here's a list of all possible wild-card characters when searching for names or text strings within product features.

```js
//# (Pound)
Matches any numeric digit

//@ (At)
Matches any alphabetic character

//. (Period)
Matches any non-alphanumeric character

//* (Asterisk)
Matches any string and can be used anywhere in the search string

//? (Question mark)
Matches any single character; for example, /?BC/ matches ABC, 3BC, and so on

//~ (Tilde)
Matches anything but the pattern; for example; /~*AB*/ matches all strings that don't contain AB

//[ ]
Matches any one of the characters enclosed; for example, /[AB]C/ matches AC and BC

//[~]
Matches any character not enclosed; for example, /[~AB]C/ matches XC but not AC

//[-]
Specifies a range for a single character; for example, /[A-G]C/ matches AC, BC, and so on to GC, but not HC

//` (Reverse quote)
Reads the next character literally; for example, /`~AB/ matches ~AB
```


AutoCAD 颜色索引 (ACI)

ACI 颜色是在基于 AutoCAD 的产品中使用的标准颜色。每种颜色均通过 ACI 编号（1 到 255 之间的整数）标识。标准颜色名称仅用于颜色 1 到 7。颜色指定如下：1 红、2 黄、3 绿、4 青、5 蓝、6 洋红、7 白/黑。

优点：出现在下拉列表框种的颜色文本是简单的数字比如10,20,..240，而不是RGB表示值255,255,255






