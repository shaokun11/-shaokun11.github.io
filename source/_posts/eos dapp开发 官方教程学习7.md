---
title: eos dapp开发学习 第五课 
date: 2018-11-11 19:30:46
---
#### 前言
> * 这次将更新完官方所有的教程，但是不会一一讲解了，因为我也没怎么看懂这个算法，因为这个算法不是我要完成这篇文章的重点，所以感兴趣的同学可以自行探索  
> * 上一篇文章虽然完成了数据上链和读取，但是由于我的大意，有些数据得做调整，具体的可以参考本篇的代码    
> * 更改了前端的显示结构，更加清晰
> * 说说我遇到的坑，以免你犯同样的错误 
 
>> * 坑1==> `void playcard(account_name username, uint8_t player_card_idx);` 这是官方的方式，当我调用方法正确传参的时候，会出现 *player_card_idx*这个参数错误，我采取的办法是重命名为 index可解决，至于原因我猜测是大于12个字符了，而且 _ 应该不允许使用  

>>* 自己挖的坑1 ==> 困扰了我1天，该打。  
> 按照上篇文章的代码，当执行开始的游戏，我的手里牌仅会产生一张，结果是我在定义dict_card的时候，使用copy的方式，所有的类型全部是empty造成  
>>* 自己挖的坑2 ==> 困扰了我2天，该打。   
> 当我写了endgame的方法，怎么执行都不报错，但是数据就是无法更改进able，后面发现没有加入abi_dispatch中。这是个大坑，简直坑死我了  
>>* 自己挖的坑3 ==> 这个是eos issues中看到的
> 当更新合约时，出现这种错误，请使用绝对路径即可   
> ![eos](/img_eos1/eos12.png)

#### 结果展示
> 这里仅做简单展示，想具体玩可以使用去官网玩  
> 登录->开始游戏->选择手里牌和ai进行对比，进行血量的计算->再进行下一轮选牌对比，直至谁的life先到达0则为输  
> 可以使用endgame 结束这场游戏

![eos](/img_eos1/eos_react6.gif)

#### 代码演示
这里只贴了我很仔细写了代码，剩下的代码 计算ai选牌，分数计算是copy官方的

	void cardgame::login(eosio::name user)
	{
		eosio::require_auth(user);
		auto itr = _users.find(user.value);
		if(itr == _users.end()){
			_users.emplace(get_self(),[&](auto&  row){
					row.name = user;
				});
		} 
		// 否则登录成功
	}
	
	void cardgame::startgame(eosio::name user) {
		eosio::require_auth(user);
		auto& itr = _users.get(user.value,"User not exist");
		_users.modify(itr, get_self(), [&](auto& row){
			game game_data;
			for(uint8_t i = 0; i < 4; i++){
				draw_card(game_data.deck_player, game_data.hand_player);
				draw_card(game_data.deck_ai, game_data.hand_ai);
			}
			row.game_data = game_data;
		});
	}
	
	void cardgame::endgame(eosio::name user) {
		eosio::require_auth(user);
		auto& itr = _users.get(user.value,"User not exist");
		_users.modify(itr, get_self(), [&](auto& row){
			row.game_data = game();
		});
	}
#### 总结
* 根据eos官方的elemental battle的dapp开发教程到这里我就简单的结束了，如果你按照第一篇文章来写，应该能够走得到最后，完成编写。
* eosio.cdt又更新到1.4了，估计有新的变化，当你按照别人的教程写合约时，请注意版本
* 上面演示时，可以看到使用eosjs进行与链上的交互很慢，而且有时还会报错，这是因为我的这边的梯子不是很好的原因，所以请不要担心
* 对于我个人来说，我觉得写合约这块:
  1. 先了解一下其基本的术语，比如account，action，permission
  2. 就是增删改查 multi_index，记住这个数据储存的特殊性
  3. 知道系统提供的一些特殊的变量和方法，比如“name”_n 和现在的get_self()
  4. 合约中一些地方权限的验证，这非常重要。
  5. 养成良好的代码风格。
  6. 当能够编写基本的合约后，应该思考一下代码的结构，
  7. 编写dapp的思想,不能按照传统的应用的模式来编写。而应该根据区块链的特性来编写应用，就例如现在在区块链上运行，储存数据都是需要有费用的。
  8. 目前来说，区块链上的由于其特性，相对公平，不容易作假，因为所有的交易历史均可以查询，但前提是你的**智能合约**足够足够安全，而合约毕竟是用代码写的，难免有bug,所以只有多写，多看，多总结了
* 目前eos的教程有很多版本，语法上面都有改变，即使是目前官方的demo，也有很多歌版本。所以你认准你编译合约的版本，不然你会越写越烦。我这个系列经历了eoiocpp eosio.cdt1.2 eosio1.3.2等等，写着也非常不顺，不过好歹也写完了。  

[第五课源码](https://github.com/shaokun11/eoslearning/tree/eos-dve5)
#### 接下来的计划
> 目前我的文章基本上都是比较简单的demo或者概念性的文章，这样看多了也没有意思，我写着也觉得没有意思了。所以接下来准备找一些具有代表性的基于以太坊，波场和eos合约与大家分享了。

#### 关于我
区块链技术痴迷的程序猿一枚，如果你喜欢我的文章，可以加上微信共同学习，共同进步。  
   
email: <skunny@163.com>