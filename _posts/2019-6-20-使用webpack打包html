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
3. 在index.js中
4.
