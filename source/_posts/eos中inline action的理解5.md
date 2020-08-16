---
title: eos的dispatcher的使用(2)
date: 2019-01-19 20:25:13
---

#### 前言
 > 希望阅读本篇文章的同学去看一下上一篇文章,不然思路断了接不上就有点麻烦了    
 
#### 多说两句
 * 话说以太坊的分叉计划又延期了,而此次的升级主要是针对底层的安全机制,所以可见写智能合约的最关键点还是安全第一吧,
 * eos一直被攻击,从未被停止.最近的eos因为一个derferred action的问题的漏洞又被黑客利用了,哎... 受伤的总是开发者
 * 大家写的合约还有一个点,就是安全.这里一个朋友已经开发了一款应用,也已经上线到主网,各种测试也做了,什么逻辑,压力,数据,前端页面的跳转等等... 然而当项目上线后,他找到我对我说,别人可以绕过前端代码直接调用的它的合约...  我只能说,这就是智能合约,你所写的合约是全世界的人(当然,如果你的项目很成功,那么黑客也是由兴趣来光顾的)都可以调用的,除非你做一些权限的验证.在eos上还好,可以通过升级合约来弥补,而且code还不用开源.但是在eth,code is law,code还要开源,所以各位同学想想这个合约安全的重要性.eos底层虽然还有bug,我们无法避免,但是我们还是应该对我们的应用层的逻辑进行严格的测试,这样开发的智能合约才经得住世界人民的考验,你说是不是呢?

#### 自定义dispatcher 方式2
>按照上篇文章的做法,那么我们实现了自己定义apply函数来取代EOSIO_DISPATCHER,但是实现的功能是一模一样的,那我们这样做的目的是什么呢?

* 那么接下来我们继续按照官方的教程走,通过另外一种方式来实现.
* 那么此种方式相对于上一中方式有什么区别呢?

```
This pattern provides more control over security at the expense of maintainability. Utilizing if...else if statements as opposed to a switch inherently provides more granularity.
此模式以可维护性为代价提供了对安全性的更多控制。利用if ... else if语句而不是switch提供更精细的控制。
```
好吧,既然看起来优点这么多,那么我们就看看是怎么样实现的吧

```
extern "C" {
  void apply(uint64_t receiver, uint64_t code, uint64_t action) {
    addressbook _addressbook(receiver);
    if(code==receiver && action==name("upsert").value) {
      execute_action(name(receiver), name(code), &addressbook::upsert );
    }
    else if(code==receiver && action==name("notify").value) {
      execute_action(name(receiver), name(code), &addressbook::notify );
    }
    else if(code==receiver && action==name("erase").value) {
      execute_action(name(receiver), name(code), &addressbook::erase );
    }
    else if(code==name("eosio.token").value && action==name("transfer").value) {
      execute_action(name(receiver), name(code), &addressbook::transfer );
    }
  }
};
```
* 初看和上一篇文章简直是一模一样,就是把switch换成了if...else,看来官方真实说的大实话呢
* 不过,最后一个else多了一个判断,addressbook去调用transfer方法,可是我们没有啊,既然是这样,那我们就依样画葫芦,待会写一个吧
* 第二点,最后一个else判断变成了code == eosio.token,这就是我们上篇文章说的这个code的意义了.而eosio.token是系统的账号,当发生eos或者其他代币时才会调用这个方法呢,那么我猜测一下这个判断是当发生eosio.token执行transfer的时候,将会执行我们合约transfer方法

#### 结果验证
>按照上面说的,我们就按照我们得思路先写一个吧,那么就加一个transfer方法如下.

```
 void transfer(uint64_t receiver, uint64_t code){
    send_summary(name(code), "eosio.token transfer");
  }
```
* 很简单,当发生transfer时,我们出发一个inline action来记录一下数据
* 那我们看下结果,结果报错了???? 
* why? 我们仅仅是转账啊,自己转出去或者别人转进来,现在都失败了

#### 结果分析
*Authorization failure with inline action sent to self*

* 权限不允许给自己发送内联action?
* 这是什么情况?给自己发送内联action还要权限?
* 想想想... 对了,要让自己的合约调用其他合约需要给eosio.code发送权限,那我们接下来试试看看是不是这个原因呢

![shaokun](/img_eos1/eos_react9.gif)

#### 再次验证分析结果
* 使用owner权限,给eosio.code授予active权限
* 执行相互转账
* 查看是否能够出发我们自定义的内联action(经过上一步的操作,和我们之前的经验,它应该是调用了我们得inline action,不过还是得看事实说话呢)
* 去kylin,确实达到了我们的结果,无论转账转出或者转入,那么我们都发送了一个inline action  
![shaokun](/img_eos1/eos_react10.gif)

[本课源码](https://github.com/shaokun11/eosabout/tree/eos-dispatcher-02)  

* 这里说一下这个源码,我已经编译好的abi和wasm文件,各位同学可以直接部署,如果要修改源码,就得自己编译了.不同的编译版本可能会有不同的eos语法,我这里用的是cdt1.3.2
* 还有,部署的时候,最好文件夹名字和主合约的名字一样,这样可以可以避免一些想不到的奇奇怪怪的错误.


#### 结论
* 关于上一篇文章开篇提到的类似以太坊中payable关键字的功能,我想各位同学应该知道怎么实现了吧
* 通过我们目前自己实现apply的方式,不用eosio.code授予权限,我们是可以拒绝或者转出eos的,当然这样的方式达到了目的,但是不够优雅
* 通过结果,我们发现了我们也可以在合约中检查监听到eos的转出和转入,那么我们是否可以通过此做出相应的动作呢?答案是可以的,具体怎么使用我们以后的文章再说
* 那么我们怎么优雅的实现拒绝账户eos的交易呢,我觉得最简单的方法就是调用eosio_assert(1==2);哈哈,^_^
* 问题又来了?如果我不调用inline action,那么是不是我不用授权给eosio.code就可以实现转账呢?答案是可以的,本篇文章主要是为了看到具体的转账结果,所以用了inline action,相当于也给大家复习一下这个权限吧.至于这个不使用inline action的,这个可以交由各位同学自己去实现了哈
* 问题又来了?那我要怎么拿取转账的信息呢?

#### 关于我
区块链技术痴迷的程序猿一枚，如果你喜欢我的文章，可以加上微信共同学习，共同进步。  
   
email: <skunny@163.com>