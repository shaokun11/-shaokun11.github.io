---
title: material-ui之自定义字体
date: 2019-08-13 21:02:00
---

##### 因由
> 因近期使需项目需要构建前端页面,故用reactjs作为前端的技术框架,ui选择material-ui v4.3.1作为一个基础.使用create-react-app初始化项目,其中踩了一些坑,记录在此,以做后续查看...

##### 问题
根据项目需求,第一步需使用自定义的字体,根据官方文档[typography](https://material-ui.com/zh/customization/typography/).按照此方式配置,皆无法使字体问题文件生效.(是本地的字体,不是Google font)在一番搜索后,也许是应为版本原因,或者其他原因,可以在此[github issues](https://github.com/google/material-design-icons/issues/205)看到有很多朋友均遇见了此番问题,参照之后,也没有实际解决

##### 转机出现
按照官方使用Roboto的字体,需要在index.html引入此文件  
 
```
<link rel="stylesheet" href="https://fonts.googleapis.com/css?family=Roboto:300,400,500,700&display=swap" />
```
 
使用Chrome浏览器看了一下,其加载了一个css文件,内容如下

``` css 
/* cyrillic-ext */
@font-face {
  font-family: 'Roboto';
  font-style: normal;
  font-weight: 400;
  src: local('Roboto'), local('Roboto-Regular'), url(https://fonts.gstatic.com/s/roboto/v20/KFOmCnqEu92Fr1Mu72xKKTU1Kvnz.woff2) format('woff2');
  unicode-range: U+0460-052F, U+1C80-1C88, U+20B4, U+2DE0-2DFF, U+A640-A69F, U+FE2E-FE2F;
}
/* cyrillic */
@font-face {
  font-family: 'Roboto';
  font-style: normal;
  font-weight: 400;
  src: local('Roboto'), local('Roboto-Regular'), url(https://fonts.gstatic.com/s/roboto/v20/KFOmCnqEu92Fr1Mu5mxKKTU1Kvnz.woff2) format('woff2');
  unicode-range: U+0400-045F, U+0490-0491, U+04B0-04B1, U+2116;
}
...
```
那我是不是也可以建立一个文件按照此种方式引入呢?

于是仿照上述,在index.css加入字体的应用,最终实现自定义的字体加入material-ui

```css
@font-face {
    font-family: 'Gliber';
    font-style: normal;
    font-weight: 400;
    font-display: swap;
    src: url('./theme/Gilbert_Bold-Preview_1004.woff2') format('woff2');
  }
```

#### 总结
* 经过摸索,如在index.css中加入了自定义字体,则在theme中无需按照官方的方法再次在theme.js中引入字体了,按照如下格式引入即可

```JavaScript
const typography = {
  fontFamily: ["Gliber","Roboto"].join(","),//这里引入自定义的字体名字,并放在第一
};
```

* 如果需要寻找更多的字体,可在[google fonts(自带梯子)](https://fonts.google.com/)寻找,按照Roboto的字体方式引入即可.经过测试,字体的使用可不用梯子
* 顺带在stackoverflow开启了第一个解答[Self-host font added in Material-UI not working?
](https://stackoverflow.com/questions/57108085/self-host-font-added-in-material-ui-not-working/57450653#57450653)

##### 结果演示
[源码](https://github.com/shaokun11/material-ui-learn/tree/v1.0)  
![material-ui self host font](/react/material-ui-font.gif) 
##### 关于我
区块链技术痴迷的程序猿一枚，如果你喜欢我的文章，可以加上微信共同学习，共同进步。  
（加好友时，请备注 区块链，谢谢）  
![jungle](/common/wx.png) 


