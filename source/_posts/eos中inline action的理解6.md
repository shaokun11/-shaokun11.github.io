---
title: eos的dispatcher的使用(3 完结)
date: 2019-01-26 15:08:04
---

#### 前言
经过前面两篇文章的练习,相信各位同学都已经对自定义dispatcher已经有了自己的看法了,那么今天我们继续跟着官方的教程走,把接下来的知识点完整一下

#### 自定义dispatcher 方式3
* 这种方式官方说了,不能用于eosio-cpp生成abi,所以我也不详细去探究了,有兴趣的同学可以自己研究一下.官方最后总结这是最为灵活的一种方式呢

```
This dispatcher places most of the security logic and control inside the action handler, however, cannot use eosio-cpp's ABI generator.
此调度程序将大多数安全逻辑和控件放在操作处理程序中，但是，不能使用eosio-cpp的ABI生成器。 
```
```
extern "C" void apply(uint64_t receiver, uint64_t code, uint64_t action) { 
  switch(action) {
      case name("upsert").value: return upsert(receiver, code);
      case name("notify").value: return notify(receiver, code);
      case name("erase").value: return erase(receiver, code);
  }
}
```

```
To handle these requests, we move security logic that was otherwise in the dispatcher, into the action. This can provide more control, but may introduce redundancy, particularly for larger contracts. In the end, it accomplishes all the same points as the above patterns while allowing more information into the scope (namely code). Having access to code inside your action may be useful in some situations. This pattern provides the most flexibility in both the dispatcher and the action, hence "Fully Flexible."
为了处理这些请求，我们将调度程序中的安全逻辑移动到操作中。这可以提供更多控制，但可能会引入冗余，特别是对于大型合同。最后，它完成了与上述模式相同的所有点，同时允许更多信息进入范围（即代码）。在某些情况下，访问操作中的代码可能很有用。这种模式在调度程序和操作中提供了最大的灵活性，因此“完全灵活”。
```

#### 自定义dispatcher 方式4
* 官方说,上面自定义apply函数的时候,如果action少还是可以接受的,如果很多呢?就编程了体力劳动了,而且有可能出现漏洞.这显然和我们程序猿的做法不一致,所以官方最后给我们推荐了新的方法,功能类似,且减少体力劳动,不得不说还是很贴心的

```
#define EOSIO_DISPATCH_CUSTOM( TYPE, MEMBERS ) \
extern "C" { \
   void apply( uint64_t receiver, uint64_t code, uint64_t action ) { \
      if( code == receiver ) { \
         switch( action ) { \
            EOSIO_DISPATCH_HELPER( TYPE, MEMBERS ) \
         } \
         /* does not allow destructor of thiscontract to run: eosio_exit(0); */ \
      } \
   } \
} \
```

* 就是这么简单,虽然简单,但是我们还是得看看有什么魔法(以下内容为官方的介绍)

1. 一定要使用这个 code == receiver的判断,而这个可以自动允许eosio.token进行transfer的方法呢

```
Checks if code==receiverand that the action is not transfer. If this condition is omitted, you may introduce a vulnerability into your contract. It allows the action through if the code is eosio.token and the action is transfer. This check prevents another contract with a transfer function from exploiting your contract.
``` 

2. 合约的实例化,可以看出和之前的apply方法是一样的,之前是在apply函数中实例化,这里直接传入一个实例

```
Instantiates the TYPE (this would be your standard C++11 class)
```

3. 说内部调用了execute_action方法,和我们之前的设计一样

```
Use the EOSIO_DISPATCH_HELPER macro, which inserts a case for each MEMBER into your switch. Inside this case, it calls execute_action using a pattern very similar to the one demonstrated in the Flexible/Compatible Dispatcher defined above.
```

4. 直接替换就可以了.不用做任何改变  

```
You can then use this macro the same way you would with the provided EOSIO_DISPATCH macro.
```

