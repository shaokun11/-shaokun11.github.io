---
title: 结合scatter学习eos dapp开发，看这篇就够了（下）
date: 2018-10-14 09:24:19
---
 
#### eos智能合约开发 c++合约编写
> 这里需要说明的一下，这个例子和官网提供的hello没有多大的区别,仅仅是带你多了解一下如何编写。其中遇到有些概念难以理解，建议还是参考官网的hello 从头看起


	#include <eosiolib/eosio.hpp>  
	#include <eosiolib/print.hpp>
	#include<string>

	using namespace eosio;
	using std::string;

	class todolist : public contract {  //继承 contract

  	public:
      using contract::contract;           
		
      todolist(account_name self):contract(self){}   //构造函数
		// account_name 是eoslib中定义的一种类型
		
      [[eosio::action]]   // 以前的版本使用的 ///abi action ，eoscdt使用这种方式标识，更加的优雅，也就是你要对外开放的方法都加上才会生成在abi中
      void create(account_name author, const uint32_t id, const std::string &description)
      {
      // 记住，这些action方法的返回值只能是void，不然会报错，只有实际做过的人才会知道 ，那你要问我如何读取区块链的内容呢？ 这个问题也困扰了我蛮久，继续看下面
        todo_index todos(_self, _self);
        // 实例化多索引列表，第一个参数是这张表是谁的 code ,即谁有权限去读写它
        // 第二个参数scope，相当于缩小范围去找到这张表
        // _self 是构造函数中的self
        // todo_index 是Multi_index的一个具体的类型，在下面定义的
        
        todos.emplace(author, [&](auto &new_todo) {
            new_todo.id = id;
            new_todo.description = description;
            new_todo.completed = 0;
        });
        // 使用todos这个表的实例，去添加一条todo
        // 第一个参数表示为这条内容的写入付费的account，第二个参数是一个lambda方法，拿到的参数是这个表中的一行的实例，然后赋值对应的数据即可
        
        print("todo#", id, " created");
        // 这里，如果你使用命令行方式 执行，执行成功后会打印这个方法，但是如果是dapp开发，这是看不到的，又有一个疑问，那我们怎么知道执行成功了呢？
      }

      [[eosio::action]]
      void complete(account_name author,const uint32_t id)
      {  
      // 更改todo的状态为完成
	  todo_index todos(_self, _self);
	  
    	  auto todo_look = todos.find(id);
    	  // 是否用find()方法，去查找这条方法对应的实例
    	  eosio_assert(todo_look != todos.end(), "todo does not exit");
		// 这里 如果找不到，todos.end() 会返回 null
    	  todos.modify(todo_look,author,[&](auto &t) {
    		  t.completed = 1;
    	  });

    	  print("todo #" ,id ,"marked as complted");
      }

      [[eosio::action]]
      void destroy(account_name author, const uint32_t id)
      {	
      	  require_auth(author);
      	  // 权限验证，只能够删除自己的todo
      	  
	  	  todo_index todos(_self, author);
    	  auto todo_look = todos.find(id);
    	  todos.erase(todo_look);
    	  print("todo#", id, " destroyed");
      }
  	private:
		// 在eoscdt中 注意加上[[eosio::table]] ，否则abi中不会生成这个table
      struct [[eosio::table]] todo {
    	  uint64_t id;
    	  string description;
    	  uint64_t completed;
		 
    	  uint64_t primary_key() const {
    	  // 这个方法是必须存在的，上面方法能够查找就是因为这个方法的存在，  
    	  // 所以说查找只用传入 id即可，  
    	  // 如果要使用其他的查找方式，得多定义几个查找的方法，最多可以定义16个  
    	  // 如何定义看官网，这里不演示了  
    		  return id;
    	  }
			
    	  EOSLIB_SERIALIZE(todo,(id)(description)(completed));
      };

      typedef eosio::multi_index<N(todo),todo> todo_index;
      // 这里定义了多索引列表的具体实现，N(todo)是将你的account 转换为 uint64_t 类型，第二个参数传入表名字， 定义的具体的类型是todo_index

	};

	EOSIO_ABI(todolist, (create)(complete)(destroy))

