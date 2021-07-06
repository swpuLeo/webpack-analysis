# webpack 源码分析

使用 webpack 的两种方式：

1、webpack-cli 这种方式也会使用 require('webpack')

2、require('webpack')

所以从 webpack() 函数开始：

> webpack.js

```js
const webpack = (options, callback) => {
  const compiler = createCompiler(options);
  
  if (callback) {
    compiler.run((err, states) => {
        compiler.close(err2 => {
        callback(err || err2, states);
      });
    })
  }
  
  return compiler;
}
```

webpack 函数接受两个参数 options 和 callback，返回一个 compiler 实例。

在这个过程中 webpack 主要做：

1、规范化 options：以函数 getNormalizedWebpackOptions 为入口去分析

2、生成一个 compiler 实例：Compiler 类会初始化钩子

3、注册 plugins（自定义插件和内置插件）：webpack 对插件强制规定必须为一个函数，或者一个包含 apply 方法的对象。apply 接受 compiler 实例作为参数，从而使插件可以在 webpack 的工作流程中注册不同的 hook

4、如果存在 callback，那么会直接 run，否则需要手动 run


run 就开始了 webpack 的工作，每一次 run 都会生成一个 compilation 实例。不同于 compiler 只会生成一次，compilation 在 watch 模式下，每一次变动都会重新生成。

run 过程会触发下面的钩子：
**beforeRun -> run -> beforeCompile -> compile -> make -> finishMake -> compilation.finish -> compilation.seal -> afterCompile  -> done -> afterDone**

整个主流程就大概是这样，后续会先分析这些钩子的注册。

基于 [Tapable](https://github.com/webpack/tapable) 库，webpack 完成事件的注册，并保证触发时他们的回调函数按序执行。

关于 Tapable 需要提前知道的是：

1、它提供同步和异步两种钩子

2、注册方式：tap / tapAsync / tapPromise。触发方式：call
