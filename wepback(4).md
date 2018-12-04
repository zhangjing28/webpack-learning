# 1. html-webpack-plugin
## 1.1 相关介绍
### 1.1.1 作用
该插件主要用于自动生成一个能作为系统入口的HTML5的名为index.html文件，之所以index.html文件需要动态生成，而并非直接写死，主要是因为webpack打包后生成的js文件名中有可能会带有每次编译打包都会改变的hash值，所以需要动态引入webpack打包后生成的js文件；通过该插件，会将webpack打包生成的bundles通过script标签引入到生成的index.html文件中。
### 1.1.2  简单使用
```
const path = require("path");
const HtmlWebpackPlugin = require("html-webpack-plugin")
module.exports = {
    context: path.resolve(__dirname, 'src')
    entry: {
        app: './index.js'
    },
    output: {
        filename: '[name].bundle.js',
        path: path.resolve(__dirname, 'dist')
    },
    plugins:[
        new HtmlWebpackPlugin({
            text: 'html demo'
        })
    ]
}
```
上述文件会生成dist/index.html文件
```
<!DOCTYPE html>
<html>
    <head>
        <meta charset="utf-8">
        <title>html demo</title>
    </head>
    <body>
        <script src="app.bundle.js"></script>
    </body>
</html>
```
**说明**

    如果有多个bundle文件，那么将会生成多个script标签来引入这些bundle
    使用ExtractTextPlugin生成的css文件也会用link标签包裹放入html的head中
    如果在项目中使用了多个插件，html-webpack-plugin插件一定要在插件数组的第一个
### 1.1.3 相关options
* title： 生成的html上的title标签对应的取值 `string`
* filename：生成的html文件的名称，默认为index.html `string`
* template：可以将该属性对应的路径的文件作为模板文件创建一个html文件 `string`
* templateParameters: 允许修改模板中的参数 `boolean | object | function`
* inject: 将所有资源添加到模板文件或者模板内容中 `boolean | string`

    *  `true | 'body'`: 将所有的js资源添加到body标签的底部
    * `head`: 将所有js资源添加到head标签中
* favicon: 在输出的html文件中添加给定的favicon地址 `string`
* meta: 允许添加meta标签 `object`
* minify: 是否压缩输出的html文件`boolean | object`，只有当`mode: 'production'`时，为`true`；其他时候都是`false`
* hash: 为`true`时，webpack会生成一个带有hash值的js和css文件 `boolean`
* showErrors: 是否在生成的html上展示错误信息 `boolean`
* chunks: 允许用户仅添加某些chunks到生成的html中，而不是默认加载所有chunks `Array<string>`
* chunksSortMode: 控制生成的html中chunks加载顺序 `string | function`
* excludeChunks: 控制生成的文件中那些chunks不进行引入 `Array<string>`
* xhtml: 生成的html文件是否采用xhtml模式

**注**：html模板默认采用的是ejs模板

