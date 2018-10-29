# webpack-learning
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


