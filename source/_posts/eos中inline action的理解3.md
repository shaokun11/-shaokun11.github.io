---
title: eos中inline action的使用方式 (续)
date: 2018-12-08 11:47:24
---

#### 填坑
 这里填一个一直以来对于我来说的一个坑,也在之前的文章的文章误导了大家关于这个event.   
  
1. 之前我说以太坊区块链会主动通知我们当有数据发生变化,这是经过几天的琢磨发现使用了web3.js库造成的错觉.
2. 也正是由于这样,我没有在eos中找到类似的方法,所以去看看了web3.js的源码,发现它是才用轮询最新区块的内容而进行的事件的匹配然后产生的回调.  
3. [requestmanager.js](https://github.com/ethereum/web3.js/blob/develop/lib/web3/requestmanager.js),如果你感兴趣,可以看看这个文件就大概知道了,更加进一步,你可以发现一些更有趣并且非常熟悉的东西,那就是在[methods.js](https://github.com/ethereum/web3.js/blob/develop/lib/web3/methods/eth.js)这个文件中 
 
4. 这是web3提供的方法,既然这样,那我们是不是不调用web3的方法也可以得到同样的效果呢?答案肯定是可以的.通过以上的两个文件,那我们用node也来写一个监听最新块的方法

		const axios = require('axios')
		const obj = { "jsonrpc": "2.0", "method": "eth_getBlockByNumber", "params": ["latest", true], "id": 1 }
		
		getBlockInfo()
		
		function getBlockInfo() {
		    axios.request('https://mainnet.infura.io/v3/e8947e207ac142519554d382200e663b', {
		        method: 'post',
		        header: {
		            'Content-type': 'application/json'
		        },
		        data: JSON.stringify(obj)
		    }).then(response => response.data).then(res => {
		            console.log(res)
		        })
		}
5. 上面的方式可以监听到最新一个块的所有交易信息,然后根据条件筛选即可达到我们要的目的了.这里注意两点:一是json-rpc的请求格式,二是json-rpc提供的endpoint,我这里使用的是infrua提供的.其实metamask也是使用的这个端点,所以使用web3的时候会让你提供provider,其实就是提供的这个端点

#### 前言
> 填完坑后,给出个结论   

> * 区块链从来不会主动推送消息给各位同学的
> * 所有的数据都是在区块里面的,只要在区块链发生的所有操作,均可以通过查询区块得到结果.
> * 至于web3或者是其他库提供的监听的功能,实际上是封装了这个轮询的过程

> 本篇文章还是属于inline action的范畴.就是如何在合约中进行eos的交易

#### 在合约中交易eos
1. 直接调用eosjs提供的api,这种方式比较简单,也不是我要说的重点(这里使用的是scatter官网提供的这个例子)

		const transactionOptions = { authorization:[`${account.name}@${account.authority}`] };
	
	        eos.transfer(account.name, 'helloworld', '1.0000 EOS', 'memo', transactionOptions).then(trx => {
	            // That's it!
	            console.log(`Transaction ID: ${trx.transaction_id}`);
	        }).catch(error => {
	            console.error(error);
	        });

2. 这里有一种需求,就是在执行某个action的时候,需要预先支付一定费用的eos,当达到某种条件时候,再返回给调用者一定的量的eos.
3. 根据需求我们写出了如下的代码,(注意asset这个类型,必须这样传入,关于它的更详细的用法得去看看api了哈;二是now()这个函数返回当前的时间戳,真实环境中这都是固定的,不能用来产生随机数).
4. 还记得之前的权限操作吗? 如果要在合约中转账,必须给合约添加eosio.code的权限,我这个合约现在部署在shaokun11113中的,那么就要给shaokun11113添加eosio.code权限.
5. 添加了权限之后,我们使用shaokun11112来调用这个合约,可以看到当shaokun11113向shaokun11112转账可以成功,但是shaokun11112向shaokun11113转账是没有权限的.
6. 那么,我们在shaokun11112中给shaokun11113添加eosio.code的权限
7. 添加权限后,可以看到一切转账成功了
8. 那么就实现了在合约中进行transfer eos的操作


		#include <eosiolib/eosio.hpp>
		#include <eosiolib/asset.hpp>	
		using namespace eosio;
		
		class [[eosio::contract]] lucky : eosio::contract {
		
		public:
			using contract::contract;
			lucky(name r,name c, datastream<const char*> ds):contract(r,c,ds){}
	
		[[eosio::action]]
		void play(name player, const asset& quantity) {
			require_auth(player);
			if(now() % 2 == 1){
				action(
			permission_level{get_self(),"active"_n},  //所需要的权限结构
			"eosio.token"_n,						  // 调用的合约名称
			"transfer"_n,							  // 合约的方法
			std::make_tuple(get_self(),player, quantity, std::string("shao kun game")) // 传递的参数
				).send();
			} else {
				action(
				permission_level{player,"active"_n},
				"eosio.token"_n,
				"transfer"_n,
				std::make_tuple(player,get_self(), quantity, std::string("shao kun game"))
				).send();
			}
		};
		};
		EOSIO_DISPATCH(lucky,(play))
![shaokun](/eosinline/inline3.gif)

#### 以太坊中eth使用合约进行交易
* 在以太坊中,如果某个方法要接收eth,必须给这个方法加上payable的关键字即可,合约中可以通过msg.value获取到交易的金额.如果此方法不加上payable而调用时传递了金额,那么调用此方法会失败  
* 如果要在初始化的时候传入eth,必须给构造方法加上payable

		contract Test4 {
	    
	    constructor() public payable {
	        
	    }
	    
	    function withPayable() public payable {
	        if(now % 2 == 0 ){
	            msg.sender.transfer(msg.value * 2);
	        }
	    }
	    
	     function withoutPayable() public {
	         
	     }
		}
![shaokun](/eosinline/inline4.gif)

#### 源码与总结
* [合约transfer](https://github.com/shaokun11/eosabout/tree/eos-transfer)
* 相对于eth来说,个人觉得eos中在合约中转账过于麻烦,这需要用户手动去给合约设置转账权限
* 如果哪位同学有更好的关于转账的方法,请告知一下,个人觉得在合约中进行转账还是很常见的需求
* 官方的eos dice合约现在已经移除了,但是可以在历史中找到[eos dice](https://github.com/EOSIO/eos/blob/v1.0.0/contracts/dice/dice.cpp) 有兴趣的同学可以研究一下

#### 关于我
区块链技术痴迷的程序猿一枚，如果你喜欢我的文章，可以加上微信共同学习，共同进步。  
（加好友时，请备注 区块链，谢谢）  
![jungle](/common/wx.png) 

当然也可以通过以下方式联系我  
email_1: <shaokuning@gmail.com>   
email_2: <skunny@163.com>