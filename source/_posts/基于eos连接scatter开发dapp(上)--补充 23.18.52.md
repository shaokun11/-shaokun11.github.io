---
title: 结合scatter学习eos dapp开发，看这篇就够了（上）--补充  
date: 2018-10-13 23:23:17
---
 
#### eos智能合约开发 在jungle上创建account 续
>之前由于网络的原因，导致中断了，那我们继续接着来。自带梯子最重要

![jungle](/images/jungle03.png)   
打开网站，输入之前的信息如图所示,点击左下角create  
![jungle](/images/jungle4.png)   
创建成功后，会返回这样的信息，看见最下面有绿色的应该就是成功了吧
接下来我们查看一下我们创建的account的信息，回到主页，选择account info，输入刚才创建的account ，我这里输入shaokun11113 ,点击 get  
![jungle](/images/jungle5.png)
![jungle](/images/jungle6.png)  
可以看到返回了我们创建账号的信息:    
 
* balance  什么都没有   
* activity key 和 owner key为我们刚才创建账户的时候输入的key  
* RAM 目前账号的RAM情况，这里可以看到刚创建账号就已经消耗了一部分的RAM,为什么呢？ 因为你的账号信息要占用内存啊，所以说，创建账号得找到一个有eos的账号的用户帮你创建，他要为你支付这部分费用
* net Bandwidth 存了100个eos,目前没有使用
* cpu Bandwidth 存了100个eos，目前没有使用

由于作为dapp开发者，那么我们得准备点eos,那么接下来我们再去获取一点eos吧   
回到主页，选择 faucet，输入 你的账号，如图所示，通过验证，选择send coins 就好  
![jungle](/images/jungle7.png)  
返回的结果如上所示，有兴趣的同学可以仔细看看这些内容。  
那么我们如何知道我们是否收到了eos呢？  
还是回到主页，进入 account info查询就好

![jungle](/images/jungle8.png) 
 
 这里注意一下 balance 那一项就好，可以看到已经有100个eos和100个jungle token了

#### eos智能合约开发 导入Account到scatter中
>经过上一步骤，我们已经创建好了账户，接下来就讲账户导入到scatter中吧。在scatter中，这里叫做identity（身份）

![jungle](/images/scatter17.png)   
进入scatter，选择身份。点击左下角的编辑的icon    
如果你关闭过浏览器，请输入最开始的密码进入就好，如果忘记了，请输入助记词找回，如果丢了，就从头来一遍就好.  
![jungle](/images/scatter18.png)   
这里我们注意一下 账户选项中的内容就好     
![jungle](/images/scatter19.png)   
这里都是选择题，账户的第一项勾选我们创建网络时输入的名字，第二项勾选我们生成密匙对的名字就好，点击导入  
![jungle](/images/scatter23.png)   
如果你看到了这个结果，不好意思，我也不知道你哪一步有问题。你可以从头开始再来一遍就好  
![jungle](/images/scatter20.png) 
当然，我希望的是你看到的这个界面，通常我们选择active账号，然后点击右上角保存按钮   
![jungle](/images/scatter21.png)   
回到这个界面，可以看到我们刚才创建的账户已经导入成功了，这里多一步的设置，把此账户设置为scatter的默认账户，这样以后调用scatter的时候就会默认选择此账户进行智能合约的调用了  勾选左下角的那个小圆圈的icon  
![jungle](/images/scatter22.png) 
我们就可以看到我们账户的信息啦

**文章允许转载，但请注明出处，谢谢**

**这样一点点的敲下来，内容还是蛮多的。本打算一次写完，结果遇到网络的问题，暂停了一会，这篇文章算是对第一篇的补充吧。到目前为止，我们只是创建好了账户，还没有涉及到dapp的开发，接下来的具体怎么开发dapp估计要等到下周了吧。**


#### 关于我
区块链技术痴迷的程序猿一枚，如果你喜欢我的文章，可以加上微信共同学习，共同进步。  
