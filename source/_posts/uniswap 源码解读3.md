---
title: 一步一步教你打造自己的defi-xswap(3) 
date: 2020-10-27 20:15:25
---
#### welcome
> 本篇文章主要看看UniswapV2Pair的父类合约UniswapV2ERC20,主要学习一下eip712

#### UniswapV2ERC20
> 此合约如果熟悉ERC20 token的同学一定知道,她其实是标准的erc20合约(其标准的erc20合约的内容此处就不再详细介绍了)  
> 但是相对标准的erc20合约多了一个permit方法,那么这个方法的作用是什么什么呢?其实是eip721的一个实现

```bash
contract UniswapV2ERC20 is IUniswapV2ERC20 {
    using SafeMath for uint;

    string public constant name = 'Uniswap V2';
    string public constant symbol = 'UNI-V2';
    uint8 public constant decimals = 18;
    uint  public totalSupply;
    mapping(address => uint) public balanceOf;
    mapping(address => mapping(address => uint)) public allowance;

    bytes32 public DOMAIN_SEPARATOR;
    // keccak256("Permit(address owner,address spender,uint256 value,uint256 nonce,uint256 deadline)");
    bytes32 public constant PERMIT_TYPEHASH = 0x6e71edae12b1b97f4d1f60370fef10105fa2faae0126114a169c64845d6126c9;
    mapping(address => uint) public nonces;

    event Approval(address indexed owner, address indexed spender, uint value);
    event Transfer(address indexed from, address indexed to, uint value);

    constructor() public {
        uint chainId;
        assembly {
            chainId := chainid
        }
        DOMAIN_SEPARATOR = keccak256(
            abi.encode(
                keccak256('EIP712Domain(string name,string version,uint256 chainId,address verifyingContract)'),
                keccak256(bytes(name)),
                keccak256(bytes('1')),
                chainId,
                address(this)
            )
        );
    }

    function _mint(address to, uint value) internal {
        totalSupply = totalSupply.add(value);
        balanceOf[to] = balanceOf[to].add(value);
        emit Transfer(address(0), to, value);
    }

    function _burn(address from, uint value) internal {
        balanceOf[from] = balanceOf[from].sub(value);
        totalSupply = totalSupply.sub(value);
        emit Transfer(from, address(0), value);
    }

    function _approve(address owner, address spender, uint value) private {
        allowance[owner][spender] = value;
        emit Approval(owner, spender, value);
    }

    function _transfer(address from, address to, uint value) private {
        balanceOf[from] = balanceOf[from].sub(value);
        balanceOf[to] = balanceOf[to].add(value);
        emit Transfer(from, to, value);
    }

    function approve(address spender, uint value) external returns (bool) {
        _approve(msg.sender, spender, value);
        return true;
    }

    function transfer(address to, uint value) external returns (bool) {
        _transfer(msg.sender, to, value);
        return true;
    }

    function transferFrom(address from, address to, uint value) external returns (bool) {
        if (allowance[from][msg.sender] != uint(-1)) {
            allowance[from][msg.sender] = allowance[from][msg.sender].sub(value);
        }
        _transfer(from, to, value);
        return true;
    }

    function permit(address owner, address spender, uint value, uint deadline, uint8 v, bytes32 r, bytes32 s) external {
        require(deadline >= block.timestamp, 'UniswapV2: EXPIRED');
        bytes32 digest = keccak256(
            abi.encodePacked(
                '\x19\x01',
                DOMAIN_SEPARATOR,
                keccak256(abi.encode(PERMIT_TYPEHASH, owner, spender, value, nonces[owner]++, deadline))
            )
        );
        address recoveredAddress = ecrecover(digest, v, r, s);
        require(recoveredAddress != address(0) && recoveredAddress == owner, 'UniswapV2: INVALID_SIGNATURE');
        _approve(owner, spender, value);
    }
}
```
#### UniswapV2ERC20中的eip712

