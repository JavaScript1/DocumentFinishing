##  Webpack 整理文档



版本：4.41.2 

cli版本：3.3.9

项目由 webpack-cli 创建自动生成 webpack.config.js 等必要文件和配置





#### 入口 entry

在 webpack.config.js 中设置 entry字段，该字段描述整个项目的入口 (JS, IMG , CSS)  等等

```javascript
entry: './src/index.js'       						// 单一入口
entry: ['./src/index1.js' , './src/index2.js]       // 多入口 -- 两个JS相互依赖情况下
entry: {																						// 多入口 -- 多页面
	page1: './src/page1.js',
    page2: './src/page2.js'
}                           

```



#### 出口 output

打包输出

```javascript
	// 文件名称使用 [name] [hash] [chunkhash] 等别名 方便不冲突
	// [name] -- 打包之前JS名称
	// [hash] -- 打包时生成的hash值
	// [chunkhash] -- 类似于版本号 打包时生成和hash不同的是只有修改过后才会产生新的chunkhash
	// 文件名称写死的话 多入口打包 后续文件会自动替换已经打完包的文件 最后值留下一个JS文件
output: {
		filename: '[name].[chunkhash].js',  			// 文件名称
		path: path.resolve(__dirname, 'dist')			// 文件路径
	},
```



#### loader

将非JS的文件 用对应的loader包 转换成webpack可以理解的JS模块 最后进行打包

```javascript
// 这个Demo可以实现 html 热更新
// 但会与 htmlWebpackPlugin.options.title 冲突
module: {
    rules: [
      { 
        test: /\.html$/,   					// 哪些文件
        /* 使用什么 loader 包来转换为JS模块  */
				use:['html-loader']			// 方式1 loader无配置
        use:{								// 方式2 loader有配置
        	loader: 'html-loader',
        	options:{
        			// 配置项 根据loader不同 配置项也不同具体看对应文档    
      		}
      	}
      }  
    ]
  }
```



#### 插件 Plugins

和loader相比 Plugins可以用于更广泛的业务场景 类似于 node的 npm包

```JavaScript
const HtmlWebpackPlugin =  require('html-webpack-plugin');

...
// plugins 字段是一个数组类型，描述了该项目都需要哪些插件
plugins: [
    new HtmlWebpackPlugin() // 下载插件之后直接实例化就可以使用 同时也可以传递参数
  ]
```



#### 模式 

```javascript

```



#### 配置webpack.config.js , package.json

##### 	webpack-dev-server

​		为静态文件提供 web 服务

​		自动刷新和热替换(HMR)

​		npm install --save-dev webpack-dev-server

```javascript

	devServer:{
        contentBase: './build',    					// 设置服务器访问的基本目录 , 一般为打包目录 
        host: 'localhost',    	 					// 服务器 IP 地址
        port: 8000,									// 端口号
        open: true,									// 自动打开浏览器
	},
```



##### 	style-loader , css-loader

​		处理css文件 转化成JS模块

​		npm install --save-dev style-loader css-loader

```javascript
module:{
    rules:[
      {
          test: /\.css$/,
          // 注意引用顺序 style在前 css在后
          use: ['style-loader','css-loader']
      },  
    ]
}
```



##### 	postcss-loader , autoprefixer

​		为css 代码自动添加浏览器前缀 

​		一般为解决浏览器兼容性

​		npm install --save-dev postcss-loader autoprefixer

```javascript
module:{
    rules:[
      {
          test: /\.css$/,
          // 注意引用顺序 style在前 css在后
          use: ['style-loader','css-loader',{
          	  loader: 'postcss-loader',
              options: {
                  plugins: [ require('autoprefixer') ]
              } 
          }]
      },  
    ]
}
```



##### 	file-loader

​		文件处理	

​		npm install --save-dev file-loader

```javascript
module:{
    rules:[
		{
            test: /\.(png|jpg|gif|jpeg)$/,
            use: [{
                loader: 'file-loader',
                options: {
                    name: '[hash].[ext]',				// 文件名。[hash]-hash值 [ext]-文件后缀 
                    context: 'webpack.config.js',		// 上下文路径
                    publicPath: '',						// 发布目录
                    outputPath: './public/img', 	  	// 输出目录
                }
            }]
        },
    ]
}
```



