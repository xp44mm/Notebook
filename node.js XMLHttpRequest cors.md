node.js的根为`global`，它没有属性`XMLHttpRequest`。这个属性在浏览器中有，并且支持cors。

node.js需要安装包'xmlhttprequest'，提供与浏览器端同样的功能。rxjs/ajax会寻找node.js的`global.XMLHttpRequest`发送请求。调用之前应提供如下代码：

```javascript
import { XMLHttpRequest } from 'xmlhttprequest'
global.XMLHttpRequest || (global.XMLHttpRequest = XMLHttpRequest)
```

