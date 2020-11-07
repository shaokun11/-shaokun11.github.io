---
title: 一步一步教你打造自己的defi-xswap(5) 
date: 2020-11-07 19:05:23
---
#### welcome
> 本篇文章主要介绍UniswapV2Pair中的以及一个名词,三个方法,四个变量,理解完后,实现xswap的add,remove的功能

#### 一个名词

- liquidity (LP)流动性,即pair的token的数量,代表着你所有这个pair价值的量,其价值为你的LP占总LP的百分比乘以pair的总价值

#### 四个变量

```
    address public token0;			//	交易对中的token0 的地址
    address public token1;			// 交易对中的token1 的地址
    uint112 private reserve0;    // 交易对地址中的 token0的数量,即token0的价值    
    uint112 private reserve1; 	// 交易对地址中的 token1的数量 ,即token1的价值
```

#### 三个方法
- mint 即生产LP,其规则根据你向这个pair中按照**固定比例(第一笔设定比例)**转入一定数量token0,token1.当然你可以不按照这个比例转入,但是你得到的LP是按照这个比例中的小的给你算的,所以不要做这样的事呢

```javascrip
    function mint(address to) external lock returns (uint liquidity) {
        (uint112 _reserve0, uint112 _reserve1,) = getReserves(); // gas savings
        uint balance0 = IERC20(token0).balanceOf(address(this));
        uint balance1 = IERC20(token1).balanceOf(address(this));
        uint amount0 = balance0.sub(_reserve0);
        uint amount1 = balance1.sub(_reserve1);
        bool feeOn = _mintFee(_reserve0, _reserve1);
        uint _totalSupply = totalSupply; // gas savings, must be defined here since totalSupply can update in _mintFee
        if (_totalSupply == 0) {
            liquidity = Math.sqrt(amount0.mul(amount1)).sub(MINIMUM_LIQUIDITY);
           _mint(address(0), MINIMUM_LIQUIDITY); // permanently lock the first MINIMUM_LIQUIDITY tokens
        } else {
            liquidity = Math.min(amount0.mul(_totalSupply) / _reserve0, amount1.mul(_totalSupply) / _reserve1);
        }
        require(liquidity > 0, 'UniswapV2: INSUFFICIENT_LIQUIDITY_MINTED');
        _mint(to, liquidity);

        _update(balance0, balance1, _reserve0, _reserve1);
        if (feeOn) kLast = uint(reserve0).mul(reserve1); // reserve0 and reserve1 are up-to-date
        emit Mint(msg.sender, amount0, amount1);
    }

```

- burn 即销毁你转入的LP,即你先把LP转入pair合约中,然后根据你转入的LP数量按照占总量LP的比例返回pair中token0,token1到你指定的地址
``` javascript
  function burn(address to) external lock returns (uint amount0, uint amount1) {
        (uint112 _reserve0, uint112 _reserve1,) = getReserves(); // gas savings
        address _token0 = token0;                                // gas savings
        address _token1 = token1;                                // gas savings
        uint balance0 = IERC20(_token0).balanceOf(address(this));
        uint balance1 = IERC20(_token1).balanceOf(address(this));
        uint liquidity = balanceOf[address(this)];
        bool feeOn = _mintFee(_reserve0, _reserve1);
        uint _totalSupply = totalSupply; // gas savings, must be defined here since totalSupply can update in _mintFee
        amount0 = liquidity.mul(balance0) / _totalSupply; // using balances ensures pro-rata distribution
        amount1 = liquidity.mul(balance1) / _totalSupply; // using balances ensures pro-rata distribution
        require(amount0 > 0 && amount1 > 0, 'UniswapV2: INSUFFICIENT_LIQUIDITY_BURNED');
        _burn(address(this), liquidity);
        _safeTransfer(_token0, to, amount0);
        _safeTransfer(_token1, to, amount1);
        balance0 = IERC20(_token0).balanceOf(address(this));
        balance1 = IERC20(_token1).balanceOf(address(this));

        _update(balance0, balance1, _reserve0, _reserve1);
        if (feeOn) kLast = uint(reserve0).mul(reserve1); // reserve0 and reserve1 are up-to-date
        emit Burn(msg.sender, amount0, amount1, to);
    }
```