[html-webpack-plugin官网](https://github.com/jantimon/html-webpack-plugin)

# 2. 实现一个plugin
## 2.1 举例
```
class HelloPlugin{
    constructor(options){
        this.options = options || {}
    }
    apply(compiler) {
        compiler.hooks.done.tap('helloPlugin', () => {
            console.log('Hello Plugin!')
            console.log('options=', this.options)
        })  
    }
}
// 修改webpack.config.js文件
const HelloPlugin = require('./scr/HelloPlugin')
module.exports = {
    //...
   plugins: [
        new HelloPlugin({test: 'testOption'})
   ] 
}
```
![例1的结果](https://user-gold-cdn.xitu.io/2018/12/3/16773bc85950d0cc?w=752&h=430&f=png&s=132469, "例1的结果")
### 2.2 编写plugin的基本步骤
 #### 2.2.1 定义`apply`方法
 webpack进行打包时的入口是`webpack.js`
 ```
    compiler = new Compiler(options.context);
    compiler.options = options;
    new NodeEnvironmentPlugin().apply(compiler);
    if (options.plugins && Array.isArray(options.plugins)) {
    	for (const plugin of options.plugins) {
    		if (typeof plugin === "function") {
    			plugin.apply(compiler);
    		} else {
    			plugin.apply(compiler);
    		}
    	}
    }
 ```
 由上述代码可知，webpack会循环`webpack.config.js`文件中的`plugins`属性中的所有插件，并调用插件中的`apply`方法，同时会将`compiler`对象传入`apply`方法中
 
 
#### 2.2.2 指定一个绑定到webpack自身的事件钩子
这里说的webpack自身的事件钩子指的就是`compiler`对象上的事件钩子,即在源码`Compiler.js`文件中的`hooks`字段对应的属性即`compiler`对象上所有的钩子
>`Compiler`是webpack引擎的支柱，它通过`CLI`或者`NODE API`传递所有选项，创建一个`Compilation`实例。它扩展自`Tapable`，提供了很多钩子函数

关于以下代码中钩子函数执行的阶段参见[官网](https://webpack.docschina.org/api/compiler-hooks/)
```
// Compiler.js
class Compiler extends Tapable {
    constructor(context) {
		super();
		this.hooks = {
			/** @type {SyncBailHook<Compilation>} */
			shouldEmit: new SyncBailHook(["compilation"]),
			/** @type {AsyncSeriesHook<Stats>} */
			done: new AsyncSeriesHook(["stats"]),
			/** @type {AsyncSeriesHook<>} */
			additionalPass: new AsyncSeriesHook([]),
			/** @type {AsyncSeriesHook<Compiler>} */
			beforeRun: new AsyncSeriesHook(["compiler"]),
			/** @type {AsyncSeriesHook<Compiler>} */
			run: new AsyncSeriesHook(["compiler"]),
			/** @type {AsyncSeriesHook<Compilation>} */
			emit: new AsyncSeriesHook(["compilation"]),
			/** @type {AsyncSeriesHook<Compilation>} */
			afterEmit: new AsyncSeriesHook(["compilation"]),

			/** @type {SyncHook<Compilation, CompilationParams>} */
			thisCompilation: new SyncHook(["compilation", "params"]),
			/** @type {SyncHook<Compilation, CompilationParams>} */
			compilation: new SyncHook(["compilation", "params"]),
			/** @type {SyncHook<NormalModuleFactory>} */
			normalModuleFactory: new SyncHook(["normalModuleFactory"]),
			/** @type {SyncHook<ContextModuleFactory>}  */
			contextModuleFactory: new SyncHook(["contextModulefactory"]),

			/** @type {AsyncSeriesHook<CompilationParams>} */
			beforeCompile: new AsyncSeriesHook(["params"]),
			/** @type {SyncHook<CompilationParams>} */
			compile: new SyncHook(["params"]),
			/** @type {AsyncParallelHook<Compilation>} */
			make: new AsyncParallelHook(["compilation"]),
			/** @type {AsyncSeriesHook<Compilation>} */
			afterCompile: new AsyncSeriesHook(["compilation"]),

			/** @type {AsyncSeriesHook<Compiler>} */
			watchRun: new AsyncSeriesHook(["compiler"]),
			/** @type {SyncHook<Error>} */
			failed: new SyncHook(["error"]),
			/** @type {SyncHook<string, string>} */
			invalid: new SyncHook(["filename", "changeTime"]),
			/** @type {SyncHook} */
			watchClose: new SyncHook([]),

			// TODO the following hooks are weirdly located here
			// TODO move them for webpack 5
			/** @type {SyncHook} */
			environment: new SyncHook([]),
			/** @type {SyncHook} */
			afterEnvironment: new SyncHook([]),
			/** @type {SyncHook<Compiler>} */
			afterPlugins: new SyncHook(["compiler"]),
			/** @type {SyncHook<Compiler>} */
			afterResolvers: new SyncHook(["compiler"]),
			/** @type {SyncBailHook<string, Entry>} */
			entryOption: new SyncBailHook(["context", "entry"])
		};
		//...
	}
	//...
}
```

#### 2.2.3 使用webpack提供的plugin API操作构建结果
>这里主要是使用`compilation`上的钩子来操作构建结果，达到插件的作用
`Compilation`模块被`Compiler`用来创建新的编译或者构建，`Compilation`实例能够访问所有的模块和他们的依赖(大部分都是循环依赖)，它会对应用程序依赖图中的所有模块进行字面上的编译，在编译过程中，模块会被加载(loaded),封存(sealed),优化(optimized),分块(chunked),哈希(hashed)和重新构建(restored)

关于以下代码中钩子函数执行的阶段参见[官网](https://webpack.docschina.org/api/compilation-hooks/)
```
//Compilation.js
class Compilation extends Tapable {
	/**
	 * Creates an instance of Compilation.
	 * @param {Compiler} compiler the compiler which created the compilation
	 */
	constructor(compiler) {
		super();
		this.hooks = {
			/** @type {SyncHook<Module>} */
			buildModule: new SyncHook(["module"]),
			/** @type {SyncHook<Module>} */
			rebuildModule: new SyncHook(["module"]),
			/** @type {SyncHook<Module, Error>} */
			failedModule: new SyncHook(["module", "error"]),
			/** @type {SyncHook<Module>} */
			succeedModule: new SyncHook(["module"]),

			/** @type {SyncHook<Dependency, string>} */
			addEntry: new SyncHook(["entry", "name"]),
			/** @type {SyncHook<Dependency, string, Error>} */
			failedEntry: new SyncHook(["entry", "name", "error"]),
			/** @type {SyncHook<Dependency, string, Module>} */
			succeedEntry: new SyncHook(["entry", "name", "module"]),

			/** @type {SyncWaterfallHook<DependencyReference, Dependency, Module>} */
			dependencyReference: new SyncWaterfallHook([
				"dependencyReference",
				"dependency",
				"module"
			]),

			/** @type {SyncHook<Module[]>} */
			finishModules: new SyncHook(["modules"]),
			/** @type {SyncHook<Module>} */
			finishRebuildingModule: new SyncHook(["module"]),
			/** @type {SyncHook} */
			unseal: new SyncHook([]),
			/** @type {SyncHook} */
			seal: new SyncHook([]),

			/** @type {SyncHook} */
			beforeChunks: new SyncHook([]),
			/** @type {SyncHook<Chunk[]>} */
			afterChunks: new SyncHook(["chunks"]),

			/** @type {SyncBailHook<Module[]>} */
			optimizeDependenciesBasic: new SyncBailHook(["modules"]),
			/** @type {SyncBailHook<Module[]>} */
			optimizeDependencies: new SyncBailHook(["modules"]),
			/** @type {SyncBailHook<Module[]>} */
			optimizeDependenciesAdvanced: new SyncBailHook(["modules"]),
			/** @type {SyncBailHook<Module[]>} */
			afterOptimizeDependencies: new SyncHook(["modules"]),

			/** @type {SyncHook} */
			optimize: new SyncHook([]),
			/** @type {SyncBailHook<Module[]>} */
			optimizeModulesBasic: new SyncBailHook(["modules"]),
			/** @type {SyncBailHook<Module[]>} */
			optimizeModules: new SyncBailHook(["modules"]),
			/** @type {SyncBailHook<Module[]>} */
			optimizeModulesAdvanced: new SyncBailHook(["modules"]),
			/** @type {SyncHook<Module[]>} */
			afterOptimizeModules: new SyncHook(["modules"]),

			/** @type {SyncBailHook<Chunk[], ChunkGroup[]>} */
			optimizeChunksBasic: new SyncBailHook(["chunks", "chunkGroups"]),
			/** @type {SyncBailHook<Chunk[], ChunkGroup[]>} */
			optimizeChunks: new SyncBailHook(["chunks", "chunkGroups"]),
			/** @type {SyncBailHook<Chunk[], ChunkGroup[]>} */
			optimizeChunksAdvanced: new SyncBailHook(["chunks", "chunkGroups"]),
			/** @type {SyncHook<Chunk[], ChunkGroup[]>} */
			afterOptimizeChunks: new SyncHook(["chunks", "chunkGroups"]),

			/** @type {AsyncSeriesHook<Chunk[], Module[]>} */
			optimizeTree: new AsyncSeriesHook(["chunks", "modules"]),
			/** @type {SyncHook<Chunk[], Module[]>} */
			afterOptimizeTree: new SyncHook(["chunks", "modules"]),

			/** @type {SyncBailHook<Chunk[], Module[]>} */
			optimizeChunkModulesBasic: new SyncBailHook(["chunks", "modules"]),
			/** @type {SyncBailHook<Chunk[], Module[]>} */
			optimizeChunkModules: new SyncBailHook(["chunks", "modules"]),
			/** @type {SyncBailHook<Chunk[], Module[]>} */
			optimizeChunkModulesAdvanced: new SyncBailHook(["chunks", "modules"]),
			/** @type {SyncHook<Chunk[], Module[]>} */
			afterOptimizeChunkModules: new SyncHook(["chunks", "modules"]),
			/** @type {SyncBailHook} */
			shouldRecord: new SyncBailHook([]),

			/** @type {SyncHook<Module[], any>} */
			reviveModules: new SyncHook(["modules", "records"]),
			/** @type {SyncHook<Module[]>} */
			optimizeModuleOrder: new SyncHook(["modules"]),
			/** @type {SyncHook<Module[]>} */
			advancedOptimizeModuleOrder: new SyncHook(["modules"]),
			/** @type {SyncHook<Module[]>} */
			beforeModuleIds: new SyncHook(["modules"]),
			/** @type {SyncHook<Module[]>} */
			moduleIds: new SyncHook(["modules"]),
			/** @type {SyncHook<Module[]>} */
			optimizeModuleIds: new SyncHook(["modules"]),
			/** @type {SyncHook<Module[]>} */
			afterOptimizeModuleIds: new SyncHook(["modules"]),

			/** @type {SyncHook<Chunk[], any>} */
			reviveChunks: new SyncHook(["chunks", "records"]),
			/** @type {SyncHook<Chunk[]>} */
			optimizeChunkOrder: new SyncHook(["chunks"]),
			/** @type {SyncHook<Chunk[]>} */
			beforeChunkIds: new SyncHook(["chunks"]),
			/** @type {SyncHook<Chunk[]>} */
			optimizeChunkIds: new SyncHook(["chunks"]),
			/** @type {SyncHook<Chunk[]>} */
			afterOptimizeChunkIds: new SyncHook(["chunks"]),

			/** @type {SyncHook<Module[], any>} */
			recordModules: new SyncHook(["modules", "records"]),
			/** @type {SyncHook<Chunk[], any>} */
			recordChunks: new SyncHook(["chunks", "records"]),

			/** @type {SyncHook} */
			beforeHash: new SyncHook([]),
			/** @type {SyncHook<Chunk>} */
			contentHash: new SyncHook(["chunk"]),
			/** @type {SyncHook} */
			afterHash: new SyncHook([]),
			/** @type {SyncHook<any>} */
			recordHash: new SyncHook(["records"]),
			/** @type {SyncHook<Compilation, any>} */
			record: new SyncHook(["compilation", "records"]),

			/** @type {SyncHook} */
			beforeModuleAssets: new SyncHook([]),
			/** @type {SyncBailHook} */
			shouldGenerateChunkAssets: new SyncBailHook([]),
			/** @type {SyncHook} */
			beforeChunkAssets: new SyncHook([]),
			/** @type {SyncHook<Chunk[]>} */
			additionalChunkAssets: new SyncHook(["chunks"]),

			/** @type {AsyncSeriesHook} */
			additionalAssets: new AsyncSeriesHook([]),
			/** @type {AsyncSeriesHook<Chunk[]>} */
			optimizeChunkAssets: new AsyncSeriesHook(["chunks"]),
			/** @type {SyncHook<Chunk[]>} */
			afterOptimizeChunkAssets: new SyncHook(["chunks"]),
			/** @type {AsyncSeriesHook<CompilationAssets>} */
			optimizeAssets: new AsyncSeriesHook(["assets"]),
			/** @type {SyncHook<CompilationAssets>} */
			afterOptimizeAssets: new SyncHook(["assets"]),

			/** @type {SyncBailHook} */
			needAdditionalSeal: new SyncBailHook([]),
			/** @type {AsyncSeriesHook} */
			afterSeal: new AsyncSeriesHook([]),

			/** @type {SyncHook<Chunk, Hash>} */
			chunkHash: new SyncHook(["chunk", "chunkHash"]),
			/** @type {SyncHook<Module, string>} */
			moduleAsset: new SyncHook(["module", "filename"]),
			/** @type {SyncHook<Chunk, string>} */
			chunkAsset: new SyncHook(["chunk", "filename"]),

			/** @type {SyncWaterfallHook<string, TODO>} */
			assetPath: new SyncWaterfallHook(["filename", "data"]), // TODO MainTemplate

			/** @type {SyncBailHook} */
			needAdditionalPass: new SyncBailHook([]),

			/** @type {SyncHook<Compiler, string, number>} */
			childCompiler: new SyncHook([
				"childCompiler",
				"compilerName",
				"compilerIndex"
			])	
		}
		//...
	}
	//...
}
```
### 2.3 Tapable
`Compiler`和`Compilation`均继承了`Tapable`，`Tapable`中存在多种用于在插件使用的钩子
```
const {
	SyncHook,
	SyncBailHook,
	SyncWaterfallHook,
	SyncLoopHook,
	AsyncParallelHook,
	AsyncParallelBailHook,
	AsyncSeriesHook,
	AsyncSeriesBailHook,
	AsyncSeriesWaterfallHook
 } = require("tapable");
```
所有的钩子函数都接收一个可选参数，参数为字符串组成的数组

Tapable主要分为两种同步钩子和异步钩子

![Tapable的分类](https://user-gold-cdn.xitu.io/2018/12/3/16774ae052299224?w=1406&h=662&f=png&s=157810, "Tapable的分类")
关于不同的钩子需要知道其订阅方式不同，对于`sync`方法只能通过`tap`订阅，对应的触发方式为`call`,而对于`async`方法可以通过`tap`, `tapPromise`和`tapAsync`三种方式订阅，其对应的触发方式为`call`, `callPromise`和`callAsync`三种

[相关参考](https://juejin.im/post/5abf33f16fb9a028e46ec352)

### 2.1.2 例2
```
class FileListPlugin{
    constructor(options){
        this.options = options || {};
    }
    apply(compiler){
        compiler.hooks.emit.tapAsync('fileList', (compilation, callback) =>{
            var fileList = 'In this build: \n\n';
            for(var filename in compilation.assets){
                fileList += ( '- ' + filename + '\n')
            }
            compilation.assets['fileList.md'] = {
                source(){
                    return fileList
                },
                size(){
                  return fileList.length  
                }
            }
        })
    }
}
// 修改webpack.config.js文件
const FileListPlugin = require('./src/FileListPlugin')
module.exports = {
    //...
    output: {
        filename:'[name].bundle.[hash:5].js'
        path: './src/dist'   
    }
    plugins: [
        new FileListPlugin()
    ]
}

// 结果--在dist目录下生成了fileList.md文件
// fileList.md文件内容

In this build: 

- app.bundle.00ab2.js
- print.bundle.00ab2.js
- index.html

```

