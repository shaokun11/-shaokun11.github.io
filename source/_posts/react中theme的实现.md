---
title: antd实现theme的切换
date: 2020-11-28 19:53:18
---

#### 目标
-  在react的开发中,如果使用在antd作为其基础的ui库,根据官方文档,没有提供类似material-ui的themeProvider来非常方便的实现动态的切换当前页面的主题的方式(如果有有方便的方式,请告诉我一下...),那么接下来我们将采用非常规的方式来实现这个目的
- 本篇文章中并不是在介绍如何自定义主题呢,请注意一下

#### 实现流程
1. 首先按照介绍安装antd,
2. 进入antd/dist/中,将**antd.dark.min.css / antd.min.css**文件拷贝到public目录中(一般我们仅需要这两个主题),注意是css
3. 修改public目录下的index.html,并将其加入到样式表中

```css
   	<link type="text/css" rel="StyleSheet" href="./antd.dark.min.css" />
    <link type="text/css" rel="StyleSheet" href="./antd.min.css"/>
```
   
4. 实现切换主题(由于这个css文件基本不会改变,故根据名称加载即可,当然其他方法也可以),其原理就是动态覆盖css变量而已咯

```javascript
import {Button} from 'antd';
import React, {useState} from 'react';

function switchTheme(theme: number) {
	let cssArr = ["antd.min.css", "antd.dark.min.css"];
	let headHTML = document.getElementsByTagName('head')[0];
	let link = document.createElement("link");
	link.type = "text/css";
	link.rel = "Stylesheet";
	link.href = cssArr[theme % 2];
	headHTML.append(link);
}

function App() {
	const [theme, updateSet] = useState(0);
	React.useEffect(() => {
		switchTheme(theme);
	}, [theme]);
	return (
		<div className="App">
			<Button type={"primary"} onClick={() => {
				updateSet(x => x + 1);
			}}> switch theme </Button>
		</div>
	);
}
```

##### 结果演示
![graphql query demo](/react/antdTheme.gif) 

#### 总结
- 对没有提供如react-material-ui中themeProvider来实现theme的切换的ui框架,采用此种方式实现虽然操作上不怎么优雅,但是也能够达到目的,在没有更好的方式来实现的时候,可以一试
- 目前验证antd,React Suite均能够实现通过此种方式,理论上提供css样式修改的ui框架均可以采用此方式是来实现实现主题的切换
- 建议将所有的主题提前在header中加载,这样在切换主题时直接从缓存中加载,避免切换的过程中出现闪烁的情况

##### 关于我
区块链技术痴迷的程序猿一枚，如果你喜欢我的文章，可以加上微信共同学习，共同进步。  



