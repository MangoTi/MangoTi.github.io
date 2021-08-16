## 前言

webpack是一个js应用程序的静态模块打包器，它会递归地构建一个依赖关系图，包含应用程序需要的每个模块，然后将所有模块打包成一个或多个bundle。

## 核心概念

1. entry：入口起点指示webpack从哪个模块作为构建依赖图的开始，进入入口起点后，递归处理每个依赖项。
2. output：在哪里输出所创建的bundles，以及如何命名，默认值为./dist
3. module：在webpack中一切皆模块，一个模块对应一个文件
4. chunk：代码块，一个chunk由多个模块组合而成，用于代码分割和合并
5. loader：loader帮助webpack处理非js文件，将所有类型的文件转换成webpack可以处理的模块
6. plugin：功能广泛，从打包优化和压缩，到定义环境变量，可以用来处理各种任务
7. compiler：执行编译的对象，包含了完整的webpack配置，全局只有一个实例

```
//简易示例
class Compiler {
  constructor(options) {
    // webpack 配置
    const { entry, output } = options
    // 入口
    this.entry = entry
    // 出口
    this.output = output
    // 模块
    this.modules = []
  }
  // 构建启动
  run() {}
  // 重写 require函数,输出bundle
  generate() {}
}
```

## 构建流程

1. 初始化参数：从配置文件和shell语句读取与合并参数，得出最终参数，还会执行配置文件中的new Plugin()
2. 开始编译：用上一步得到的参数初始化compiler对象，加载所有配置的插件，执行对象的run方法开始执行编译
3. 确定入口：根据entry找出所有入口文件
4. 编译模块：从入口文件出发，调用所有配置的loader对模块进行翻译，再根据import（ESM）或require（CommonJS）找出依赖的模块，递归直到所有入口依赖的文件都执行好
5. 完成模块编译：得到每个模块被翻译后的最终内容以及它们之间的依赖关系
6. 输出资源：根据依赖关系图，组装成包含多个模块的chunk，转换成单独的文件加入输出列表
7. 输出完成：确定好输出内容后，根据配置确定输出的路径和文件名，写入文件系统。

## plugin原理

1. 在webpack运行的生命周期中会广播许多事件，plugin可以监听这些事件，在合适的时机通过webpack提供的API改变输出结果。

```
class BasicPlugin{
  // 在构造函数中获取用户给该插件传入的配置
  constructor(options){
  }

  // Webpack 会调用 BasicPlugin 实例的 apply 方法给插件实例传入 compiler 对象
  apply(compiler){
         compiler.plugin('compilation',function(compilation) {
    })
  }
}

// 导出 Plugin
module.exports = BasicPlugin;

// 使用方法
const BasicPlugin = require('./BasicPlugin.js');
module.export = {
  plugins:[
    new BasicPlugin(options),
  ]
}
```

webpack启动后，在读取配置的过程中会先执行new BasicPlugin(options)初始化一个BasicPlugin获得实例。在初始化compiler对象后，再调用basicPlugin.apply(compiler)给插件实例传入compiler对象。插件实例在获取到compiler对象后，就可以通过compiler.plugin(事件名称，回调函数)监听到webpack广播出来的事件。并且可以通过compiler对象去操作webpack。

2. 
complication对象包含了当前的模块资源、编译生成资源、变化的文件等，当webpack以开发模式运行时，每当检测到一个文件变化，一个新的complication将被创建，该对象提供了很多事件回调供插件做拓展，也能读取到compiler对象。

3. 
webpack通过tapable来组织文件输出的生产线，插件只需要监听它关心的广播事件，就能加入到生产线中去改变生产线的运作，compiler和compilation都继承自tapable，都可以广播和监听事件。


## Tree Shaking

1. 用来提出JS中用不上的死代码，依赖静态的ES6模块化语法，最先在rollup中出现，webpack在2.0版本中引入，例如如果只用到了import的utils.js中的方法a，tree shaking之后utils.js中的其他方法会被剔除。
2. 配置方法：

- 配置babel，保留ES6模块化语句，修改.babelrc文件：

```
{
  "presets": [
    [
      "env",
      {
        "modules": false
      }
    ]
  ]
}
```

- 接入uglifyJS（js代码压缩工具）去剔除用不上的代码，使用UplifyJSPlugins实现
- 大部分的npm中的组件都是用commonjs语法，导致tree shaking无法正常工作，有些库考虑到了这点，发布时也提供ES6模块化语法，使用mainFields配置哪个字段作为模块的入口

```
node_modules/redux/package.json
{
  "main": "lib/index.js", // 指明采用 CommonJS 模块化的代码入口
  "jsnext:main": "es/index.js" // 指明采用 ES6 模块化的代码入口
}
// 配置入口优先
module.exports = {
  resolve: {
    // 针对 Npm 中的第三方模块优先采用 jsnext:main 中指向的 ES6 模块化语法的文件
    mainFields: ['jsnext:main', 'browser', 'main']
  },
};
```

如果不存在jsnext:main 就采用brower或者main作为入口，能优化的就有话

## webpack和rollup的对比

1. rollup是下一代JavaScript模块打包工具，可以在应用或库中使用ES6模块，然后高效地将它们打包成一个单一的文件用于浏览器和nodejs使用，tree-shaking变得更容易，比webpack2.0+做得更干净。
2. webpack：对代码分割和静态资源导入较优秀，支持热模式替换，适合开发中大型应用使用
3. rollup：只需要打包一个简单的bundle包，并使用ES6模块开发，适合用来开发库使用

## babel原理

1. 核心是编译器（parse）、转换过程（transform）、生成器（generator）三个部分。
2. 过程如下：

- babylon将源代码经过编译器得到抽象语法树AST
- plugin用babel-traverse将抽象语法树转换成目标代码的AST
- babel-generator将AST生成转换后的代码

## 参考

[https://webpack.wuhaolin.cn](https://webpack.wuhaolin.cn)

webpack： [https://segmentfault.com/a/1190000021494964](https://segmentfault.com/a/1190000021494964)

rollup： [https://baijiahao.baidu.com/s?id=1619539794578155307&wfr=spider&for=pc](https://baijiahao.baidu.com/s?id=1619539794578155307&wfr=spider&for=pc)

babel： [https://baijiahao.baidu.com/s?id=1619539794578155307&wfr=spider&for=pc](https://baijiahao.baidu.com/s?id=1619539794578155307&wfr=spider&for=pc)
