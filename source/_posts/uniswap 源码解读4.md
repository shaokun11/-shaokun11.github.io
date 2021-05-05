---
title: 一步一步教你打造自己的defi-xswap(4) 
date: 2020-11-02 23:17:23
---
#### welcome
> 本篇文章编写智能合约中的create2的使用,也就是uniswap中的**UniswapV2Factory**合约中的createPair方法.具体的信息可参考[eip1014](https://eips.ethereum.org/EIPS/eip-1014)与[the-magical-world-of-create2](https://blog.ricmoo.com/wisps-the-magical-world-of-create2-5c2177027604).


#### create2的使用方式

```javascript
 		bytes memory bytecode = type(UniswapV2Pair).creationCode;
 		bytes32 salt = keccak256(abi.encodePacked(token0, token1));
       assembly {
            pair := create2(0, add(bytecode, 32), mload(bytecode), salt)
      }
        IUniswapV2Pair(pair).initialize(token0, token1);
```

  - 此处主要注意看以上两行代码,按照字面意思我们需要**UniswapV2Pair**的合约, **token0, token1**组成的salt,最后面再调用UniswapV2Pair的初始化函数,即完成了整个create2的使用,
  - 对的,就是这么简单.那么这样单独抽出一个**UniswapV2Factory**的作用是什么呢?
  - 因为我们是批量创建相同的合约,只是每个合约的里面记录的数据不同而已,所以这里我们还得引入另一个合约函数来配合使用,他就是
  [UniswapV2Library](https://github.com/Uniswap/uniswap-v2-periphery/blob/master/contracts/libraries/UniswapV2Library.sol)中的pairFor方法,此方法可以根据创建的参数返回正确的合约地址.其中factory参数就是上述使用create2的合约地址
  
  ```javascript
    // calculates the CREATE2 address for a pair without making any external calls
    function pairFor(address factory, address tokenA, address tokenB) internal pure returns (address pair) {
        (address token0, address token1) = sortTokens(tokenA, tokenB);
        pair = address(uint(keccak256(abi.encodePacked(
                hex'ff',			// 固定值
                factory,			// 使用create2的合约地址
                keccak256(abi.encodePacked(token0, token1)), // 确定唯一的合约
                hex'96e8ac4277198ff8b6f785478aa9a39f403cb768dd02cbee326c3e7da348845f' // init code hash
            ))));
    }
  ```
  
#### 学以致用
> 此处我们使用create2实现一个功能,记录Apple手机的制作流程,具体使用AppleFactory定义手机的信息,然后把指定具体的工厂来加工生产出Apple,并将数据保存在区块链上


1. 一台手机包括唯一序列号,外观颜色,运行内存大小,以及存储空间,这里加了一个加工次数来记录整个流程,其中核心方法**initialize**为初始化此产品的基本信息,
**processAction**记录加工流程, **updatePlayer**交接给下一个工厂完成继续制造

```javascript
// SPDX-License-Identifier: MIT
pragma solidity ^0.7.0;
pragma experimental ABIEncoderV2;

contract Apple {
    struct Action {
        uint step;
        string from;
        string to;
        uint256 time;
        address player;
    }

    uint256 public id;                      // 序列号
    string public color;                    //颜色
    uint256 public flashMemory;             // 运行内存大小 4g
    uint256 public diskMemory;              //  储存空间大小 256g
    address public player;                  // 可以更新此合约数据的用户
    uint256 public actionCount;             // 总共加工次数
    Action[] public actions;
    address public factory;

    function getApple()
    public
    view
    returns (uint _id, uint _memory, uint _disk, uint _count, address _player, string memory _color){
        _id = id;
        _memory = flashMemory;
        _disk = diskMemory;
        _count = actionCount;
        _player = player;
        _color = color;

    }

    constructor() {
        factory = msg.sender;
    }

    event Trace(address player, string from, string to, uint256 time);
    event PlayerUpdate(address _pre, address newPlayer, uint256 time);

    function initialize(
        address _player,
        uint256 _id,
        uint256 _memory,
        uint256 _disk,
        string memory _color)
    external {
        require(msg.sender == factory, "permission deny");
        player = _player;
        id = _id;
        color = _color;
        flashMemory = _memory;
        diskMemory = _disk;
        _processAction("apple", "init");
    }

    function getActions(uint _start, uint _end) public view returns (Action[] memory atcs){
        uint end = _end + 1 >=  actions.length ? actions.length : _end;
        atcs = new Action[](end - _start);
        for (uint i = _start; i < end; i++) {
            atcs[i - _start] = actions[i];
        }
    }

    modifier onlyPlayer {
        require(player == msg.sender, "permission deny");
        _;
    }

    function updatePlayer(address _player) onlyPlayer external {
        address oldPlayer = player;
        player = _player;
        emit PlayerUpdate(oldPlayer, _player, block.timestamp);
    }

    function _processAction(string memory _from, string memory _to) internal {
        actionCount += 1;
        actions.push(Action({from : _from, to : _to, time : block.timestamp, step : actionCount,player:msg.sender}));
        emit Trace(player, _from, _to, block.timestamp);
    }

    function processAction(string memory _from, string memory _to) onlyPlayer public {
        _processAction(_from, _to);
    }

}

```

2. factory合约应由苹果公司根据实际的生产情况进行管理,并录入基本信息(此处由于测试,任何人都可以创建),此处的**makeApple**即使生产apple, **getApple**可以用来得到生产出来的apple的具体合约地址,

```javascript
// SPDX-License-Identifier: MIT
pragma solidity ^0.7.0;

import "./Apple.sol";

interface IApple {
    function initialize(address _player, uint256 _id, uint256 _mem, uint256 _disk, string memory _c) external;
}

contract AppleFactory {

    address[] public apples;

    mapping(address => bool) public checkApple;
    
    bytes32 public hash;
    
    function getAllApples(uint _start, uint _end) public view returns(address[] memory _apples){
        _apples = new address[](_end - _start);
        for(uint i = _start; i < _end ; i++){
            _apples[i - _start] = apples[i];
        }
    }

    function totalApple() external view returns (uint256) {
        return apples.length;
    }

    event AppleCreated(address maker, uint256 _id, uint256 _memory, uint256 _disk, string _color, address _apple);

    function makeApple(
        uint256 _memory,
        uint256 _disk,
        string memory _color)
    external returns (address apple) {
        uint256 _id = apples.length + 1;
        bytes memory bytecode = type(Apple).creationCode;
        if (hash == bytes32(0)) {
            hash = keccak256(bytecode);
        }
        bytes32 salt = keccak256(abi.encodePacked(_id, _memory, _color));
        assembly {
            apple := create2(0, add(bytecode, 32), mload(bytecode), salt)
        }
        require(checkApple[apple] == false, 'this apple have been made');
        IApple(apple).initialize(msg.sender, _id, _memory, _disk, _color);
        apples.push(apple);
        emit AppleCreated(msg.sender, _id, _memory, _disk, _color, apple);
    }
    /// 可以通过同样的参数计算出对应的合约地址
    function getApple(int256 _id, uint256 _memory, uint256 _disk, string calldata _color)
    external
    view
    returns (address apple){
        apple = address(uint(keccak256(abi.encodePacked(
                hex'ff',
                address(this),
                keccak256(abi.encodePacked(_id, _memory, _disk, _color)),
                hash
            ))));
    }
}
```

![demo](/xswap/ether4.gif)

#### demo演示
- [前端源码](https://github.com/shaokun11/xswap/tree/v2),[合约源码](https://github.com/shaokun11/solidity-env-hardhat/tree/v3)
- 首先制造成产了一台iPhone,填入颜色,内存与储存空间
- 进入加工环节,当执行process时,自动进入到下一个加工环节
- ...
- 跟随者加工流程指导加工生产完成,这样可以记录每一台手机的生产流程,且具体的流程可信
            
#### 总结
1. 可以按照编程语言中的面向对象思想进行理解,create2传入的参数bytecode就是class的字节码,而create2相当于new,而我们又可以根据传入create2加上其factory的地址以及salt恢复找到以这salt为key的合约地址且唯一,而且合约的地址提前可知呢.Ó´
2. 那么按照介绍使用create2,我们需要两个合约,一个template合约(类),一个factory合约(负责new)即可,是不是很简单的?
3. 如果使用create2创建了一个合约,再使用同样的参数进行调用,是不能成功的,除非已创建的合约调用selfdestruct(address)释放掉才行
4. 接下来就可以进入到在第一篇文章中所提到的三个合约中的**UniswapV2Pair**,也即使最后一个合约
5. 其实到这里来说,合约知识中的难点已经讲解的差不多了,剩下的其实是关于uniswap中的创新思想的复习咯

#### 关于我
区块链程序猿一枚  
