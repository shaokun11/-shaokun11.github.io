---
title: eos dapp开发学习 第二课 续
date: 2018-10-21 13:16:25
---
#### 前言
> 到此我这里发现了多个问题，这里列出来，希望帮到遇到同样问题的同学
> 
> 1. 目前eoscdt git已经更新到1.4版本了，其中的结构，语法更改了很多，我这边mac系统是10.12.6，弄了一周还是没办法升级到13，而新的cdt最低需求已经要13了，忧伤。。。 
> 而我从源码编译，总是clone不下来，试过各种方法均失败了。这里提醒一下，最好按照他的系统要求去安装，已经有打包好的二进制文件了 
> 2. 目前的官网的版本使用的eoscdt1.2，而经过上面的方式失败了，我再次尝试了这个，成功了
> 3. 这里如果按照昨天的cpp文件使用eosio-cpp进行编译，是没有问题的。但是无法部署合约，这里需要将table name 由user_info 改为 userinfo   注意 不能改成userInfo,UserInfo等等，即所有的需要写入abi的action和table name必须 仅能包含   
> `.12345abcdefghijklmnopqrstuvwxyz`   
> 这些字符
> 4. 莫名的感觉忧伤而兴奋，忧伤是跟不上新版本的步伐，兴奋的是开发越来越规范，现在可以接着上一篇文章继续写作了
> 5. 带着梯子办事，总会减少很多很多没有必要的问题

#### 编译、部署合约
> 不同的版本有着不同的变化，这里仅以eoscdt1.2进行操作，如果后续使用eoscdt1.3或者使用之前的eosiocpp来编译出来的abi和wasm文件肯定是用不了的

* 更改cardgame.hpp  
![contract](/img_eos1/eos8.png)  

		///@abi action  => [[eosio::action]]
		///@abi table => [[eosio::table]]
		user_info => userinfo
* 再次生成abi和wasm文件，这里要切换到c++文件所在的目录
 
		eosio-cpp -o cardgame.wasm cardgame.cpp --abigen

* 部署合约，这里在任何目录执行都可以，合约的目录输入绝对路径
	
		cleos -u http://jungle.cryptolions.io:18888 set contract shaokun11113 /Users/shaokun/Desktop/eoslearning/cardgame -p shaokun11113@active

* 当部署遇到图中红色的提示时，请注意table,action,class，struct的名字所含字符  
![contract](/img_eos1/eos9.png)   
由之前的结论可以看到，我们合约成功部署了

#### 与前端共同交互合约 
* 这里使用了最新的eosjs,请更新 [eosjs](https://github.com/EOSIO/eosjs)

		npm install eosjs@beta	
  
* 更新后package.json ,请注意eosjs的版本
	
		{
		  "dependencies": {
		    "eosjs": "^20.0.0-beta2",
		    "react": "^16.5.2",
		    "react-dom": "^16.5.2",
		    "react-redux": "^5.0.7",
		    "react-scripts": "2.0.5",
		    "redux": "^4.0.1",
		    "redux-logger": "^3.0.6"
		  },
		}

* 新版的eosjs的引入有更改，请仔细对比
* 由于此处的合约只要是合法的注册到jungle上的account 都可以登录，所以前端框的private key的输入框可以随意输入，这里请注意风险 
*  登录成功后，跳转界面！！！本篇文章的代码[lesson2 code](https://github.com/shaokun11/eoslearning/tree/eos-dev02-update)  
![contract](/img_eos1/eos_react3.gif)  

**文章允许转载，但请注明出处，谢谢**

#### 结尾
>总算完成了之前的坑。^o^   
>中间有很多前端的知识，这不是这个课程讲解的重点，只能靠各位同学自己去补习了。这个系列使用react，redux也是一个很庞大的工程，所以可以简单一点，展示一下交互就好。  
>应该说到此，区块链与前端的交互已经完成，接下来就是充实智能合约的功能了。如果你做到了这一步，建议去[eos c++ api](https://developers.eos.io/eosio-cpp/reference)这里了解一下api，这样对于后面的合约编写会有一个更深层次的掌握

#### 关于我
区块链技术痴迷的程序猿一枚，如果你喜欢我的文章，可以加上微信共同学习，共同进步。  
  
email: <skunny@163.com>