* 既然这样,这里多了一个EOSIO_DISPATCH_HELPER的宏,我们还是得去源码看看它究竟做了什么我们才能放心呢
* 这里出现了一个小的惊喜,由于我目前的源码都是在eos去找,我居然没有找到EOSIO_DISPATCH_HELPER这个宏,这就尴尬了,转而我转向cdt的源码,总算找到了这个宏

```
// Helper macro for EOSIO_DISPATCH
#define EOSIO_DISPATCH_HELPER( TYPE,  MEMBERS ) \
   BOOST_PP_SEQ_FOR_EACH( EOSIO_DISPATCH_INTERNAL, TYPE, MEMBERS )

```

```
// Helper macro for EOSIO_DISPATCH_INTERNAL
#define EOSIO_DISPATCH_INTERNAL( r, OP, elem ) \
   case eosio::name( BOOST_PP_STRINGIZE(elem) ).value: \
      eosio::execute_action( eosio::name(receiver), eosio::name(code), &OP::elem ); \
      break;
```

```
#define EOSIO_DISPATCH( TYPE, MEMBERS ) \
extern "C" { \
   void apply( uint64_t receiver, uint64_t code, uint64_t action ) { \
      if( code == receiver ) { \
         switch( action ) { \
            EOSIO_DISPATCH_HELPER( TYPE, MEMBERS ) \
         } \
         /* does not allow destructor of thiscontract to run: eosio_exit(0); */ \
      } \
   } \
} \
```
* 这里一下子三个宏了,我们关心的EOSIO_DISPATCH其实内部也是调用了EOSIO_DISPATCH_HELPER,所以这第三种方式定义的宏我觉得没有必要了哈,
* 哎,官方的文档貌似又落后了,不过已经习以为常了呢.

####  Security, Security, Security...
```
Your contract's first line of security begins at your dispatcher. Understanding how the dispatching of actions to your contracts is handled is imperative to limiting exposure to logic inside your actions. Always take great caution when writing a custom dispatcher, and be aware of the security implications of each individual implementation method.
```

* 安全第一,所以应该小心使用apply函数,所以官方最后建议还是直接使用EOSIO_DISPATCHER就可以了,至于原因则如下

```
For simple contracts that only execute internal public actions, the EOSIO_DISPATCH is more than suitable, eliminates cruft and greatly decrease the chance of introducing logical errors.
```

#### 猜想
* 既然使用cdt中的EOSIO_DISPATCH就能够达到需求了,那我们该如何实现前面课程的监听呢?
* 通过查看EOSIO_DISPATCHER的判断,这里是不允许code为eosio.token的的操作的记录的,既然这样那我们改造一下EOSIO_DISPATCHER如下所示

```
#define EOSIO_DISPATCH_CUSTOM( TYPE, MEMBERS ) \
extern "C" { \
   void apply( uint64_t receiver, uint64_t code, uint64_t action ) { \
      if( code == receiver || code == "eosio.token"_n.value && action == "transfer"_n.value) { \
         switch( action ) { \
            EOSIO_DISPATCH_HELPER( TYPE, MEMBERS ) \
         } \
         /* does not allow destructor of thiscontract to run: eosio_exit(0); */ \
      } \
   } \
} \
```

* 那么最终结果和上篇文章的内容是一样的,只是我们也没有自己实现apply函数了,而是借助于helper这个宏帮助我们完成

