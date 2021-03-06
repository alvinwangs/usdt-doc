# USDT交易

目前对接波场，支持币种 如下：TRX、TRC20,其中TRC20 在测试时币种名称为：BBT，正式为USDT

## 手续费

波场交易手续费收取是以能量、带宽为载体，当账户能量带宽余额不足以支付交易所需的能量带宽时，将直接从TRX 账户余额扣减。

每笔转账所需的能量、带宽如下：

|      | 消耗  |
| ---- | ----- |
| 能量 | 14000 |
| 带宽 | 350   |

能量获得方式：

> 质押TRX获得能量 ，质押100 TRX 可获得约 28,93 能量

带宽获得方式：

> 1. 每个账户每日可自动获得1500 带宽
> 2. 也可以通过质押的方式获得带宽，质押100TRX 可获得150 带宽

**注意**：能量、带宽质押兑换比例并不固定，每笔交易所消耗的能量、带宽也不固定，跟随市场的波动变化而变化。

如果账户能量、带宽不足以支付抵扣交易所需要、则在扣减完能量/带宽余额后，不足的部分直接从账户的TRX 余额里扣减。而如果TRX 余额也不足以抵扣手续费则交易失败

而如果完全通过TRX 来抵扣手续费，则一笔交易大概花费：4TRX 抵扣能量+0.4 抵扣带宽，约4.4 TRX 手续费。

|      | 消耗  | TRX抵扣 |
| ---- | ----- | ------- |
| 能量 | 14000 | 4       |
| 带宽 | 350   | 1       |



## 账户

账户实际上是一个公开地址+ 一个私钥。私钥会以文件方式保存在服务器（需要严格保管）。账号创建并不会立即激活，但是会提供一个立即激活的选项。立即激活会自动向新建的账号转一笔1.1 个 TRX 来实现激活。

建议立即激活账户使用于每个用户固定一个地址的场景，非激活状态的地址可以用于一次性交易的地址

### 创建账户

#### 创建流程

![image-20220329220930018](./img/image-20220329220930018.png)



**接口地址**:  `/v1/account`

 **请求方法**:    `POST`

**请求数据类型**：`application/json`

