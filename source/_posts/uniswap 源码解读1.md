---
title: 一步一步教你打造自己的defi-xswap(1) 
date: 2020-10-24 18:31:22
---
#### welcome
>[uniswap合约源码 etherscan](https://etherscan.io/address/0x5C69bEe701ef814a2B6a3EDD4B1652CB9cc5aA6f#code)    
>[uniswap合约源码 github](https://github.com/Uniswap/uniswap-v2-core)  
>由于在工作中已经基于uniswap实现了更多玩法的defi,在此把其中的流程再梳理一下  
>本系列文章仅作为uniswap的源码理解以及带你如何实现一个自己的xswap,其他玩法请各位同学自己发挥了

#### 准备工作
1. 编程语言 
 * **solidity**(写以太坊合约的语言),
 * **nodejs** (开发环境)
 * **react**(写web前端)
2. 工具
	* metamask(以太坊钱包)
	* 以太坊账号,由于计划合约部署在rinkeby上,所以得去领一些ether
	* 梯子,这个你懂的

#### 搭建以太坊开发环境
1.	[remix](http://remix.ethereum.org/#optimize=false&version=soljson-v0.5.1+commit.c8a2cb62.js),
2.	[truffle](https://www.trufflesuite.com/),
3.	[waffle](https://getwaffle.io/),
4.	[Hardhat](https://hardhat.org/)
5. [演示用的](https://github.com/shaokun11/solidity-env-hardhat)  (主要是将官方demo改为ts版本)  

>  至于怎么选择,合适自己的就是最好的

#### 代码实现
此部分代码为demo代码,如果你和我选用的是同一个框架,应该和下面是一样的  
  
```solidity
//SPDX-License-Identifier: Unlicense
pragma solidity ^0.7.0;

import "hardhat/console.sol";


contract Greeter {
  string greeting;

  constructor(string memory _greeting) {
    // 这里可以打印console,所以选择了这个框架进行开发
    console.log("Deploying a Greeter with greeting:", _greeting);
    greeting = _greeting;
  }

  function greet() public view returns (string memory) {
    return greeting;
  }

  function setGreeting(string memory _greeting) public {
    console.log("Changing greeting from '%s' to '%s'", greeting, _greeting);
    greeting = _greeting;
  }
}

```

测试脚本

```javascript
const {expect} = require("chai");
//@ts-ignore
import {ethers} from "hardhat";
import chai from "chai";
import {Wallet} from "ethers";
import {deployContract, solidity} from "ethereum-waffle";

describe("Greeter", function () {
	it("Should return the new greeting once it's changed", async function () {
		const Greeter = await ethers.getContractFactory("Greeter");
		const greeter = await Greeter.deploy("Hello, world!");

		await greeter.deployed();
		expect(await greeter.greet()).to.equal("Hello, world!");

		await greeter.setGreeting("Hola, mundo!");
		expect(await greeter.greet()).to.equal("Hola, mundo!");
	});
});

```
#### 环境搭建结果演示
1. 本地开发启动节点
2. 编写测试脚本
![demo](/xswap/ether1.gif)
#### 关于我
区块链技术痴迷的程序猿一枚