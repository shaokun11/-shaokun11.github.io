---
title: 一步一步教你打造自己的defi-xswap(2) 
date: 2020-10-25 11:22:06
---
#### welcome
> 本篇文章主要看看uniswap的合约的源码和结构,以及如何找到源码的入口(当然也可以直接阅读uniswap的doc来达到目的)

#### 获取源码
从[uniswap合约源码 github](https://github.com/Uniswap/uniswap-v2-core)把源码colne到本地,进入contract目录  

* 可以看到合约的代码量不多,那么我们该如何开始呢?
* 从目录的命名来看,我们貌似只需要关注暴露在外面的三个合约 UniswapV2ERC20.sol, UniswapV2Factory.sol, UniswapV2Pair.sol即可,其他的文件夹的内容为interface和libraries

> 这里如果不熟悉的同学可以试着run一下这个这个test,然后根据test去查看项目如何执行的

```bash
-rw-r--r--  1 kun  staff  3435 10 25 11:14 UniswapV2ERC20.sol
-rw-r--r--  1 kun  staff  1867 10 25 11:14 UniswapV2Factory.sol
-rw-r--r--  1 kun  staff  9788 10 25 11:14 UniswapV2Pair.sol
drwxr-xr-x  7 kun  staff   238 10 25 11:14 interfaces
drwxr-xr-x  5 kun  staff   170 10 25 11:14 libraries
drwxr-xr-x  3 kun  staff   102 10 25 11:14 test

./interfaces:
total 40
-rw-r--r--  1 kun  staff   823 10 25 11:14 IERC20.sol
-rw-r--r--  1 kun  staff   159 10 25 11:14 IUniswapV2Callee.sol
-rw-r--r--  1 kun  staff  1148 10 25 11:14 IUniswapV2ERC20.sol
-rw-r--r--  1 kun  staff   661 10 25 11:14 IUniswapV2Factory.sol
-rw-r--r--  1 kun  staff  2424 10 25 11:14 IUniswapV2Pair.sol

./libraries:
total 24
-rw-r--r--  1 kun  staff  602 10 25 11:14 Math.sol
-rw-r--r--  1 kun  staff  563 10 25 11:14 SafeMath.sol
-rw-r--r--  1 kun  staff  578 10 25 11:14 UQ112x112.sol

./test:
total 8
-rw-r--r--  1 kun  staff  187 10 25 11:14 ERC20.sol
```
#### 试玩uniswap

1. [eth兑换uni](https://rinkeby.etherscan.io/tx/0x4f1551395b26afa5b270c42bf165e9a013db010b3e585bf9419a63e83866081d),从etherscan中可以看到函数签名为**swapExactETHForTokens**   
2. [添加uni->eth的资金池](https://rinkeby.etherscan.io/tx/0x4f1551395b26afa5b270c42bf165e9a013db010b3e585bf9419a63e83866081d),调用的函数签名为**addLiquidityETH**,(中间approve了两次此合约可以转移我钱包中的uni的操作,第二次是重复的)  
3. 那么我们可以从这两个函数开始阅读其合约代码   

![demo](/xswap/ether2.gif)

#### router合约 
1. 在我们试玩找入口时,我们发现其调用的函数签名并没有在我们Colne的项目中有找到,那是怎么回事呢?按照文档,这是uniswap为我们的sdk,我们可以根据这基础的合约创建属于我们的swap(是的,我们正在做这件事),而他们自己也做了一个swap [uniswap合约源码](https://github.com/Uniswap/uniswap-v2-periphery),这个才是他们基于sdk开发的一个应用呢
2. 那我们根据[eth兑换uni](https://rinkeby.etherscan.io/tx/0x4f1551395b26afa5b270c42bf165e9a013db010b3e585bf9419a63e83866081d)这笔交易,找到其在etherscan上验证过的的合约,发现其实际调用的是*UniswapV2Router02*这个函数名的合约,全局搜索找到其函数签名如下  

```solidity
   function swapExactETHForTokens(uint amountOutMin, address[] calldata path, address to, uint deadline)
        external
        virtual
        override
        payable
        ensure(deadline)
        returns (uint[] memory amounts)
    {
        require(path[0] == WETH, 'UniswapV2Router: INVALID_PATH');
        amounts = UniswapV2Library.getAmountsOut(factory, msg.value, path);
        require(amounts[amounts.length - 1] >= amountOutMin, 'UniswapV2Router: INSUFFICIENT_OUTPUT_AMOUNT');
        IWETH(WETH).deposit{value: amounts[0]}();
        assert(IWETH(WETH).transfer(UniswapV2Library.pairFor(factory, path[0], path[1]), amounts[0]));
        _swap(amounts, path, to);
    }
```
3. 先忽略**UniswapV2Library**方法,观察此合约,其核心方法集中在**_swap(amounts, path, to)**这个方法上,继续搜索可以发现很多方法都调用了此方法

```solidity
    // **** SWAP ****
    // requires the initial amount to have already been sent to the first pair
    function _swap(uint[] memory amounts, address[] memory path, address _to) internal virtual {
        for (uint i; i < path.length - 1; i++) {
            (address input, address output) = (path[i], path[i + 1]);
            (address token0,) = UniswapV2Library.sortTokens(input, output);
            uint amountOut = amounts[i + 1];
            (uint amount0Out, uint amount1Out) = input == token0 ? (uint(0), amountOut) : (amountOut, uint(0));
            address to = i < path.length - 2 ? UniswapV2Library.pairFor(factory, output, path[i + 2]) : _to;
            IUniswapV2Pair(UniswapV2Library.pairFor(factory, input, output)).swap(
                amount0Out, amount1Out, to, new bytes(0)
            );
        }
    }
```
4.  发现其最终落在了**IUniswapV2Pair(UniswapV2Library.pairFor(factory, input, output)).swap(
                amount0Out, amount1Out, to, new bytes(0)
            );**这个方法上,这个方法就在我们的clone项目中了,那么其核心方法我们就只要关心**IUniswapV2Pair. swap()**  
            5. 为什么我们colne项目中没有这个router的合约? (本系列结束后回答)  
      
> 只有在etherscan上验证过的合约才能够找到其合约元am,这也是最真实的,如果做defi,一般会将其验证,增加其去中心化的公信力
            
#### 总结
1. 通过一系列的操作,我们总算找到了合约入口啦,接下来应该进入到正式的合约阅读步骤啦,是不是很激动?

2. 其中有些细节我觉得通过文字来描述就有点啰嗦了,然后就跳过了,如果你感觉到其中哪些地方迷糊,可以给我发邮件
            
#### 关于我
区块链程序猿一枚  
**email:skunny@163.com**