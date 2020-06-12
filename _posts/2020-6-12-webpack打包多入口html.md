### 前言
如果是多入口的html项目，有一些js依赖如vue等，并且项目较复杂，使用webpack打包能自动压缩，便于区分环境管理项目，后期开发也能随时展开，不过第一次配置有点麻烦，记录如下
## 配置过程
1. 使用npm init -y初始化项目，会生成package.json文件
2. 修改package.json
```javascript
"scripts": {
	  "start_test": "cross-env NODE_ENV=development HOST=test webpack-dev-server --hot --inline --progress",
	  "start_prod": "cross-env NODE_ENV=development HOST=prod webpack-dev-server --hot --inline --progress",
	  "start_pre": "cross-env NODE_ENV=development HOST=pre webpack-dev-server --hot --inline --progress",
	  "build_test": "cross-env NODE_ENV=production HOST=test webpack -p --progress",
	  "build_prod": "cross-env NODE_ENV=production HOST=prod webpack -p --progress",
	  "build_pre": "cross-env NODE_ENV=production HOST=pre webpack -p --progress"
},
"author": "",
"license": "MIT",
"browserslist": [
	  "> 1%",
	  "last 2 versions",
	  "not ie <= 8"
],
"dependencies": {
	  "copy-webpack-plugin": "^5.0.3",
	  "expose-loader": "^0.7.5",
	  "file-loader": "^3.0.1",
	  "jquery": "^3.4.1",
	  "url-loader": "^1.1.2",
	  "vconsole-webpack-plugin": "^1.4.2"
},
"devDependencies": {
	  "@babel/core": "^7.4.4",
	  "@babel/plugin-transform-runtime": "^7.4.4",
	  "@babel/preset-env": "^7.4.4",
	  "@babel/runtime": "^7.4.4",
	  "babel-core": "^7.0.0-bridge.0",
	  "babel-loader": "^8.0.6",
	  "babel-plugin-transform-runtime": "^6.23.0",
	  "babel-preset-env": "^1.7.0",
	  "babel-runtime": "^6.26.0",
	  "clean-webpack-plugin": "^2.0.2",
	  "copy-webpack-plugin": "^5.1.1",
	  "cross-env": "^5.2.0",
	  "css-loader": "^2.1.1",
	  "html-webpack-plugin": "^3.2.0",
	  "mini-css-extract-plugin": "^0.6.0",
	  "postcss-import": "^12.0.1",
	  "postcss-loader": "^3.0.0",
	  "postcss-preset-env": "^6.6.0",
	  "style-loader": "^0.23.1",
	  "webpack": "^4.31.0",
	  "webpack-cli": "^3.3.2",
	  "webpack-dev-server": "^3.4.1"
}
```
script中是使用了环境变量来区分三个环境

