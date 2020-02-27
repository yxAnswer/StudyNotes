[toc]

# 一、webpack快速体验

## 1.webpack是什么

简介：webpack其实就是一个JavaScript应用程序的静态模块打包器。

官方文档：[https://www.webpackjs.com/](https://www.webpackjs.com/)  官方文档啥都有，详细内容看文档，快速入门看笔记。具体的概念看官方文档。

四个**核心概念**：

- 入口(entry)  ：指明入口起点，webpack处理入口起点直接间接依赖的模块
- 输出(output) ：指明输出文件路径，以及如何命名。即打完包去哪里找。
- loader  ： *loader* 让 webpack 能够去处理那些非 JavaScript 文件（webpack 自身只理解 JavaScript）。 即将所有类型文件转换依赖图可以直接引用的模块。
- 插件(plugins) ：就是`require()`各种插件呗，处理各种各样的任务。

## 2.webpack有什么作用

模块化打包：
		webpack会将项目的资源文件当成一个一个模块，模块之间会有依赖关系，webpack将会对这些有依赖关系的文件进行处理，让浏览器能够识别，最后将应用程序需要的每个模块打包成一个或者多个bundle。

## 3、Webpack环境配置和简单打包 

###  3.1、安装node

node官网地址: https://nodejs.org/zh-cn/

### 3.2、创建package.json文件

在项目目录下启动命令行，创建package.json文件，**初始化项目**

命令：`npm init -y`    -y是使用默认配置，如果不加-y ,可以手动输入配置信息。 默认生成的文件内容如下：
```js
{
  "name": "webpackdemo", //项目名
  "version": "1.0.0",  //版本
  "description": "",
  "main": "index.js",
  "scripts": {  //脚本命令
    "test": "echo \"Error: no test specified\" && exit 1"
  },
  "keywords": [], //关键词
  "author": "",  //作者
  "license": "ISC"
}
```

### 3.3、 安装webpack

3.1 本地安装：（推荐）

```shell
npm install --save-dev webpack
npm install --save-dev webpack-cli
```

3.2全局安装：

 ```shell
npm install --global webpack webpack-cli
 ```

如果有下载比较慢的情况，可以使用淘宝的镜像：

```shell 
 npm install -g cnpm --registry=https://registry.npm.taobao.org
```

安装完成后，项目根目录下会有以下文件夹

——node_modules  //文件夹,webpack生成

——package.json 文件,npm init生成

——package-lock.json //文件,webpack生成

`--save -dev`会将依赖包下载安装以后，自动修改项目的package.json文件夹下，自动添加到`devDependencies` 声明下：比如

```json
  "devDependencies": {
    "css-loader": "^3.2.1",
    "style-loader": "^1.0.1",
    "webpack": "^4.41.2",
    "webpack-cli": "^3.3.10",
    "webpack-dev-server": "^3.9.0"
  }
```



### 3.4、打包

默认entry入口 `src/index.js`
默认output出口 `dist/main.js `

在项目文件夹下创建入口文件对应的src文件夹及文件。比如：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20191206113440775.png)

打包模式：

webpack 有个mode概念，可以设置打包模式，可以在配置文件添加mode属性写明，也可以通过脚本用命令行的方式，如下更方便：  两个参数 `development` 和`production`

```shell
webpack --mode development   //开发阶段
webpack --mode production   //生产阶段
```

编辑package.json 文件，添加打包模式命令。(为了方便不用手动去命令行敲webpack命令了)

```json
{
  "name": "webpackdemo",
  "version": "1.0.0",
  "description": "",
  "main": "index.js",
  "scripts": {  //添加这两个命令,在命令行执行npm run dev  或者npm run build会执行对应命令
    "dev": "webpack --mode development",
    "build": "webpack --mode production"
  },
  "keywords": [],
  "author": "",
  "license": "ISC",
  "devDependencies": {
    "webpack": "^4.41.2",
    "webpack-cli": "^3.3.10"
  }
}

```

设置好后在命令行程序中运行npm run dev或者npm run build来进行打包。打包完成后会自动生成dist文件夹，这个是webpack的默认出口。

`npm run dev`    调试模式，不压缩代码

`npm run build`  生产模式下，会压缩文件，代码去除空格压缩成一行

# 二、webpack 配置

## 1、webpack 配置文件的初使用

### 1.1、配置webpack.config.js

（1）项目根目录--新建一个webpack.config.js

（2）修改配置文件 webpack.config.js