#### eos智能合约开发  编译智能合约
>经过以上的步骤，我们编写好了智能合约，这个合约是一个todolist，可以增加todo,可以将todo变为完成的状态，还可以删除todo,只能删除自己的todo

这里编译智能合约，需要下载好eoscdt,如果没有下载好，建议完整安装好eoscdt再继续往下走了  

1. ![contract](/images/contract1.png)    
新建一个todolist文件夹，将上面写好的todolist.cpp放进去    
2.  ![contract](/images/contract2.png)  
打开终端，进入文件夹，输入   

		eosio-cpp -o todolist.wasm todolist.cpp --abigen


3.  ![contract](/images/contract3.png)  
	成功后会在当前文件夹目录下生成 todolist.wasm 和 todolist.abi
	
#### eos智能合约开发 部署智能合约
> 这里部署智能合约我暂时只找到使用 命令行的方式部署，如果哪位同学知道其他的方式部署，请告诉我，谢谢

>合约的部署又得创建钱包导入账户，和我们在scatter的方式一样，只不过这次是在命令行上操作了，那接下来继续走


* ![contract](/images/contract4.png)   
	
		//显示当前的钱包，如果是按照教程来，应该如图显示
		cleos wallet list 
  
* ![contract](/images/contract5.png)  

		// 创建钱包，记住将返回的钱包密匙保存起来
		cleos wallet create —-to-console 
		// 显示钱包
		cleos wallet list 

* ![contract](/images/contract6.png)  

		// 导入我们之前的导入scatter的密匙对的私钥
		// 执行后粘贴 私钥就好
		cleos wallet import

* ![contract](/images/contract6.png)  

		// 导入我们之前的导入scatter的密匙对的私钥
		// 执行后粘贴 私钥就好
		cleos wallet import
		
* ![contract](/images/contract7.png)  

		// 部署合约 eee,注意看提示内容，RAM不够了，怎么办？去买
		// 参数解释
		// -u 指定部署合约的网络
		// set contract  部署合约的命令
		// shaokun11113  部署到哪个账户上，注意 这里这个账户的私钥必须被导入钱包且已在 -u指定网络上进行注册
		// /Users/shaokun/Documents/todolist  abi 和 wasm的 绝对路径 ，建议不要使用相对路径
		//-p 指定权限
		cleos -u http://jungle.cryptolions.io:18888 set contract shaokun11113 /Users/shaokun/Documents/todolist -p shaokun11113@active

* ![contract](/images/contract8.png)  

		// 购买RAM
		// -u 去哪里买
		// system buyram  购买ram的命令
		// shaokun11113  谁去买，谁付钱
		// shaokun11113  谁受益 买来的ram 给谁
		// -k 买多少 kbytes 
		cleos -u http://jungle.cryptolions.io:18888 system buyram shaokun11113 shaokun11113 -k 1024
		
* ![contract](/images/contract9.png)  
	有时候你也许会遇到这种情况 ，那就需要之前创建钱包时的密匙进行解锁钱包
		
* ![contract](/images/contract10.png)  

		// 解锁默认钱包，输入钱包秘钥就好， 可以使用-n 指定解锁的钱包名字
		cleos unlock wallet
		
* ![contract](/images/contract11.png)  

		// 再次部署合约，可以看到结果eosio::setcode eosio::sebabi说明部署成功
		cleos -u http://jungle.cryptolions.io:18888 set contract shaokun11113 /Users/shaokun/Documents/todolist -p shaokun11113@active
		
#### eos智能合约开发 使用scatter.js进行交互
>这里使用 create-react-app 脚手架创建项目，删除无用的文件，启动项目。这里有疑问的请百度

* ![contract](/images/contract12.png) ![contract](/images/contract13.png) 

创建react项目,打开项目，删除无用的文件

* ![contract](/images/contract14.png) 

启动项目，保证项目的正确性

* ![contract](/images/contract15.png) 
		
		// 安装scatter sdk
		// 这里使用的是eosjs1,尝试了多种方式，未找到eosjs2的使用方式，如有知道的同学，还请告知
		npm i -S scatterjs-core  scatterjs-plugin-eosjs

*  ![contract](/images/contract16.png)

