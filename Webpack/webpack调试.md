## webpack 配置



#### devtool	

​	用来控制是否生成，以及如何生成 source map ( 用来调试代码在打包后依然能在控制台查看源代码文件 )

devtool 是 webpack 本身自带的配置项 也可以使用插件来进行项目调试 	`SourceMapDevToolPlugin`*/*`EvalSourceMapDevToolPlugin` 

上述插件拥有比 devtool更加细致的选项 但禁止devtool和插件同时使用

​	![image-20191108111755007](/Users/javascript/Library/Application Support/typora-user-images/image-20191108111755007.png)



在webpack.config.js 中的配置

```javascript
module.exports = {
    ...
    devtool: "cheap-eval-source-map"      // 根据环境不同 可配置不同选项 
    ...
}
```



#### 模块热替换

​	模块热替换 和 网页自动刷新的区别 可以理解为 Ajax的异步请求和同步请求 

```javascript

	devServer:{
        contentBase: './build',    					// 设置服务器访问的基本目录 , 一般为打包目录 
        host: 'localhost',    	 					// 服务器 IP 地址
        port: 8000,									// 端口号
        open: true,									// 自动打开浏览器
            
        hot: true 									// 开启模块热替换 
        // 不设置hotOnly:
        hotOnly: true								// 只有热更新 不会刷新页面
	},
    ...    
    plugins:[
        new webpack.NamedModulesPlugin(),   		// 当开启HMR时是用该插件 (折行)
        new webpack.HotModuleReplacementPlugin(),   // 会显示模块的相对路径
    ]


///////// 需要在入口文件中 index.js 补充
	if( module.hot ){
        // 手动刷新模块
        module.hot.accept()
    }
```



#### 环境的区分

​	开发环境和生产环境项目构建有很大差异 比如模块热替换 devServer 放在开发环境 而生产环境则关注更小的Bundle以及资源的优化 比如压缩HTML，CSS，JS  所以建议为每一个环境创建独立的webpack配置

​	安装 webpakc-merge 

npm install --save-dev webpack-merge

​	拆分webpack.config.js 如果你的项目只区分开发和生产两种环境的话 那么就将webpack.config.js 拆分为三个文件 分别是 webpack.common.config.js（基本的公用配置） webpack.dev.config.js（开发环境配置）webpack.pro.config.js（生产环境配置）

##### 如果拆分文件后路径和原来不一致 要注意修改配置项的路径（loader，plugin）

##### webpack.common.config.js

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
    // 插件配置
    plugins: [
        // 自动生成html
        new HtmlWebpackPlugin({
            template: 'index.html', 					// 模版路径 
            filename: 'index,htnl',						// 生成的html名称
            hash: true									// 生成hash值 避免缓存
            minify:{
            		// html 压缩选项 具体查看文档
        	}
       }),
    ]
}
```



##### webpack.dev.config.js

```javascript
const webpack = require( 'webpack' );
const merge = require( 'merge' );
const common = require( './webpack.common.config.js' );

module.exports = merge( common , { // 此时使用 merge 模块来合并配置文件
    devtool: 'cheap-module-eval-source-map',
    devServer: {
        contentBase: './build',    				
        host: 'localhost',    	 					// 服务器 IP 地址
        port: 8000,									// 端口号
        open: true,									// 自动打开浏览器
        hot: true 									// 热更新
    },
    plugins:[
        new webpack.NameeMoudlesPlugin(),   		// 当开启HMR时是用该插件 (折行)
        new webpack.HotModuleReplacementPlugin(),   // 会显示模块的相对路径
    ]
})
```



##### webpack.pro.config.js

```javascript
const merge = require( 'merge' );
const MiniCssPlugin = require( 'mini-css-extract-plugin' );
const OptimizeCssAssetsPlugin = require( 'optimize-css-assets-webpack-plugin' );
const common = require( './webpack.common.config.js' );

module.exports = merge( common , { // 此时使用 merge 模块来合并配置文件
	devtool: 'source-map',
    plugins:[
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
})
```



##### 最后注意package.json的配置

```javascript
{
  "version": "1.0.0",
  "description": "My webpack project",
  "scripts": {
    /******************** 打包模式切换 ********************/
	"build-dev": "webpack --mode development --config ./webpack.dev.config.js", 
    "build-pro": "webpack --mode production --config ./webpack.pro.config.js",	 		
    /************** 配置webpack-dev-server **************/
	"start-dev": "webpack-dev-server --mode development --config ./webpack.dev.config.js" 
	"start-pro": "webpack-dev-server --mode production  --config ./webpack.pro.config.js"      
    
    
  },
	"devDependencies":{ ... },
  "dependencies":{ ... }
}
```

