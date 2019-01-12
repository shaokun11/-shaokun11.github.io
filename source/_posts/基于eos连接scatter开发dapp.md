---
title: 结合scatter 学习eos dapp开发, 看这篇就够了（上）
date: 2018-10-13 15:14:24
---
 
#### eos安装 方式1
[eos安装1](https://developers.eos.io/eosio-home/docs/setting-up-your-environment)  
这是eos官网推荐的安装方式。

官网推荐docker，可以减少不必要的麻烦 ，而且步骤超级详细，超级详细，要是安装不成功的话，请确认是否使用mac电脑或者官网推荐的其他操作系统，千万不要使用Windows系统来玩eos,因为官方说过不支持的哈。

#### eos安装 方式2

[eos 安装2](https://developers.eos.io/eosio-nodeos/docs/getting-the-code) 
 
	git clone https://github.com/EOSIO/eos --recursive
	cd eos
	./eosio_build.sh
	./eosio_install.sh
	// 从源码编译 准备好梯子很重要
	
#### eoscdt安装  
[eosio.cdt 安装1](https://developers.eos.io/eosio-home/docs/installing-the-contract-development-toolkit) 这里遵循官网的步骤
 
	git clone --recursive https://github.com/eosio/eosio.cdt --branch 	v1.2.1 --single-branch
	cd eosio.cdt 
	./build.sh eos
	sudo ./install.sh
	//这是新的生成wasm 和 abi的工具，以后官方会维护的，之前方式会在后续被移除 
[eosio.cdt 安装2](https://developers.eos.io/eosio-home/docs/installing-the-contract-development-toolkit) 这里是最新的步骤
#### eos安装 方式总结
简单，超级简单。  
但是，受限于我们得环境，自备梯子是成功的第一步，第二步就是要一个适合的硬件，两者缺一不可。

#### eos智能合约开发 开发体会
最好的学习资料就是官方提供的资料了，而且eos官方的资料具详细，在此送上  
[eos官网](https://developers.eos.io/)

安装不是我要说的重点，过程艰辛就不说了，有问题Google或者度娘吧，我这里主要想要与大家分享的是如何使用scatter连接eos的测试网或者主网进行dapp的开发。  
  
上面的环境安装了可以启动本地的节点来进行测试，官网也是如此，而现在我能够搜索到的文章基本上也是基于本地节点使用命令行来进行开发，这样只能保证你学习开发eos的流程，但是具体怎样开发dapp你可能还是不清楚，因为你不知道如何连接scatter，如何与智能合约进行交互，所以这篇文章重点解决的问题是在这里。

这里再区分一下钱包开发和dapp的开发。钱包开发需要用户提供他的私钥给你，或者你创建好给用户。而我们此处所要使用的是官方的钱包scatter来进行开发dapp，不需要用户提供私钥给我们，他只需要接入官方提供的scatter就好，如果你有以太坊的经验，这就是metamask

#### eos智能合约开发 scatter安装

scatter，官方提供的eos钱包，这样用户才会放心把他们的密钥放进去，所以先安装它吧  
scatter是chrome的插件，所以你先得安装它[chrome浏览器下载](https://www.google.com/chrome/)    
接下来把scatter 插件安装上 [scatter Chrome插件](https://chrome.google.com/webstore/search/scatter?hl=zh-CN)  


![scatter](/images/scatter00.png)  
搜索scatter安装就好，我这里是安装好的界面  
* 	![scatter](/images/scatter0.png)     
点击插件的scatter图标弹出如图界面，输入你的钱包密码然后点击生成钱包就好   
* 	![scatter](/images/scatter1.png)   
助记词，抄写下来吧，以后恢复钱包用的   
* 	![scatter](/images/scatter2.png)   
这里我选择skip basic setup，后续再来设置也是可行的  
*  ![scatter](/images/scatter3.png)     
这个界面，点击右上角的⚙图标  
*  ![scatter](/images/scatter4.png)    
这个界面，选择language，设置语言  
*  ![scatter](/images/scatter5.png)  
设置好后，切换回来，成功了  
#### eos智能合约开发 使用scatter生成密匙对
1. 经过以上的步骤，scatter已经安装好了，也就是我们创建了钱包，但是我们的钱包还没有内容的，为什么呢？这里我还是看看官网的这张图吧   
 ![eos钱包关系](/images/eosio2.png)
2. 基于上张图的方式，我们得创建一对密匙，那就按照接下来的方式做吧
	*  ![scatter](/images/scatter6.png)  
	点击密匙对，点击新建
	*  ![scatter](/images/scatter7.png)  
	点击生成密匙对  
	*  ![scatter](/images/scatter8.png)  
	重点，记住这里的步骤不能省略  
	重点，记住这里的步骤不能省略  
	重点，记住这里的步骤不能省略  
	点击   复制  
	*  ![scatter](/images/scatter9.png)    
	新开一个文本文档，点击粘贴，这样就把生成好的密匙对保存下来了，待会要用的  
	*  ![scatter](/images/scatter10.png)  
	为这个密匙对取个名字，随便写，随意几好 
	*  ![scatter](/images/scatter11.png)  
	完成了这些步骤，你应该能够看到上面的信息，如果没有，就再从头来一遍了。  
	这样我们的密匙对就创建好了，但是只是在本地创建好了的啊，怎么连接到测试网络和主网络呢？  
	
#### eos智能合约开发 设置scatter连接 jungle test net 
*  ![scatter net setting](/images/scatter5.png)  
 回到这个界面，选择设置图标，选择
*  ![scatter net setting](/images/scatter12.png)  
这个界面选择网络  
*  ![scatter net setting](/images/scatter13.png)  
这里可以看到已经有一个eos和eth的主网的网络节点，那么我们来把我们的测试网络的节点加入进去，点击右上角的新建  
*  ![scatter net setting](/images/scatter14.png)  
这个界面就是设置网络所需要的一些必要信息，我们一个一个来填  
	1. 由于我们是连接eos网络，所以这里第一项默认不变，下拉框可以选择eth,以后还可以支持tron
	2. 名称： 随便写一个，你自己记得住就好  
	3. 改成http,因为我们测试网络的endpoint就是用的这个协议，照改就好。
	4. 域名或ip地址：jungle.cryptolions.io	
	5. 端口 ：18888
	6. chaninId:  038f4b0fc8ff18a4f0842a8f0564611f6e96e8535901dd45e43ac8691a1c4dca
	7. 如果你对以上填入的这些信息想知道为什么，那么可以补一下rpc协议，不想了解直接填入就好，以后就懂了  
*  ![scatter net setting](/images/scatter15.png)  
按照上述的步骤填完后，信息如上，点击右上角保存  
*  ![scatter net setting](/images/scatter16.png)  
没有问题的话，会看到如上红框的显示内容，代表我们得测试网络已经设置好了

#### eos智能合约开发 在jungle测试网络上创建account
>为什么要创建账户呢？这就是eos的一大特点。BM说，你创建了一个密匙对，谁记得住呢，所以就设计了一个这样的账户系统，这个账户 长12位，只能由a-z12345这些字符组成。简而言之，方便记忆。

* [jungle net account ](http://jungle.cryptolions.io/#home)  
打开jungle测试网络地址
* ![jungle net account ](/images/jungle1.png)  
来到jungle的主页，点击 Create Account  
* ![jungle net account ](/images/jungle2.png)  
	1. 	Account name : 看清楚要求 *小写字母 加上12345，总共12个字符*   
	这里我写作 shaokun11113
	2. owner public key : 将我们用scatter生成的那个密匙对的公钥填入这里
	3. active public key: 将我们用scatter生成的那个密匙对的公钥填入这里
	4. 我这里用一对就好，在主网申请账号的时候建议使用两对密匙对。想要知道为什么？可以看看eos第二版的白皮书
* ![jungle net account ](/images/jungle03.png)  
这次我就这样填写，然后人机验证，通过后点击左下角的create,确保网络质量优良  

**文章允许转载，但请注明出处，谢谢**

**这里我的网络质量太差了。。。先写到这里，，， 请继续关注第二篇**  

#### 关于我
区块链技术痴迷的程序猿一枚，如果你喜欢我的文章，可以加上微信共同学习，共同进步。  
（加好友时，请备注 区块链，谢谢）  
![jungle](/common/wx.png)          
当然也可以通过以下方式联系我  
email_1: <shaokuning@gmail.com>   
email_2: <skunny@163.com>