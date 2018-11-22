# 1. loader
## 1.1 loader的定义
    通过使用不同的loader，webpack可以将不同的文件都转成js文件
## 1.2 loader的相关属性
**module.noParse**

* 对于与给定的正则表达式相匹配的文件不使用webpack进行解析，因此忽略的文件中不应该包含import，require等的调用，
* 忽略大型的插件可以提高构建性能
    ```
    module: {
        noParse: /jquery|lodash/,
        noParse: function(content) {
            console.log(content);
            return /jquery|lodash/.test(content)
        }
    }
    ```
![content打印结果](https://user-gold-cdn.xitu.io/2018/11/21/16736c5923fa55a6?w=2664&h=648&f=png&s=317911, "content打印结果")
**module.rules**:  array
>创建模块时，匹配请求的规则(`Rule`)数组，这些规则能够修改模块的创建方式。这些规则能够对模块(`module`)应用`loader`

> 解释：对于`webpack`而言，所有类型的文件都会被解析打包成`js`模块,这里的`rules`就是用来对等不同的文件(如`css`文件,`js`文件,`png`文件)配置不同的解析打包成模块的方式或者修改文件对应的解析器的

**Rule**: 每个规则分成三个部分-条件(condition), 结果(result)和嵌套规则(nested rule)
* Rule Condition

    条件有两种输入值：
    * resource：请求文件的绝对路径
    * issuer： 请求文件所在模块的文件的绝对路径
    
    例如：在`app.js`文件中`import ./style.css`，那么此时的请求文件就是`style.css`,resource就是`style.css`文件所在的绝对路径，issuer就是`app.js`文件所在的绝对路径
    
    对于上述两个输入下的文件，可以通过配置规则来进行条件限定。如果将条件配置在`recource`属性下，就表示该条件用来约束请求文件(上例中的`style.css`)；如果将条件配置在`issuer`属性下，就表示该条件用来约束请求文件所在的文件(上例中的`app.js`)
    
    ```
    // 简单的resource配置
    module.exports = {
        //...
        module:{
            rules:[{
                // 这里的resource下的test属性就是匹配对于所有的请求文件中以.css结尾的文件
                resource:{
                    test: /\.css$/
                },
            }]
        }
    }
    ```
    **条件**
    
    常用的条件(condition)：
    
        字符串： 匹配输入必须以提供的字符串开始，字符串为目录的绝对路径或者文件的绝对路径
        正则表达式： 就是一个正则表达式,如`/\.css$/`
        函数：调用输入的函数，必须返回一个真值(truthy value)以匹配
        条件数组：至少一个匹配条件
    ```
    { test: condition}: 匹配特定条件，一般是提供一个正则表达式或者正则表达式数组
    { include: condition}: 匹配特定条件，一般是提供一个字符串或者字符串数组
    { exculde: condition}: 排除特定条件，一般是提供一个字符串或者字符串数组
    ```
    
**Rule.test**: `Rule.resource.test`的简写，如果提供了`Rule.test`选项，就不能再提供`Rule.resource`选项

**Rule.include**: `Rule.resource.include`的简写，同样提供了`Rule.include`选项，就不能再提供`Rule.resource`选项

**Rule.exculde**: `Rule.resource.exclude`的简写，同样提供了`Rule.exclude`选项，就不能再提供`Rule.resource`选项

有上述可知，如果要配置关于`issuer`的条件同样可以用`test`, `include`和`exclude`等属性

举个🌰
```
// 只有在.js文件中引入的.css文件才使用style-loader, css-loader
module.exports = {
    //...
    module:{
        rules:[{
            resource: {
                test: /\.css$/
            },
            issuer: {
                test: /\.js$/
            },
            use: ['style-loader', 'css-loader']
        }]
    }
}
```

* Rule Result

在上述例子中，不仅包括了Rule Condition(`resource`和`issuer`的配置),还有Rule Result的内容--> `use:['style-loader', 'css-loader']`

    规则结果只在规则条件匹配时使用，有两种输入值：
        应用的loader： 应用在resource上的loader数组
        parser选项，用于为模块创建解析器的选项对象
**Rule.use**: 数组，数组内可以是字符串或者对象，表示对于匹配的规则，采用的处理结果

`use:['style-loader', 'css-loader']`是
```
    use:[{
        loader: 'style-loader'
        
    },{
        loader: 'css-loader'
    }]
```
的简写形式

**Rule.loaders**: 是`Rule.use`的别名，已经被废弃

**Rule.loader**: 是`Rule.use:[{loader}]`的简写
```
    module.exports = {
        //...
        module:{
            rules:[{
                resource: {
                    test: /\.css$/
                },
                issuer: {
                    test: /\.js$/
                },
                loader: ['style-loader', 'css-loader']
            }]
        }
    }
```


* Rule Nested Rule

>可以使用属性`rules`和`oneOf`指定嵌套规则,这些规则用于在条件匹配时进行取值

**Rule.rules**
```
    module.exports = {
        //...
        module:{
            rules:[{
                resource: {
                    test: /\.css$/
                },
                issuer: {
                    test: /\.js$/
                },
                rules: [
                    {
                        issuer: {
                            test: /\.jsx$/
                        },
                        use: 'style-loader',
                    }, {
                        use: 'css-loader',
                    },
                ],
            }]
        }
    }
```
采用rules的优势就是可以在每个`rules`中的每个对象上再进行条件筛选(如例子中的`issuer:{test: /\.jsx$/}`),进而确定使用的`loader`

**Rule.oneOf**

webpack.config.js
```
    module.exports = {
        //...
        module:{
            rules:[{
                resource: {
                    test: /\.css$/
                },
                issuer: {
                    test: /\.js$/
                },
                oneOf: [
                    {
                        resourceQuery: /aain/, 
                        use: ['style-loader', 'css-loader']
                    },{
                        resourceQuery: /inline1/,
                        use: ['style-loader', 'css-loader']
                    }
                ]
            }]
        }
    }
```
index.js
```
// 这里的?inline用来匹配配置文件中的resourceQuery部分
import './css/common.css?inline'
```
由上述例子可以知道Rule.resourceQuery的用法是用来匹配引入的文件`?`后的内容的,即上例子中的`?inline`
 

## 1.3 loader的链式调用
* loader的职责是单一的，只能完成一种转化
* 当配置多个loader时，loader的执行顺序为从右到左，右边的执行结果作为参数传到左边
* `less-loader`把`less`文件转化为`css`，传给`css-loader`,`css-loader`处理后的结果传给`style-loader`，`style-loader`返回javascript模块
## 1.4 less-loader
### 1.4.1 less-loader的官网介绍
**安装**
`npm install --save-dev less-loader less`

**相关options**

 * [less options官网](http://lesscss.org/usage/#less-options)

**webpack resolver 和 less 内置的resolver**

webpack提供了一种解析文件的高级机制，`less-loader`应用了一个less插件，并将所有的查询参数传递给webpack resolver，所以可以从`node_modules`导入less模块，只要加一个`~`前缀，告诉webpack去查询模块

`@import "~bootstrap/less/bootstrap";`
```
module.exports = {
  //...
  resolve: {
    // modules的默认值
    modules: ['node_modules']
  }
};
```
* resolve.modules

    取值是数组
    告诉webpack从那开始查找模块
如果想要查找模块的地址，优先于默认的`node_modules`，可以采用下面的方式
    ```
        module.exports = {
            //...
            resolver:{
                modules: [path.resolve(__dirname, 'src'), 'node_modules']
            }
        }
    ```
**less-loader中的resolver属性**

在`less-loader`中如果配置了paths属性，那么模块的查找路径将从paths中获取，而不是从webpack的`resolver.modules`属性中获取，`paths`属性必须是一个只包含绝对路径的数组
```
module.exports = {
    //...
    module:{
        rules:[{
            test: /\.less$/,
            use:[{
                loader: 'style-loader'
            },{
                loader: 'css-loader'
            },{
                loader: 'less-loader',
                options: {
                    paths:[
                        path.resolve(__dirname, 'node_modules')
                    ]
                }
            }]
        }]
    }
}
```
**提取样式文件**
>在开发环境利用webpack打包css有很多优点比如，但是在生产环境将css样式通过js加载会出现展示出来的网页没有样式的情况，因此在生产环境下提取css文件是一个比较好的方式
```
// 提取样式文件的简单的配置
var ExtractTextWebpackPlugin = require('extract-text-webpack-plugin')
var path = require('path')
module.exports = {
    entry: {
        app: './src/index.js'
    },
    output: {
        filename: [name].[hash:5].js,
        path: path.resolve(__dirname, 'dist')
    },
    module: {
        rules:[{
            test: /\.less$/,
            use: ExtractTextWebpackPlugin.extract({
                fallback: 'style-loader',
                use: ['css-loader', 'less-loader']
            })
        }]
    },
    plugins:[
        new ExtractTextWebpackPlugin({
            filename: '[name].css'
        })
    ]
}
```
**css sourceMap**

要启用`css`的`sourceMap`，你需要将`sourceMap`选项传递给`less-loader`和`css-loader`，因此`webpack.config.js`文件应该是这样的：
```
module.exports = {
    // ...
    module: {
        rules: [{
            test: /\.less$/,
            use: [{
                loader: "style-loader"
            }, {
                loader: "css-loader", options: {
                    sourceMap: true
                }
            }, {
                loader: "less-loader", options: {
                    sourceMap: true
                }
            }]
        }]
    }
};
```
>当webpack打包源代码时，可能很难源代码中错误和警告的位置；为了更容易的跟踪错误和警告， webpack提供了source map功能，将编译后的代码映射为原始源代码，source map在devtool属性上进行配置，并且有很多不同的选项，选项的不同，对构建和重新构建的速度的影响有所不同，而且有些选项区在生产环境上不可用

![devtool选项](https://user-gold-cdn.xitu.io/2018/11/21/16735c98fe5520d8?w=961&h=1252&f=png&s=281536, "devtool选项")