**配置入口entry(所需打包的文件路径)**  ,比如我们建的文件夹，src/index.js 

**配置出口output**  ,打包后输出的文件夹，及文件名称，如下。

```js
const path = require('path');

module.exports = {
  entry: './src/index.js', 
  output: {
    filename: 'bundle.js',
    path: path.resolve(__dirname, 'build')
  }
};
```

- path指文件打包后的存放路径
- path.resolve()方法将路径或路径片段的序列处理成绝对路径
- __dirname 表示当前文件所在的目录的绝对路径
- filename是打包后文件的名称

### 1.2、命令行程序运行

`npm run dev或者npm run build`

运行打包命令后，会按照我们配置文件中写的，输出到build文件夹下的 bundle.js 。



## 2、入口entry和出口output进阶用法

###  2.1入口entry

（1）单入口

**单文件：**

例如：` entry: './src/index.js'`

**多文件：**

在你想要多个依赖文件一起注入，并且将它们的依赖导向到一个“chunk”时，传入数组的方式就很有用。

例如：`entry:['./src/index.js',./src/index2.js]`

（2）多入口

例如：

```js
entry:{
    pageOne:'./src/pageOne/index.js',
    pageTwo:'./src/pageTwo/index.js'
  }
```

**注意：多入口要和多出口同步设置，设置了多入口必须设置多出口**

### 2.2出口output

（1）单出口

```js
 output: {
    filename: 'bundle.js',
    path: path.resolve(__dirname, 'build')
  }
```

（2）多出口

[name] 原文件的名字. 只需要把 filename  改为 `[name].js`

```js
  output: {
    filename: '[name].js',
    path: path.resolve(__dirname, 'build')
  }
```

### 2.3 配置文件示例

**单入口(多文件) +单出口 示例：将两个入口文件index.js和index2.js打包到同一个文件bundle.js中**

```js
const path = require('path');

module.exports = {
  entry: ['./src/index.js','./src/index2.js'], 
  output: {
    filename: 'bundle.js',
    path: path.resolve(__dirname, 'build')
  }
};
```



**多入口+多出口： 打包后会生成pageOne.js和pageTwo.js两个文件**

```js
const path = require('path');
module.exports = {
  entry:{
    pageOne:'./src/pageOne/index.js',
    pageTwo:'./src/pageTwo/index.js'
  },
  output: {
    filename: '[name].js',
    path: path.resolve(__dirname, 'build')
  }
};
```

## 3、配置webpack-dev-server

### 3.1 webpack-dev-server介绍

webpack-dev-server是webpack官方提供的一个小型Express服务器。使用它可以为webpack打包生成的资源文件提供web服务。

**webpack-dev-server 主要提供两个功能：**

（1）为静态文件提供服务

（2）自动刷新和热替换(HMR)

### 3.2 安装webpack-dev-server

在项目根目录命令行运行安装命令：

 ` npm install --save-dev webpack-dev-server`

### 3.3 配置webpack.config.js文件

添加 devServer部分如下:

```js
const path = require('path');

module.exports = {
  entry: './src/index.js', 
  output: {
    filename: 'bundle.js',
    path: path.resolve(__dirname, 'build')
  },
  devServer:{
      contentBase:'./build', //设置服务器访问的基本目录-也就是我们打包输出路径
      host:'localhost', //服务器的ip地址
      port:'8080', //设置访问端口
      open:true //是否自动打开页面
  }
};
```

### 3.4 配置package.json启动命令

```json
"scripts": {
  "start": "webpack-dev-server --mode development"
 }
```

这样就可以在命令行`npm run start` 来启动服务

### 3.5  测试一下

在build文件夹下新建index.html文件

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <meta http-equiv="X-UA-Compatible" content="ie=edge">
    <title>Document</title>
</head>
<body>
    测试一下
</body>
</html>
```

命令行运行 `npm run start` ,就会自动打开浏览器，显示此网页。然后我们改一下文字，然后刷新网页，网页就会跟着刷新为最新内容。

#  三、webpack   Loader

## 1、使用loader加载css 

### 1.1 loader是什么？

   loader 让 webpack 能够去处理那些非 JavaScript 文件（webpack 自身只理解 JavaScript）。loader 可以将所有类型的文件转换为 webpack 能够处理的有效模块，然后你就可以利用 webpack 的打包能力，对它们进行处理。（**所以使用loader可以将css打包进 js文件**）

### 1.2 安装style-loader和css-loader

要将css文件打包，需要使用`style-loader`和`css-loader`   ,配置是注意顺序

下载安装所需的loader：

```shell
npm install style-loader css-loader --save-dev
```

### 1.3 配置loader

在webpack.config.js文件里配置module中的rules

在 webpack 的配置中 loader 有两个目标：

- **test 属性**，用于标识出应该被对应的 loader 进行转换的某个或某些文件。

- **use 属性**，表示进行转换时，应该使用哪个 loader。

代码：看module属性配置。

```js
const path = require('path');

