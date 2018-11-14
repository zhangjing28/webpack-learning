#### 1. bable-loader
在webpack中babel-loader的相关配置信息可以写在package.json文件的babel属性中或者在项目根目录下创建一个.babelrc文件，建议单独写到.babelrc文件中

babel中主要包括两部分内容

**presets**

    表示babel中的预设，取值可以是数组或者字符串
```
presets:[
    "env",
    "stage-2"
]
// 或者
presets:[
    ["env",{
       "targets": {
           "browsers":["last 2 versions", "ie >= 7"]
       },
       //是否将ES6的模块化语法转译成其他类型
         //参数："amd" | "umd" | "systemjs" | "commonjs" | false，默认为'commonjs'
        "modules": false,
        //是否进行debug操作，会在控制台打印出所有插件中的log，已经插件的版本
        "debug": false,
        //强制开启某些模块，默认为[]
        "include": ["transform-es2015-arrow-functions"],
        //禁用某些模块，默认为[]
        "exclude": ["transform-es2015-for-of"],
        //是否自动引入polyfill，开启此选项必须保证已经安装了babel-polyfill
        //参数：Boolean，默认为false.
        "useBuiltIns": false
    }]
    "stage-2"
]
```

**关于preset常见的配置项** 

* **targets**: 指定要转译到那个环境

