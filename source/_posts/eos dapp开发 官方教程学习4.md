---
title: eos dapp开发学习 第三课
date: 2018-10-27 13:12:11
---
#### 前言
>上一篇文章登录的时候，你如果做了，虽然也未报错，按照结论貌似登录上了，但是，其实是没有登录上的，这里给大家说一声对不起，因为其中的某些参数未完全按照官方的例子敲，所以造成了参数不对。  
>既然上篇文章登录错了，那么这篇文章还是得纠正过来，并且找到错在哪里

#### 教程内容
>在本课中，我们将学习如何从区块链中获取数据。数据通过来自前端的交易进入区块链，我们在第2.2课和第2.3课中查看了这一点。调用操作可以更改状态，该状态存储在多索引表中。此外，此操作记录在区块链中。
#### 教程结果
>首先声明，所有的结果均未完全百分之百按照官方教程走，省略了其中的ui，而只涉及了其中正常开发所必须掌握的内容，如果觉得页面不够美丽，建议你可以按照官方的例子去学，这是地址[elemental battles](https://battles.eos.io/)  
>本次调整了部分逻辑，方便我操作，如果你觉得不合适，可以修改  
>再来强调一下你需要哪些知识才能看懂：
>> * 区块链共识算法，这是区块链的核心
>> * eos 账户,权限，cpu,net,ram等相关知识
>> * eos 本地环境，得安装在电脑上，安装方式自行查找（也就是部署合约用，如果你没有而又想走捷径，这里有个好东西，拿去不谢，[eos4js](https://github.com/itleaks/js4eos)，这个库开源出来才2天，也许后面还有更多类似的优秀的库，你都可以使用）
>> * eoscdt 得安装，去下载，然后下一步就好。（这个东西是用来编译成wasm和abi的，单独抽出来的，建议使用，当前时间我使用的eoscdt1.2，语法是旧的，如果你后续用了高版本，按照这篇文章敲语法会有问题的，但是思想是不变的）
>> * react 这是一个前端的库，为什么要用这个？因为第一，我会，第二，官方也是用的他，第三，用起来很爽，你如果用的别的如vue活着原生来写，只要你把流程搞懂了，也是一样的
>> * react-redux 这东西有点难懂，可能会看得你一脸蒙蔽，和上面一点是配套使用的，数据流的管理方式。简单理解就是数据的传递方式，写起来很复杂，理解起来还是很烧脑。我一时半会也说不完，来这里去看看吧[react-redux](https://cn.redux.js.org/docs/react-redux/)。（我也考虑过不用这个写，这样简单很多，对大家的理解也会有帮助，但是考虑到官方用了，我也就顺便用了） 
>> * c++ 这个东西我也是为了写eos智能合约才开始学的，还好我之前有java的基础，学点基础够我们用了，难点的在它的指针上面，（话说，写合约貌似不怎么用指针了，eos已经帮我们封装的够多了，用得多的是引用），这个就看自己了  
>>* node 基础 够用就好  
>>* 当然 js基础  同样够用就好
>>* 梯子
>>* 系统，反正不能是Windows系统

为了避免上一篇文章犯错，这次先把结果展示了
[eos第三课源码](https://github.com/shaokun11/eoslearning/tree/eos-dev03)
>这里尽量放git图片了，先看图后看文字解释。  

![eos](/img_eos1/eos_react4.gif) 
#### 布局更新
* 更改了采取内置账号和秘钥的方式，直接点击不用输入了
* 当登录成功，去区块链上拿到用户的信息，刷新ui
* 下面的代码已经有注释了

		import {Api, JsonRpc, JsSignatureProvider} from 'eosjs';
		
		async function takeAction(action, account, pk, dataValue) {
		    console.log("takeAction", action, dataValue)
		    const rpc = new JsonRpc(process.env.REACT_APP_EOS_HTTP_ENDPOINT);
		    const signatureProvider = new JsSignatureProvider([pk]);
		    const api = new Api({rpc, signatureProvider, textDecoder: new TextDecoder(), textEncoder: new TextEncoder()});
		    try {
		        return await api.transact({
		            actions: [{
		                account: "shaokun11113",  // 合约的拥有者
		                name: action,             // 执行的合约方法
		                authorization: [{
		                    actor: account,       // 执行合约的账户
		                    permission: 'active', // 执行合约的账户所需要的权限
		                }],
		                data: dataValue,          // 所需要的参数,以json字符串进行传递，注意key为合约中定义的参数的名字，也可以通过abi进行查看
	
	            }]
	        }, {
	            blocksBehind: 3,              // 多少个块后确认
	            expireSeconds: 30,            // 超时时间
	        });
	    } catch (err) {
	        throw(err)
	    }
		}
	
		class ApiService {
	
	    static async getUserByName(username) {
	        const rpc = new JsonRpc(process.env.REACT_APP_EOS_HTTP_ENDPOINT);
	        const res = await rpc.get_table_rows({
	            code: process.env.REACT_APP_EOS_CONTRACT_NAME,             //由于在合约中写死了，这里也可以写死
	            scope: process.env.REACT_APP_EOS_CONTRACT_NAME,             //由于在合约中写死了，这里也可以写死
	            table: "userinfo",                                           //由于在合约中写死了，这里也可以写死
	            json: true,                                                // 默认值 ，可以不填
	            lower_bound: username,                                      // 匹配规则，不填默认返回全部
	        })
	        return res.rows[0];
	    }
	
	    static login({name, key}) {
	        return new Promise((resolve, reject) => {
	            takeAction("login", name, key, {user: name}).then(res => {
	                // 执行成功会返回默认的交易hash
	                console.log("login success", res);
	                resolve();
	            }).catch(err => {
	                console.log(err)
	                reject(err);
	            })
	        });
	    }
		}
	
		export default ApiService;
* 当登录成功显示的组件,这是当跳转后立即获取最新数据。结合上面的代码，就知道了。

	![eos](/img_eos1/eos10.png) 
	
* 上面最后，我使用postman拿取了合约中所有的数据，可以看到，目前的数据储存是两条，当然，你在代码中也可以实现，至于怎么实现，我想你应该知道的。至于上一篇文章错在哪里，你仔细看了上面的代码就知道了。

**文章允许转载，但请注明出处，谢谢**

#### 结尾
> 对个人的总结：  
> 目前分享的经验不是很足，严格算起来，就一个多月而已,还是经过一个外国老哥的建议才开始写的，目前文章也比较粗糙。  
> 但我还是会尽力来记录这些。一来这样记录下来可以让自己对知识把握得更牢靠，二来也可以帮助到他人。  
> 从零开始搭博客，学习markdown语法，搜集图片，上传图片，传动图或者视频，都是一步一步摸索出来，中间坑很多，但是乐趣也在此.

至于后续的文章，尽量跟着官方的教程走，所以到这里，第三课结束了。  
小小的总结一下前三课的内容    

* 第一课  搭环境  
* 第二课  搭ui与合约的编写
* 第二课续 部署，测试合约（合约部署成功，连接合约是失败的）
* 第三课  更改上课的错误，连接合约，增加新的展示界面
			

#### 关于我
区块链技术痴迷的程序猿一枚，如果你喜欢我的文章，可以加上微信共同学习，共同进步