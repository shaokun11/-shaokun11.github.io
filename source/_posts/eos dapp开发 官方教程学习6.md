---
title: eos dapp开发学习 第四课 续  
date: 2018-11-09 21:48:45
---
#### 前言
>很幸运，总算把新版本的eos和cdt装上了，踩过了很多坑，说多了都是泪。 
>
>* 首先，目前本地我的环境eos1.4 eosio.cdt1.3.2 所以你按照接下来的教程走，那么请核对你的版本
>* 坑1 ==> 如果在原来的账号上部署过类似的table的合约，再次更新合约如果在table中增加修改了字段，能够部署，但是不能够得到得到table的内容和执行action，建议用新的账号进行部署解决
>* 坑2 ==> [helloworld](https://developers.eos.io/eosio-home/docs/your-first-contract)很流畅，一路走下去完美。[table数据储存](https://developers.eos.io/eosio-home/docs/data-persistence)这里不知道知道是我的操作不对还是怎么回事，跟着官网的教程，abi和wasm都能够正常生成，这里合约能够部署上去。但是来了，无法get table，也无法push action。
>* 坑3 ==> 语法变了哈，这里强调一下，支持vector，enume，map 和任何自定义的类型哈，剩下的这个就大家自行琢磨了
>* 坑4 ==> 目前来看，尽量把合code在一个cpp中，不然有想不到的意外等着的哈

#### 合约代码
`cardgame.hpp`

	#pragma once
	#include <eosiolib/eosio.hpp>
	using namespace std;
	
	class [[eosio::contract]]cardgame : public eosio::contract {
	  public:
	    cardgame(eosio::name reciever,eosio::name code,eosio::datastream<const char*> ds )
	                                :contract(reciever,code,ds),
	                                _users(reciever,code.value),
	                                _seed(reciever,code.value){};
	                                
	    [[eosio::action]] void login(eosio::name user);
	    [[eosio::action]] void startgame(eosio::name user);
	    [[eosio::action]] void playcard(eosio::name user,uint8_t player_card_index);
	
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
	
	    const  map<uint8_t,card> card_dict = {
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
	     vector<uint8_t>  deck_player = {1,2,3,4,5,6,7,8,9,10,11,12,13,14,15,16,17}; // 玩家待选的卡牌id
	     vector<uint8_t> deck_ai = {1,2,3,4,5,6,7,8,9,10,11,12,13,14,15,16,17};     // ai待选的卡牌id
	     vector<uint8_t> hand_player = {0,0,0,0}; // 玩家的手里牌
	     vector<uint8_t> hand_ai = {0,0,0,0};     // ai的手里牌
	     uint8_t selected_card_player = 0;          // 从待选牌中选中的牌
	     uint8_t selected_card_ai = 0;              // 从待选牌中选中的牌
	    };
	    
	
	  	struct [[eosio::table]] userinfo
	  	{
	  		eosio::name  name;			
	  		uint16_t win_count = 0;		
	  		uint16_t lost_count = 0;
	      game game_data;         
	
	  		auto primary_key() const {return name.value;}  
	  	};
	    
	    struct [[eosio::table]] seed 
	    {
	      uint64_t key = 1;
	      uint32_t seed_value = 1;
	
	      auto primary_key() const {return key;};
	    };
	
	  	typedef eosio::multi_index<"userinfo"_n,userinfo> user_index;
	    typedef eosio::multi_index<"seed"_n,seed> seed_index;
	    user_index _users; //声明表的实例
	    seed_index _seed;
	
	  int random(const int range);
	  void draw_card(vector<uint8_t>& deck, vector<uint8_t>& hand);
	};

`cardgame.cpp`

	#include "cardgame.hpp"
	
	void cardgame::login(eosio::name user)
	{
		_users.emplace(get_self(),[&](auto&  u){
				u.name = user;
		});
	}
	
	void cardgame::startgame(eosio::name user) {
		auto& itr = _users.get(user.value,"User not exist");
		_users.modify(itr,get_self(),[&](auto& _to_modify_user){
			game game_data;
			for(uint8_t i = 0; i < 4; i++){
				draw_card(game_data.deck_player,game_data.hand_player);
				draw_card(game_data.deck_ai,game_data.hand_ai);
			}
			_to_modify_user.game_data = game_data;
		});
	}
	
	void cardgame::playcard(eosio::name user, uint8_t player_card_index){
		eosio::require_auth(user);
		eosio_assert(player_card_index < 4,"invalid hand index");// 手上牌最多四张
		// 通过user找到数据表中数据
		auto& player = _users.get(user.value,"User not exist");
		eosio_assert(player.game_data.status == ONGOING,"game have ended");
		eosio_assert(player.game_data.selected_card_player == 0,"the player have selected car in this turn");
		// 修改数据表
		_users.modify(player,get_self(),[&](auto& _to_modify_user){
			game& game_data = _to_modify_user.game_data;
			// 设定选中的卡片id
			game_data.selected_card_player = game_data.hand_player[player_card_index];
			// 将手中卡片对应位置置位empty
			game_data.hand_player[player_card_index] = 0;
		});
	};
	int cardgame::random(const int range) {
		auto seed_itr = _seed.begin();
		if (seed_itr  == _seed.end()){	// 如果没有随机数据，就用默认值初始化
			seed_itr = _seed.emplace(get_self(),[&](auto& s){
				s = seed{};
			});
		};
		int prime = 65535;
		int new_seed_value = (seed_itr->seed_value + now()) % prime;
		_seed.modify(seed_itr,get_self(),[&](auto& s){
				s.seed_value = new_seed_value;
		});
		// 随机范围  0 ~ range
		int random_res = new_seed_value % range;
		return random_res;
	};
	void cardgame::draw_card(vector<uint8_t>& deck, vector<uint8_t>& hand) {
		  int deck_card_idx = random(deck.size());
		  int first_empty_slot = -1;
		  for (int i = 0; i <= hand.size(); i++) {
		      auto id = hand[i];
		      if (card_dict.at(id).type == EMPTY) {
		          first_empty_slot = i;
		          break;
		      }
		    }
		  eosio_assert(first_empty_slot != -1, "No empty slot in the player's hand");
		  hand[first_empty_slot] = deck[deck_card_idx];
		  deck.erase(deck.begin() + deck_card_idx);
	 };
	EOSIO_DISPATCH(cardgame,(login)(startgame)(playcard))

* 由于上面的坑，用shaokun11113是无法正确的，所以我用了shaokun11114进行部署  
* 终于能够写数据到链上了![eos](/img_eos1/eos11.png)   
* [更新的合约源码](https://github.com/shaokun11/eoslearning/tree/eos-dve4-update)，这次就没有更新前端的mock数据了，相信聪明的你一定知道怎样调整过来

#### 关于我
区块链技术痴迷的程序猿一枚，如果你喜欢我的文章，可以加上微信共同学习，共同进步