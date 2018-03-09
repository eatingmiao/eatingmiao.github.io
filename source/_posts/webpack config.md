title: Webpack Loader and Plugin
tags: [JavaScript]
date: 2018-03-08
---

Webpack是当前流行的JavaScript静态资源打包工具，可以根据四个核心设置Entry，Output，Loaders，Plugins来处理资源文件，构建各个module的依赖脉络。

## Webpack Loader
Webpack的loader可以针对不同类型的文件进行有效处理，转化成应用可以直接引用的模块，比如可以把sass转化成css，把ts或者es6代码转化成大部分浏览器都支持的es5。

在Webpack中，loader有两个主要的作用属性:

- test：需要进行转换的文件类型
- use：转换需要的对应loader

在webpack.config.js中配置loader：

	const path = require('path');
	
	const config = {
	  entry: './path/to/my/entry/file.js',
	  output: {
	    path: path.resolve(__dirname, 'dist'),
	    filename: 'my-webpack.bundle.js'
	  },
	  module: {
	    rules: [
	      { test: /\.scss$/, use: 'sass-loader' }
	    ]
	  }
	};
	
	module.exports = config;

loader如果有多个，use的参数可以是数组，也可以设置各个loader的参数：

	module: {
	  rules: [
	    {
	      test: /\.css$/,
	      use: [
	        { loader: 'style-loader' },
	        {
	          loader: 'css-loader',
	          options: {
	            sourceMap:true,
	            modules: true
	          }
	        }
	      ]
	    }
	  ]
	}
	
常用的loader有:

- 样式：style-loader、css-loader、less-loader、sass-loader等

- 文件：raw-loader、file-loader 、url-loader等

- 编译：babel-loader、coffee-loader 、ts-loader等

- 校验测试：mocha-loader、jshint-loader 、eslint-loader等

babel-loader可以把ES6的代码转换成ES5代码。

file-loader可以复制和放置资源位置，指定文件名模板，也可以用hash命名以便缓存利用。

url-loader可以将小于配置limit参数的文件转换成data-url的方式，以便减少请求。

raw-loader可以读取文件，以字符串的形式返回。

imports-loader可以把文件以变量的形式向模块注入，比如注入jQuery：  
	
	$,imports-loader?$=jquery // 相当于var $ = require("jquery")

expose-loader可以设置对象为全局变量。

可以由此看出，loader本质就是接收字符串或者buffer，再返回处理完的字符串或者buffer。

除了常用的loader外，也可以根据实际生产环境自定义loader。比如需要把资源加载进模板的占位符：
	
	module.exports = function (source) {
	  return getTemplete().replace('{__templete_content__}', source)
	}

## Webpack Plugin
Webpack的plugin与loader的作用有点相似，都扩展了打包的功能，但是loader专注于资源转换，而plugin则可以执行更丰富或者复杂的任务，比如压缩，优化，重新定义环境变量。

配置plugin先用require方式引入，再用new操作符创建实例作为参数插入plugins数组，大部分plugin都可以通过选项进行定制。

在webpack.config.js中配置plugins：

	const HtmlWebpackPlugin = require('html-webpack-plugin');
	const webpack = require('webpack');
	const path = require('path');
	
	const config = {
	  entry: './path/to/my/entry/file.js',
	  output: {
	    path: path.resolve(__dirname, 'dist'),
	    filename: 'my-first-webpack.bundle.js'
	  },
	  module: {
	    rules: [
	      { test: /\.txt$/, use: 'raw-loader' }
	    ]
	  },
	  plugins: [
	    new webpack.optimize.UglifyJsPlugin(),
	    new HtmlWebpackPlugin({template: './src/index.html'})
	  ]
	};
	
	module.exports = config;

在loader中虽然也可以进行html模板的替换，但是HtmlWebpackPlugin的接口配置比loader要丰富得多：

- title：用于生成的HTML文件的标题。
- filename：用于生成的HTML文件的名称，默认是index.html，可以在这里指定子目录。
- template：模板文件路径，支持加载器，比如 html!./index.html
- inject：true | 'head' | 'body' | false，注入所有的资源到特定的template或templateContent中，如果设置为true或者body，所有的javascript资源将被放置到body元素的底部，'head'将放置到head元素中。
- favicon：添加特定的favicon路径到输出的 HTML 文件中。
- minify：压缩HTML文件，removeComments为true移除HTML中的注释，collapseWhitespace为true删除空白符与换行符。
- hash：true | false，如果为true，将添加一个唯一的webpack编译hash到所有包含的脚本和CSS文件，对于cache管理很有用。
- cache：true | false，如果为true，仅在文件修改之后才会发布文件。
- showErrors：true | false，如果为true，错误信息会写入到页面中。
- chunks：允许只添加某些块（比如unit、test）。
- chunksSortMode：用于控制块在添加到页面之前的排序方式。
- excludeChunks：允许跳过某些块（比如unit、test）。

同样plugin也可以根据生产环境进行定制，通过plugin你可以访问compliler和compilation过程，通过钩子控制webpack的执行流程，从这里可以看出plugin会比loader更适合做复杂的任务。

一个完整的webpack插件需要满足以下几条规则和特征：

- 是一个独立的模块。
- 模块对外暴露一个js函数。
- 函数的原型prototype上定义了一个注入compiler对象的apply方法。
- apply函数中需要有通过compiler对象挂载的webpack事件钩子，钩子的回调中能获取当前编译的compilation对象，如果是异步编译的话则可以获取callback。
- 完成自定义子编译流程并处理complition对象的内部数据。
- 如果异步编译插件的话，数据处理完成后执行callback。