module.exports = {
  entry: './src/index.js', 
  output: {
    filename: 'bundle.js',
    path: path.resolve(__dirname, 'build')
  },
  devServer:{
      contentBase:'./build', //设置服务器访问的基本目录
      host:'localhost', //服务器的ip地址
      port:'8080', //设置访问端口
      open:true //是否自动打开页面
  },
  module:{
    rules:[
      {
        test:/\.css$/,
        use:['style-loader','css-loader']
      }
    ]
  }
};
```

注意： **在 webpack 配置中定义 loader 时，要定义在 `module.rules` 中，而不是 `rules`**

 上面的意思是

>  “嘿，webpack 编译器，当你碰到「在 `require()`/`import` 语句中被解析为 '.txt' 的路径」时，在你对它打包之前，先**使用** `style-loader`  和 `css-loader`转换一下。” 

这样，我们可以将css文件打包进.js文件中了。比如在 index.js中引入 index.css 文件，然后打包到bundle.js文件。通过代码看到已经引入了。这个时候如果有html文件使用index.js文件，那么index.css所设置的样式就会起作用。

##  2、webpack编译 less 和sass

 当使用less或者sass的时候，我们可以像使用css一样，通过loader 提供支持。

### 2.1 使用less

**安装less-loader   和 less**

命令行安装：

```shell
npm install less-loader less --save-dev
```

配置文件修改：

```js
 module:{
    rules:[
      {
        test:/\.less$/,   //注意这里
        use:['style-loader','css-loader','less-loader']
      }
    ]
  }
```

### 2.2  使用Sass

**安装sass-loader 和 node-sass**

```shell
npm install sass-loader node-sass --save-dev
```

配置文件修改：

```js
 module:{
    rules:[
      {
        test:/\.scss$/,   //注意这里
        use:['style-loader','css-loader','sass-loader']
      }
    ]
  }
```

## 3、使用PostCSS处理浏览器前缀

 使用postcss-loader和autoprefixer同一处理 浏览器前缀

### 3.1 安装loader

安装postcss-loader和autoprefixer

```shell
npm install --save-dev postcss-loader autoprefixer
```

### 3.2 配置loader

需要和autoprefixer一起用

```shell
  module: {
    rules: [
      {
        test: /\.css$/,
        use: [
          'style-loader',
          'css-loader',
          {
            loader: 'postcss-loader',
            options: {
              plugins: [
                require("autoprefixer")(
                  {
                    browsers: [
                      'ie >= 8',//ie版本大于等于ie8
                      'Firefox >= 20',//火狐浏览器大于20版本
                      'Safari >= 5',//safari大于5版本
                      'Android >= 4',//版本大于Android4
                      'Ios >= 6',//版本大于ios6
                      'last 4 version'//浏览器最新的四个版本
                    ]

                  }
                )
              ]
            }
          }
        ]
      }
    ]
  }
```

这里重点是配置`postcss-loader` 以及插件`autoprefixer`  ,直接复制就可以了。

在loader中设置浏览器：

```js
                  {
                    browsers: [
                      'ie >= 8',//ie版本大于等于ie8
                      'Firefox >= 20',//火狐浏览器大于20版本
                      'Safari >= 5',//safari大于5版本
                      'Android >= 4',//版本大于Android4
                      'Ios >= 6',//版本大于ios6
                      'last 4 version'//浏览器最新的四个版本
                    ]

                  }
```

还有一种方式，不需要再 webpack.config.js中 生命浏览器设置，而是将浏览器设置生命到package.json文件中。

修改package.json  添加如下配置

```json
  "browserslist": [
    "ie >= 8",
    "Firefox >= 20",
    "Safari >= 5",
    "Android >= 4",
    "Ios >= 6",
    "last 4 version"
  ]
```

然后，webpack.config.js就没有必要再声明了，可以为如下：

```js
 module: {
    rules: [
      {
        test: /\.css$/,
        use: [
          'style-loader',
          'css-loader',
          {
            loader:'postcss-loader',
            options:{
              plugins:[
                require("autoprefixer")
              ]
            }
          }
        ]
      }
    ]
  }
