---
title: 一步一步教你打造自己的defi-xswap(6) 
date: 2020-11-08 17:10:49
---
#### welcome
> 本篇文章主要配合前端实现xswap的最后一个功能swap

#### swap公式
- 此公式由 UniswapV2Library提供
-  getAmountOut 即我付出一定量的token,可以得到另一种token的多少,比如我只有100个tokenA,这个就是查看我这个100tokenA可以换取到多少个tokenB
-  getAmountIn, 即我要得到一定量的token,我得付出多少另一种token,比如我想要100个tokenA,那么这个就是查看我要付出多少个tokenB
-  每次交易,均有千分之三fromToken的损失,这部分费用是按照LP的比例分给提供LP的做市商

```javascript
	 function getAmountOut(uint amountIn, uint reserveIn, uint reserveOut) internal pure returns (uint amountOut) {
        require(amountIn > 0, 'UniswapV2Library: INSUFFICIENT_INPUT_AMOUNT');
        require(reserveIn > 0 && reserveOut > 0, 'UniswapV2Library: INSUFFICIENT_LIQUIDITY');
        uint amountInWithFee = amountIn.mul(997);
        uint numerator = amountInWithFee.mul(reserveOut);
        uint denominator = reserveIn.mul(1000).add(amountInWithFee);
        amountOut = numerator / denominator;
    }

    // given an output amount of an asset and pair reserves, returns a required input amount of the other asset
    function getAmountIn(uint amountOut, uint reserveIn, uint reserveOut) internal pure returns (uint amountIn) {
        require(amountOut > 0, 'UniswapV2Library: INSUFFICIENT_OUTPUT_AMOUNT');
        require(reserveIn > 0 && reserveOut > 0, 'UniswapV2Library: INSUFFICIENT_LIQUIDITY');
        uint numerator = reserveIn.mul(amountOut).mul(1000);
        uint denominator = reserveOut.sub(amountOut).mul(997);
        amountIn = (numerator / denominator).add(1);
    }
```

#### demo验证
- [本次演示前端代码](https://github.com/shaokun11/xswap/tree/v4)
- 目前有两步操作,一是转入tokenA到pair中,二是根据上述的算法,动态计算出tokenA转入可以得到的tokenB的数量,调用pair的swap方法将tokenB提到自己的钱包中
- 前端需使用bignumber.js这个lib来计算大数,可以看到和合约的实现是一样的,只有这样才能通过验证  

```javascrip
	import BN from 'bignumber.js'
	function getAmountIn(amountOut: string, reserveIn: string, reserveOut) {
	    return new BN('1000')
	        .times(amountOut)
	        .times(reserveIn)
	        .div(new BN(reserveOut).minus(amountOut).times(997).plus(1))
	        .toFixed()
	}
	
	function getAmountOut(amountIn: string, reserveIn: string, reserveOut: string) {
	    let feeIn = new BN(amountIn).times(997)
	    return feeIn
	        .times(reserveOut)
	        .div(feeIn.plus(new BN(1000).times(reserveIn)))
	        .toFixed()
	}
```

![xswap](/xswap/ether7.gif)  

#### 总结
- xswap到这里已经结束了.
- 本系列只重点着重于合约代码层面的实现,采用react全家桶实现的前端界面(文中未做过多介绍前端代码的实现).
- 个人觉得uniswap的精髓在其整个的设计与其创新,理论上来讲,是真正实现了一个去中心化的交易所的功能
- 目前前端页面只提供了tokenA,与tokenB的操作,且只适用于rinkeby网络,如果各位同学想要进一步发挥,可以完全参考uniswap的实现[uniswap-peripher](https://github.com/Uniswap/uniswap-v2-periphery),[uniswap前端页面](https://github.com/Uniswap/uniswap-interface)自己实现一套属于自己的swap吧

#### 关于我
区块链程序猿一枚  