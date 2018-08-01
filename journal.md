
### 性能与异常监控

#### 1.概述

对前端进行性能和异常的监控探索是有必要的

#### 2.异常捕获
  + 接口调用错误
  + 页面逻辑的错误

  **异常捕获的方式**

   + 1. 全局捕获

   ```
    window.onerror = function(errorMessage, scriptURI, lineNo, columnNo, error) {
    console.log('errorMessage: ' + errorMessage); // 异常信息
    console.log('scriptURI: ' + scriptURI); // 异常文件路径
    console.log('lineNo: ' + lineNo); // 异常行号
    console.log('columnNo: ' + columnNo); // 异常列号
    console.log('error: ' + error); // 异常堆栈信息
    // ...
    // 异常上报
    };
    throw new Error('这是一个错误');

 ```  
  + 2. `window.addEventListener`方式
  + 3. `try catch `方式

#### 3. 常见问题

  + 1. **跨域脚本无法准确捕获异常**

  通常情况下 `javascript`脚本是放在静态资源服务器上的，或者是**CDN**

  在`index.html`页面中添加全局监控后，发现是捕获不到正确的异常信息,而是统一返回一个`script`错误

  **解决方法：**

  对`script`标签添加一个`crossorigin=”anonymous`并且服务器添加`Access-Control-Allow-Origin`

  `<script src="http://cdn.xxx.com/index.js" crossorigin="anonymous"></script>`

  + 2. 通常情况下我们的代码是经过`webpack`打包压缩混淆的代码, 所以全局捕获的时候，返回的信息行数都是**1**.

  解决办法是开启`webpack`的`source-map`，利用打包生成的`.map`脚本文件,可以让浏览器追踪到位置.
  但是问题是只有`chrom`和`firefox`支持,不过也有解决办法,就是引入`npm`库，推荐使用`nodejs`对接受到的日志时使用`source-map`解析,避免源代码造成风险

  ```
    const express = require('express');
    const fs = require('fs');
    const router = express.Router();
    const sourceMap = require('source-map');
    const path = require('path');
    const resolve = file => path.resolve(__dirname, file);
    // 定义post接口
    router.get('/error/', async function(req, res) {
        // 获取前端传过来的报错对象
        let error = JSON.parse(req.query.error);
        let url = error.scriptURI; // 压缩文件路径
        if (url) {
            let fileUrl = url.slice(url.indexOf('client/')) + '.map'; // map文件路径
            // 解析sourceMap
            let consumer = await new sourceMap.SourceMapConsumer(fs.readFileSync(resolve('../' + fileUrl), 'utf8')); // 返回一个promise对象
            // 解析原始报错数据
            let result = consumer.originalPositionFor({
                line: error.lineNo, // 压缩后的行号
                column: error.columnNo // 压缩后的列号
            });
            console.log(result);
        }
    });
    module.exports = router;

  ```

  ### 4. vue捕获异常
  在vue中，使用`window.onerror`可能会捕获不到错误，原因是被`vue`自身的`try catch`捕捉了，
  在这里我们可以用`Vue.config.errorHandler`全局配置,可以指定组件的渲染和观察期间为捕获的处理函数,这个处理函数给调用时,可获取错误信息和vue实例

  ```
  Vue.config.errorHandler = function (err, vm, info) {
  // handle error
  // `info` 是 Vue 特定的错误信息，比如错误所在的生命周期钩子
  // 只在 2.2.0+ 可用
}

```

### 5. react 捕获异常

```
在React中，可以使用ErrorBoundary组件包括业务组件的方式进行异常捕获，配合React 16.0+新出的componentDidCatch API，可以实现统一的异常捕获和日志上报。
```
 **组件**

 ```
 class ErrorBoundary extends React.Component {
  constructor(props) {
    super(props);
    this.state = { hasError: false };
  }

  componentDidCatch(error, info) {
    // Display fallback UI
    this.setState({ hasError: true });
    // You can also log the error to an error reporting service
    logErrorToMyService(error, info);
  }

  render() {
    if (this.state.hasError) {
      // You can render any custom fallback UI
      return <h1>Something went wrong.</h1>;
    }
    return this.props.children;
  }
}
```
 **使用方式**
```
<ErrorBoundary>
  <MyWidget />
</ErrorBoundary>
```


[文章参考](https://juejin.im/post/5b5dcfb46fb9a04f8f37afbb)