```

这两种方式都可以添加css3浏览器前缀。

## 4、文件处理

### 4.1图片处理

（1）安装loader

     下载安装file-loader
    
      ```shell
 npm install --save-dev file-loader
      ```

（2）配置config文件

```js
  module: {
    rules: [
      //file-loader
      {
        test:/\.(png|jpg|gif|jpeg)$/,
        use:{
          loader:'file-loader'
        }
      }
    ]
  }
```

（3）选项配置

```js
  module: {
    rules: [
      //file-loader
      {
        test:/\.(png|jpg|gif|jpeg)$/,
        use:{
          loader:'file-loader',
          options:{
           //这里可以设置属性 比如：
              // name:'xxx.jpg'
              //outputPath:'./img'
          }
        }
      }
    ]
  }
```

**配置options：**

**name**：为你的文件配置自定义文件名模板（默认值[hash].[ext]）

**context**：配置自定义文件的上下文，默认为 webpack.config.js

**publicPath**：为你的文件配置自定义 public 发布目录

**outputPath**：为你的文件配置自定义 output 输出目录

          [ext]：资源扩展名
    
          [name]：资源的基本名称
    
          [path]：资源相对于 context的路径
    
          [hash]：内容的哈希值

 ```js
//这里单独注释下这些属性
use:{
   loader:'file-loader',
   options:{
      name:'test.jpg'  //直接在输出文件夹将文件重命名为test.jpg,注意这里只是一张图片测试而已
       
      name:'[path]test.jpg'//源文件相对于webpack.config.js相对路径为src/源文件.jpg，使用path打包后，会在 输出文件夹下，生成 src/test.jpg。  也就是会在输出文件夹里也按照相对路径生成
       
      name:'[hash]_test.jpg' //[hash] 可以自定义文件的重命名规则，这样多张图片也会按照我们的规则，自动重命名
      context:'../' //这个例子就是将context上移一层到项目根目录。默认是webpack.config.js所在文件夹 这一层级
      publicPath:'http://www.xxxxxx.com/public'  //自定义发布目录，随便指定，最终打包的图片url会替换。 比如，将文件放到cdn
      outputPath:'./img' //自定义图片的输出路径。。跟[path]相比，会先生成比如同时设置name：[path]和outputPath，会先生存img,在img下生成path的相对路径和重命名后的图片
      
   }   
}

 ```



### 4.2 字体文件处理

（1）下载字体文件

          以bootstrap字体文件为例子
    
          Boostrap字体文件下载地址：https://v3.bootcss.com/getting-started/

（2）在index.js中引入bootstrap.css，在html中使用bootstrap字体图标

（3）配置config文件

```js
  module: {
    rules: [
      //css  loser
      {  
        test: /\.css$/,
        use: [ 'style-loader', 'css-loader',
          {
            loader:'postcss-loader',
            options:{
              plugins:[ require("autoprefixer") ]
            }
          }
        ]
      },
      //file-loader
      {
        test:/\.(png|jpg|gif|jpeg)$/,
        use:{
          loader:'file-loader',
          options:{
            outputPath:'./img'
          }
        }
      },
      //配置字体
      {
        test:/\.(eot|svg|ttf|woff|woff2)/,
        use:{
          loader:'file-loader',
          options:{
            outputPath:'./fonts'
          }
        }
      }
    ]
  } 
```

### 4.3 第三方 js库处理

    以jquery库为例子

#### （1）本地导入

       编写配置文件：
    
       `webpack.ProvidePlugin`参数是键值对形式，键就是我们项目中使用的变量名，值就是键所指向的库。`webpack.ProvidePlugin`会先从npm安装的包中查找是否有符合的库。

如果`webpack`配置了`resolve.alias`选项（理解成“别名”），那么`webpack.ProvidePlugin`就会顺着设置的路径一直找下去

     使用`webpack.ProvidePlugin`前需要引入`webpack`

```js
const webpack = require('webpack');
```

然后在`module.exports = {}`中配置`resolve` 和`plugins` :

```js
  //本地导入方式配置 第三方js库jquery
  resolve: {
    alias: {
      jQuery: path.resolve(__dirname, 'src/js/jquery.min.js') //找到本地库路径
    }
  },
  plugins: [
    new webpack.ProvidePlugin({
      jQuery: "jQuery"  //键为我们使用的变量别名，值就是我们使用的库
    })

  ]
