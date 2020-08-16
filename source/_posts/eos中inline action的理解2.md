---
title: eos中inline action的使用方式 (下)
date: 2018-12-01 09:34:33
---

#### 前言
 * 关于上一篇的inline action关于作为log的用法,我觉得这是其一,其二是可以action调用action,这样的话应该也算作action的第二种用法了吧  
 * 本篇文章介绍inline action的第三种用户,外部合约的调用  
 * 记住一点,这些调用都是在一个事务中进行的,所以可以放心大胆的使用
 * 由于eos的体系,需要你完全明白eos中 account 和permission level这两个东东的作用,不然理解起来会有点困难

#### 据说
* 听说BM又要离开eos了,我觉得无所谓了,毕竟eos是开源的,很多人维护的
* eos1.5发布了,是不是又得跟上步伐呢?感觉学不动了
* [js4eos介绍](https://mp.weixin.qq.com/s/IovpLgfvcWhu3L3tGbKiZg)这工具可以简化一些工作,喜欢的同学可以试试[js4eos github](https://github.com/itleaks/js4eos) 

#### eos中inline action 结果展示与说明
* 这里官方的例子合约(cpp文件)和账户名字是一样的,让我困扰了一会,这里给大家一个结论就好:一个账户就只能部署一个合约,而这个合约的名字就是get_self(),当你部署合约的时候,就是账号的名字了.在写line action的时候不要弄混了
* 使用shaokun11111部署合约 addressbook
* 使用shaokun11112部署合约 adcounter
* 在addressbook中使用inline action调用adcounter中的count方法
* 这样设置后,多个合约就可以相互调用了
* 关键点,如果要actor shaokun11111能够拥有调用其他合约的权限,必须给shaokun11111账号授予权限eosio.code,这是个系统的权限.


		cleos -u https://api-kylin.eosasia.one:443 set account permission shaokun11111 active '{"threshold":1,"keys":[{"key":"EOS5694dNS19CXwNMgQ3nd8eVG4gnqcihdstnmsChM8AdHNaDCC82","weight":1}],"accounts":[{"permission":{"actor":"shaokun11111","permission":"eosio.code"}}]}' -p shaokun11111@owner
* 关于permission的格式可以参考我以前的文章,注意这里使用的owner权限
* `EOS5694dNS19CXwNMgQ3nd8eVG4gnqcihdstnmsChM8AdHNaDCC82` 这个是原来创建账号的权限的public key,当然按照上面的写法,这样其实可以更改原来账号的active的权限,建议保持原来权限创建的形式就好
* 授权成功后,账号应该有如图中的显示
* ![shaokun](/eospermission/per1.png)
* ![shaokun](/eospermission/per3.gif)

#### eth中合约相互调用
>如果你理解了eos合约的相互调用,那么接下来eth中的合约相互调用你应该很好理解了  
>个人看来两者的机制大同小异,因为eth中是以address中作为id的,而eos中是以account做为id的

* 基于上篇文章的eth合约,我们稍加改造一下
* 新添加了一个合约 Test3,其中我们将Test2的合约地址用来实例化了Service
* 这个Service中需要包含Test2合约中你需要调用的方法的签名(这里关于是怎样找到的可以去看看官方文档,其实  `function notify(string _notification) external;` 就是这个函数的签名,包括方法名字和参数类型,返回值类型hash后 取了四个字节)
* 部署Test3合约,部署成功后再Test3合约中的callNotify调用Test2合约中的notify方法,可以看到Test合约中的内容生效了
* 这里有同学可能有疑问了,你这样的合约岂不是没有安全?拿到Test2合约地址,任何人新建一个合约就可以调用其合约的方法?是的,理论上是这样的.但是,凡事都有个但是哈.还记得我在之前的文章中关于eth中的权限如果验证吗?如果你看了那篇文章,那聪明的你一定知道该怎么做了哈  
* ![shaokun](/eospermission/per4.gif)

#### 源码献上
> `addressbook.cpp`,没什么说的,注意写法,变化的部分如果你认真阅读了应该知道变化在哪里
	#include <eosiolib/eosio.hpp>
	#include <eosiolib/print.hpp>
	
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
	            counter(user,"emplace");
	        } else {
	            addresses.modify(itr, user, [&](auto& row){
	                        row.first_name = first_name;
	                        row.age = age;
	            });
	            send_summary(user, "modify data");
	            counter(user,"modify");
	        }
	    }
	
	    [[eosio::action]]
	    void erase(name user){
	        require_auth(user);
	        address_index addresses(_self, _code.value);
	        auto itr = addresses.find(user.value);
	        eosio_assert(itr != addresses.end(),"record not exist");
	        addresses.erase(itr);
	        // 调用inline action
	        send_summary(user, " erase data");
	        counter(user,"erase");
	    }
	
	    [[eosio::action]]
	    void notify(name user, std::string){
	        // inline action
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
	    
	    // inline action的具体实现
	    void send_summary(name user, std::string msg){
	        action(
	            permission_level{get_self(),"active"_n},
	                get_self(),
	                "notify"_n,
	                std::make_tuple(user, name{user}.to_string() +msg)
	            ).send();
	    }
	    void counter(name user, std::string type){
	        action counter = action(
	            permission_level{get_self(),"active"_n},// 执行这个inline aciton所需要的权限,即本合约调用
	            "shaokun11112"_n,                       // 账号名称
	            "count"_n,                              // action 名称
	            std::make_tuple(user,type)              // 传递的参数
	            );
	        counter.send();
	    }
	};
	EOSIO_DISPATCH(addressbook,(upsert)(erase))

