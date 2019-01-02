# ONTO Scan

## 概述

本文用于指导DApp方如何接入ONTO，并使用扫码登陆，扫码调用智能合约等服务。
流程中涉及到的参与方包括：

* DApp方：对ONT生态内的用户提供dApp，是本体生态中重要的组成部分。
* ONTO：第一个实现daApi mobile规范的综合客户端。

## 交互流程说明

### 登录交互流程


![Login](./img/login.png)

- 1 dApp web端向dApp server请求登录数据。
- 2 dapp web根据请求回来的登录数据生成二维码（[登陆二维码标准](#登陆二维码标准)）。
- 3 使用ONTO扫码
- 4 用户对扫码获取的数据签名（[DApp服务端登陆接口](#DApp服务端登陆接口)）
- 5 ONTO将签名信息发往dApp server验证（[签名验证方法](#签名验证方法)）
- 6 dApp web端向server端请求验证结果

### 调用智能合约交互流程

![Login](./img/invokeSC.png)

- 1 dApp web端向dApp server请求智能合约数据。
- 2 dapp web根据请求回来的智能合约数据生成二维码（[调用合约二维码标准](#调用合约二维码标准)）。
- 3 使用ONTO扫码
- 4 ONTO构造交易，用户签名，预执行交易，用户确认，发送到链上，返回交易hash
- 5 ONTO将签名交易信息发往dApp server验证（[合约查询方法](#合约查询方法)）
- 6 dApp web端向server端请求验证结果

## 接入步骤

### 前提条件
使用前，你需要联系[本体机构合作](https://info.ont.io/cooperation/en)

### 登陆接入步骤

#### 登陆二维码标准
扫码获取

```
{
	"action": "login",
	"version": "v1.0.0",
	"params": {
		"type": "ontid or account",
		"dappName": "dapp Name",
		"dappIcon": "dapp Icon",
		"message": "helloworld",
		"expired": "20181215152730", // QR Code expire time
		"callback": "http://101.132.193.149:4027/blockchain/v1/common/test-onto-login"
	}
}
```

|字段|类型|定义|
| :---| :---| :---|
| action   |  string |  定义此二维码的功能，登录设定为"Login"，调用智能合约设定为"invoke" |
| type   |  string |  定义是使用ontid登录设定为"ontid"，钱包地址登录设定为"account" |
| dappName   | string  | dapp名字 |
| dappIcon   | string  | dapp icon信息 |
| message   | string  | 随机生成，用于校验身份  |
| expired   | string  | 可选  |
| callback   | string  |  用户扫码签名后发送到DApp后端URL |

### DApp服务端登陆接口
method: post

```
{
	"action": "login",
	"version": "v1.0.0",
	"params": {
		"type": "ontid or account",
		"user": "did:ont:AUEKhXNsoAT27HJwwqFGbpRy8QLHUMBMPz",
		"message": "helloworld",
		"publickey": "0205c8fff4b1d21f4b2ec3b48cf88004e38402933d7e914b2a0eda0de15e73ba61",
		"signature": "01abd7ea9d79c857cd838cabbbaad3efb44a6fc4f5a5ef52ea8461d6c055b8a7cf324d1a58962988709705cefe40df5b26e88af3ca387ec5036ec7f5e6640a1754"
	}
}
```

|字段|类型|定义|
| :---| :---| :---|
| action | string | 操作类型，登录设定为"login"，调用智能合约设定为"invoke" |
| params | string | 方法要求的参数 |
| type   |  string |  定义是使用ontid登录设定为"ontid"，钱包地址登录设定为"account" |
| user | string | 用户做签名的账户，比如用户的ontid或者钱包地址 |
| message   | string  | 随机生成，用于校验身份  |
| publickey | string | 账户公钥 |
| signature  |  string |  用户签名 |

#### Response
* Success

```
{
  "action": "login",
  "error": 0,
  "desc": "SUCCESS",
  "result": true
}
```

* Failed

```
{
  "action": "login",
  "error": 8001,
  "desc": "PARAMS ERROR",
  "result": 1
}
```


### 调用合约二维码标准
扫码获取

```
{
	"action": "invoke",
	"version": "v1.0.0",
	"params": {
		"login": true,
		"callback": "http://101.132.193.149:4027/invoke/callback",
		"qrcodeUrl": "http://101.132.193.149:4027/qrcode/AUr5QUfeBADq6BMY6Tp5yuMsUNGpsD7nLZ"
	}
}
```

|字段|类型|定义|
| :---        | :---    | :---                                                              |
| action      | string  | 操作类型，登录设定为"Login"，调用智能合约设定为"invoke" |
| qrcodeUrl         | string  | 二维码参数地址                                           |
| callback         | string  | 选填，返回交易hash给dApp服务端                                           |

根据二维码中qrcodeUrl链接，GET的的数据如下：

```
{
	"action": "invoke",
	"version": "v1.0.0",
	"params": {
		"invokeConfig": {
			"contractHash": "16edbe366d1337eb510c2ff61099424c94aeef02",
			"functions": [{
				"operation": "method name",
				"args": [{
					"name": "arg0-list",
					"value": [true, 100, "Long:100000000000", "Address:AUr5QUfeBADq6BMY6Tp5yuMsUNGpsD7nLZ", "ByteArray:aabb", "String:hello", [true, 100], {
						"key": 6
					}]
				}, {
					"name": "arg1-map",
					"value": {
						"key": "String:hello",
						"key1": "ByteArray:aabb",
						"key2": "Long:100000000000",
						"key3": true,
						"key4": 100,
						"key5": [100],
						"key6": {
							"key": 6
						}
					}
				}, {
					"name": "arg2-str",
					"value": "String:test"
				}]
			}],
			"payer": "AUr5QUfeBADq6BMY6Tp5yuMsUNGpsD7nLZ",
			"gasLimit": 20000,
			"gasPrice": 500
		}
	}
}


```


ONTO 构造交易，用户签名，预执行交易，发送交易，POST 交易hash给callback url。

* 发送交易成功POST给callback

```
{
  "action": "invoke",
  "error": 0,
  "desc": "SUCCESS",
  "result": "tx hash"
}
```

* 发送交易失败给callback

```
{
  "action": "invoke",
  "error": 8001,
  "desc": "SEND TX ERROR",
  "result": 1
}
```

##### 预执行交易

ONTO将会对交易做预执行，主要作用是提醒用户该交易中包含的ONT/ONG转账数量。

预执行交易返回的Notify结果可以查看用户在这笔交易中会花费多少ONT/ONG。因为当前节点没升级，需要连接到固定节点预执行才会有返回Notify信息：主网：http://dappnode3.ont.io， 测试网：http://polaris5.ont.io

> 需要遍历Notify做判断，因为该交易可能有多笔转账事件或其他合约事件，如果是其他合约事件不需做处理，通过合约地址判断是ONT还是ONG，再判断transfer方法和转出方。建议UI显示转入转出方和amount，还有交易手续费大约0.01 ONG。
ONT:0100000000000000000000000000000000000000
ONG:0200000000000000000000000000000000000000

如果预执行成功，节点返回的结果是：
```
{
    "Action": "sendrawtransaction",
    "Desc": "SUCCESS",
    "Error": 0,
    "Result": {
              	"Notify": [{
              		"States": ["transfer", "AUr5QUfeBADq6BMY6Tp5yuMsUNGpsD7nLZ", "AecaeSEBkt5GcBCxwz1F41TvdjX3dnKBkJ", 1],
              		"ContractAddress": "0100000000000000000000000000000000000000"
              	}],
              	"State": 1,
              	"Gas": 20000,
              	"Result": "01"
     },
    "Version": "1.0.0"
}
```

如果预执行失败，Error 的值> 0。


## 代码参考

##### 签名验证方法
* [java sdk验签](https://github.com/ontio/ontology-java-sdk/blob/master/docs/cn/interface.md#%E7%AD%BE%E5%90%8D%E9%AA%8C%E7%AD%BE)
* [ts sdk验签](https://github.com/ontio/ontology-ts-sdk/blob/master/test/message.test.ts)

##### 合约查询方法
* [java sdk 合约查询](https://github.com/ontio/ontology-java-sdk/blob/master/docs/cn/basic.md#%E4%B8%8E%E9%93%BE%E4%BA%A4%E4%BA%92%E6%8E%A5%E5%8F%A3)
* [ts sdk 合约查询](https://github.com/ontio/ontology-ts-sdk/blob/master/test/websocket.test.ts)

##### ONTO
* [ONTO](https://onto.app)
