---
title: 一步一步教你在eos主网上发行自己的tokens
date: 2019-02-23 13:33:56
---

#### 前言
* 2019年来了,祝大家新年快乐
* 新的一年我们得继续努力呢,当然我也得继续来填坑
* 接着上篇文章来,本篇文章会带着你一步一步在测试网(主网要真金白银呢)上发行自己的token
* 本来官网的教程中已经有教怎么样发行自己的token了 [eosio token issue](https://developers.eos.io/eosio-home/docs/token-contract),为什么我还要再写一遍呢?
* 我觉得在我学习的过程中遇到了以下两个问题:
	1. 我没有eosio.token的账号呢,我怎么发行token呢? 
	2. 那我就用自己的account进行发布,那这样的token可以使用吗?
	3. 发布token的合约是怎么样的呢?
* 请自备有足够ram的kylin的账号呢,不然后面没法走哈

#### 获取token源码
* 相信跟着官网走得同学都是在本地的目录中去看源码的,这个源码版本目前只能使用eosiocpp编译,而我目前需要使用cdt来编译呢.
* 所以我们得来这里获取支持cdt的源码[eosio.contracts](https://github.com/EOSIO/eosio.contracts),这里存了系统的的contract源码,我们需要eosio.token的合约源码,我们第一步先把它download下来
* 建议不用修改,除非增加新的业务逻辑

#### token源码分析
* 文中有注释就不啰嗦了,各位同学自己看就好
* 如果有同学看过旧版token的合约,可以发现还有有些变化的.不过我们不用care之前的旧版token合约,使用新版的就好
* token中主要有两个table,account 和 currency_stats  
	 1. account主要负责记录用户的余额,这个table储存余额很有趣. 使用owner作为scope(这里可以理解为在这个就是一个单独的小table),然后在里面找有没有对应的symbol(前提这symbol得通过currency_stats table的检查,即已经create的)
	 2. currency_stats主要负责记录发行的token的信息,主要包括最大发行量,和已经在市面上流通的token数量
* token合约中含有create,issue,retire,transfer,open,close六个action,和get_balance,get_supply两个静态方法.其中创建一个token,只有create, issue, transfer是必须的,其他的可根据需求进行增添
	1. retire 燃烧自己的自己的token,这里token燃烧后总量不会变,相当于又返回给issuer了,如果需要此功能建议减少totalSupply而不是suppy
	2. open和close这两个方法我猜测是为了当issuer token给新账号的时候,可以设置这笔费用,个人觉得没什么鸟用
	3. get_balance和get_supply可以直接获取账户余额和已经issue的token

#### 编译,部署,验证
* 编译,部署
* 购买ram
* craete SHAOKUN token,总量10000个,比较值钱哈
* issue 900个给eostoday1235,然后在retire掉(可以理解为销毁,不要这个action也可以) 
* 后续再转账,都可以在浏览器上查到对应的结果
* ![演示结果](/img_eos1/eos_gif_2.gif)
* [源码](https://github.com/shaokun11/eoslearning/tree/create-token)

#### 总结
* 按照上面的步骤,我们已经发行了一种token,注意是使用我们自己的账号
* 回想一下dispatcher的文章,我们为什么要去判断 code == eosio.token的原因呢? 其实就是确保我们这个token是市面上流通的eos代币,而不是其他某个个人发布的token
* 一个账号可以发行很多token的,只要symbol不一致就可以了,当然,这个账号也可以发行以eos为symbol的token
* 最终再和以太坊的erc20 token对比一下,其实这都是一个官方的标准,我们如果想直接使用完全不用修改的,直接往主网部署就可以了.
* 有时候有新的业务需求,我们完全可以根据自己的需求来更改token内部的某些业务逻辑,毕竟它只是一份部署在你账号的一个智能合约而已,而且这个合约你完全有权限随时更改呢


#### 关于我
区块链技术痴迷的程序猿一枚，如果你喜欢我的文章，可以加上微信共同学习，共同进步。  
（加好友时，请备注 区块链，谢谢）  
![jungle](/common/wx.png) 

当然也可以通过以下方式联系我  
email_1: <shaokuning@gmail.com>   
email_2: <skunny@163.com>