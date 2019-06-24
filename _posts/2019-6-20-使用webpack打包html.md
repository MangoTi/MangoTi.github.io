---
layout:     post   				    # 使用的布局（不需要改）
title:      使用webpack打包html 				# 标题 
subtitle:   多文件的html项目打包 #副标题
date:       2019-6-20 				# 时间
author:     XMT 						# 作者
header-img: img/post-bg-2015.jpg 	#这篇文章标题背景图片
catalog: true 						# 是否归档
tags:								#标签
    - html
---
## 前言
webpack是常用的静态资源打包器，之前跟vue-cli配套使用过，最近用来打包html项目，遇到了一些问题，在此记录一下，便于学习。

## webpack核心概念
### 1.入口【entry】
即webpack应该使用哪个模块来作为构建内部依赖图的开始，一般是项目首页的js文件路径，可以是单个入口即单页项目，或是多页面应用程序，也可以是第三方库入口，entry中罗列的入口所创建的依赖图彼此完全分离、互相独立。

    
	//单个入口
    entry: {
		main: './path/to/my/entry/file.js'
	}
	//多个入口
	entry: {
		pageOne: './src/pageOne/index.js',
		pageTwo: './src/pageTwo/index.js',
		pageThree: './src/pageThree/index.js'
	}

如果首页js对其他的js有依赖，就可以在entry中如下设置，但是只限于自己写的js，如果是例如jquery，单纯这样引入就无法生效，需要先安装，然后用别的方式引入。



    entry: {
    	pagehead:path.resolve(__dirname, 'src/js/pagehead2.0.js'),
    	main: path.resolve(__dirname, 'src/js/index.js')
    },

### 2.出口【output】
即webpack输出打包后的内容，以及如何命名这些文件，通过output.filename 和 output.path 属性，还可以通过设置publicPath控制文件访问路径前缀，如下就是解决css中引用图片打包后路径不对，但是如果改成./又会导致开发环境图片路径不对，因此如此判断。

    publicPath: process.env.NODE_ENV === 'development'? '/' : './'

### 3.loader
loader让webpack处理非JS文件，因为webpack自身只理解JavaScript。loader 将所有类型的文件，转换为应用程序的依赖图（和最终的 bundle）可以直接引用的模块。

### 4.插件【plugins】
插件的范围包括，从打包优化和压缩，一直到重新定义环境中的变量，想要使用一个插件，你只需要 require() 它，然后把它添加到 plugins 数组中。


    plugins: [
		new HTMLWebpackPlugin({
			inject: true,
			template: path.resolve(__dirname, 'public/index.html')
		})
    ]

## 遇到的问题
### 1.引入jQuery.js
- 方式1：
1. npm install jquery
2. npm install expose-loader
3. 在index.js中require("expose-loader?$!jquery")
该方法可以理解为将$解释成jquery，但是如果有些js中使用jquery是用jQuery，就会无法解析，如layerui-layer中，应该可以指定多个字符。
- 方式2：
1. npm install jquery
2. 在webpack.config.js的plugins中
```javascript
	new webpack.ProvidePlugin({
    	$:"jquery",
    	jQuery:'jquery',
    	'window.jQuery':'jquery'
    })
```
这种方式比较好，因为能够声明多种使用方式

### 2.自定义全局变量
开发项目时通常需要全局变量还区分环境使用不同的接口域名。
在webpack.config.js的plugins中
```javascript
new webpack.DefinePlugin({
	'HOST': JSON.stringify(process.env.HOST)
})
```
注意全局变量不可以是proscess.evn.NODE_ENV，因为NODE_EVN本来就是一个全局变量，再次定义取值也不受控，而是根据是取值，开发环境(development)还是生产环境(production)，无法对应其余新增环境，如预发布等，就无法用来区分更多环境。

定义HOST后用如下方法来控制不同环境，在package.json中定义命令
```javascript
"scripts": {
    "start_test": "cross-env NODE_ENV=development HOST=test webpack-dev-server --hot --inline --progress",
    "start_prod": "cross-env NODE_ENV=development HOST=prod webpack-dev-server --hot --inline --progress",
    "start_pre": "cross-env NODE_ENV=development HOST=pre webpack-dev-server --hot --inline --progress",
    "build_test": "cross-env NODE_ENV=production HOST=test webpack -p --progress",
    "build_prod": "cross-env NODE_ENV=production HOST=prod webpack -p --progress",
    "build_pre": "cross-env NODE_ENV=production HOST=pre webpack -p --progress"
  },
```
前三种是本地运行，后三种是打包，然后在接口域名或其他需要区分环境的地方根据HOST的值做不同判断，例如：
```javascript
switch (HOST) {
	case 'test':{
		hostPath = "https://t-paygate.lepass.cn/pay/qr";
		domain = "http://pos.lepass.cn";
		break;
	}
	case 'prod':{
		hostPath = "https://qr.leshuazf.com/o2o";
		domain = "https://pos.yeahka.com";
		break;
	}
	case 'pre':{
		hostPath = "https://qrpre.lepass.cn/o2o";
		domain = "https://pospre.yeahka.com";
		break;
	}
}
```

### 3.layer的使用
layer通过entry引入js以及安装layer再引入的方式都不能生效，几番研究发现以下方式：
1. yarn add layui-layer
2. index.js中import layer from "layui-layer"

### 4.多页面跳转生效
按照以上描述的方法，项目只有index.js这一个页面入口，打包只对首页作用，如果要跳转到其余页面就需要将其余页面添加到打包后的文件夹中，webpack提供了拷贝的方法：
1. yarn add copy_webpack_plugin
2. 在webpack.config.js的plugins中：
```javascript
new CopyPlugin([
	{ from: __dirname+'/public/scanFail.html', to: __dirname+'/build' },
	{ from: __dirname+'/public/qrWarning.html', to: __dirname+'/build' },
	{ from: __dirname+'/public/simplePay.html', to: __dirname+'/build' },
	{ from: __dirname+'/public/sytPayResult.html', to: __dirname+'/build' },
	{ from: __dirname+'/static', to: __dirname+'/build/static' },
]),
```
除了将页面拷贝，还将static文件中的内容直接拷贝，会和webpack生成的合并在一起，这些是上方页面中使用到的资源文件。
除此之外实现多页面入口的方法可见大神demo: http://gitlab.yeahka.com/Web/pay-ui-new/tree/cagee


### css中使用的图片打包路径
因为在css文件中使用图片通常是通过（../img）的方式访问，而webpack打包时outputPath是.static/，是将所需的资源文件放build下的static文件下，并且css和图片路径前的../都会变成./static,所以图片的路径就会变成static/css/static/img，因此要设置publicPath属性为../，图片路径即恢复../img，就可以正常访问到static文件夹中的文件。
可能你会觉得是不是很多此一举，加上了static又给要去掉，但是如果一开始设置outputPath为../会导致资源文件放置的路径不对，就不在build下了。
具体是在webpack.config.js中
```javascript
module: {
	rules: [
	...
	{
		test: /\.(jpg|png|jpeg|gif|svg)$/,
		use: [{
			loader: 'url-loader',
			options: {
				limit: 8192, // 小于8KB转为base64 Data
				name: 'imgs/[name].[hash:8].[ext]',
				outputPath:'./static',
				publicPath:'../'
			}
		}]
	}
	]
}
```