3. 创建webpack.config.js
```javascript
const path = require('path');
const CleanWebpackPlugin = require('clean-webpack-plugin'); // 用于清除历史打包文件
const HTMLWebpackPlugin = require('html-webpack-plugin'); // 用于生成目标html文件
const MiniCssExtractPlugin = require('mini-css-extract-plugin'); // 压缩css
const webpack = require('webpack');
const vConsolePlugin = require('vconsole-webpack-plugin'); // vconsole控制台
const CopyPlugin = require('copy-webpack-plugin'); // 直接将文件或文件夹复制到打包文件夹中
var hostPath;
switch (process.env.HOST) {
	  case 'test':{
		hostPath = "https://t-saas-coupon.lepass.cn";
		break;
  }
	  case 'prod':{
		hostPath = "https://saas-coupon.leshuazf.com";
		break;
  }
	  case 'pre':{
		hostPath = "https://p-saas-coupon.lepass.cn";
		break;
  }
}
let openVconsole = process.env.HOST !== 'prod'? true : false; // 根据环境动态判断是否需要vconsole
module.exports = {
// 入口js，这里有两个页面的入口，注意引号中的页面路径不需要./等前置路径，scr在根目录下
// path.resolve()方法可以将路径或者路径片段解析成绝对路径,可以传入2个参数，从右至左解析，如果第二个参数是绝对路径，即/src...
// 会忽略第一个参数，否则会在第一个参数的目录下根据第二个路径生成绝对路径
// __dirname代表的是当前文件的绝对路径
	  entry: {
		index: path.resolve(__dirname, 'src/js/index.js'),
		activity: path.resolve(__dirname, 'src/js/activity.js')
	  },
// 输出路径，最终会输出到dist文件夹中，打包后的js文件名会是index-bundle.js和activity-bundle.js
	  output: {
		path: path.resolve(__dirname, 'dist'),
		filename: './js/[name]-bundle.js',//多文件name不能写死
  },
	  mode: 'development',
  	module: {
// 依赖包中的js使用babel处理，能够兼容处理es6等写法
    	rules: [{
      	test: /\.js$/,
     	 exclude: /node_modules/,
      	use: {
        	loader: 'babel-loader'
      	}
    	}, {
      	test: /\.css$/,
      	use: [
        	MiniCssExtractPlugin.loader,
        	{
          	loader: 'css-loader',
          	options: {
            	importLoaders: 1
          	}
        	},
        	'postcss-loader'
      	]
    	}, {
      	test: /\.(jpg|png|jpeg|gif|svg)$/,
      	use: [{
        	loader: 'url-loader',
        	options: {
          	limit: 8192, // 小于8KB转为base64 Data
          	name: 'image/[name].[hash:8].[ext]',
          	outputPath:'./',
          	publicPath:'../', 图片使用时路径前缀，如果是在css中使用，打包后就会找../+name路径下的图片
        	}
      	}]
    	}, {
      	test: /\.(woff2?|eot|ttf|otf)(\?.*)?$/,
      	use: [{
        	loader: 'url-loader',
        	options: {
          	limit: 50000,
          	name: 'src/css/font/[name].[hash:8].[ext]',
          	publicPath:'../'
        	}
      	}]
    	}]
  	},
  	resolve: {
    	modules: [ __dirname, 'node_modules' ]
  	},
  	plugins: [
    	new HTMLWebpackPlugin({
      	filename: "index.html",
      	template: path.resolve(__dirname, './index.html'), // 生成html模板，有几个入口就要写几个，会将打包后的js、css等引入
      	chunks: ['index'],// 页面对应的入口js
    	}),
    	new HTMLWebpackPlugin({
      	filename: "activity.html",
      	template: path.resolve(__dirname, './activity.html'),
      	chunks: ['activity'],
    	}),
    	new MiniCssExtractPlugin({
      	filename: './css/[name].[contenthash:8].css',
      	chunkFilename: './css/[name].[contenthash:8].chunk.css'
    	}),
    	new CopyPlugin([
      	{ from: __dirname+'/src/js/rem.js', to: __dirname+'/dist/js' }, // 复制文件夹或页面到打包后的文件夹中
// 不会经过处理
    	]),
    	new vConsolePlugin({
      	filter: [],  // 需要过滤的入口文件
      	enable: openVconsole // 根据环境控制是否使用vconsole
    	}),
    	new CleanWebpackPlugin(),
    	new webpack.DefinePlugin({
      	'HOST': JSON.stringify(process.env.HOST),// 添加环境变量
      	'HOSTPATH': JSON.stringify(hostPath),// 接口域名地址
    	})
	],
	devServer: {
  		host: '0.0.0.0',
  		port: 9003,
  		contentBase: './dist',
  		historyApiFallback: true,
  		watchOptions: {
    		aggregateTimeout: 300,
    		poll: 1000
  		},
  		proxy:{
    		'/open-api/':{
      		target: 'https://t-saas-coupon.lepass.cn/',
      		changeOrigin:true
    	}
  	}
},
// devtool: 'source-map',
externals: []
```

4. 运行npm install --registry=https://registry.taobao.org
安装所有依赖包

5. 报错Error: No PostCSS Config found in...
解决办法：根目录下新增文件postcss.config.js，内容如下
```javascript
module.exports = {
  plugins: {
    'autoprefixer': {browsers: 'last 5 version'}
  }
}
```

6. html中不引入js和css,采用在入口js中import的方式：
```html
import './../css/base.css'
import Vue from './vue.min.js'
import layer from './layer.js'
import './../css/layer.css'
import axios from './axios.min.js'import './../css/base.css'
import Vue from './vue.min.js'
import layer from './layer.js'
import './../css/layer.css'
import axios from './axios.min.js'
```
但是如果有使用rem，在此处引入，首屏渲染体验会不好，解决办法是在html的head中直接引入，并且将rem不经过压缩直接复制到目标文件加中
见上方CopyPlugin插件配置
<script src="./js/rem.js"></script>
路径是打包后的路径，即使是本地运行也能找到文件，应该是运行有临时生成，本来文件的路径是src/js/rem.js

7. 使用vue时，当js未加载好，页面中的变量会暴露，影响体验，解决办法是最外层div初始隐藏，然后在js中mounted中
document.getElementById('container').style.display = 'block'
