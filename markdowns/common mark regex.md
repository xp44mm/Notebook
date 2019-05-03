line ending

```javascript
let re = /(?<!\r)\r?\n/
```

blank line

```js
let re = /^[ \t]*$/m
let re = /^[\s-[\f\n\r\v]]*$/m
```

whitespace character

```js
let ws = /[ \t\n\v\f\r]/
```

link文本是textual content

plain string content

不支持raw html

Shortcut

第一优先级

Code spans, HTML tags, and autolinks have the same precedence. 

Backslash escapes do not work in code spans, autolinks