1. [eip712](https://github.com/0xProject/EIPs/blob/master/EIPS/eip-712.md)其实是也是用你的私钥对一段文字进行签名,但是签名的过程中更加语义化,目前metamask已经支持,接下来看看premit中的核心方法

```javascirp
   bytes32 digest = keccak256(
            abi.encodePacked(
                '\x19\x01',
                DOMAIN_SEPARATOR,
                keccak256(abi.encode(PERMIT_TYPEHASH, owner, spender, value, nonces[owner]++, deadline))
            )
        );
```

2. 第一个参数 **\x19\x01** 固定值,为什么选择这个?请参考上面的链接
3. 第二个参数:**DOMAIN_SEPARATOR** 其签名一般如下,固定值,在合约的构造函数中初始化,标志着只是eip712的实现

```bash
 // EIP712Domain(string name,string version,uint256 chainId,address verifyingContract)
  constructor() public {
        uint chainId;
        assembly {
            chainId := chainid
        }
        DOMAIN_SEPARATOR = keccak256(
            abi.encode(
                keccak256('EIP712Domain(string name,string version,uint256 chainId,address verifyingContract)'),
                keccak256(bytes(name)),	
                keccak256(bytes('1')),
                chainId,
                address(this)
            )
        );
    }
```

4. 第三个参数的前段部分 **PERMIT_TYPEHASH**,与DOMAIN_SEPARATOR的含义类似,代表着这个函数的签名
5. 第三个参数的后端部分统一理解为传进来signature对应的数据,此处用v,r,s代替  

#### eip712的简单实现  

- 此处采用的结构为 **keccak256('Test(address owner,uint256 amount,uint256 nonce)')**,此处注意数据类型和字段名中间有一个空格,参数之间的逗号之间没有空格     

- eip712主要功能是能够离线签名,让任意的人可以代替你做同样的事情,相当于授权.此处仅做演示签名后的地址正确性  

- 此处仅测试验证结果成功即增加用户输入的amount 

- [合约源码](https://github.com/shaokun11/solidity-env-hardhat/blob/master/contracts/TestEip712.sol)


```javascript

pragma solidity ^0.7.0;

contract Test {
    bytes32 public DOMAIN_SEPARATOR;
    address public rc;
    mapping(address => uint) public nonces;
    mapping(address => uint) public amounts;
    bytes32 public constant TEST_TYPEHASH = keccak256('Test(address owner,uint256 amount,uint256 nonce)');
    constructor() {
        uint chainId;
        assembly {
            chainId := chainid() // 大于6.0以上是个方法,不是属性
        }
        DOMAIN_SEPARATOR = keccak256(
            abi.encode(
                keccak256('EIP712Domain(string name,string version,uint256 chainId,address verifyingContract)'),
                keccak256(bytes("shaokun")), // 此处一般填写此合约有意义的值
                keccak256(bytes('1')),
                chainId,
                address(this)
            )
        );
    }

    function testEIP712(address owner, uint256 amount, uint8 v, bytes32 r, bytes32 s) external {

        bytes32 digest = keccak256(
            abi.encodePacked('\x19\x01',
            DOMAIN_SEPARATOR,
            keccak256(abi.encode(TEST_TYPEHASH, owner, amount, nonces[owner]++))
            )
        );
        address recoveredAddress = ecrecover(digest, v, r, s);
        require(recoveredAddress != address(0) && recoveredAddress == owner, "account not fit");
        rc = recoveredAddress;
        amounts[owner] += amount;

    }
}

```
#### 流程演示
> 这一周花了点时间用react做了个UI,使用了react + react-material开发UI,web3-react实现与区块链的交互[xswap前端](https://github.com/shaokun11/xswap)

1. 使用remix部署合约在rinkeby测试链上,合约地址**0x9F8C390b7048395d4DeBc7636031aD992115C303** [ethersacn](https://rinkeby.etherscan.io/address/0x9F8C390b7048395d4DeBc7636031aD992115C303)
2. 从链上读出当前签名所用的nonce
3. 根据nonce与amount使用账号进行签名,并发送到链上
4. 当交易执行成功,获取新的nonce和amount,验证结果
5. 再次执行签名,将获取的签名数据(其中如果数量为25,发送将会提醒失败,改为签名的数据19则正常弹出metamask)通过remix发送,等待交易结果后返回页面查询数据已经更改  

![demo](/xswap/ether3.gif)

#### js签名流程
> 目前web3不支持此类签名方式,metamask是支持的,请注意一下签名的请求方式

```javascript
// 两个字段 types与domain
const eip712Obj = {
    types: {
        EIP712Domain: [
            { name: 'name', type: 'string' },
            { name: 'version', type: 'string' },
            { name: 'chainId', type: 'uint256' },
            { name: 'verifyingContract', type: 'address' },
        ],
        // 签名的结构,与合约保持一致
        Test: [
            { name: 'owner', type: 'address' },
            { name: 'amount', type: 'uint256' },
            { name: 'nonce', type: 'uint256' },
        ],
    },
   	
    domain: (chainId: number) => ({
        name: 'shaokun',
        version: '1',
        chainId: chainId,
        verifyingContract: '0x9F8C390b7048395d4DeBc7636031aD992115C303',
    }),
}
		
	// 签名所要的数据
  const data = JSON.stringify({
            types: eip712Obj.types,		
            domain: eip712Obj.domain(web3.chainId!!), // 获取当前的id,并返回与合约一致的数据
            primaryType: 'Test',	// 最开始hash的类型,如果有多个类型,需指定按照合约的顺序的第一个
            // test的数据具体内容
            message: {
                owner: web3.account,
                amount: inputV,
                nonce,
            },
        })


// 此处使用的web3-react,provider 为metamask
 web3.library.provider.request(
            {
                method: 'eth_signTypedData_v4',
                params: [web3.account, data],   // 两个参数,第一个必须为账号地址,第二个
            }).then((result: string) => {
            const signature = result.substring(2)
            const r = '0x' + signature.substring(0, 64)
            const s = '0x' + signature.substring(64, 128)
            const v = parseInt(signature.substring(128, 130), 16)
            // 得到签名结果的r,s,v,发送到链上校验即可
            console.log('---sign result -',{r,s,v})
        })

```

#### 总结
- 关于语义化签名的就介绍到此了,那么uniswap中的permit的作用就是也就是如此,至于怎么使用它是怎么使用的呢,后续再看吧
- UniswapV2ERC20 基础合约就没有什么难点了,其余的方法为标准的erc20 token的方法
- 接下来讲解*UniswapV2Factory*,又是一个开发智能合约很重要的一个知识点呢
            
#### 关于我
区块链程序猿一枚  