| 参数名称    | 参数说明                                                    | 是否必须 | 数据类型 |
| ----------- | ----------------------------------------------------------- | -------- | -------- |
| active      | 是否立即激活                                                | false    | boolean  |
| callBackUrl | 该账户地址的所有交易都会通过该url 异步[回调](#交易回调)通知 | true     | string   |
| chain       | 该地址隶属哪个链。值: Tron                                  |          |          |
| requestId   | 请求id，唯一性。每一个请求都要使用不同的id                  | false    | string   |
| sign        | 验签，统一签名参考 [签名](#签名) 章节                       | true     | string   |

**请求示例**:


```javascript
{
  "active": true,
  "callBackUrl": "https://you.server.com/api/callback",
  "requestId": "",
  "chain":"Tron",
  "sign": ""
}
```

**响应参数**:


| 参数名称    | 参数说明   | 类型   |
| ----------- | ---------- | ------ |
| address     | 创建的地址 | string |
| callBackUrl | 回调地址   | string |

**响应示例**:

```javascript
{
  "address": "",
  "callBackUrl": ""
}
```



### 查询账户资产

**接口描述**:  查询地址的所有资产信息；可能存在的资产包括：BBT（测试币，测试链上才有）、TRX(测试网和主网)、USDT（主网，如果有的话）

**接口地址**:`/v1/asset/{address}`

**请求方式**:`GET`

**请求数据类型**:`application/json`

**响应数据类型**:`*/*`

**请求参数**:


| 参数名称 | 参数说明 | 请求类型 | 是否必须 | 数据类型 | schema |
| -------- | -------- | -------- | -------- | -------- | ------ |
| address  | address  | path     | true     | string   |        |

| 状态码 | 说明                           | schema    |
| ------ | ------------------------------ | --------- |
| 200    | OK，非200 的请求均代表请求失败 | Json body |


**响应参数**:


| 参数名称     | 参数说明                      | 类型           | schema         |
| ------------ | ----------------------------- | -------------- | -------------- |
| balance      | 余额                          | string         |                |
| contractName | 合约缩写，合约类型:TRX，TRC20 | string         |                |
| precision    | 精度                          | integer(int32) | integer(int32) |

**响应示例**:

```javascript
[
  {
    "balance": "",
    "contractName": "",
    "precision": 0
  }
]
```

### 分页查询流水

**接口地址**:`/v1/transaction`


**请求方式**:`GET`


**请求数据类型**:`application/x-www-form-urlencoded`


**响应数据类型**:`*/*`


**接口描述**:

**请求参数**:


| 参数名称       | 参数说明                       | 请求类型 | 是否必须 | 数据类型       |
| -------------- | ------------------------------ | -------- | -------- | -------------- |
| page.pageNum   | 当前页码                       | query    | false    | integer(int32) |
| page.pageSize  | 每页条数                       | query    | false    | integer(int32) |
| page.startTime | 起始时间 (yyyy-MM-dd HH:mm:ss) | query    | false    | string         |
| page.endTime   | 结束时间 (yyyy-MM-dd HH:mm:ss) | query    | false    | string         |
| page.sort      |                                | query    | false    | string         |
| page.order     | Ordering - asc or desc         | query    | false    | string         |
| keywords       | 关键字                         | query    | false    | string         |
| ownerAddress   | 转出账号                       | query    | false    | string         |
| toAddress      | 转入账号                       | query    | false    | string         |
| contractName   | 资产类型：TRX、TRX20           | query    | false    | string         |
| type           | 流水类型                       | query    | false    | integer(int32) |
| status         |                                | query    | false    | integer(int32) |


**响应状态**:


| 状态码 | 说明 |
| ------ | ---- |
| 200    | OK   |

**响应参数**:


| 参数名称                  | 参数说明                                                 | 类型           | schema           |
| ------------------------- | -------------------------------------------------------- | -------------- | ---------------- |
| endRow                    |                                                          | integer(int32) | integer(int32)   |
| firstPage                 |                                                          | integer(int32) | integer(int32)   |
| hasNextPage               |                                                          | boolean        |                  |
| hasPreviousPage           |                                                          | boolean        |                  |
| isFirstPage               |                                                          | boolean        |                  |
| isLastPage                |                                                          | boolean        |                  |
| lastPage                  |                                                          | integer(int32) | integer(int32)   |
| list                      |                                                          | array          | TransactionLogPo |
| &emsp;&emsp;amount        | 转账金额                                                 | string         |                  |
| &emsp;&emsp;contractName  | 资产类型：TRX、TRC20                                     | integer(int32) |                  |
| &emsp;&emsp;createTime    |                                                          | Timestamp      | Timestamp        |
| &emsp;&emsp;fee           | 转账手续费                                               | string         |                  |
| &emsp;&emsp;id            | id                                                       | integer(int64) |                  |
| &emsp;&emsp;message       | 转帐信息                                                 | string         |                  |
| &emsp;&emsp;ownerAddress  | 付款地址                                                 | string         |                  |
| &emsp;&emsp;precision     | 转账金额精度                                             | integer(int32) |                  |
| &emsp;&emsp;startBlockNum | 转账时块号                                               | integer(int64) |                  |
| &emsp;&emsp;status        | 转账状态                                                 | integer(int32) |                  |
| &emsp;&emsp;toAddress     | 收款地址                                                 | string         |                  |
| &emsp;&emsp;transactionId | 交易流水                                                 | string         |                  |
| &emsp;&emsp;type          | 转账类型：1：创建账户激活费用；2：转出；3：转入；4：归集 | integer(int32) |                  |
| &emsp;&emsp;updateTime    |                                                          | Timestamp      | Timestamp        |
| navigateFirstPage         |                                                          | integer(int32) | integer(int32)   |
| navigateLastPage          |                                                          | integer(int32) | integer(int32)   |
| navigatePages             |                                                          | integer(int32) | integer(int32)   |
| navigatepageNums          |                                                          | array          |                  |
| nextPage                  |                                                          | integer(int32) | integer(int32)   |
| pageNum                   |                                                          | integer(int32) | integer(int32)   |
| pageSize                  |                                                          | integer(int32) | integer(int32)   |
| pages                     |                                                          | integer(int32) | integer(int32)   |
| prePage                   |                                                          | integer(int32) | integer(int32)   |
| size                      |                                                          | integer(int32) | integer(int32)   |
| startRow                  |                                                          | integer(int32) | integer(int32)   |
| total                     |                                                          | integer(int64) | integer(int64)   |


**响应示例**:
```javascript
{
	"endRow": 0,
	"firstPage": 0,
	"hasNextPage": true,
	"hasPreviousPage": true,
	"isFirstPage": true,
	"isLastPage": true,
	"lastPage": 0,
	"list": [
		{
			"amount": "",
			"contractAddress": "",
			"contractName": "",
			"contractType": 0,
			"createTime": 121312312313112,
			"fee": "",
			"id": 0,
			"message": "",
			"ownerAddress": "",
			"precision": 0,
			"startBlockNum": 0,
			"status": 0,
			"toAddress": "",
			"transactionId": "",
			"type": 0,
			"updateTime":121312312313112,
			"writeBlockNum": 0
		}
	],
	"navigateFirstPage": 0,
	"navigateLastPage": 0,
	"navigatePages": 0,
	"navigatepageNums": [],
	"nextPage": 0,
	"pageNum": 0,
	"pageSize": 0,
	"pages": 0,
	"prePage": 0,
	"size": 0,
	"startRow": 0,
	"total": 0
}
```

### 查询单笔流水

**接口地址**:`/v1/transaction`/{transactionId}


**请求方式**:`GET`

**请求数据类型**:`application/x-www-form-urlencoded`

**响应数据类型**:`*/*`

**接口描述**:  查询单笔流水

**请求参数**:

| 参数名称      | 参数说明                 | 请求类型 | 是否必须 | 数据类型 |
| ------------- | ------------------------ | -------- | -------- | -------- |
| transactionId | 流水id（回调、转账返回） | path     | true     | string   |

**响应参数**:


| 参数名称                  | 参数说明                                                 | 类型           | schema |
| ------------------------- | -------------------------------------------------------- | -------------- | ------ |
| &emsp;&emsp;amount        | 转账金额                                                 | string         |        |
| &emsp;&emsp;contractName  | 资产类型：TRX、TRC20                                     | integer(int32) |        |
| &emsp;&emsp;fee           | 转账手续费                                               | string         |        |
| &emsp;&emsp;id            | id                                                       | integer(int64) |        |
| &emsp;&emsp;message       | 转帐信息                                                 | string         |        |
| &emsp;&emsp;ownerAddress  | 付款地址                                                 | string         |        |
| &emsp;&emsp;precision     | 转账金额精度                                             | integer(int32) |        |
| &emsp;&emsp;startBlockNum | 转账时块号                                               | integer(int64) |        |
| &emsp;&emsp;status        | 转账状态                                                 | integer(int32) |        |
| &emsp;&emsp;toAddress     | 收款地址                                                 | string         |        |
| &emsp;&emsp;transactionId | 交易流水                                                 | string         |        |
| &emsp;&emsp;type          | 转账类型：1：创建账户激活费用；2：转出；3：转入；4：归集 | integer(int32) |        |
| &emsp;&emsp;tradeTime     | 交易时间                                                 | string         |        |

## 交易

### 创建交易

**交易流程**

​	![image-20220329222625731](./img/image-20220329222625731.png)



**接口地址**:`/v1/transaction/transfer`


**请求方式**:`POST`


**请求数据类型**:`application/json`

**响应数据类型**:`*/*`

**接口描述**: 该接口会产生手续费，具体参考[手续费](#手续费)章节

**请求示例**:


```javascript
{
  "amount": "",
  "contractName": "",
  "toAddress": ""，
  "requestId": "",
  "sign": ""
}
```


**请求参数**:


| 参数名称     | 参数说明                                                     | 是否必须 |        |
| ------------ | ------------------------------------------------------------ | -------- | ------ |
| &emsp;amount | 转账金额，不同资产类型精度不一样，业务方不需要特殊处理精度，例如支付100USDT，传入100即可 | false    | string |
| contractName | 资产类型：TRX、TRC20                                         | false    | string |
| requestId    | 请求id，唯一性。每一个请求都要使用不同的id                   | false    | string |
| sign         | 验签。统一签名参考 [签名](#签名) 章节                        | true     | string |
| toAddress    | 收款人地址                                                   | false    | string |

**响应状态**:


| 状态码 | 说明                         |
| ------ | ---------------------------- |
| 200    | OK，非200 状态均视为失败请求 |

**响应参数**:


| 参数名称      | 参数说明                         | 类型   |
| ------------- | -------------------------------- | ------ |
| transactionId | 交易流水，可记录用于查询交易结果 | string |

**响应示例**:

```javascript
{
  "transactionId": ""
}
```

### 账户资产归集--所有账户

![image-20220329224257957](./img/image-20220329224257957.png)



**接口描述**: 将所有的地址的资金归集到母账户。（需要指定资产类型），该接口每个账户归集均会产生[手续费](#手续费)

**接口地址**:`/v1/accounts/collect`

**请求方式**:`POST`

**请求数据类型**:`application/json`

**响应数据类型**:`*/*`

**请求示例**:


```javascript
{
  "contractName": "",
  "requestId": "",
  "sign": "",
  "sortParam": ""
}
```


**请求参数**:


| 参数名称                 | 参数说明                                   | 是否必须 | 数据类型 |
| ------------------------ | ------------------------------------------ | -------- | -------- |
| &emsp;&emsp;contractName | 资产类型：TRX、TRC20                       | true     | string   |
| &emsp;&emsp;requestId    | 请求id，唯一性。每一个请求都要使用不同的id | false    | string   |
| &emsp;&emsp;sign         | 验签。统一签名参考 [签名](#签名) 章节      | true     | string   |


**响应状态**:


| 状态码 | 说明                    |
| ------ | ----------------------- |
| 200    | OK,非200 均视为请求失败 |


**响应参数**:


| 参数名称      | 参数说明                  | 类型   | schema |
| ------------- | ------------------------- | ------ | ------ |
| ownerAddress  | 客户账户                  | string |        |
| result        | 请求结果：SUCCESS, FAILED | string |        |
| transactionId | 流水Id                    | string |        |

**响应示例**:

```javascript
[
  {
    "ownerAddress": "",
    "sign": "",
    "sortParam": "",
    "transactionId": ""
  }
]
```



### 账户资产归集--指定账户

**接口描述**: 将指定地址的资金归集到母账户。（需要指定资产类型），该接口每个账户归集均会产生[手续费](#手续费)

**接口地址**:`/v1/accounts/collect/_partial`

**请求方式**:`POST`

**请求数据类型**:`application/json`

**响应数据类型**:`*/*`

**请求示例**:


```javascript
{
  "contractName": "",
  "requestId": "",
   "accountAddressList":[]
  "sign": "",
  "sortParam": ""
}
```


**请求参数**:


| 参数名称                       | 参数说明                                   | 是否必须 | 数据类型 |
| ------------------------------ | ------------------------------------------ | -------- | -------- |
| &emsp;&emsp;contractName       | 资产类型：TRX、TRC20                       | true     | string   |
| &emsp;&emsp;accountAddressList | 指定账户地址列表                           | true     | Array    |
| &emsp;&emsp;requestId          | 请求id，唯一性。每一个请求都要使用不同的id | false    | string   |
| &emsp;&emsp;sign               | 验签。统一签名参考 [签名](#签名) 章节      | true     | string   |

**响应状态**:


| 状态码 | 说明                    |
| ------ | ----------------------- |
| 200    | OK,非200 均视为请求失败 |


**响应参数**:


| 参数名称     |                  参数说明 | 类型   | schema |
| ------------ | ------------------------: | ------ | ------ |
| ownerAddress |                  客户账户 | string |        |
| result       | 请求结果：SUCCESS, FAILED | string |        |
| message      |                      信息 | string |        |

**响应示例**:

```javascript
[
  {
    "ownerAddress": "",
    "result": "",
    "message":""
  }
]
```



## 签名

对接初期，系统会为**业务方**生成一对**公私钥** ，**私钥**保存于业务方服务器中（妥善保管）用于签名，公钥保存于支付服务，用于验签，从而实现身份验证。

签名步骤：

1. 请求参数除**sign** 之外的所有参数按照字典升序排序；如果某个字段是Array 字段则array 的内容以","分隔
2. 对排序后的内容进行 hash计算
4. 将hash转换成16进制

### 示例

****

**私钥**：83f15af3f6c2e5f719996d1689085cc02a89f9df16e76fac5eb4fd3bcb6962e8

**公钥**：748c4638fb9950ae4ba906868d510ffdad5cb97bd914f57588dd7deddbcafa70acc46e849874ef364d5688d11dc1cc80cf1d233b208f5f0143dcb8b6603021d4

**请求参数**：

```
{	
  "amount": "",
  "contractName": "",
  "ownerAddress": "",
  "toAddress": ""，
  "requestId": "",
  "sign": ""
}
```

**排序后的内容**：

```
amount=10.001&contractName=USDT&ownerAddress=TAbcDeFGKJSKDTrdfgTFD&requestId=uuid&toAddress=TCCstFdfgSKDTrdfgTFD
```

**JAVA签名**代码示例一：

```java
				String secret=83f15af3f6c2e5f719996d1689085cc02a89f9df16e76fac5eb4fd3bcb6962e8
        String plainText = "amount=10.001&contractName=TRC20&ownerAddress=TAbcDeFGKJSKDTrdfgTFD&requestId=uuid&toAddress=TCCstFdfgSKDTrdfgTFD"+secret;	
        //计算hash
        SHA256.Digest digest = new SHA256.Digest();
        digest.update(plainText.getBytes());
        byte[] hash = digest.digest();
        String sign = Hex.toHexString(hash);
        System.out.println("signature -------- >>>>>> " + sign);
```

**JAVA签名**代码示例二：

以指定账户归集资产为例：

```java
String secret=83f15af3f6c2e5f719996d1689085cc02a89f9df16e76fac5eb4fd3bcb6962e8
String plainText = "contractName=TRC20&accountAddressList=TAbcDeFGKJSKDTrdfgTFD,Bcsdfe4456sdfd4565&requestId=uuid"+secret;	
        //计算hash
SHA256.Digest digest = new SHA256.Digest();
digest.update(plainText.getBytes());
byte[] hash = digest.digest();
String sign = Hex.toHexString(hash);
System.out.println("signature -------- >>>>>> " + sign);
```

## 交易回调

**请求路径**：业务方提供，自定义

**请求方法**：`POST`


**请求数据类型**:`application/json`

**响应数据类型**:`*/*`

**请求参数**:


| 参数名称      | 参数说明                                                     | 是否必须 | 数据类型   |
| ------------- | ------------------------------------------------------------ | -------- | ---------- |
| id            | 流水row id                                                   | true     | string     |
| transactionId | 流水id，可在区块链上查到该笔交易的详细信息                   | true     | string     |
| ownerAddress  | 付款方 地址                                                  | true     | string     |
| toAddress     | 收款方地址                                                   | true     | string     |
| contractName  | 资产类型，**TRX**、**TRC20**                                 | true     | string     |
| amount        | 交易金额，业务方不需要特殊处理精度，例如支付100USDT，传入100即可 | true     | BigDecimal |
| precision     | 该资产的精度，不同的资产类型有不同的精度                     | true     | Integer    |
| status        | 交易状态。0：默认状态，交易提交后的初始状态，1：正在确认；2： 成功；3 失败 | true     | Integer    |
| message       | 交易信息。失败原因                                           | true     | String     |
| fee           | 手续费                                                       | true     | Integer    |
| energyFee     | 能量费                                                       | true     | Integer    |
| type          | 转账类型：1：创建账户激活费用；2：转出；3：转入；4：归集     | true     | Integer    |
| tradeTime     | 交易时间：**yyyy-MM-dd HH:mm:ss**                            |          |            |
| sign          | 交易的签名。参考 [签名](#签名) 章节                          |          |            |

**响应状态**:


| 状态码 | 说明                    |
| ------ | ----------------------- |
| 200    | OK,非200 均视为请求失败 |



**异常处理**：业务方需要在收到支付回调时响应 200 的http 状态码，如果响应非200 状态，则会进入重试队列，每个30秒重试一次，最多重试6次。6次失败后不再重试。



## 错误码

接口规范遵循restful 风格，http status code =200 请求成功，http status code ！=200，则请求失败，并返回错误信息，错误信息如下：

| 错误码 | 错误信息                   |
| ------ | -------------------------- |
| 1      | success                    |
| 202000 | 重复请求                   |
| 202001 | 数据录入重复, 请检查后重试 |
| 400000 | 参数异常                   |
| 400001 | 签名错误                   |
| 404000 | 请求的资源不存在           |
| 401000 | 权限不足                   |
| 500000 | 服务端异常, 请稍后再试     |
| 500001 | 系统繁忙, 请稍后再试       |
| 500002 | 服务端异常, 请稍后再试     |
| 600001 | 用户余额不足               |
| 600002 | 用户FeeLimit不足           |