```
#include <eosiolib/eosio.hpp>
#include <eosiolib/print.hpp>

using namespace eosio;

class [[eosio::contract]] addressbook : public eosio::contract {

public:
  using contract::contract;
  
  addressbook(name receiver, name code,  datastream<const char*> ds): contract(receiver, code, ds) {}

  [[eosio::action]]
  void upsert(name user, std::string first_name, std::string last_name) {
    require_auth(user);
    address_index addresses(_code, _code.value);
    auto iterator = addresses.find(user.value);
    if( iterator == addresses.end() )
    {
      addresses.emplace(user, [&]( auto& row ) {
       row.key = user;
       row.first_name = first_name;
       row.last_name = last_name;
      });
      send_summary(user, " successfully emplaced record to addressbook");
    }
    else {
      std::string changes;
      addresses.modify(iterator, user, [&]( auto& row ) {
        row.key = user;
        row.first_name = first_name;
        row.last_name = last_name;
      });
      send_summary(user, " successfully modified record to addressbook");
    }
  }

  [[eosio::action]]
  void erase(name user) {
    require_auth(user);

    address_index addresses(_self, _code.value);

    auto iterator = addresses.find(user.value);
    eosio_assert(iterator != addresses.end(), "Record does not exist");
    addresses.erase(iterator);
    send_summary(user, " successfully erased record from addressbook");
  }

  void transfer(uint64_t receiver, uint64_t code){
    send_summary(name(code), "eosio.token transfer");
  }

private:
  struct [[eosio::table]] person {
    name key;
    std::string first_name;
    std::string last_name;
    uint64_t primary_key() const { return key.value; }
  };

  void send_summary(name user, std::string message) {
    action(
      permission_level{get_self(),"active"_n},
      get_self(),
      "notify"_n,
      std::make_tuple(user, name{user}.to_string() + message)
    ).send();
  };

  typedef eosio::multi_index<"people"_n, person> address_index;
};

#define EOSIO_DISPATCH_CUSTOM( TYPE, MEMBERS ) \
extern "C" { \
   void apply( uint64_t receiver, uint64_t code, uint64_t action ) { \
      if( code == receiver || code == "eosio.token"_n.value && action == "transfer"_n.value) { \
         switch( action ) { \
            EOSIO_DISPATCH_HELPER( TYPE, MEMBERS ) \
         } \
         /* does not allow destructor of thiscontract to run: eosio_exit(0); */ \
      } \
   } \
} \

EOSIO_DISPATCH_CUSTOM( addressbook, (upsert)(erase)(transfer) )
```

#### 结果验证
* 通过我们自定义的dispatcher,我们已经实现了和上篇文章同样的功能,且不用自己实现apply函数,极大的简化了操作
* 当然,目前的方式我们允许的是eosio.token执行transfer操作,那么我们也可以仿照实现其他特殊合约的记录,比如说eosio.system(用来操作cpu,ram)的或者其他我们自定义的合约

![shaokun](/img_eos1/eos_gif_1.gif)

[本课源码](https://github.com/shaokun11/eosabout/tree/eos-dispatcher-03)

#### 总结
* 上一篇最后一个问题,我们已经记录了当我们账户上eos的交易的数据,但是具体的数据我们还不知道呢?比如说转给谁?谁转的?转了多少?还有memo(这个也很重要的,有时候很多数据的有效性可以通过它来进行验证)
* 第二个问题,我们怎么在链上发我们的自己的代币呢?而且这个代币要符合eos代币的标准标准(这里为什么要抛出这个问题呢?因为我只在本地的环境按照教程走,发过行过代币,但是在测试网,我没有eosio.token的账号,我居然懵逼了,不知道怎么发了),过程很简单,但是留给各位同学自己也思考一下
* 三篇文章下来,可以看到跟着官网走,有好处,权威,也有坑,文档总是落后实际情况,那么我们能做点什么吗?我觉得能的,虽然语法变了,但是核心不变,所以我们掌握它的基础性内容,变来变去我们只需要去补一点它变化的内容就好了,如果你之前有基础,这应该就是手到擒来的事情了,只是,看我们还能否学得动哈^_^

#### 啰嗦一句
* 莫名其妙,就叫收拾东西走人了,具体原因都没有,寒冬的果实也落在了我的头上了...(ps:猜测是俺站错了队吧,整个团队一起走人了...)
* 换个方向想,何尝不是又给了我们人生中多一次选择的机会呢?
* 这应该是年前的最后一篇文章了,希望年后能够继续给大家分享一些dapp开发的知识,谢谢你的阅读

#### 关于我
区块链技术痴迷的程序猿一枚，如果你喜欢我的文章，可以加上微信共同学习，共同进步。  
email: <skunny@163.com>