改造项目的布局

	render() {
        return (
            <div>

                <div>
                    <button onClick={() => this.createTodo()}>create todo</button>
                    <button onClick={() => this.deleteTodo()}>delete todo</button>
                    <input type="text" onChange={e => {
                        this.setState({
                            deleteId: Number.parseInt(e.target.value)
                        })
                    }}/>

                    <button onClick={() => this.completeTodo()}>complete todo</button>
                    <input type="text" onChange={e => {
                        this.setState({
                            competedId: Number.parseInt(e.target.value)
                        })
                    }}/>
                    <button onClick={() => this.showTodo()}>show todo</button>
                </div>
                <div>
                    <p>below is data</p>
                    <ul>
                        {
                            this.state.rows.map((todo, index) => {
                                return <li key={index}>
                                    <p>id : {todo.id}</p>
                                    <p>description : {todo.description}</p>
                                    <p>completed : {todo.completed}</p>
                                </li>
                            })
                        }
                    </ul>
                </div>
            </div>
        );
    }
* 引入scatter sdk 

		import React, {Component} from 'react';
		import ScatterJS from 'scatterjs-core';
		import ScatterEOS from 'scatterjs-plugin-eosjs';
		import Eos from 'eosjs';
	
		ScatterJS.plugins(new ScatterEOS());
	
		const network = {
	    blockchain: 'eos',
	    protocol: 'http',
	    host: 'jungle.cryptolions.io',
	    port: 18888,
	    chainId: 		'038f4b0fc8ff18a4f0842a8f0564611f6e96e8535901dd45e43ac8691a1c4dca'
		}
		

* 初始化

	
		componentDidMount() {
        	setTimeout(() => {
            	this.init()
        	}, 2000)
    	}

    	init() {
        ScatterJS.scatter.connect('eos').then(connected => {
            if (!connected) return false;
            const scatter = ScatterJS.scatter;

            this.setState({
                scatter
            });
            alert("scatter load success")
        });
    	}	
    	
* ![contract](/images/contract17.png)	
	
	增加 曾 删 改 查的功能
	
		createTodo() {
        this.state.scatter.getIdentity(requiredFields).then(() => {
            const account = this.state.scatter.identity.accounts.find(x => x.blockchain === 'eos');
            const eos = this.state.scatter.eos(network, Eos);
            const transactionPermission = {authorization: [`${account.name}@${account.authority}`]};
            const num = Math.floor(Math.random() * 100000);
            eos.contract(account.name).then(ins => {
                ins.create(account.name, num, "this is " + num, transactionPermission).then(res => {
                    console.log(res)
                })
            })

        }).catch(error => {
            console.error(error);
        });
    	}
	
* ![contract](/images/eos11.gif)	

启动项目，看看演示效果。可以看见与eos网络的交互还是很流畅，速度也很快。到此，基于eos网络开发dapp的流程应该算结束了，谢谢你的阅读。

#### eos智能合约开发 总结
> 本打算下周更新此篇文章，恰好今天原来的计划取消了，那就不拖了，下周有下周的事情呢  
> 这里有几点需要大家注意：

*  一定要有梯子，不然根本无从谈起
*  eos要求硬件比较高，这个需要自己准备了
*  一定要先理解eos中的相关概念，和以太坊还是有很大的区别的
*  一定要先跟着官网的hello走一遍
*  如果哪一步遇到问题了，可以反复翻看这两篇文章，你一定能够成功的

#### eos智能合约开发 源码
2018-11-15 22:18:58更新
[源码送上...]
(https://shaokun11.github.io/2018/11/15/%E5%9F%BA%E4%BA%8Eeos%E8%BF%9E%E6%8E%A5scatter%E5%BC%80%E5%8F%91dapp(%E4%B8%8B)%20%E7%BB%AD)
#### 接下来的计划
* 继续分享智能合约的知识，主要是基于eos,tron,eth三个平台，希望能够帮助到大家
* 分享一些编程语言知识，eth 和 tron 是用solidity写合约，eos是用c++写，前端展示页面是用react写的 

**文章允许转载，但请注明出处，谢谢**

#### 关于我
区块链技术痴迷的程序猿一枚，如果你喜欢我的文章，可以加上微信共同学习，共同进步。 