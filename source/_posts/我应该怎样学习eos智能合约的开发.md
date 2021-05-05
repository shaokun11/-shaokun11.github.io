---
title: 我应该怎样学习eos智能合约的开发?
date: 2019-06-12 21:19:18
---

#### 前言
* 最近还有许多朋友通过微信加我好友,看来越来越多的人在学习区块链了,希望文章能够帮助到你们

#### 初衷
* 当时eos出来的时候,我才开始学习eos的智能合约开发,那时候我找得到的唯一的学习资料就是官网的hello world了.但是eos的版本(目前1.8了,cdt 1.6的更新得非常快,而官网的文章也没来的及更新,那时候Google也找不到给新手入门的资料,所以我就准备把自己学习的一些经验和资料分享给大家,希望你们能够快速入门,少走弯路

#### 踩过的坑
* 至于第一篇eos的文章,是讲述scatter如何使用,刚开始写文章,怕文字描述得不够清楚,就一步一步截图(现在想想真傻).由于mac的原因,截的图片是高清的,最终导致图片上传上去太大了,基本上一张图就一个视窗了,看起来真揪心.
* 最终想了想,这样还是不行呢,所以又去找了个gif的图片放上去,这样基本上就减少了文章的长度.但是是无声的,虽然还是不够视频来得有感觉,但是我尽量做到详细,大家如果跳过了,可以多看两遍应该比直接看图来得好
* 工具就这样定了,然后就是eosio的版本,由于之前对版本的认知不够,也怪自己的技术弱鸡,这其中就在纠结怎么装好eos,eosio.cdt,现在总结起来其实就是梯子不够高
* ...

#### 你应该做的

0. **不用再看本系列的文章了,所有的文章都已经过时了**
1. 如果你是入门eos的智能合约的开发,官网的教程永远是最好的入门资料--->[eosio developter](https://developers.eos.io/eosio-home/docs) 
2. 官网的例子学习完,你可以参考这位大佬-->[eos智能合约开发教程](https://shimo.im/docs/jt2MhxPYTKI6tWnp/read)的系列文章.这位大佬的文章既有源码分析,也有c++知识,Linux的讲解,还有web前端的知识,更有node后端的分析,甚至还有IPFS的分析,filecoin的玩法.所以你是不应该错过这一些的文章的
3. 现在eos的安装也方便了,docker直接搞定,如果不喜欢走docker,可以来这里看看[eosio github](https://github.com/EOSIO/eos)
4. 小工具,可以完全代替cleos,主要是安装比较简单,如果不跑本地的网络,完全在测试网络上运行,可以一试[The most flexible & powerful command line tool for any developer to interact with an EOSIO chain](https://eosc.app/)
5. 网络选择,测试网国内建议[kylin test](https://kylin.eosx.io/),国外建议[jugle2 testnet ](https://monitor.jungletestnet.io/),梯子足够高的随意选择,如果安装了eos,直接本地运行吧
6. 开发IDE 第一个-->[eos stdudio](https://www.eosstudio.io/),这个是本地版本的,各个操作系统都有,  第二个-->[beosin](https://beosin.com/BEOSIN-IDE/index.html) 类似以太上的remix, 我选择的是vscode
7. 小工具,js操作eos [js4eos]([js4eos](https://github.com/itleaks/js4eos))

#### 总结
eos网络已经运行一年了,然而我的技术却还是没有进步,纠结ing ^_^