##### 	webpack.ProvidePlugin()  ！！！在项目中 直接  import 也生效

​		处理第三方 JS 库 

​		参数 是以键值对形式 Key 表示项目中的变量名 Value 表示 本地索引 在resolve配置

​		以jquery为例

​		npm install --save-dev jquery

```javascript
	const webpack = require( 'webpack' );
...
	resolve: {
        alias: { // 别名 一般用于项目引用本地文件 
            jQuery: path.resolve( __dirname,'public/js/jquery.min.js' )
        }
    },
    // 插件配置
    plugins: [
        // 配置 第三方JS
        new webpack.ProvidePlugin({
            jQuery: 'jQuery'
        }),
    ]
```



##### 	ES6 --> ES5

​		将 es6 代码 转换成为 es5

​		需要三个依赖包

​		babel-loader: 负责es6语法转化

​		babel-code: babel核心包

​		babel-preset-env: 告诉babel使用哪种转码规则进行文件处理

​		方式1：需要在根目录创建 .babelrc 文件

​		方式2：在babel 的loader 中设置

```javascript
// .babelrc文件。和webpack.config.js 同一级别
{
    "presets": ["@babel/preset-env"]
}
```

​		npm install babel-loader @babel/core @babel/preset-env --save-dev

```javascript
module:{
    rules:[
        {
            test: /\.js$/,
            exclude: /node_modules/, // 排除这个文件夹
            use: {
                loader: 'babel-loader',
                options: {
                    "presets": ["@babel/preset-env"]	  // 方式2
                }
            }
        }
    ]
}
```



##### 	html-webpack-plugin

​		自动生成HTML文件 同时也可以设置模版

​		可以对html进行压缩 删除无用元素

​		npm install --save-dev html-webpack-plugin

```javascript
const HtmlWebpackPlugin = require( 'npm install --save-dev html-webpack-plugin' );
...
plugins: [
    // 自动生成html
    new HtmlWebpackPlugin({
        template: 'index.html', 					// 模版路径 
        filename: 'index,htnl',						// 生成的html名称
        hash: true									// 生成hash值 避免缓存
        minify:{									// 压缩HTML
            crashWhitespace: true,
            removeComments: true,
            removeRedundantAttributes: true,
            removeScriptTypeAttributes: true,
            removeStyleLinkTypeAttributes: true,
            useShortDoctype: true
    	}
     }) 
]
```

​	

##### 	mini-css-text-plugin

​	抽离css文件单独打包

​	npm install --save-dev mini-css-extract-plugin css-loader

```javascript
const MiniCssPlugin = require( 'mini-css-extract-plugin' );
...
module:{
    rules:[
		{
        	test: /\.css$/,
            use:[MiniCssPlugin.loader,'css-loader']
        }
    ]
}
...
plugins:[
	new MiniCssPlugin({
        filename: './css/[name]-[hash].css'
    })
]
```

#### 

#####  	optimize-css-assets-webpack-plugin

​	压缩 优化 css

​	npm install --save-dev optimize-css-assets-webpack-plugin

```javascript
const OptimizeCssAssetsPlugin = require( 'optimize-css-assets-webpack-plugin' );
...
plugins:[
    new OptimizeCssAssetsPlugin({
    	assetNameRegExp: /\.css$/g, 		// 匹配文件
        cssProcessor: require('cssnano'), 	// 用于压缩和优化的处理器
        cssProcessorPluginOptions:{
            preset: ['default' , {discardComments:{removeAll:true}}] // 去掉注释
        },
        canPrint: true
    })
]
```



##### 	clean-webpack-plugin

​	删除打包文件 避免多次打包造成文件越来越多

​	npm install --save-dev clean-webpack-plugin

```javascript
const cleanWebpackPlugin = require( 'clean-webpack-plugin' );
plugins:[
	new cleanWebpackPlugin(['./build'])   //删除路径
]
```



##### 	html-loader

​	处理HTML文件	

