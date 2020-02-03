---
title: 在typescript中使用eosjs-ecc
date: 2020-02-02 16:19:14
---
#### 前言
- ts开发后端项目,考虑到维护的需要,还是需要ts来支持.其中涉及到的lib都能找到对应的types,但是也有例外比如eosjs-ecc

#### 解决方法
1. 简单快速,但是没有任何提示了,IDE基本的提示都没有了,对于直接将.js-->.ts可以使用

```javascript
	// tsconfig.json中将此字段设为false
   "noImplicitAny": false,
```

2. 自己写 **d.ts,喜欢的同学可以尝试一下
3. 工具自动生成,推荐方式  
	-这里使用微软提供的[dts-gen](https://github.com/microsoft/dts-gen)进行生成,放到项目的任意目录即可,但是还是无法使用,这时需要在最外面包裹一层 declare module 'eosjs-ecc'{}即可,如果使用ts-node执行项目,需要在导入的时候添加ts-ignore,直接使用tsc编译则可以不用
	
	```javascript
	// @ts-ignore
	import ecc from 'eosjs-ecc'
	ecc.randomKey(0).then((privateKey:string) => {
    	console.log(privateKey )
	});
	```
4. 优化  
	- 细心的同学发现,里面的类型全是any,一点用处都没有啊?   
	但是这样可以有IDE的提示了,ts可以正常写代码了呢  
	- 由于一个lib我们不会完全用它的方法,那么我们只需要去修改一下我们需要使用的类型即可.比如randomKey这个方法,我们将它的declare中的按照如下更改即可,再次调用就达到了正常使用lib的目的啦

	```javascript
	// export function randomKey(cpuEntropyBits: number ): Promise<string>;
	export function randomKey(cpuEntropyBits?: number ): Promise<string>;
	```

#### 总结
  最开始自己写*.d.ts,后续发现了居然还有dts-gen这样的工具.回头想想,利用这些工具,真香 ^0^,
#### 关于我
区块链技术痴迷的程序猿一枚，如果你喜欢我的文章，可以加上微信共同学习，共同进步。  
（加好友时，请备注 区块链，谢谢）  
![jungle](/common/wx.png) 

当然也可以通过以下方式联系我  
email_1: <shaokuning@gmail.com>   
email_2: <skunny@163.com>