- swap 即使用token0交换token1或者相反交易.注意一下,swap是不会改变pair的LP的量.这个呢会改变当前两种token对应的价格.所以一个token的价格也即是通过这种方式来定义的呢!
- 这里具体的操作呢?这里设计得非常巧妙!
- *function swap(uint amount0Out, uint amount1Out, address to, bytes calldata data) external lock * 函数签名中,amount0Out与amount1Out一定有一个值为0,所以首先得转入你需要交易的卖掉的token,如果是token0,那么调用此方法的第二个参数即amount1Out设置为大于0的值即可得到你想要的另一种token,即完成了一次交易,至于这个值填多少? 值为理论转出的99.7%

```javascript
    function swap(uint amount0Out, uint amount1Out, address to, bytes calldata data) external lock {
        require(amount0Out > 0 || amount1Out > 0, 'UniswapV2: INSUFFICIENT_OUTPUT_AMOUNT');
        (uint112 _reserve0, uint112 _reserve1,) = getReserves(); // gas savings
        require(amount0Out < _reserve0 && amount1Out < _reserve1, 'UniswapV2: INSUFFICIENT_LIQUIDITY');

        uint balance0;
        uint balance1;
        { // scope for _token{0,1}, avoids stack too deep errors
        address _token0 = token0;
        address _token1 = token1;
        require(to != _token0 && to != _token1, 'UniswapV2: INVALID_TO');
        if (amount0Out > 0) _safeTransfer(_token0, to, amount0Out); // optimistically transfer tokens
        if (amount1Out > 0) _safeTransfer(_token1, to, amount1Out); // optimistically transfer tokens
        if (data.length > 0) IUniswapV2Callee(to).uniswapV2Call(msg.sender, amount0Out, amount1Out, data);
        balance0 = IERC20(_token0).balanceOf(address(this));
        balance1 = IERC20(_token1).balanceOf(address(this));
        }
        uint amount0In = balance0 > _reserve0 - amount0Out ? balance0 - (_reserve0 - amount0Out) : 0;
        uint amount1In = balance1 > _reserve1 - amount1Out ? balance1 - (_reserve1 - amount1Out) : 0;
        require(amount0In > 0 || amount1In > 0, 'UniswapV2: INSUFFICIENT_INPUT_AMOUNT');
        { // scope for reserve{0,1}Adjusted, avoids stack too deep errors
        uint balance0Adjusted = balance0.mul(1000).sub(amount0In.mul(3));
        uint balance1Adjusted = balance1.mul(1000).sub(amount1In.mul(3));
        require(balance0Adjusted.mul(balance1Adjusted) >= uint(_reserve0).mul(_reserve1).mul(1000**2), 'UniswapV2: K');
        }

        _update(balance0, balance1, _reserve0, _reserve1);
        emit Swap(msg.sender, amount0In, amount1In, amount0Out, amount1Out, to);
    }
```
#### demo验证
- [本次演示前端代码](https://github.com/shaokun11/xswap/tree/v3)
-  相关合约地址: factory:**0xfed46C22C2C39126F3312fBe739a0D9cD29AD1D1** tokenA: **0x425CBf6caF564FfFe1629ACADBD17Ee421B1f6aE** tokenB: **0x7A759B6cb32Ba3098530909aD5178dc6125A2c26**
- 由于执行添加或者swap需要一对token,所以顺带创建了2个测试token,[xtoken模板](https://github.com/shaokun11/solidity-env-hardhat/blob/v4/contracts/XToken.sol)提供了*faucet*方法,每次调用将会空投100个token到操作账户的地址
- 这里我初始定价为 2 tokenA = 1 tokenB,
- 使用faucet方法分别得到一定数量的tokenA tokenB(这里只演示了获取TokenA)
- add 转账tokenA 都pair合约中,转账tokenB到pair合约中,换取LP到自己的账户 (这里一共执行了三次合约)
- remove 转账LP到pair合约中,再根据转进去的LP按照总的比例取出换回来的tokenA,tokenB   
![xswap](/xswap/ether5.gif)  

#### 总结
- 注意一下,本系列所有的合约仅部署在了rinkeby测试链上,如果需要源码演示请切换网络才
- 此合约部署只需要[UniswapV2Factory.sol](https://github.com/Uniswap/uniswap-v2-core/blob/master/contracts/UniswapV2Factory.sol)这一个合约即可,这里我flatten一下这个合约,各位同学可以直接使用[faltten UniswapV2Factory.sol](https://github.com/shaokun11/solidity-env-hardhat/blob/v4/contracts/XSwapFactory.sol)
- 看完pari这个合约,合约代码方面的技术难题了,本篇文章只实现了add的功能,下一篇实现其最后一个功能swap

#### 关于我
区块链程序猿一枚  
**email:skunny@163.com**