```

使用的时候，直接用jQuery这个变量名代替我们原来的$ ，例如：

```js
//$ 改为我们设置的jQuery 别名
jQuery('#header').addClass('one');  
```

#### （2）npm安装模块

     1、安装jquery库：
    
        ```shell
npm install jquery --save-dev
        ```

     2、直接在js里import引入

   ```js
 import $ from‘jquery’
   ```

 	3、啥也不说了，直接用吧

```js
import $ from 'jquery'

$('#header').addClass('one');  //同样可以使用
```

注意：这两种方式都可以，一般npm安装比较省事，然后的，在安装使用过程中可能会出现这个找不到，那个找不到的情况，这个时候，不要慌，`npm install`先试试，不行再百度google。

##  5、使用babel 编译es6

### 5.1 了解babel

目前，ES6（ES2015）这样的语法已经得到很大规模的应用，它具有更加简洁、功能更加强大的特点，实际项目中很可能会使用采用了ES6语法的模块，但浏览器对于ES6语法的支持并不完善。为了实现兼容，就需要使用转换工具对ES6语法转换为ES5语法，babel就是最常用的一个工具

官网：[https://babel.docschina.org/docs/en/index.html](https://babel.docschina.org/docs/en/index.html)

 例如：es6语法

```js
let a=10;
console.log(a);
```

转换为：el5语法

```js
var a=10;
console.log(a);
```

`babel`转化语法所需依赖项：

`babel-loader`： 负责 es6 语法转化

`babel-core`： babel核心包

`babel-preset-env`：告诉babel使用哪种转码规则进行文件处理

### 5.2 安装依赖

```shell
npm install babel-loader @babel/core @babel/preset-env --save-dev
```

 babel-loader官方文档：[https://www.webpackjs.com/loaders/babel-loader/](https://www.webpackjs.com/loaders/babel-loader/)

### 5.3 配置config文件

 ```js
 module: {
    rules: [
      //babel配置
      {
        test:/\.js$/,
        exclude:/node_modules/,
        use:'babel-loader'
      }
    ]
  }
 ```

**exclude表示不在指定目录查找相关文件**，排除`node_modules`文件夹

 因为要确保转译尽可能少的文件 ，不然babel-loader就会很慢。

### 5.4 使用 .babelrc 文件配置转换规则

**在根目录创建文件`.babelrc` 文件，输入下面代码**

 ```js
{
    "presets": ["@babel/preset-env"]
}
 ```

### 5.5 使用options方式配置转换规则

```js
 module: {
    rules: [
      //babel配置
     {
        test:/\.js$/,
        exclude:/node_modules/,
        use:{
          loader:'babel-loader',
          options:{
            presets:['@babel/preset-env']
          }
        
        }
      }
    ]
  } 
```



# 四、webpack插件

## 1、html-webpack-plugin 生成HTML

1.1 了解html-webpack-plugin

HtmlWebpackPlugin会自动为你生成一个HTML文件，根据指定的index.html模板生成对应的 html 文件。

根据src下的index.html自动在打包后的目录下生成html文件，相关引用关系和文件路径都会按照正确的配置被添加进生成的html里

文档：[https://www.webpackjs.com/plugins/html-webpack-plugin/](https://www.webpackjs.com/plugins/html-webpack-plugin/)

1.2 安装依赖

```shell
npm install html-webpack-plugin --save-dev
```

1.3 配置config文件

 ```js
const HtmlWebpackPlugin = require('html-webpack-plugin');
 ```

//配置插件

```js
 plugins:[
    new HtmlWebpackPlugin({
      template:'./src/index.html',
      filename:'webpack.html',
      minify:{
        minimize:true,  //是否打包为最小值
        collapseWhitespace: true, //去除空格
        removeComments: true, //去除注释
        useShortDoctype: true,
				removeAttributeQuotes:true, //去除引号
				minifyCSS:true, //压缩html内的样式
				minifyJS:true, //压缩html内的js
				removeEmptyElements:true  //清理内容为空的元素
      },
      hash:true  //引入产出资源的时候加上哈希，避免缓存
  
    })
  ]

```

## 2、提取分离CSS

将所有的入口 chunk(entry chunks)中引用的 css，移动到独立分离的 CSS 文件。不用再像之前将css打入最后的bundle.js这个文件 。

### 2.1 mini-css-extract-plugin插件

更详细的使用看文档

官方文档：[https://webpack.js.org/plugins/mini-css-extract-plugin/#root](https://webpack.js.org/plugins/mini-css-extract-plugin/#root)

**注意：mini-css-extract-plugin 好像和style-loader 不能共存**

（1）安装（下载）

 ```shell