```javascript
module:{
    rules:[
        {
            test: /\.(html)$/,
            use:{
                loader: 'html-loader',
                option: {
                    attrs:['img:src','img:data-src']  // 处理html内嵌图片
                }
            }
        }
    ]
}

<img src="./img/1.jpg" data-src="./img/1.jpg"/> // 添加data-src属性
```



#### 在webpack.config.js文件中配置

```javascript
const path = require( 'path' );
const webpack = require( 'webpack' );
const HtmlWebpackPlugin = require( 'html-webpack-plugin' );
const MiniCssPlugin = require( 'mini-css-extract-plugin' );
const OptimizeCssAssetsPlugin = require( 'optimize-css-assets-webpack-plugin' );

module.exports = {
    entry: './src/index.js', 							// 文件入口
    output:{											// 文件出口 
        path: path.resolve( __dirname , 'dist' ),		// 打包路径
        fileName: '[name]-[chunkhash].js'				// 文件名称
    },
    // 配置 dev-sercer
    devServer:{
        contentBase: './build',    					// 设置服务器访问的基本目录 , 一般为打包目录 
        host: 'localhost',    	 					// 服务器 IP 地址
        port: 8000,									// 端口号
        open: true,									// 自动打开浏览器
        hot: true 									// 热更新
    },
    // Loader 配置
    module:{
        rules: [
            // 配置 css-loader style-loader
            {
                test: /\.css$/,
                // 注意引用顺序 style在前 css在后
                use: [MiniCssPlugin.loader,'style-loader','css-loader',{
                    // 配置 css前缀
                    loader: 'postcss-loader',
                    options: {
                        plugins: [ require('autoprefixer') ]
                    }
                }]
            },
            // 配置文件处理（图片）
            {
                test: /\.(png|jpg|gif|jpeg)$/,
                use: [{
                    loader: 'file-loader',
                    options: {
                        name: '[hash].[ext]',			// 文件名。[hash]-hash值 [ext]-文件后缀 
                        context: 'webpack.config.js',	// 上下文路径
                        publicPath: '',					// 发布目录
                        outputPath: './public/img', 	// 输出目录
                    }
                }]
            },
            // 配置 babel
            {
                test: /\.js$/,
                exclude: /node_modules/, // 排除这个文件夹
                use: {
                    loader: 'babel-loader',
                    options: {
                        "presets": ["@babel/preset-env"]	  // 方式2
                    }
                }
            }
        ]
    },
    resolve: {
        alias: { // 别名 一般用于项目引用本地文件 
            jQuery: path.resolve( __dirname,'public/js/jquery.min.js' )
        }
    },
    // 插件配置
    plugins: [
        // 配置 第三方JS
        new webpack.ProvidePlugin({
            jQuery: 'jQuery'
        }),
        // 自动生成html
        new HtmlWebpackPlugin({
            template: 'index.html', 					// 模版路径 
            filename: 'index,htnl',						// 生成的html名称
            hash: true									// 生成hash值 避免缓存
            minify:{
            		// html 压缩选项 具体查看文档
        	}
       }),
        // 抽离css 单独打包
       new MiniCssPlugin({
       		filename: './css/[name]-[hash].css'
	   }),
       // 压缩css
       new OptimizeCssAssetsPlugin({
           assetNameRegExp: /\.css$/g, 							// 匹配文件
           cssProcessor: require('cssnano'), 					// 用于压缩和优化的处理器
           cssProcessorPluginOptions:{
               preset: ['default' , {discardComments:{removeAll:true}}] // 去掉注释
           },
           canPrint: true
       })
    ]
}
```



#### 在package.json文件中配置

```javascript
{
  "version": "1.0.0",
  "description": "My webpack project",
  "scripts": {
    /******************** 打包模式切换 ********************/
		"build-dev": "webpack --mode development", 						   // 开发环境
    "build-pro": "webpack --mode production",	 						   // 正式环境
      
    /************** 配置webpack-dev-server **************/
    "start-dev": "webpack-dev-server --mode development"     // 启动开发环境
    "start-pro": "webpack-dev-server --mode production"      // 启动开发环境
    
    
  },
	"devDependencies":{ ... },
  "dependencies":{ ... }
}
```



