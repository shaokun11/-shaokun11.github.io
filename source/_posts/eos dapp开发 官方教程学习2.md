---
title: eos dapp开发学习 第二课
date: 2018-10-20 22:11:25
---
##### 学习目标
>我们将学习如何在智能合约中存储玩家信息;使用登录操作。然后我们将开始构建Web前端，添加Login页面。 要使登录页面与智能合约交互，我们将使用eosjs连接到EOSIO区块链。我们使用eosjs库来调用智能合约中的登录操作。接下来，我们将React连接到Redux以存储应用程序状态。最后，我们将所有这些结合起来并添加一个游戏页面，以便我们有一个工作登录屏幕。 在本课程结束时，玩家应该能够登录游戏并进入游戏中的第一个屏幕。


#### step 1 定义表结构
对于元素战斗，我们需要存储每个玩家的游戏进度的详细信息或状态。为此，我们将使用多索引表，这是EOSIO的一个功能。您可以将多索引表视为内存数据库。要使用多索引表，我们需要告诉EOSIO我们要在其中存储哪些数据。 对于游戏，我们需要存储玩家信息，所以让我们在之前创建的cardgame.hpp中创建一个名为user_info的结构。我们要存储：

* the player’s name
* win count (default to 0)
* lost count (default to 0)

* cardgame.hpp（这里传图片得了，看得仔细一点）   
![lessons](/img_eos1/eos5.png)

#### step 2 写入数据到多索引表_user中
> 现在我们可以使用我们的多索引表了。所以让我们存储我们的合同状态。为此，我们将创建第一个操作，即登录操作。

![lessons](/img_eos1/eos6.png)

#### step 3 前端页面组件布局
> 这里还是使用Jungle test net来做，account shaokun11113。  
> 我这里出现了一个问题，就是之前我把eos升级了，但是我的电脑没有升级升级到eoscdt。导致现在能够生成abi和wasm文件，但是部署不到链上去，出现的是如下错误，哪位同学知道可以告诉我，感谢。  
> ![lessons](/img_eos1/eos7.png)

这里卡住我太久了，那部署合约这一步就先跳过，这一篇文章我们先根据官方的步骤先把前端页面展示出来吧！  
![lessons](/img_eos1/eos_react.gif)
主要是精简了官方的图片和ui，这一部分先把功能实现，有心思的同学如果觉得我的这个丑就按照官方的来吧，这里用到了几个react的库，建议你们先了解一下

	 "eosjs": "^16.0.9",   
    "react": "^16.5.2",
    "react-dom": "^16.5.2",
    "react-redux": "^5.0.7",
    "react-scripts": "2.0.5",
    "redux": "^4.0.1",
    "redux-logger": "^3.0.6"

#### step 4 页面交互
由于合约没有部署上，所以没有办法上链进行演示，故此处仅仅是将整个前端的ui交互成功，如果你看过我之前的两篇文章，那么到这里你应该可以理解的。 
![lessons](/img_eos1/eos_react2.gif)

####  结尾
这篇文章写的很不顺，没有完成最关键的上链一步给大家演示，真是抱歉了。那么只能够走思路了。附上本篇文章所用到的源码 [elementel battels dev2](https://github.com/shaokun11/eoslearning/tree/eos-dev02)

**文章允许转载，但请注明出处，谢谢**

#### 关于我
区块链技术痴迷的程序猿一枚，如果你喜欢我的文章，可以加上微信共同学习，共同进步。  
（加好友时，请备注 区块链，谢谢）  
![jungle](/common/wx.png) 

当然也可以通过以下方式联系我  
email_1: <shaokuning@gmail.com>   
email_2: <skunny@163.com>