> `adcounter.cpp` 注意其中的权限验证部分,其他没什么好说的

	#include <eosiolib/eosio.hpp>
	
	using namespace eosio;
	
	class [[eosio::contract]] adcounter: public eosio::contract {
	public:
		using contract::contract;
		adcounter(name receiver, name code, datastream<const char*> ds):contract(receiver,code,ds),
																		counts(receiver,code.value){}
	
		void count(name user, std::string type) {
			// 设定这个方法的调用权限
			// 只有 shaokun11111 这个账号部署的合约有这个权限更改这个合约的内容
			require_auth(name("shaokun11111"));
			auto itr = counts.find(user.value);
			if(itr == counts.end()){
				counts.emplace("shaokun11111"_n, [&](auto& row){
					row.key = user;
					row.emplaced = type == "emplace" ? 1 : 0;
					row.modified = type == "modify" ? 1 : 0;
					row.erased = type == "erase" ? 1 : 0;
				});
			} else {
				counts.modify(itr,"shaokun11111"_n,[&](auto& row){
					if (type == "emplace")
					{
						row.emplaced += 1;
					}
					if (type == "modify")
					{
						row.modified += 1;
					}
					if (type == "erase")
					{
						row.erased += 1;
					}
				});
			}
		}
	private:
		struct [[eosio::table]] counter
		{
			name key;
			uint64_t emplaced;
			uint64_t modified;
			uint64_t erased;
	
			uint64_t primary_key() const {return key.value;}
		};
	
		typedef eosio::multi_index<"counter"_n,counter> counter_index;
		counter_index counts;
	};
	EOSIO_DISPATCH(adcounter,(count));
前端代码就不展示了,因为所有的操作都是在addressbook这个合约中进行操作的,adcounter只是展示了合约中的记录  
[本课源码](https://github.com/shaokun11/eosabout/tree/eos-inline-action2)

#### 总结
* 建议各位同学一定要仔细敲一遍,虽然例子简单,但是再复杂的内容也是由这些简单的例子所产生的
* eos inline action的作用已经说完了.这样掌握之后,就可以写很多合约,壮大你的dapp了.inline action很重要的一点 是可以回滚交易,好好利用,发挥意想不到的作用
* 话说还有个deferred action,有时间再和大家分享一下这货是怎使用了
* dapp的开发不仅仅是写合约,目前主要应用还是在web上.所以建议各位同学还是得了解一下js,node,webpack,react,css这些基础的概念和用法吧.不要多熟练,大概了解一下怎么用就好了
* 还有一点,哪位同学知道eos中有类似eth中的event的功能,即合约主动推送信息给我们,我觉得这很重要.我觉得eosjs应该提供类似的功能呢.知道的同学麻烦告知一声

#### 关于我
区块链技术痴迷的程序猿一枚，如果你喜欢我的文章，可以加上微信共同学习，共同进步。  
 
email: <skunny@163.com>