* **targets.browsers**: 指定浏览器环境

    取值： `last 2 versions, ie >= 7, safari >= 7，> 5%`等；具体参见[browserlist](https://github.com/browserslist/browserslist)

* **targets.node**: 指定node版本，如6.10，current等
* **modules**: 启动将es6模块语法转换成另一种模块
    取值： "amd"|"umd"|"commonjs"|"systemjs"|false, 默认为"commonjs"
    将其设置为false就不会转换模块，由于现在webpack已经可以转化es6，就不需要babel进行转换了，所以设置为false即可
* **useBuiltIns**: 
    * 'usage': 监测代码中的es6的使用情况，仅仅加载代码中用到的polyfills
    
![举例](https://user-gold-cdn.xitu.io/2018/11/14/1670fa3c77d84419?w=1340&h=426&f=png&s=86451, "举例")

![](https://user-gold-cdn.xitu.io/2018/11/14/1670fa1fef7dc586?w=1688&h=614&f=png&s=148254, "打包结果")
    * false： 默认值，不进行任何polyfill
    
![](https://user-gold-cdn.xitu.io/2018/11/14/1670f9e28a9c774d?w=1464&h=252&f=png&s=69256, "为false时，打包结果")

    * 'entry': 根据运行环境引入插件，只用在import'@babel/polyfill'或者require（'@babel/polyfill'）时使用
    
![](https://user-gold-cdn.xitu.io/2018/11/14/1670fa0d787d9a9c?w=1986&h=316&f=png&s=92557, "为entry的打包结果")
* **loose**: boolean,默认为false，允许为这个preset下的任何插件启用loose转换
* **debug**: boolean,默认为false，将使用的目标浏览器/插件等指定的版本用console.log输出
```
@babel/preset-env: `DEBUG` option

Using targets:
{
  "android": "4.4.3",
  "chrome": "69",
  "edge": "16",
  "firefox": "62",
  "ie": "10",
  "ios": "11.3",
  "safari": "11.1"
}

Using modules transform: auto

Using plugins:
  transform-template-literals { "android":"4.4.3", "ie":"10" }
  transform-literals { "android":"4.4.3", "ie":"10" }
  transform-function-name { "android":"4.4.3", "edge":"16", "ie":"10" }
  transform-arrow-functions { "android":"4.4.3", "ie":"10" }
  transform-block-scoped-functions { "android":"4.4.3", "ie":"10" }
  transform-classes { "android":"4.4.3", "ie":"10" }
  transform-object-super { "android":"4.4.3", "ie":"10" }
  transform-shorthand-properties { "android":"4.4.3", "ie":"10" }
  transform-duplicate-keys { "android":"4.4.3", "ie":"10" }
  transform-computed-properties { "android":"4.4.3", "ie":"10" }
  transform-for-of { "android":"4.4.3", "ie":"10" }
  transform-sticky-regex { "android":"4.4.3", "ie":"10" }
  transform-dotall-regex { "android":"4.4.3", "edge":"16", "firefox":"62", "ie":"10" }
  transform-unicode-regex { "android":"4.4.3", "ie":"10" }
  transform-spread { "android":"4.4.3", "ie":"10" }
  transform-parameters { "android":"4.4.3", "edge":"16", "ie":"10" }
  transform-destructuring { "android":"4.4.3", "edge":"16", "ie":"10" }
  transform-block-scoping { "android":"4.4.3", "ie":"10" }
  transform-typeof-symbol { "android":"4.4.3", "ie":"10" }
  transform-new-target { "android":"4.4.3", "ie":"10" }
  transform-regenerator { "android":"4.4.3", "ie":"10" }
  transform-exponentiation-operator { "android":"4.4.3", "ie":"10" }
  transform-async-to-generator { "android":"4.4.3", "ie":"10" }
  proposal-async-generator-functions { "android":"4.4.3", "edge":"16", "ie":"10", "ios":"11.3", "safari":"11.1" }
  proposal-object-rest-spread { "android":"4.4.3", "edge":"16", "ie":"10" }
  proposal-unicode-property-regex { "android":"4.4.3", "edge":"16", "firefox":"62", "ie":"10" }
  proposal-json-strings { "android":"4.4.3", "chrome":"69", "edge":"16", "firefox":"62", "ie":"10", "ios":"11.3", "safari":"11.1" }
  proposal-optional-catch-binding { "android":"4.4.3", "edge":"16", "ie":"10" }
```
* **include**: Array<string>, 默认为[]; 一定包含的插件

    取值：
    
    * [babel plugins](https://github.com/babel/babel-preset-env/blob/master/data/plugin-features.js) 如: babel-plugin-transform-es2015-spread或者无前缀的transform-es2015-spread
    * [built-ins](https://github.com/babel/babel-preset-env/blob/master/data/built-in-features.js) 如：map，set， object.assign等
* **exclude**：取值和include相同，表示排除哪些插件
    
[preset的配置项](https://babeljs.io/docs/en/babel-preset-env/#options)

**plugin**
* plugins和presets的配置项完全相同，不同之处在于preset的配置项控制预设中包含所有插件，plugins的配置项只针对一个插件
* plugins会运行在presets之前，而且顺序执行，ordering is first to last
* preset的顺序是逆序的，ordering is last to first；逆序的原因是为了保证向后兼容，因为大多数用户都会在`stage-0`之前列出`es-2015`，具体内容参见[关于隐式遍历api变化的说明](https://github.com/babel/notes/blob/master/2016/2016-08/august-01.md#potential-api-changes-for-traversal)
```
{
  "plugins": [
    ["transform-async-to-module-method", {
      "module": "bluebird",
      "method": "coroutine"
    }]
  ]
}
```
[编写自己的babel-plugin手册](https://github.com/jamiebuilds/babel-handbook/blob/master/translations/zh-Hans/plugin-handbook.md)

**babel-polyfill**

模拟一个完整的es2015的环境，并意图运行于一个应用而不是一个库或者工具，需要运行在应用的源代码之前

`npm install --save babel-polyfill`

引用时需要在入口文件的上方引入，要确保它在任何代码或者依赖声明之前调用

`import babel-polyfill`

**babel-plugin-transform-runtime**

需要安装babel-runtime和babel-plugin-transform-runtime分别作为生产依赖和开发依赖
```
npm install --save-dev babel-plugin-transform-runtime
npm intall --sace babel-runtime
```
>转译插件通常只用于开发，但你已部署/已发布的代码依赖于运行时（runtime）

**配置选项**
[相关官网地址](https://babel.docschina.org/docs/en/babel-plugin-transform-runtime#options) [中文babel6链接](https://www.babeljs.cn/docs/plugins/transform-runtime#%E9%80%89%E9%A1%B9)
* **core.js**: boolean or number 默认为`false`
 
    取值：2 | '2' | false | undefined

    `The 'corejs' option must be undefined, false, 2 or '2', but got 4.`
    `npm install --save @babel/runtime-corejs2`
    
![](https://user-gold-cdn.xitu.io/2018/11/13/1670d37302e339cc?w=970&h=688&f=png&s=101377, "2取值可能代表的含义")
>使用新的内置函数，例如 Map，Set，Promise 等。使用这些函数的唯一方式通常是引入一个污染全局的 polyfill。
>注意： 例如 "foobar".includes("foo") 等实例方法将不会正常工作

* **polyfill**: boolean removed in v7 

    是否切换新的内置插件（Promise，Set，Map等）为使用非全局污染的 polyfill。
    
* **helpers**: boolean 默认为`true`

    是否替换将内联(inline)的babel helper( ClassCallCheck, extends等)替换为对moduleName的调用
    
    * **input**
    `class Person{}`
    * **output**
    >当helpers为true时
    ```
    "use strict";
    Object.defineProperty(__webpack_exports__, "__esModule", { value: true });
    /* harmony import */ var __WEBPACK_IMPORTED_MODULE_0__babel_runtime_helpers_classCallCheck__ = __webpack_require__(1);
    /* harmony import */ var __WEBPACK_IMPORTED_MODULE_0__babel_runtime_helpers_classCallCheck___default = __webpack_require__.n(__WEBPACK_IMPORTED_MODULE_0__babel_runtime_helpers_classCallCheck__);
    
    
    var Person = function Person() {
      __WEBPACK_IMPORTED_MODULE_0__babel_runtime_helpers_classCallCheck___default()(this, Person);
    };
    
    /***/ }),
    /* 1 */
    /***/ (function(module, exports) {
    
    function _classCallCheck(instance, Constructor) {
      if (!(instance instanceof Constructor)) {
        throw new TypeError("Cannot call a class as a function");
      }
    }
    
    module.exports = _classCallCheck;
    ```
    > 当helper为false时
    ```
    
    (function(module, exports) {

        function _classCallCheck(instance, Constructor) { if (!(instance instanceof Constructor)) { throw new TypeError("Cannot call a class as a function"); } }
        
        var Person = function Person() {
          _classCallCheck(this, Person);
        };
    })
    ```
* **regenerator**: boolean 默认为`true`

    是否切换 generator 函数为不污染全局作用域的 regenerator 运行时。
* **useBuiltIns**： removed in v7
* **useESModules**: boolean 默认值为`false`
#### 2. 补充说明
##### 2.1 hash说明
>hash是工程级别的缓存，即没修改一个文件，所有的缓存都将失效，这样就失去了缓存的意义，所以会用到后期讲到的chunkHash和contentHash
此部分内容参见[链接](http://www.cnblogs.com/giggle/p/9583940.html)
##### 2.2 npm版本语义化说明
* 语义化的版本控制(semver)
>semver设计的初衷在于允许依赖的版本是动态的，因而可以快捷地获取到一定范围内的最新版本。

>版本号为a.b.c的形式，其中a是大版本号，b是小版本号，c是补丁号
```
^version：对应所有小版本号和补丁号。比如 ^1.0.1 可以理解为所有形式为 1.x.x的最新版本。
 ~version：对应所有补丁版本号。比如 ~1.0.1可以理解为所有形式为 1.0.x 的最新版本。
version：对应特定版本。比如版本号为 1.0.1 ，则安装的模块版本只能为 1.0.1。
```
动态版本使得开发者无需更改 pakcage.json 中的依赖版本信息，便可以获取到他所定义的范围内最新的版本。比如当我们通过 npm install jspdf 安装了这个包之后，package.json 中它的版本号为 ^1.3.5。当 split 这个包的作者发布了一个新的版本 1.3.6 的时候，我们下次重新执行 npm install就会装这个 1.3.6 的版本了。
>semver 带来的优势是可以使 npm 包的发布者很方便地补丁推送到使用者。比如当有一些小的 补丁 时候，用户不需要有任何感知，便可以将其中的问题修复。其实 npm 对于各个版本号的意义也有约定。
```
补丁版本发布（如 1.0.1 -> 1.0.2）：bugfix 以及微小的改动
小版本发布（如 1.0.1 -> 1.1.0）：添加了新的特性，同时并不会破坏原有的功能或更改接口
大版本发布（如 1.0.1 -> 2.0.0）：添加了破坏性的改动，有功能或接口不再兼容
```
>npm 允许使用特殊符号，指定所要使用的版本范围，假定当前版本是1.0.4
```
只接受补丁包：1.0 或者 1.0.x 或者 ~1.0.4
只接受小版本和补丁包：1 或者 1.x 或者 ^1.0.4
接受所有更新：* or x
```
Beta：也是测试版，这个阶段的版本会一直加入新的功能。在Alpha版之后推出。

RC：(Release　Candidate) 顾名思义么 ! 用在软件上就是候选版本。系统平台上就是发行候选版本。RC版不会再加入新的功能了，主要着重于除错。

[参考内容](https://www.qdtalk.com/2018/09/27/%E6%80%BB%E7%BB%93%E4%BA%86%E4%B8%80%E4%BA%9B%E5%85%B3%E4%BA%8Enpm%E7%9A%84%E4%B8%80%E4%BA%9B%E5%B0%8F%E6%8A%80%E5%B7%A7/)