npm install --save-dev mini-css-extract-plugin
 ```

（2）配置config文件

引入插件

```js
const MiniCssExtractPlugin = require('mini-css-extract-plugin');
```

Rules设置：

 **webpack.config.js** 

```js
const MiniCssExtractPlugin = require('mini-css-extract-plugin');

module.exports = {
  module: {
    rules: [
      {
        test: /\.css$/,
        use: [
          {
            loader: MiniCssExtractPlugin.loader,
            options: {
              //可以设置属性，非必须
            },
          },
          'css-loader',
        ],
      },
    ],
  },
   plugins: [
    new MiniCssExtractPlugin({
      filename: './css/[name].css'
    }),
  ],
};
```

##  3、压缩css及优化css结构

地址：[https://github.com/NMFR/optimize-css-assets-webpack-plugin](https://github.com/NMFR/optimize-css-assets-webpack-plugin)

 它将在Webpack构建期间搜索CSS资源，并优化\最小化CSS（默认情况下，它使用[cssnano，](http://github.com/ben-eb/cssnano)但可以指定自定义CSS处理器）。 

###  3.1 optimize-css-assets-webpack-plugin插件

（1）安装（下载）

 ```shell
npm install --save-dev optimize-css-assets-webpack-plugin
 ```

（2）配置config文件

引入插件：

 ```js
const OptimizeCSSAssetsPlugin = require("optimize-css-assets-webpack-plugin"); 
 ```

Plugins设置

`assetNameRegExp`: 正则表达式，用于匹配需要优化或者压缩的资源名。默认值是 /\.css$/g

`cssProcessor`: 用于压缩和优化CSS 的处理器，默认是 cssnano.

`cssProcessorPluginOptions`:传递给cssProcessor的插件选项，默认为{}

`canPrint`:表示插件能够在console中打印信息，默认值是true

`discardComments`:去除注释

```js
new OptimizeCSSAssetsPlugin({
 	assetNameRegExp:/\.css$/g,
    cssProcessor: require('cssnano'),
    cssProcessorPluginOptions: {
        preset: ['default', { discardComments: { removeAll: true } }],
    },
    canPrint: true
})
```

## 4、拷贝静态文件copy-webpack-plugin

将未使用的静态资源比如图片、文档等，一同打包到最后的输出文件夹下。

文档地址：[https://webpack.js.org/plugins/copy-webpack-plugin/#root](https://webpack.js.org/plugins/copy-webpack-plugin/#root)

4.1 安装(下载)

```shell
npm install --save-dev copy-webpack-plugin
```

4.2 配置config文件

引入插件：

```js
const CopyWebpackPlugin = require('copy-webpack-plugin'); 
```

Plugins设置

```js
   new CopyWebpackPlugin([
     {
       from:__dirname+"/src/assets",
       to:__dirname+"/build/assets"
     }
   ])
```

**官方文档示例最简单的配置：**

一个from 和一个to ,就可以直接将要打包的静态文件copy过去。

```js
const CopyPlugin = require('copy-webpack-plugin');

module.exports = {
  plugins: [
    new CopyPlugin([
      { from: 'source', to: 'dest' },
      { from: 'other', to: 'public' },
    ]),
  ],
};
```

## 5、用clean-webpack-plugin来清除文件

当我们修改带hash的文件并进行打包时，每打包一次就会生成一个新的文件，而旧的文件并没有删除。

因为hash会每次都生成新的，每次名字都不同。所以打包次数越来越多，旧文件也就越多。

为了解决这种情况，我们可以使用`clean-webpack-plugin`在打包之前将文件先清除，之后再打包出最新的文件 

文档地址：[https://www.npmjs.com/package/clean-webpack-plugin](https://www.npmjs.com/package/clean-webpack-plugin)

5.1安装插件

 ```shell
 npm install --save-dev clean-webpack-plugin
 ```

5.2 配置config文件

  引入插件

```js
const CleanWebpackPlugin = require("clean-webpack-plugin")

//如果报错那么就按下面的方式写--二者选其一，那个不报TypeError: CleanWebpackPlugin is not a constructor 这个错，就用哪个。这个可能跟GitHub上文档和代码是否同步有关，我自己用上面的报错，下面的正常

const {CleanWebpackPlugin}  = require('clean-webpack-plugin');
```

  Plugin配置

```js
  plugins: [
    new CleanWebpackPlugin() //使用
  ]
```





 

 

 