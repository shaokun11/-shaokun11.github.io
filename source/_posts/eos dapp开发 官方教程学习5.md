---
title: eos dapp开发学习 第四课
date: 2018-11-02 21:02:05
---
#### 前言
>官方又发布新版本了，目前写的合约也部署不上了，新版本的语法变了确实蛮多的，目前仅按照官网的教程把流程过一遍了
> 很幸运，目前的官方的游戏第一次和第二次已经被我玩了，这里也把第二次的游戏流程分享给大家，对于接下来编写代码心里有点底，因为本课主要是智能合约的编写。  
> Elemental Battles是一款单人纸牌游戏，其中包括元素之间的战斗。对手将通过在EOSIO上运行的智能合约来代表。  
![eos](/img_eos1/eos_react5.gif)

#### 学习目标
>在本课中，我们将准备我们的游戏。我们还将在智能合约中构建startgame和playcard动作，并从前端触发。在本课程结束时，您将能够开始玩元素战斗并迈出第一步！  

#### 合约编写
> 这里我暂时无法部署上去，只能先把代码放在这里了。看了上面的游戏玩法，各位同学结合注释应该看得明白了。
> 玩家和ai进行卡片游戏对战，都从各自的待选17张牌中选出四张，每张牌有不同的攻击力。根据攻击力进行判断一次对决的胜负，各有5条命，输一次减少一条命，谁先到达0谁就输了。游戏简单，主要是了解一下智能合约的开发流程   
> 这里并没有成功布置到链上，以后补上了 
[第四课源码](https://github.com/shaokun11/eoslearning/tree/eos-dev4)

`cardgame.hpp`

	#include <eosiolib/eosio.hpp>
	using namespace std;
	class cardgame : public eosio::contract {
	// 继承contract
	  public:
	    //构造方法,并实例化了multi_index表users
	    cardgame( account_name self):contract(self),
	                                _users(self,self),
	                                _seed(self,self){};
	
	    // table和action 必须加上@abi 的注解，这样才能生存对应的abi，
	    // 在eosio.cdt中，使用 [[eosio::action]] 更加简洁 
	    [[eosio::action]]
	    void login(account_name user);
	
	    // 开始游戏，分发手里牌
	    [[eosio::action]]
	    void startgame(account_name user);
	
	    //从手上牌选择卡片进行对战
	    [[eosio::action]]
	    void playcard(account_name user,uint8_t player_card_index);
	
	  private:
	    enum game_status:int8_t
	    {
	      ONGOING = 0,      // 游戏正在进行中
	      PLAYER_WON = 1,   // 游戏已经结束，玩家获得胜利
	      PLAYER_LOST = -1 // 游戏结束， 完结失败
	    };
	    enum card_type : uint8_t  // 卡片的类型 总共五种属性类型,总共17张卡牌
	    {
	      EMPTY = 0,    // 不存在的卡片
	      FIRE = 1,     // 火属性， 克木  攻击力为 1 和 2 的各两张  攻击力为3的一张    总共5张
	      WOOD = 2,     // 木属性， 克水  攻击力为 1 和 2 的各两张  攻击力为3的一张    总共5张
	      WATER = 3,    // 水属性   克火  攻击力为 1 和 2 的各两张  攻击力为3的一张    总共5张
	      NEUTRAL = 4,  // 中立属性       攻击力为3 总共1张 
	      VOID = 5      // 平局属性       共计力为0 总共1张
	    };
	    struct card
	    {
	      uint8_t type;   // 卡片类型
	      uint8_t attack_point; // 卡片的攻击力
	    };
	    typedef uint8_t card_id;
	    const map<card_id,card> card_dict = {
	        {0 , {EMPTY , 0}},
	        {1 , {EMPTY , 1}},
	        {2 , {EMPTY , 1}},
	        {3 , {EMPTY , 2}},
	        {4 , {EMPTY , 2}},
	        {5 , {EMPTY , 3}},
	        {6 , {EMPTY , 1}},
	        {7 , {EMPTY , 1}},
	        {8 , {EMPTY , 2}},
	        {9 , {EMPTY , 2}},
	        {10 , {EMPTY , 3}},
	        {11 , {EMPTY , 1}},
	        {12 , {EMPTY , 1}},
	        {13 , {EMPTY , 2}},
	        {14 , {EMPTY , 2}},
	        {15 , {EMPTY , 3}},
	        {16 , {EMPTY , 3}},
	        {17 , {EMPTY , 0}},
	    };
	    struct game
	    {
	     int8_t status = ONGOING;   // 只要登录 默认游戏正在进行
	     int8_t life_player = 5;    // 游戏玩家 5条生命
	     int8_t ai_player = 5;      // 游戏玩家 5条生命
	     vector<card_id>  deck_player = {1,2,3,4,5,6,7,8,9,10,11,12,13,14,15,16,17}; // 玩家待选的卡牌id
	     vector<card_id> deck_ai = {1,2,3,4,5,6,7,8,9,10,11,12,13,14,15,16,17};     // ai待选的卡牌id
	     vector<card_id> hand_player = {0,0,0,0}; // 玩家的手里牌
	     vector<card_id> hand_ai = {0,0,0,0};     // ai的手里牌
	     card_id selected_card_player = 0;          // 从待选牌中选中的牌
	     card_id selected_card_ai = 0;              // 从待选牌中选中的牌
	    };
	
	  	[[eosio::table]]
	  	struct userinfo
	  	{
	  		account_name name;			//玩家的名字    account_name 是uint64_t的一个别名
	  		uint16_t win_count = 0;		
	  		uint16_t lost_count = 0;
	      game game_data;         // game 
	
	  		auto primary_key() const {return name;}  
	  		//多索引表查找名为primary_key（）的getter函数。这必须使用结构中的第一个字段，编译器将使用它来添加主键。让我们定义这个功能。
	  	};
	    [[eosio::table]]
	    struct seed // 这个是用来生成随机数的，全局储存，只要有人玩就会改变，没办法，eos里只能用table来持久化数据在链上
	    {
	      uint64_t key = 1;
	      uint32_t value = 1;
	
	      auto primary_key(){return key;};
	    }
	
	  	typedef eosio::multi_index<N(userinfo),userinfo> user_index;
	  	//多索引表定义，它有两个参数： 表名 
	  	//						 结构定义了我们打算在多索引表中存储的数据。 
	  	// 新类型名称作为我们的多索引表定义的别名
	  	// 
	  	// 多索引表中的数据由四条信息标识： code（account_name） 
	  	// 							 scop 
	  	// 							 table name
	  	// 							 primary key 
	    typedef multi_index<N(seed),seed> seed_index;
	    user_index _users; //声明表的实例
	    seed_index _seed;
	
	    int random(const int range);
	    void draw_one_card(vector<uint8_t>& deck,vector<uint8_t>& hand);
	};	



`cardgame.cpp` 
 
	#include "cardgame.hpp"
	void cardgame::login(account_name user)
	{
		require_auth(user); 
		//确保这个用户已经被授权
		// 这里我理解了很久，把我的想法记录在这里
		// 这里是调用此合约的account name，即提供私钥的account，这里传入的这个account必须拥有它的active权限，
		// 当然，这里还可以要求有其他的权限，这样就可以给特殊的账号特殊的权限
		auto user_iterator = _users.find(user);  // 查找
		if (user_iterator == _users.end()) {
			user_iterator = _users.emplace(user, [&](auto& new_user){
				new_user.name = user;
			});
		}
		// 这里使用multi_index的find 和 emplace(插入)两种方法，
		// 这些方法的形式都是一样的，记住就好，可以去官网查看详细的方法说明
	}
	
	void cardgame::startgame(account_name name) {
		require_auth(name);
		auto& info = _users.get(name, "user not exist");
		_users.modify(info,name,[&](auto& _to_modify_user){
			game game_data;
			for(uint8_t i = 0; i < 4; i++){
				draw_one_card(game_data.deck_player,game_data.hand_player);
				draw_one_card(game_data.deck_ai,game_data.hand_ai);
			}
			_to_modify_user.game_data = game_data;
		});
	}
	
	void cardgame::playcard(account_name user,uint8_t player_card_index){
		require_auth(user);
		eosio.assert(player_card_index < 4,"invalid hand index"); // 手上牌最多四张
		// 通过user找到数据表中数据
		auto& player = _users.get(user,"User not exist");
		eosio_assert(player.game_data.status == ONGONING,"game have ended"); // 确保本轮游戏没有结束
		eosio_assert(player.game_data.selected_card_player == 0,"the player have selected car in this turn"); // 确保手上没有选牌
		// 修改数据表
		_users.modify(player,user,[&](auto& _to_modify_user){
			game& game_data = _to_modify_user.game_data;
			// 设定选中的卡片id
			game_data.selected_card_player = game_data.hand_player[player_card_index];
			// 将手中卡片对应位置置位empty
			game_data.hand_player[player_card_index] = 0;
		});
	}
	EOSIO_ABI(cardgame,(login)(playcard))

`gameplay.cpp`

	#include "cardgame.hpp"
	int cardgame::random(const int range) {
		auto seed_itr = _seed.begin();
		if (seed_itr  == _seed.end())
		{	// 如果没有随机数据，就用默认值初始化
			seed_itr = _seed.emplace(_self,[&](auto& seed){})
		};
		int prime = 65535;
		// 新值得范围 0 ~ 65535
		auto new_seed_value = (seed_itr.value + now()) % prime;
		_seed.modify(seed_itr,_self,[&](auto& s){
				s.value = new_seed_value;
		});
		// 随机范围  0 ~ range
		int random_res = new_seed_value % range;
		return random_res;
	}
	// 根据待选卡片和手上已有的卡片，随机进行选择新的卡片
	void cardgame::draw_one_card(vector<uint8_t> &deck,vector<uint8_t> &hand){
		// 0 ~ 16
	 	int deck_card_index = random(deck.size());
	 	int first_emprty_slot = -1;
	 	for (int i = 0; i < hand.size; ++i)
	 	{	
	 		auto id = hand[i];
	 		if (card_dict.at(id).type == EMPTY){
	 			first_emprty_slot = i;
	 			break;
	 		}
	 	}
	 	eosio_assert(first_emprty_slot !=- 1, "No empty slot in the players hand");
	 	// 如果手上的牌 有空位，进行随机赋值
	 	hand[first_emprty_slot] = deck[deck_card_index];
	 	// 待选牌中移除掉已经被赋值给手上的牌
	 	deck.erase(deck.began() + deck_card_index);
	 }
#### 关于我
区块链技术痴迷的程序猿一枚，如果你喜欢我的文章，可以加上微信共同学习，共同进步。  
  
email: <skunny@163.com>