---
title: antd实现theme的切换(2)
date: 2020-11-29 15:51:05
---

#### 目标
-  如果只有一套可以按照官方的介绍的方式,配置在项目中即可达到目的
-  本篇文章实现的是根据ui提供的less文件使用webpack导出自定义的css

#### 实现流程

- 目录结构

```bash
custom-less-theme
package-lock.json	src
package.json		webpack.config.js

./src:
main.js
```

- 安装依赖

```javascript
	{
	  "scripts": {
    	"build": "webpack "
  	},
	 "dependencies": {
    "antd": "^4.8.6",
    "css-loader": "^5.0.1",
    "css-minimizer-webpack-plugin": "^1.1.5",
    "less": "^3.12.2",
    "less-loader": "^7.1.0",
    "mini-css-extract-plugin": "^1.3.1",
    "style-loader": "^2.0.0",
    "webpack": "^5.8.0",
    "webpack-cli": "^4.2.0"
  	}
  }
```

- 导出less文件即main.js的内容

```javascript
	import "antd/dist/antd.less"
```

- webpack.config.js的配置,固定写法,其中用到了两个插件,一个为将less转化成的css提取出来,一个为压缩提取出来的css.
- 这里可以将要修改的变量采用一个json来配置,方便管理
	
```javascript
	const path = require("path");
const MiniCssExtractPlugin = require("mini-css-extract-plugin");
const CssMinimizerPlugin = require('css-minimizer-webpack-plugin');
module.exports = {
  mode: "production",
  entry: "./src/main.js",			//入口js
  output: {
    filename: "main.js",			// 输出的js,我们不需要,这里是个空文件
    path: path.resolve(__dirname, "dist"),
  },
  plugins: [
    new MiniCssExtractPlugin({
      filename: "myant.min.css",		// 输出的css文件名
    }),
  ],
  optimization: {
    minimizer: [
      new CssMinimizerPlugin(),
    ],
  },
  module: {
    rules: [
      {
        test: /\.less$/,
        use: [
          { loader: MiniCssExtractPlugin.loader },
          {
            loader: "css-loader",
          },
          {
            loader: "less-loader",
            options: {
              lessOptions: {
                javascriptEnabled: true,
                modifyVars: {
                  "primary-color": "#1DA57A",	// 要修改的less变量
                },
              },
            },
          },
        ],
      },
    ],
  },
};

```

- 执行打包, npm run build

##### 结果演示
- 实现的效果为从antd->antd dark -> andt的primary颜色的三个主题来回变化,其中绿色的primary即是本次实现的自定义主题的效果啦  

![graphql query demo](/react/antdTheme2.gif) 


#### 总结
> 熟悉各个工具的使用,细心跟着文档做就可以啦

##### 关于我
软件开发攻城狮一枚,熟悉dapp,web,android,node,go等软件的开发。  
**email:skunny@163.com**  



