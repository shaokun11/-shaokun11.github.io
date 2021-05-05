---
title: eos中inline action的使用方式 (上)
date: 2018-11-25 20:03:05
---

#### 前言
> eos中有两种action,一种是本文要带大家理解使用的inline action,一种deffer action.    
> 关于他们的差别我觉得主要一点就是前者一定是打包在同一个块中的,而且是事务操作.

#### surprise
> * jungle的测试网升级到2.0了,既然升级了,应该有不少的优化吧[jugnle2.0地址](https://monitor.jungletestnet.io/)  
> * 由于在jungle测试网遇到很多坑,所以试了试另外一个测试网[麒麟](https://tools.cryptokylin.io/#/tools/create),用起来很不错,不用带梯子,建议大家在国内使用这个网络
> * 在kylin创建账号和获取测试币,直接在浏览器地址栏敲进去就可以了

	http://faucet.cryptokylin.io/create_account?new_account_name
	http://faucet.cryptokylin.io/get_token?your_account_name
####  action的结果展示
> * 结果和之前的todolist内容差不多,只是多了一些额外的输出内容,而且这也是官方的例子
> * 那为什么还要记录呢?第一是我学习的过程,第二是看看具体的结果,第三是部署到测试网试试
> * 这里我也一些疑问?这样是否会多消耗ram? 如果不消耗,可以作为一些log的使用还是不错的,如果消耗,个人感觉没什么用,毕竟输出的每一个文字都是ram呢
> * 当然,它最主要的作用是可以调用其他的合约的内容,而同时保证在一个事务中,这个下篇文章给大家展示
> * 以太坊中的event有点类似本问所展示的作用,它可以监听,而且这部分log不消耗gas
> * 在交互中,也顺便展示了scatter中如何切换账号的操作

![inline](/eosinline/inline1.gif)

#### cpp源码展示
> 没什么需要多说的,注意inline action的写法

	#include <eosiolib/eosio.hpp>	
	using namespace eosio;
	
	class [[eosio::contract]] addressbook : public eosio::contract {
	public:
	    using contract::contract;
	    addressbook(name reciever, name code, datastream<const char*> ds) : contract(reciever,code,ds){}
	
	    [[eosio::action]]
	    void upsert(name user, std::string first_name, uint64_t age){
	        require_auth(user);
	        address_index addresses(_code,_code.value);
	        auto itr = addresses.find(user.value);
	        if(itr == addresses.end()){
	            addresses.emplace(user,[&](auto& row){
	                row.key = user;
	                row.first_name = first_name;
	                row.age = age;
	            });
	            send_summary(user, "emplace data");
	        } else {
	            addresses.modify(itr, user, [&](auto& row){
	                        row.first_name = first_name;
	                        row.age = age;
	            });
	            send_summary(user, "modify data");
	        }
	    }
	
	    [[eosio::action]]
	    void erase(name user){
	        require_auth(user);
	        address_index addresses(_self, _code.value);
	        auto itr = addresses.find(user.value);
	        eosio_assert(itr != addresses.end(),"record not exist");
	        addresses.erase(itr);
	        send_summary(user, "erase data");
	    }
	
	    [[eosio::action]]
	    void notify(name user, std::string){
	        require_auth(get_self());
	        require_recipient(user);
	    }
	
	private:
	    struct [[eosio::table]] person {
	        name key;
	        std::string first_name;
	        uint64_t age;
	
	        auto primary_key() const {
	            return key.value;
	        };
	    };
	    typedef multi_index<"person"_n, person> address_index;
	 
	    void send_summary(name user, std::string msg){
	        action(
	            permission_level{get_self(),"active"_n},
	                get_self(),
	                "notify"_n,
	                std::make_tuple(user, name{user}.to_string() +msg)
	            ).send();
	    }
	};
	EOSIO_DISPATCH(addressbook,(upsert)(erase))

#### eth中的event的使用
>* eth的event可以当做log使用,而且可以单独监听.这给了极大的方便了给任何一个人或则组织想知道某个合约函数的执行情况,我觉得这是一种非常好的机制.不知道eos中有没有类似的功能.
>* 也就是说合约中的信息可以主动推送给我们
>* 可以看到一个简单的node七八行代码就可以监听任何一个合约的所有事件的功能,前提你得拿到这个合约的地址和abi
>* 这里涉及了较多的以太坊的知识,只是为了向大家展示一下这个功能,不知道eos中目前有不有类似的功能,感觉它这个inline action即本篇文章描述的这个功能有点点类似,但是不知道能不能主动推送给我们呢?   
  
![inline](/eosinline/inline2.gif)

#### scatter遇到的一个小问题
>* 这里遇到了一个很纠结的问题,就是scatter无法弹出来.
>* 如果有遇到的同学,请重新安装scatter那两个lib,再次感谢某位朋友帮忙  

	yarn remove eosjs scatterjs-core scatterjs-plugin-eosjs
	yarn add eosjs scatterjs-core scatterjs-plugin-eosjs
#### 源码献上
>这里的源码包括本篇文章演示所有的合约和js文件,涉及到了eos和eth,各位同学可以选择性使用.

[inline action的使用](https://github.com/shaokun11/eosabout/tree/eos-inline-action)
#### 总结
> * 我为什么喜欢用测试网,而且总是要带梯子? 我觉得测试网与主网的环境操作一模一样,只是换了endpoint而已.如果在测试网能够按照设想运行,那么主网只需要改变一下下即可.而梯子对于我们这行的同学来说,应该是必备的.
> * 而现在用麒麟,不用梯子了哈.这点很重要
> * 由于eos的交易速度很快,所以用测试网来测试完全没有任何问题.
> * 这篇文章只讲解的eos的inline action的一个功能,下篇文章将使用它的更加实用的功能,而不仅仅是一个log的功能哈
> * 最近公司的项目很紧,所以能不能快速把新的知识点分享给大家还是未知数了呢,如果你也对dapp开发感兴趣,可以加个微信一起探讨 ^.^
> * 声明一点,我目前只对dapp的开发有点经验,至于源码分析,个人能力达不到呢.
> * eth和eos虽然实现的机制不一样,但都是区块链啊.所以学习eos的时候,当我有些地方不怎么明白,我会再回去用eth的思想来思考一下这个eos的功能怎么用,目前看来,毕竟eos个人觉得比eth上手难度大很多呢

#### 关于我
区块链技术痴迷的程序猿一枚，如果你喜欢我的文章，可以加上微信共同学习，共同进步。 