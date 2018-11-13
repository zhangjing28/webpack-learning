### webpack(1)分享
关于webpack的学习
### 1. wepack定义
webpack是当前流行的前端资源的模块打包器，当webpack处理应用程序时，它会递归地构建一个依赖关系图（dependency graph），其中包含应用程序所需的每一个模块，然后将这些模块打包成少量的bundle，通常只有一个，由浏览器加载

![webpack依赖图](https://user-gold-cdn.xitu.io/2018/10/24/166a37e046ea1718?w=1000&h=393&f=webp&s=9810 "webpack依赖图")
### 2.webpack重要的几个概念
**2.1 entry(入口文件)**

指示webpack应该使用那个模块来作为构建其内部依赖图的开始，进入入口起点后，webpack会找出有哪些模块和库是入口起点(直接或者间接)依赖的
    
**2.2 output(出口)**

output属性会告诉webpack在哪里输出它所创建的bundles，以及如何命名这些文件,默认值为./dist

**2.3 loader**
loader让webpack能够处理那些非JavaScript文件(webpack自身只理解JavaScript)。loader可以将所有类型的文件转化为webpack能够处理的有效模块，然后可以利用webpack的打包能力，对它们进行处理，本质上，webpack将所有类型的文件，转换为应用程序的依赖图(最终的bundle)可以直接引用的模块

**2.4 plugins**

loader被用于转换某些类型的模块，而插件则用于执行范围更广的任务，插件的功能极其强大，用来处理各种各样的任务
--想要使用一个插件，只需要require进来，然后将其添加到plugins数组中，多数插件通过option自定义，可以在一个配置文件中因为不同的目的使用同一个插件多次，使用插件时，需要通过new操作符来创建一个它的实例
        
webpack简单配置
```
    module.exports = {
        entry: './path/to/my/entry/file.js',
        output: {
            path: path.resolve(__dirname, 'dist'),
            filename: 'my-first-webpack.bundle.js'
        },
        module: {
            rules: [
                {
                 test: /\.txt$/,
                 use: 'row-loader'
                }
            ]
        },
        plugins: [
            new HtmlWebpackPlugin({template: './src/index.html'})
        ]
    }
```
### 3. entry配置
字符串：入口文件的地址
```
    module.exports = {
        entry: './app.js'
    }
    // 传入一个字符串或字符串数组，chunk 会被命名为 main
    // 等价于 --- webpack默认的entry的key为main
    module.exports = {
        entry: {
            main: './app.js'
        }
    }
```
数组： 当存在多个入口，还要将这些文件打包到一个文件时，采用这种写法
```
    module.exports = {
        entry: ['./app.js', './vendor.js']
    }
```

![官网对于entry为数组的解释](https://user-gold-cdn.xitu.io/2018/10/29/166be87357cf729d?w=1410&h=220&f=png&s=81916 "官网对于entry为数组的解释")
对象：推荐写法，也是webpack的可扩展写法
```
    module.exports = {
        entry: {
            'app': './app.js',
            'vendor': './vendor.js'
        }
    }
```
![官网对于entry为对象的解释](https://user-gold-cdn.xitu.io/2018/10/29/166be8ed5219b16a?w=1426&h=218&f=png&s=77397 "官网对于entry为对象的解释")
### 4. output

```
    module.exports = {
        entry: './app.js',
        output: {
            filename: 'index.bundle.js'
        }
    }
    module.exports = {
        entry: {
            index: './app.js',
            vendor: './vendor.js'
        },
        output: {
            // [name]: 使用入口名称，
            // [hash:8]: 每次构建生成的hash值，默认为20位，这里取8位
            // [id]: 每次创建生成的chunk的id
            // 同样还可以使用文件夹结构
            filename: 'js/[name]. [hash:8].js',
            path: path.resolve(__dirname, 'dist')
        }
    }
```
>对于几种路径的解释
>>__dirname: 获取当前执行文件所在目录的完整目录
>
>>__filename:获取当前执行文件的带有绝对路径的文件名
>
>>process.cwd(): 获取当前执行node命令的文件所在目录的完整目录

>webpack4默认的entry为src/index.js文件,默认的output为dist/main.js,所以如果在webpack4下直接输入webpack进行打包，会自动将src/index.js作为entry文件，并将其打包结果输出到dist/main.js;而在webpack4之前的版本则打包不成功，会报错

举例： 
![简单举例](https://user-gold-cdn.xitu.io/2018/10/29/166be94ae1a70d22?w=1284&h=712&f=png&s=191754)
引入方式对打包的影响
![简单对比](https://user-gold-cdn.xitu.io/2018/10/29/166be963d78d17c7?w=1244&h=740&f=png&s=203303)
### 5. loader
loader主要在module中的rules里配置
仅以babel-loader为例
babel中存在转译es6新的语法的预置配置babel-preset-env、babel-preset-es2015、babel-preset-es2016、babel-preset-es2017、babel-preset-latest等；转译最新的js的api和全局对象的插件如babel-polyfill,bable-plugin-transform-runtime
#### 5.1 几种常见的预置配置说明
```
    babel-preset-env：
        最常用的babel的预置配置，可以根据浏览器等信息配置语法的转译范围
        
    babel-preset-es2015：
        将es2015版本的js代码转译为es5代码，对于es2016版本的代码或者es2017版本的代码不转译
        
    stage-3: 候选(cancidate, complete spec and initial browser implementations)
        包含插件：
            transform-async-to-generator 支持async/await
            transform-exponentiation-operator 支持幂运算符语法糖
            
    stage-2：初稿(draft, initial sepc)
        stage-2除了包含stage-3，还包含了下面2个插件：
            syntax-trailing-function-commas 支持尾逗号函数
            transform-object-reset-spread 支持对象的解构赋值
            
    stage-1：提案(proposal, this is worth working on)
        stage-1除了包含stage-2,还包含了以下插件：
            transform-class-constructor-call 支持class的构造函数
            transform-class-properties 支持class的static属性
            transform-decorators 支持es7的装饰者模式即@符号引入的方法(还在讨论中的特性？)
            transform-export-extensions 支持export方法
            
    stage-0: 草案(strawman, just an idea)
        包括stage1所有插件,还包括以下插件：
            transform-do-expressions 支持在jsx中书写if/else
            transform-function-bind 支持::操作符来切换上下文，类似于es5的bind
```
[stage部分相关参考](https://www.aliyun.com/jiaocheng/989794.html)    
```
    // 常见的babel-loader的配置
    // 1. webpack.config.js文件中配置相关rules
    module.export = {
        ...
        module: {
            rules: [
                {
                    test: /\.m?js$/,
                    loader: 'babel-loader'
                }
            ]
        }
    }
    // 2. .babelrc文件中
    {
        // presets为数组时，遵循倒序，即先用stage-2中转译，然后再用env中的配置进行转译
        "presets": [
            //数组第一项表示预置的那个内容，这里选择env
            // 第二项表示对于env预置内容的相关配置
            [
                "env",
                {
                    "modules": false
                }
            ],
            "stage-2"
        ],
        "plugins": [
            "transform-runtime"
        ]
    }
```
#### 5.2 关于presets中的相关配置
>这里相关配置的具体含义并没有用代码做过测试，暂时参照官网内容

[presets中相关配置官网参考](https://www.babeljs.cn/docs/plugins/preset-env#targets)


