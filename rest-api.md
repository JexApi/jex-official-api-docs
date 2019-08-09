# 基本信息
* 本篇列出REST接口的baseurl  https://www.jex.com
* 所有接口的响应都是JSON格式
* 响应中如有数组，数组元素以时间升序排列，越早的数据越提前。
* 所有时间、时间戳均为UNIX时间，单位为毫秒
* HTTP `4XX` 错误码用于指示错误的请求内容、行为、格式。
* HTTP `429` 错误码表示警告访问频次超限，即将被封IP
* HTTP `418` 表示收到429后继续访问，于是被封了。
* HTTP `5XX` 错误码用于指示服务器侧的问题。 
* HTTP `504` 表示API服务端已经向业务核心提交了请求但未能获取响应，特别需要注意的是`504`代码不代表请求失败，而是未知。很可能已经得到了执行，也有可能执行失败，需要做进一步确认。
* 每个接口都有可能抛出异常，异常响应格式如下：
```javascript
{
  "code": -1121,
  "msg": "Invalid symbol."
}
```

* 具体的错误码及其解释在[错误代码汇总.md](./错误代码汇总.md)
* `GET`方法的接口, 参数必须在`query string`中发送.
* `POST`, `PUT`, 和 `DELETE` 方法的接口, 参数可以在 `query string`中发送，也可以在 `request body`中发送(content type `application/x-www-form-urlencoded`。允许混合这两种方式发送参数。但如果同一个参数名在query string和request body中都有，query string中的会被优先采用。
* 对参数的顺序不做要求。

# 访问限制
* 在 `/api/v1/exchangeInfo`接口中`rateLimits`数组里包含有REST接口(不限于本篇的REST接口)的访问限制。包括带权重的访问频次限制、下单速率限制。本篇`枚举定义`章节有限制类型的进一步说明。
* 违反上述任何一个访问限制都会收到HTTP 429，这是一个警告.
* 每一个接口均有一个相应的权重(weight)，有的接口根据参数不同可能拥有不同的权重。越消耗资源的接口权重就会越大。
* 当收到429告警时，调用者应当降低访问频率或者停止访问。
* **收到429后仍然继续违反访问限制，会被封禁IP，并收到418错误码**
* 频繁违反限制，封禁时间会逐渐延长，**从最短2分钟到最长3天**.

# 接口鉴权类型
* 每个接口都有自己的鉴权类型，鉴权类型决定了访问时应当进行何种鉴权
* 如果需要 API-key，应当在HTTP头中以`X-MBX-APIKEY`字段传递
* API-key 与 API-secret 是大小写敏感的
* 可以在网页用户中心修改API-key 所具有的权限，例如读取账户信息、发送交易指令、发送提现指令

鉴权类型 | 描述
------------ | ------------
NONE | 不需要鉴权的接口
TRADE | 需要有效的API-KEY和签名
USER_DATA | 需要有效的API-KEY和签名
USER_STREAM | 需要有效的API-KEY
MARKET_DATA | 需要有效的API-KEY


# 需要签名的接口 (TRADE 与 USER_DATA)
* 调用这些接口时，除了接口本身所需的参数外，还需要传递`signature`即签名参数。
* 签名使用`HMAC SHA256`算法. API-KEY所对应的API-Secret作为 `HMAC SHA256` 的密钥，其他所有参数作为`HMAC SHA256`的操作对象，得到的输出即为签名。
* 签名大小写不敏感。
* 当同时使用query string和request body时，`HMAC SHA256`的输入query string在前，request body在后

## 时间同步安全
* 签名接口均需要传递`timestamp`参数, 其值应当是请求发送时刻的unix时间戳（毫秒）
* 服务器收到请求时会判断请求中的时间戳，如果是5000毫秒之前发出的，则请求会被认为无效。这个时间窗口值可以通过发送可选参数`recvWindow`来自定义。
* 另外，如果服务器计算得出客户端时间戳在服务器时间的‘未来’一秒以上，也会拒绝请求。
* 逻辑伪代码：
  ```javascript
  if (timestamp < (serverTime + 1000) && (serverTime - timestamp) <= recvWindow) {
    // process request
  } else {
    // reject request
  }
  ```

**关于交易时效性** 
互联网状况并不100%可靠，不可完全依赖,因此你的程序本地到服务器的时延会有抖动.
这是我们设置`recvWindow`的目的所在，如果你从事高频交易，对交易时效性有较高的要求，可以灵活设置recvWindow以达到你的要求。
**不推荐使用5秒以上的recvWindow**


## POST /api/v1/spot/order 的示例

以下是在linux bash环境下使用 echo openssl 和curl工具实现的一个调用接口下单的示例
apikey、secret仅供示范

Key | Value
------------ | ------------
apiKey | vmPUZE6mv9SD5VNHk4HlWFsOr6aKE2zvsw0MuIgwCIPy6utIco14y7Ju91duEh8A
secretKey | NhqPtmdSJYdKjVHjA7PZj4Mge3R5YNiP1e3UZjInClVN65XAbvqqM6A7H5fATj0j


参数 | 取值
------------ | ------------
symbol | LTCBTC
side | BUY
type | LIMIT
timeInForce | GTC
quantity | 1
price | 0.1
recvWindow | 5000
timestamp | 1499827319559


### 示例 1: 所有参数通过 query string 发送
* **queryString:** symbol=LTCBTC&side=BUY&type=LIMIT&timeInForce=GTC&quantity=1&price=0.1&recvWindow=5000&timestamp=1499827319559
* **HMAC SHA256 签名:**

    ```
    [linux]$ echo -n "symbol=LTCBTC&side=BUY&type=LIMIT&timeInForce=GTC&quantity=1&price=0.1&recvWindow=5000&timestamp=1499827319559" | openssl dgst -sha256 -hmac "NhqPtmdSJYdKjVHjA7PZj4Mge3R5YNiP1e3UZjInClVN65XAbvqqM6A7H5fATj0j"
    (stdin)= c8db56825ae71d6d79447849e617115f4a920fa2acdcab2b053c4b2838bd6b71
    ```


* **curl 调用:**

    ```
    (HMAC SHA256)
    [linux]$ curl -H "X-JEX-APIKEY: vmPUZE6mv9SD5VNHk4HlWFsOr6aKE2zvsw0MuIgwCIPy6utIco14y7Ju91duEh8A" -X POST 'https://www.jex.com/api/v1/order?symbol=LTCBTC&side=BUY&type=LIMIT&timeInForce=GTC&quantity=1&price=0.1&recvWindow=5000&timestamp=1499827319559&signature=c8db56825ae71d6d79447849e617115f4a920fa2acdcab2b053c4b2838bd6b71'
    ```

### 示例 2: 所有参数通过 request body 发送
* **requestBody:** symbol=LTCBTC&side=BUY&type=LIMIT&timeInForce=GTC&quantity=1&price=0.1&recvWindow=5000&timestamp=1499827319559
* **HMAC SHA256 签名:**

    ```
    [linux]$ echo -n "symbol=LTCBTC&side=BUY&type=LIMIT&timeInForce=GTC&quantity=1&price=0.1&recvWindow=5000&timestamp=1499827319559" | openssl dgst -sha256 -hmac "NhqPtmdSJYdKjVHjA7PZj4Mge3R5YNiP1e3UZjInClVN65XAbvqqM6A7H5fATj0j"
    (stdin)= c8db56825ae71d6d79447849e617115f4a920fa2acdcab2b053c4b2838bd6b71
    ```


* **curl 调用:**

    ```
    (HMAC SHA256)
    [linux]$ curl -H "X-MBX-APIKEY: vmPUZE6mv9SD5VNHk4HlWFsOr6aKE2zvsw0MuIgwCIPy6utIco14y7Ju91duEh8A" -X POST 'https://www.jex.com/api/v1/order' -d 'symbol=LTCBTC&side=BUY&type=LIMIT&timeInForce=GTC&quantity=1&price=0.1&recvWindow=5000&timestamp=1499827319559&signature=c8db56825ae71d6d79447849e617115f4a920fa2acdcab2b053c4b2838bd6b71'
    ```

### 示例 3: 混合使用 query string 与 request body
* **queryString:** symbol=LTCBTC&side=BUY&type=LIMIT&timeInForce=GTC
* **requestBody:** quantity=1&price=0.1&recvWindow=5000&timestamp=1499827319559
* **HMAC SHA256 签名:**

    ```
    [linux]$ echo -n "symbol=LTCBTC&side=BUY&type=LIMIT&timeInForce=GTCquantity=1&price=0.1&recvWindow=5000&timestamp=1499827319559" | openssl dgst -sha256 -hmac "NhqPtmdSJYdKjVHjA7PZj4Mge3R5YNiP1e3UZjInClVN65XAbvqqM6A7H5fATj0j"
    (stdin)= 0fd168b8ddb4876a0358a8d14d0c9f3da0e9b20c5d52b2a00fcf7d1c602f9a77
    ```


* **curl 调用:**

    ```
    (HMAC SHA256)
    [linux]$ curl -H "X-MBX-APIKEY: vmPUZE6mv9SD5VNHk4HlWFsOr6aKE2zvsw0MuIgwCIPy6utIco14y7Ju91duEh8A" -X POST 'https://www.jex.com/api/v1/order?symbol=LTCBTC&side=BUY&type=LIMIT&timeInForce=GTC' -d 'quantity=1&price=0.1&recvWindow=5000&timestamp=1499827319559&signature=0fd168b8ddb4876a0358a8d14d0c9f3da0e9b20c5d52b2a00fcf7d1c602f9a77'
    ```

Note that the signature is different in example 3.
There is no & between "GTC" and "quantity=1".




# 公开API接口
## 术语解释
* `base asset` 指一个交易对的交易对象，即写在靠前部分的资产名
* `quote asset` 指一个交易对的定价资产，即写在靠后部分资产名


## 枚举定义
**交易对状态:**

* PRE_TRADING 盘前交易
* TRADING 正常交易中
* POST_TRADING 盘后交易
* END_OF_DAY 收盘
* HALT 交易终止(该交易对已下线)
* AUCTION_MATCH 集合竞价
* BREAK 交易暂停

**交易对类型:**

* SPOT 币币
* OPTION 期权
* CONTRACT 合约

**订单状态:**

* NEW 新建订单
* PARTIALLY_FILLED  部分成交
* FILLED  全部成交
* CANCELED  已撤销
* PENDING_CANCEL 正在撤销中(目前不会遇到这个状态)
* REJECTED 订单被拒绝
* EXPIRED 订单过期(根据timeInForce参数规则)

**订单种类:**

* LIMIT 限价单
* MARKET  市价单
* STOP_LOSS 止损单
* STOP_LOSS_LIMIT 限价止损单
* TAKE_PROFIT 止盈单
* TAKE_PROFIT_LIMIT 限价止盈单
* LIMIT_MAKER 限价做市单

**订单方向:**

* BUY 买入
* SELL 卖出

**Time in force:**

* GTC - Good Till Cancel 成交为止
* IOC - Immediate or Cancel 无法立即成交(吃单)的部分就撤销
* FOK - Fill or Kill 无法全部立即成交就撤销

**K线间隔:**

m -> 分钟; h -> 小时; d -> 天; w -> 周; M -> 月

* 1m
* 3m
* 5m
* 15m
* 30m
* 1h
* 2h
* 4h
* 6h
* 12h
* 1d
* 1w

**限制种类 (rateLimitType)**

* REQUESTS_WEIGHT  单位时间请求权重之和上限
* ORDERS    单位时间下单(撤单)次数上限
* RAW_REQUESTS  单位时间请求次数上限

**限制间隔**

* SECOND
* MINUTE
* DAY

**合约账单接口(type)**
* transfer       资金划转
* fee            手续费
* funding        资金费用
* closPosition   平仓
* liquidation    爆仓平仓
* adl            强减平仓
* autoMargin     自动减仓
* positionSize   持仓变化



## 通用接口
### 测试服务器连通性 PING
```
GET /api/v1/ping
```
测试能否联通

**权重:**
1

**参数:**
NONE

**响应:**
```javascript
{}
```

### 获取服务器时间
```
GET /api/v1/time
```
获取服务器时间

**权重:**
1

**参数:**
NONE

**响应:**
```javascript
{
  "serverTime": 1499827319595
}
```

### 交易对信息
```
GET /api/v1/exchangeInfo
```
获取此时的限制信息和交易对信息

**权重:**
1

**Parameters:**
NONE

**响应:**
```javascript
{
  "timezone": "UTC",
  "serverTime": 1551270414646,
  "rateLimits": [
    {
      "rateLimitType": "requestsWeight",
      "interval": "minute",
      "intervalNum": 1,
      "limit": 1200
    },
    {
      "rateLimitType": "orders",
      "interval": "second",
      "intervalNum": 1,
      "limit": 10
    },
    {
      "rateLimitType": "orders",
      "interval": "day",
      "intervalNum": 1,
      "limit": 100000
    },
    {
      "rateLimitType": "rawRequests",
      "interval": "minute",
      "intervalNum": 5,
      "limit": 5000
    }
  ],
  "optionSymbols": [
    {
      "symbol": "BTCCALLM",
      "status": "TRADING",
      "baseAsset": "月BTC看涨0226",
      "baseAssetPrecision": 0,
      "quoteAsset": "USDT",
      "quotePrecision": 2,
      "orderTypes": [
        "LIMIT"
      ],
      "icebergAllowed": false,
      "filters": [
        {
          "filterType": "PRICE_FILTER",
          "maxPrice": "100000"
        },
        {
          "filterType": "LOT_SIZE",
          "minQty": "0.00000000"
        }
      ]
    }
  ],
  "contractSymbols": [
    {
      "symbol": "BTCUSDT",
      "status": "TRADING",
      "baseAsset": "btc",
      "baseAssetPrecision": 4,
      "quoteAsset": "usdt",
      "quotePrecision": 0,
      "orderTypes": [
        "LIMIT",
        "MARKET"
      ],
      "icebergAllowed": false,
      "filters": [
        {
          "filterType": "PRICE_FILTER",
          "minPrice": "0.00000001000000000000",
          "maxPrice": "1000000.00000000000000000000"
        },
        {
          "filterType": "LOT_SIZE",
          "minQty": "0.00010000000000000000",
          "maxQty": "1000000.00000000000000000000"
        }
      ]
    }
  ],
  "spotSymbols": [
    {
      "symbol": "DASHUSDT",
      "status": "TRADING",
      "baseAsset": "DASH",
      "baseAssetPrecision": 4,
      "quoteAsset": "USDT",
      "quotePrecision": 2,
      "orderTypes": [
        "LIMIT"
      ],
      "icebergAllowed": false,
      "filters": [
        {
          "filterType": "PRICE_FILTER",
          "maxPrice": "100000"
        },
        {
          "filterType": "LOT_SIZE",
          "minQty": "1.00000000"
        }
      ]
    }
  ]
}
```

### 期权交易对信息
```
GET /api/v1/optionInfo
```
获取此时的限制信息和交易对信息

**权重:**
1

**Parameters:**
NONE

**响应:**
```javascript
[
  {
    "symbol": "BTCCALLM",
    "status": "TRADING",
    "type": "LONGS",
    "exertionTime": 1551344400000,
    "optionDesc": "www.jex.com",
    "canRelease": true,
    "releaseOption": {
      "convertRatio": 0.1,
      "assetMarginName": "BTC",
      "rateMargin": 0.1,
      "rateBase": 1,
      "assetBaseName": "JEX"
    }
  },
  {
    "symbol": "BTCPUTM",
    "status": "TRADING",
    "type": "SHORTS",
    "exertionTime": 1551172087000,
    "optionDesc": "www.jex.com",
    "canRelease": false,
    "releaseOption": null
  }
]
```

### 合约交易对信息
```
GET /api/v1/contractInfo
```
获取此时的限制信息和交易对信息

**权重:**
1

**Parameters:**
NONE

**响应:**
```javascript
[
  {
    "premiumIndex": 0,
    "fundingRate": 0.000002,
    "fundingInterval": 600000,
    "nextFundingTime": 1551337200000,
    "predictedFundingRate": 0.000002,
    "fairBasis": 0,
    "fairBasisRate": 0,
    "openPositions": 0,
    "turnover24": 212921.712,
    "totalVolume": 0,
    "markMethod": 2,
    "markPrice": 2300.0046,
    "lastPrice": 2300,
    "autoDeleveraging": "DISABLE",
    "symbol": "BTCUSDT"
  }
]
```




## 行情接口
### 币币深度信息
```
GET /api/v1/spot/depth
```

**权重:1**


**参数:**

名称 | 类型 | 是否必须 | 描述
------------ | ------------ | ------------ | ------------
symbol | STRING | YES |
limit | INT | NO | 默认 60; 最大 60. 可选值:[5, 10, 20, 50, 60]


**响应:**
```javascript
{
  "lastUpdateId": 1027024,
  "bids": [
    [
      "4.00000000",     // 价位
      "431.00000000",   // 挂单量
      []                // 请忽略.
    ]
  ],
  "asks": [
    [
      "4.00000200",
      "12.00000000",
      []
    ]
  ]
}
```
### 期权深度信息
```
GET /api/v1/option/depth
```
**权重:1**  
**参数:**  

名称 | 类型 | 是否必须 | 描述  
------------ | ------------ | ------------ | ------------  
symbol | STRING | YES |  
limit | INT | NO | 默认 60; 最大 60. 可选值:[5, 10, 20, 50, 60]  
**响应:**  
```javascript
{
  "lastUpdateId": 1027024,
  "bids": [
    [
      "4.00000000",     // 价位
      "431.00000000",   // 挂单量
      []                // 请忽略.
    ]
  ],
  "asks": [
    [
      "4.00000200",
      "12.00000000",
      []
    ]
  ]
}
```
### 合约深度信息
```
GET /api/v1/contract/depth
```
**权重:1**  
**参数:**  

名称 | 类型 | 是否必须 | 描述
------------ | ------------ | ------------ | ------------
symbol | STRING | YES |
limit | INT | NO | 默认 60; 最大 60. 可选值:[5, 10, 20, 50, 60]  
**响应:**  
```javascript
{
  "lastUpdateId": 1027024,
  "bids": [
    [
      "4.00000000",     // 价位
      "431.00000000",   // 挂单量
      []                // 请忽略.
    ]
  ],
  "asks": [
    [
      "4.00000200",
      "12.00000000",
      []
    ]
  ]
}
```


### 币币近期成交
```
GET /api/v1/spot/trades
```
获取近期成交

**权重:**
1

**参数:**

Name | Type | Mandatory | Description
------------ | ------------ | ------------ | ------------
symbol | STRING | YES |
limit | INT | NO | Default 60; max 60.

**响应:**
```javascript
[
  {
    "price": "81.230000000000000000",
    "qty": "0.001000000000000000",
    "time": 0
  },
  {
    "price": "81.230000000000000000",
    "qty": "0.000400000000000000",
    "time": 0
  },
  {
    "price": "81.240000000000000000",
    "qty": "0.001100000000000000",
    "time": 0
  }
]
```
### 期权近期成交
```
GET /api/v1/option/trades
```
**权重:1**  
**参数:**  

Name | Type | Mandatory | Description
------------ | ------------ | ------------ | ------------
symbol | STRING | YES |
limit | INT | NO | Default 60; max 60.
  
**响应:**  
```javascript
[
  {
    "price": "81.230000000000000000",
    "qty": "0.001000000000000000",
    "time": 0
  },
  {
    "price": "81.230000000000000000",
    "qty": "0.000400000000000000",
    "time": 0
  },
  {
    "price": "81.240000000000000000",
    "qty": "0.001100000000000000",
    "time": 0
  }
]
```
### 合约近期成交
```
GET /api/v1/contract/trades
```
**权重:1**  
**参数:**  

Name | Type | Mandatory | Description
------------ | ------------ | ------------ | ------------
symbol | STRING | YES |
limit | INT | NO | Default 60; max 60.  
**响应:**  
```javascript
[
  {
    "price": "81.230000000000000000",
    "qty": "0.001000000000000000",
    "time": 0
  },
  {
    "price": "81.230000000000000000",
    "qty": "0.000400000000000000",
    "time": 0
  },
  {
    "price": "81.240000000000000000",
    "qty": "0.001100000000000000",
    "time": 0
  }
]
```



### 查询币币历史成交(MARKET_DATA)
```
GET /api/v1/spot/historicalTrades
```

**权重:**
5

**参数:**

Name | Type | Mandatory | Description
------------ | ------------ | ------------ | ------------
symbol | STRING | YES |
limit | INT | NO | Default 200; max 500.
fromId | LONG | NO | 从哪一条成交id开始返回. 缺省返回最近的成交记录

**响应:**
```javascript
[
  {
    "id": "3489",
    "price": "0.00001457",
    "qty": "6",
    "time": 1551129538000,
    "buyerMaker": false
  },
  {
    "id": "3488",
    "price": "0.00001464",
    "qty": "2",
    "time": 1551129500000,
    "buyerMaker": true
  }
]
```
### 查询期权历史成交(MARKET_DATA)
```
GET /api/v1/option/historicalTrades
```
**权重:5**  
**参数:**  

Name | Type | Mandatory | Description
------------ | ------------ | ------------ | ------------
symbol | STRING | YES |
limit | INT | NO | Default 200; max 500.
fromId | LONG | NO | 从哪一条成交id开始返回. 缺省返回最近的成交记录
  
**响应:**  
```javascript
[
  {
    "id": "3489",
    "price": "0.00001457",
    "qty": "6",
    "time": 1551129538000,
    "buyerMaker": false
  },
  {
    "id": "3488",
    "price": "0.00001464",
    "qty": "2",
    "time": 1551129500000,
    "buyerMaker": true
  }
]
```
### 查询合约历史成交(MARKET_DATA)
```
GET /api/v1/contract/historicalTrades
```
**权重:5**  
**参数:**  

Name | Type | Mandatory | Description
------------ | ------------ | ------------ | ------------
symbol | STRING | YES |
limit | INT | NO | Default 200; max 500.
fromId | LONG | NO | 从哪一条成交id开始返回. 缺省返回最近的成交记录  
**响应:**  
```javascript
[
  {
    "id": "3489",
    "price": "0.00001457",
    "qty": "6",
    "time": 1551129538000,
    "buyerMaker": false
  },
  {
    "id": "3488",
    "price": "0.00001464",
    "qty": "2",
    "time": 1551129500000,
    "buyerMaker": true
  }
]
```



### 币币K线数据
```
GET /api/v1/spot/klines
```
每根K线的开盘时间可视为唯一ID

**权重:**
1

**参数:**

Name | Type | Mandatory | Description
------------ | ------------ | ------------ | ------------
symbol | STRING | YES |
interval | ENUM | YES | "1m":分钟"3m":分钟"5m":分钟"15m":分钟"30m":分钟"1h":小时"2h":小时"4h":小时"6h":小时"12h":小时"1d":天"3d":天"1w":星期  
startTime | LONG | NO |
endTime | LONG | NO |
limit | INT | NO | Default 500; max 1000.

* 缺省返回最近的数据

**响应:**
```javascript
[
  [
    1499040000000,      // 开盘时间
    "0.01634790",       // 开盘价
    "0.80000000",       // 最高价
    "0.01575800",       // 最低价
    "0.01577100",       // 收盘价(当前K线未结束的即为最新价)
    "148976.11427815",  // 成交量
    1499644799999,      // 收盘时间
    "2434.19055334",    // 成交额
    308,                // 成交笔数
    "1756.87402397",    // 主动买入成交量
    "28.46694368",      // 主动买入成交额
    "17928899.62484339" // 请忽略该参数
  ]
]
```
### 查询期权K线数据
```
GET /api/v1/option/klines
```
**权重:1**  
**参数:**  

Name | Type | Mandatory | Description
------------ | ------------ | ------------ | ------------
symbol | STRING | YES |
interval | ENUM | YES | "1m":分钟"3m":分钟"5m":分钟"15m":分钟"30m":分钟"1h":小时"2h":小时"4h":小时"6h":小时"12h":小时"1d":天"3d":天"1w":星期  
startTime | LONG | NO |
endTime | LONG | NO |
limit | INT | NO | Default 500; max 1000.

* 缺省返回最近的数据
  
**响应:**  
```javascript
[
  [
    1499040000000,      // 开盘时间
    "0.01634790",       // 开盘价
    "0.80000000",       // 最高价
    "0.01575800",       // 最低价
    "0.01577100",       // 收盘价(当前K线未结束的即为最新价)
    "148976.11427815",  // 成交量
    1499644799999,      // 收盘时间
    "2434.19055334",    // 成交额
    308,                // 成交笔数
    "1756.87402397",    // 主动买入成交量
    "28.46694368",      // 主动买入成交额
    "17928899.62484339" // 请忽略该参数
  ]
]
```
### 查询合约K线数据
```
GET /api/v1/contract/klines
```
**权重:1**  
**参数:**  

Name | Type | Mandatory | Description
------------ | ------------ | ------------ | ------------
symbol | STRING | YES |
interval | ENUM | YES | "1m":分钟"3m":分钟"5m":分钟"15m":分钟"30m":分钟"1h":小时"2h":小时"4h":小时"6h":小时"12h":小时"1d":天"3d":天"1w":星期  
startTime | LONG | NO |
endTime | LONG | NO |
limit | INT | NO | Default 500; max 1000.

* 缺省返回最近的数据
  
**响应:**  
```javascript
[
  [
    1499040000000,      // 开盘时间
    "0.01634790",       // 开盘价
    "0.80000000",       // 最高价
    "0.01575800",       // 最低价
    "0.01577100",       // 收盘价(当前K线未结束的即为最新价)
    "148976.11427815",  // 成交量
    1499644799999,      // 收盘时间
    "2434.19055334",    // 成交额
    308,                // 成交笔数
    "1756.87402397",    // 主动买入成交量
    "28.46694368",      // 主动买入成交额
    "17928899.62484339" // 请忽略该参数
  ]
]
```





### 币币当前平均价格
```
GET /api/v1/spot/avgPrice
```
**权重:**
1
**参数:**

Name | Type | Mandatory | Description
------------ | ------------ | ------------ | ------------
symbol | STRING | YES |
**响应:**
```javascript
{
  "mins": 5,
  "price": "9.35751834"
}
```
### 查询期权当前平均价格
```
GET /api/v1/option/avgPrice
```
**权重:1**  
**参数:**  

Name | Type | Mandatory | Description
------------ | ------------ | ------------ | ------------
symbol | STRING | YES |  
**响应:**  
```javascript
{
  "mins": 5,
  "price": "9.35751834"
}
```
### 查询合约当前平均价格
```
GET /api/v1/contract/avgPrice
```
**权重:1**  
**参数:**  

Name | Type | Mandatory | Description
------------ | ------------ | ------------ | ------------
symbol | STRING | YES |  
**响应:**  
```javascript
{
  "mins": 5,
  "price": "9.35751834"
}
```







### 币币24hr价格变动情况
```
GET /api/v1/spot/ticker/24hr
```
请注意，不携带symbol参数会返回全部交易对数据，不仅数据庞大，而且权重极高

**权重:**
带symbol为1
不带为40

**参数:**

Name | Type | Mandatory | Description
------------ | ------------ | ------------ | ------------
symbol | STRING | NO |


**响应:**
```javascript
{
  "symbol": "BNBBTC",
  "priceChange": "1.00000000",
  "priceChangePercent": "50.00000000",
  "weightedAvgPrice": "1.88",
  "lastPrice": "3.00000000",
  "bidPrice": "1.00000000",
  "bidQty": "5.00000000",
  "askPrice": "3.00000000",
  "askQty": "9.00000000",
  "openPrice": "2.00000000",
  "highPrice": "18.00000000",
  "lowPrice": "1.00000000",
  "volume": "8.00000000",
  "quoteVolume": "15.00000000",
  "openTime": 1551174049782,
  "closeTime": 1551173957144
}
```
OR
```javascript
[
  {
    "symbol": "BNBBTC",
    "priceChange": "1.00000000",
    "priceChangePercent": "50.00000000",
    "weightedAvgPrice": "1.88",
    "lastPrice": "3.00000000",
    "bidPrice": "1.00000000",
    "bidQty": "5.00000000",
    "askPrice": "3.00000000",
    "askQty": "9.00000000",
    "openPrice": "2.00000000",
    "highPrice": "18.00000000",
    "lowPrice": "1.00000000",
    "volume": "8.00000000",
    "quoteVolume": "15.00000000",
    "openTime": 1551174049782,
    "closeTime": 1551173957144
  }
]
```
### 查询期权24hr价格变动情况
```
GET /api/v1/option/ticker/24hr
```
**权重:**  
带symbol为1
不带为40  
**参数:**  

Name | Type | Mandatory | Description
------------ | ------------ | ------------ | ------------
symbol | STRING | NO |
  
**响应:**  
```javascript
{
  "symbol": "BNBBTC",
  "priceChange": "1.00000000",
  "priceChangePercent": "50.00000000",
  "weightedAvgPrice": "1.88",
  "lastPrice": "3.00000000",
  "bidPrice": "1.00000000",
  "bidQty": "5.00000000",
  "askPrice": "3.00000000",
  "askQty": "9.00000000",
  "openPrice": "2.00000000",
  "highPrice": "18.00000000",
  "lowPrice": "1.00000000",
  "volume": "8.00000000",
  "quoteVolume": "15.00000000",
  "openTime": 1551174049782,
  "closeTime": 1551173957144
}
```
OR
```javascript
[
  {
    "symbol": "BNBBTC",
    "priceChange": "1.00000000",
    "priceChangePercent": "50.00000000",
    "weightedAvgPrice": "1.88",
    "lastPrice": "3.00000000",
    "bidPrice": "1.00000000",
    "bidQty": "5.00000000",
    "askPrice": "3.00000000",
    "askQty": "9.00000000",
    "openPrice": "2.00000000",
    "highPrice": "18.00000000",
    "lowPrice": "1.00000000",
    "volume": "8.00000000",
    "quoteVolume": "15.00000000",
    "openTime": 1551174049782,
    "closeTime": 1551173957144
  }
]
```
### 查询合约24hr价格变动情况
```
GET /api/v1/contract/ticker/24hr
```
**权重:**  
带symbol为1
不带为40  
**参数:**  

Name | Type | Mandatory | Description
------------ | ------------ | ------------ | ------------
symbol | STRING | NO |
  
**响应:**  
```javascript
{
  "symbol": "BNBBTC",
  "priceChange": "1.00000000",
  "priceChangePercent": "50.00000000",
  "weightedAvgPrice": "1.88",
  "lastPrice": "3.00000000",
  "bidPrice": "1.00000000",
  "bidQty": "5.00000000",
  "askPrice": "3.00000000",
  "askQty": "9.00000000",
  "openPrice": "2.00000000",
  "highPrice": "18.00000000",
  "lowPrice": "1.00000000",
  "volume": "8.00000000",
  "quoteVolume": "15.00000000",
  "openTime": 1551174049782,
  "closeTime": 1551173957144
}
```
OR
```javascript
[
  {
    "symbol": "BNBBTC",
    "priceChange": "1.00000000",
    "priceChangePercent": "50.00000000",
    "weightedAvgPrice": "1.88",
    "lastPrice": "3.00000000",
    "bidPrice": "1.00000000",
    "bidQty": "5.00000000",
    "askPrice": "3.00000000",
    "askQty": "9.00000000",
    "openPrice": "2.00000000",
    "highPrice": "18.00000000",
    "lowPrice": "1.00000000",
    "volume": "8.00000000",
    "quoteVolume": "15.00000000",
    "openTime": 1551174049782,
    "closeTime": 1551173957144
  }
]
```









### 币币最新价格接口
```
GET /api/v1/spot/ticker/price
```
返回最近价格

**权重:**
单交易对1
无交易对2


**参数:**

Name | Type | Mandatory | Description
------------ | ------------ | ------------ | ------------
symbol | STRING | NO |

* 不发送交易对参数，则会返回所有交易对信息

**响应:**
```javascript
{
  "symbol": "LTCBTC",
  "price": "4.00000200"
}
```
OR
```javascript
[
  {
    "symbol": "LTCBTC",
    "price": "4.00000200"
  },
  {
    "symbol": "ETHBTC",
    "price": "0.07946600"
  }
]
```
### 查询期权最新价格接口
```
GET /api/v1/option/ticker/price
```
**权重:**  
单交易对1
无交易对2  
**参数:**  

Name | Type | Mandatory | Description
------------ | ------------ | ------------ | ------------
symbol | STRING | NO |

* 不发送交易对参数，则会返回所有交易对信息
  
**响应:**  
```javascript
{
  "symbol": "LTCBTC",
  "price": "4.00000200"
}
```
OR
```javascript
[
  {
    "symbol": "LTCBTC",
    "price": "4.00000200"
  },
  {
    "symbol": "ETHBTC",
    "price": "0.07946600"
  }
]
```
### 查询合约最新价格接口
```
GET /api/v1/contract/ticker/price
```
**权重:**  
单交易对1
无交易对2  
**参数:**  

Name | Type | Mandatory | Description
------------ | ------------ | ------------ | ------------
symbol | STRING | NO |

* 不发送交易对参数，则会返回所有交易对信息
  
**响应:**  
```javascript
{
  "symbol": "LTCBTC",
  "price": "4.00000200"
}
```
OR
```javascript
[
  {
    "symbol": "LTCBTC",
    "price": "4.00000200"
  },
  {
    "symbol": "ETHBTC",
    "price": "0.07946600"
  }
]
```







### 币币最优挂单接口
```
GET /api/v1/spot/ticker/bookTicker
```
返回当前最优的挂单(最高买单，最低卖单)

**权重:**
单交易对1
无交易对2

**参数:**

Name | Type | Mandatory | Description
------------ | ------------ | ------------ | ------------
symbol | STRING | NO |

* 不发送交易对参数，则会返回所有交易对信息

**响应:**
```javascript
{
  "symbol": "LTCBTC",
  "bidPrice": "4.00000000",//最优买单价
  "bidQty": "431.00000000",//挂单量
  "askPrice": "4.00000200",//最优卖单价
  "askQty": "9.00000000"//挂单量
}
```
OR
```javascript
[
  {
    "symbol": "LTCBTC",
    "bidPrice": "4.00000000",
    "bidQty": "431.00000000",
    "askPrice": "4.00000200",
    "askQty": "9.00000000"
  },
  {
    "symbol": "ETHBTC",
    "bidPrice": "0.07946700",
    "bidQty": "9.00000000",
    "askPrice": "100000.00000000",
    "askQty": "1000.00000000"
  }
]
```
### 查询期权最优挂单接口
```
GET /api/v1/option/ticker/bookTicker
```
返回当前最优的挂单(最高买单，最低卖单)
**权重:**  
单交易对1
无交易对2  
**参数:**  

Name | Type | Mandatory | Description
------------ | ------------ | ------------ | ------------
symbol | STRING | NO |

* 不发送交易对参数，则会返回所有交易对信息  
**响应:**  
```javascript
{
  "symbol": "LTCBTC",
  "bidPrice": "4.00000000",//最优买单价
  "bidQty": "431.00000000",//挂单量
  "askPrice": "4.00000200",//最优卖单价
  "askQty": "9.00000000"//挂单量
}
```
OR
```javascript
[
  {
    "symbol": "LTCBTC",
    "bidPrice": "4.00000000",
    "bidQty": "431.00000000",
    "askPrice": "4.00000200",
    "askQty": "9.00000000"
  },
  {
    "symbol": "ETHBTC",
    "bidPrice": "0.07946700",
    "bidQty": "9.00000000",
    "askPrice": "100000.00000000",
    "askQty": "1000.00000000"
  }
]
```
### 查询合约最优挂单接口
```
GET /api/v1/contract/ticker/bookTicker
```
返回当前最优的挂单(最高买单，最低卖单)
**权重:**  
单交易对1
无交易对2  
**参数:**  

Name | Type | Mandatory | Description
------------ | ------------ | ------------ | ------------
symbol | STRING | NO |

* 不发送交易对参数，则会返回所有交易对信息  
**响应:**  
```javascript
{
  "symbol": "LTCBTC",
  "bidPrice": "4.00000000",//最优买单价
  "bidQty": "431.00000000",//挂单量
  "askPrice": "4.00000200",//最优卖单价
  "askQty": "9.00000000"//挂单量
}
```
OR
```javascript
[
  {
    "symbol": "LTCBTC",
    "bidPrice": "4.00000000",
    "bidQty": "431.00000000",
    "askPrice": "4.00000200",
    "askQty": "9.00000000"
  },
  {
    "symbol": "ETHBTC",
    "bidPrice": "0.07946700",
    "bidQty": "9.00000000",
    "askPrice": "100000.00000000",
    "askQty": "1000.00000000"
  }
]
```

### 合约的指数价格，标记价格
```
GET /api/v1/contract/ticker/indicesPrice
```
返回当前最优的挂单(最高买单，最低卖单)

**权重:**
1


**参数:**

Name | Type | Mandatory | Description
------------ | ------------ | ------------ | ------------
symbol | STRING | NO |

* 不发送交易对参数，则会返回所有交易对信息

**响应:**
```javascript
{
  "symbol": "BTCUSDT",
  "price": "2600.0000",
  "indexPrice": "2300.0000",
  "markPrice": "2300.0023"
}
```
OR
```javascript
[
  {
    "symbol": "EOSUSDT",
    "price": "2301.5491",
    "indexPrice": "2300.0000",
    "markPrice": "2300.7659"
  },
  {
    "symbol": "BTCUSDT",
    "price": "2600.0000",
    "indexPrice": "2300.0000",
    "markPrice": "2300.0023"
  }
]
```




## 账户接口
### 币币下单  (TRADE)
```
POST /api/v1/spot/order  (HMAC SHA256)
```

**权重:**
1

**参数:**

Name | Type | Mandatory | Description
------------ | ------------ | ------------ | ------------
symbol | STRING | YES |
side | ENUM | YES |
type | ENUM | YES | 目前只有LIMIT
timeInForce | ENUM | NO | 暂时没用
quantity | DECIMAL | YES |
price | DECIMAL | NO |
newClientOrderId | STRING | NO | 用户自定义的orderid，如空缺系统会自动赋值
stopPrice | DECIMAL | NO | 暂时没用
icebergQty | DECIMAL | NO | 暂时没用
newOrderRespType | ENUM | NO | 指定响应类型 `ACK`, `RESULT`; 默认为`ACK`. 
recvWindow | LONG | NO |
timestamp | LONG | YES |

根据 order `type`的不同，某些参数强制要求，具体如下:

Type | 强制要求的参数
------------ | ------------
`LIMIT` | `timeInForce`, `quantity`, `price`


关于 newOrderRespType的俩种选择

**Response ACK:**
返回速度快，不包含成交信息，信息量最少
```javascript
{
  "symbol": "JEXBTC",
  "orderId": 28,
  "transactTime": 1507725176595
}
```

**Response RESULT:**
返回速度慢，返回吃单成交的少量信息
```javascript
{
  "symbol": "JEXBTC",
  "orderId": 2208,
  "transactTime": 1551184078060,
  "price": "0.00001464",
  "origQty": "1",
  "executedQty": "0",
  "cummulativeQuoteQty": "0.00000000",
  "status": "NEW",
  "timeInForce": "GTC",
  "type": "LIMIT",
  "side": "BUY"
}
```
### 期权下单  (TRADE)
```
POST /api/v1/option/order  (HMAC SHA256)
```

**权重:**  
1  
**参数:**

Name | Type | Mandatory | Description
------------ | ------------ | ------------ | ------------
symbol | STRING | YES |
side | ENUM | YES |
type | ENUM | YES | `LIMIT`,`MARKET`
timeInForce | ENUM | NO | 暂时没用
quantity | DECIMAL | YES |
price | DECIMAL | NO |
newClientOrderId | STRING | NO | 用户自定义的orderid，如空缺系统会自动赋值
newOrderRespType | ENUM | NO | 指定响应类型 `ACK`, `RESULT`; 默认为`ACK`. 
recvWindow | LONG | NO |
timestamp | LONG | YES |
**响应:**  
根据 order `type`的不同，某些参数强制要求，具体如下:

Type | 强制要求的参数
------------ | ------------
`LIMIT` | `timeInForce`, `quantity`, `price`


关于 newOrderRespType的俩种选择

**Response ACK:**
返回速度快，不包含成交信息，信息量最少
```javascript
{
  "symbol": "JEXBTC",
  "orderId": 28,
  "transactTime": 1507725176595
}
```

**Response RESULT:**
返回速度慢，返回吃单成交的少量信息
```javascript
{
  "symbol": "JEXBTC",
  "orderId": 2208,
  "transactTime": 1551184078060,
  "price": "0.00001464",
  "origQty": "1",
  "executedQty": "0",
  "cummulativeQuoteQty": "0.00000000",
  "status": "NEW",
  "timeInForce": "GTC",
  "type": "LIMIT",
  "side": "BUY"
}
```


### 合约下单  (TRADE)
```
POST /api/v1/contract/order  (HMAC SHA256)
```

**权重:**
1
**参数:**

Name | Type | Mandatory | Description
------------ | ------------ | ------------ | ------------
symbol | STRING | YES |
side | ENUM | YES |
type | ENUM | YES | `LIMIT`,`MARKET`
timeInForce | ENUM | NO | 暂时没用
quantity | DECIMAL | YES |
price | DECIMAL | NO |
newClientOrderId | STRING | NO | 用户自定义的orderid，如空缺系统会自动赋值
newOrderRespType | ENUM | NO | 指定响应类型 `ACK`, `RESULT`; 默认为`ACK`. 
recvWindow | LONG | NO |
timestamp | LONG | YES |


**Response ACK:**
返回速度快，不包含成交信息，信息量最少
```javascript
{
  "symbol": "BTCUSDT",
  "orderId": "4613019726031880200",
}
```

**Response RESULT:**
返回速度慢，返回吃单成交的少量信息
```javascript
{
  "symbol": "BTCUSDT",
  "orderId": "4613019726031880199",
  "side": "buy",
  "origQty": "1.0000",
  "executedQty": "0",
  "price": "3800",
  "status": "entrusting",
  "type": "limit"
}
```






### 币币测试下单接口 (TRADE)
```
POST /api/v1/spot/order/test (HMAC SHA256)
```
用于测试订单请求，但不会提交到撮合引擎

**权重:**
1

**参数:**

参考 `POST /api/v1/spot/order`


**响应:**
```javascript
{}
```

### 期权测试下单接口 (TRADE)
```
POST /api/v1/option/order/test (HMAC SHA256)
```
用于测试订单请求，但不会提交到撮合引擎

**权重:**
1

**参数:**

参考 `POST /api/v1/option/order`


**响应:**
```javascript
{}
```
### 合约测试下单接口 (TRADE)
```
POST /api/v1/contract/order/test (HMAC SHA256)
```
用于测试订单请求，但不会提交到撮合引擎

**权重:**
1

**参数:**

参考 `POST /api/v1/contract/order`


**响应:**
```javascript
{}
```





### 币币查询订单 (USER_DATA)
```
GET /api/v1/spot/order (HMAC SHA256)
```
查询订单状态

**权重:**
1

**参数:**

Name | Type | Mandatory | Description
------------ | ------------ | ------------ | ------------
symbol | STRING | YES |
orderId | LONG | NO |
origClientOrderId | STRING | NO | 
recvWindow | LONG | NO |
timestamp | LONG | YES |

注意:
* 至少需要发送 `orderId` 与 `origClientOrderId`中的一个


**响应:**
```javascript
{
  "symbol": "JEXBTC",
  "orderId": "2208",
  "price": "0.00001464",
  "origQty": "1.00000000",
  "executedQty": "1.00000000",
  "cummulativeQuoteQty": "0.00001464",
  "status": "FILLED",
  "timeInForce": "GTC",
  "type": "LIMIT",
  "side": "BUY",
  "time": 1551184037000,
  "updateTime": 1551184037000,
  "working": true
}
```

### 期权查询订单 (USER_DATA)
```
GET /api/v1/option/order (HMAC SHA256)
```
查询订单状态

**权重:**
1

**参数:**

Name | Type | Mandatory | Description
------------ | ------------ | ------------ | ------------
symbol | STRING | YES |
orderId | LONG | NO |
origClientOrderId | STRING | NO | 
recvWindow | LONG | NO |
timestamp | LONG | YES |

注意:
* 至少需要发送 `orderId` 与 `origClientOrderId`中的一个


**响应:**
```javascript
{
  "symbol": "BTCCALLM",
  "orderId": "61292",
  "price": "2.00",
  "origQty": "1.00",
  "executedQty": "1.00",
  "cummulativeQuoteQty": "2.00",
  "status": "FILLED",
  "timeInForce": "GTC",
  "type": "LIMIT",
  "side": "BUY",
  "time": 1551184924000,
  "updateTime": 1551234873000,
  "working": true
}
```

### 合约查询订单 (USER_DATA)
```
GET /api/v1/contract/order (HMAC SHA256)
```
查询订单状态

**权重:**
1

**参数:**

Name | Type | Mandatory | Description
------------ | ------------ | ------------ | ------------
symbol | STRING | YES |
orderId | LONG | NO |
origClientOrderId | STRING | NO | 
recvWindow | LONG | NO |
timestamp | LONG | YES |

注意:
* 至少需要发送 `orderId` 与 `origClientOrderId`中的一个


**响应:**
```javascript
{
  "symbol": "BTCUSDT",
  "orderId": "4613019726031880200",
  "updateTime": 1551257935000,
  "side": "BUY",
  "origQty": "1.0000",
  "executedQty": "0.0000",
  "price": "3800",
  "executedPrice": null,
  "status": "",
  "time": 1551257976000,
  "reject": null,
  "type": "LIMIT"
}
```





### 币币撤销订单 (TRADE)
```
DELETE /api/v1/spot/order  (HMAC SHA256)
```

**权重:**
1

**Parameters:**

Name | Type | Mandatory | Description
------------ | ------------ | ------------ | ------------
symbol | STRING | YES |
orderId | LONG | NO |
origClientOrderId | STRING | NO |
newClientOrderId | STRING | NO |  用户自定义的本次撤销操作的ID(注意不是被撤销的订单的自定义ID)。如无指定会自动赋值。
recvWindow | LONG | NO |
timestamp | LONG | YES |

`orderId` 与 `origClientOrderId` 必须至少发送一个

**响应:**
```javascript
{
  "symbol": "JEXBTC",
  "orderId": 75,
  "transactTime": 1551238738296,
  "price": "0.00001407",
  "origQty": "4",
  "executedQty": "0",
  "cummulativeQuoteQty": "0.00000000",
  "status": "CANCELED",
  "timeInForce": "GTC",
  "type": "LIMIT",
  "side": "BUY"
}
```

### 期权撤销订单 (TRADE)
```
DELETE /api/v1/option/order  (HMAC SHA256)
```

**权重:**
1

**Parameters:**  

Name | Type | Mandatory | Description
------------ | ------------ | ------------ | ------------
symbol | STRING | YES |
orderId | LONG | NO |
origClientOrderId | STRING | NO |
newClientOrderId | STRING | NO |  用户自定义的本次撤销操作的ID(注意不是被撤销的订单的自定义ID)。如无指定会自动赋值。
recvWindow | LONG | NO |
timestamp | LONG | YES |

`orderId` 与 `origClientOrderId` 必须至少发送一个




**响应:**
```javascript
{
  "symbol": "BTCCALLM",
  "orderId": 61299,
  "transactTime": 1551239198405,
  "price": "2.00",
  "origQty": "1",
  "executedQty": "0",
  "cummulativeQuoteQty": "0.00",
  "status": "CANCELED",
  "timeInForce": "GTC",
  "type": "LIMIT",
  "side": "BUY"
}
```

### 合约撤销订单 (TRADE)
```
DELETE /api/v1/contract/order  (HMAC SHA256)
```

**权重:**
1

**Parameters:**

Name | Type | Mandatory | Description
------------ | ------------ | ------------ | ------------
symbol | STRING | YES |
orderId | LONG | NO |
origClientOrderId | STRING | NO |
newClientOrderId | STRING | NO |  用户自定义的本次撤销操作的ID(注意不是被撤销的订单的自定义ID)。如无指定会自动赋值。
recvWindow | LONG | NO |
timestamp | LONG | YES |

`orderId` 与 `origClientOrderId` 必须至少发送一个



**响应:**
```javascript
{
  "symbol": "BTCUSDT",
  "orderId": "4613019726031880200",
  "side": "buy",
  "origQty": "1.00000000000000000000",
  "executedQty": "0.00000000000000000000",
  "price": "3800.00000000000000000000",
  "status": "entrusted",
  "type": "limit"
}
```








### 查看账户当前币币挂单 (USER_DATA)
```
GET /api/v1/spot/openOrders  (HMAC SHA256)
```

**权重:**  
 5


**参数:**

Name | Type | Mandatory | Description
------------ | ------------ | ------------ | ------------
symbol | STRING | YES |
orderId | LONG | NO | 只返回此orderID之后的订单，缺省返回最近的订单
startTime | LONG | NO |
endTime | LONG | NO |
limit | INT | NO | Default 500; max 500.
recvWindow | LONG | NO |
timestamp | LONG | YES |


**响应:**
```javascript
[
  {
    "symbol": "JEXBTC",
    "orderId": "76",
    "price": "0.00001406",
    "origQty": "5.00000000",
    "executedQty": "0.00000000",
    "cummulativeQuoteQty": "0.00000000",
    "status": "NEW",
    "timeInForce": "GTC",
    "type": "LIMIT",
    "side": "BUY",
    "time": 1550659007000,
    "updateTime": 1550659007000,
    "working": true
  }
]
```

### 查看账户当前期权挂单 (USER_DATA)
```
GET /api/v1/option/openOrders  (HMAC SHA256)
```

**权重:**
5

**参数:**

Name | Type | Mandatory | Description
------------ | ------------ | ------------ | ------------
symbol | STRING | YES |
orderId | LONG | NO | 只返回此orderID之后的订单，缺省返回最近的订单
startTime | LONG | NO |
endTime | LONG | NO |
limit | INT | NO | Default 500; max 500.
recvWindow | LONG | NO |
timestamp | LONG | YES |


**响应:**
```javascript
[
  {
    "symbol": "JEXBTC",
    "orderId": "76",
    "price": "0.00001406",
    "origQty": "5.00000000",
    "executedQty": "0.00000000",
    "cummulativeQuoteQty": "0.00000000",
    "status": "NEW",
    "timeInForce": "GTC",
    "type": "LIMIT",
    "side": "BUY",
    "time": 1550659007000,
    "updateTime": 1550659007000,
    "working": true
  }
]
```

### 查看账户当前合约挂单 (USER_DATA)
```
GET /api/v1/contract/openOrders  (HMAC SHA256)
```

**权重:**
5

**参数:**

Name | Type | Mandatory | Description
------------ | ------------ | ------------ | ------------
symbol | STRING | YES |
orderId | LONG | NO | 只返回此orderID之后的订单，缺省返回最近的订单
startTime | LONG | NO |
endTime | LONG | NO |
limit | INT | NO | Default 500; max 500.
recvWindow | LONG | NO |
timestamp | LONG | YES |


**响应:**
```javascript
[
  {
    "symbol": "JEXBTC",
    "orderId": "76",
    "price": "0.00001406",
    "origQty": "5.00000000",
    "executedQty": "0.00000000",
    "cummulativeQuoteQty": "0.00000000",
    "status": "NEW",
    "timeInForce": "GTC",
    "type": "LIMIT",
    "side": "BUY",
    "time": 1550659007000,
    "updateTime": 1550659007000,
    "working": true
  }
]
```

### 合约平仓 (TRADE)
```
POST /api/v1/contract/liquidation  (HMAC SHA256)
```

**权重:**
1

**参数:**

Name | Type | Mandatory | Description
------------ | ------------ | ------------ | ------------
symbol | STRING | YES |
recvWindow | LONG | NO |
timestamp | LONG | YES |


**响应:**
```javascript
{
  "symbol": "EOSUSDT",
  "orderId": "4613030721148158029",
  "side": "sell",
  "origQty": "4.0000",
  "status": "entrusting",
  "type": "market"
}
```

### 查看账户合约平仓单 (MARKET_DATA	)
```
GET /api/v1/contract/liquidationOrder  
```

**权重:**
5

**参数:**

Name | Type | Mandatory | Description
------------ | ------------ | ------------ | ------------
symbol | STRING | YES |
orderId | LONG | NO | 只返回此orderID之后的订单，缺省返回最近的订单
startTime | LONG | NO |
endTime | LONG | NO |
limit | INT | NO | Default 500; max 500.
recvWindow | LONG | NO |
timestamp | LONG | YES |


**响应:**
```javascript
[
  {
    "symbol": "EOSUSDT",
    "orderId": "2251799813685248011",
    "updateTime": 1551083505000,
    "side": "BUY",
    "origQty": "2.0515",
    "executedQty": "2.0515",
    "price": "2527.9188",
    "executedPrice": "2527.9188",
    "status": "FILLED",
    "time": 1551083505000,
    "reject": null,
    "type": "FLAT"
  }
]
```

### 查看账户合约仓位 (USER_DATA)
```
GET /api/v1/contract/position  (HMAC SHA256)
```

**权重:**
2

**参数:**

Name | Type | Mandatory | Description
------------ | ------------ | ------------ | ------------
symbol | STRING | NO |
recvWindow | LONG | NO |
timestamp | LONG | YES |


**响应:**
```javascript
[
  {
    "symbolName": "EOSUSDT",
    "direction": "longs",
    "leverage": "10.00",
    "currentQuantity": "1.0000",
    "positionMargin": "725.9045",
    "positionCost": "2300.6803",
    "costPrice": "2300.6803",
    "liquidationPrice": "1594.9068",
    "profit": "-1.5161",
    "profitRate": "-0.0020885667",
    "risk": null,
    "initialMargin": "0.1000",
    "maintenanceMargin": "0.0050",
    "buyOrderCost": "0.0000",
    "sellOrderCost": "0.0000",
    "position": null
  }
]
```

### 调整账户合约杠杆 (USER_DATA)
```
GET /api/v1/contract/position/leverage  (HMAC SHA256)
```

**权重:**
1

**参数:**

Name | Type | Mandatory | Description
------------ | ------------ | ------------ | ------------
symbol | STRING | YES |
leverage | STRING | YES | 
recvWindow | LONG | NO |
timestamp | LONG | YES |


**响应:**
```javascript
{}
```



### 账户信息 (USER_DATA)
```
GET /api/v1/account (HMAC SHA256)
```

**权重:**
1

**参数:**

Name | Type | Mandatory | Description
------------ | ------------ | ------------ | ------------
recvWindow | LONG | NO |
timestamp | LONG | YES |

**响应:**
```javascript
{
    "updateTime": 1523093528000
    "contractBalances": [
        {
            "asset": "usdt",
            "canDeposit": false,
            "canTrade": true,
            "canWithdraw": false,
            "free": "8144.0412",
            "locked": "1855.3288"
        }
    ],
    "optionBalances": [
        {
            "asset": "BCXBTC01",
            "canDeposit": false,
            "canTrade": true,
            "canWithdraw": false,
            "free": "0",
            "locked": "0"
        }
    ],
    "spotBalances": [
        {
            "asset": "BTC",
            "canDeposit": true,
            "canTrade": true,
            "canWithdraw": false,
            "free": "999616.1363",
            "locked": "0.7219"
        }
    ],
}
```


### 账户币币历史委托 (USER_DATA)
```
GET /api/v1/spot/historyOrders  (HMAC SHA256)
```
获取某交易对的成交历史

**权重:**
5

**参数:**

Name | Type | Mandatory | Description
------------ | ------------ | ------------ | ------------
symbol | STRING | YES |
startTime | LONG | NO |
endTime | LONG | NO |
orderId | LONG | NO |返回该orderId之后的成交，缺省返回最近的成交
limit | INT | NO | Default 500; max 500.
recvWindow | LONG | NO |
timestamp | LONG | YES |

**响应:**
```javascript
[
  {
    "symbol": "JEXBTC",
    "orderId": "15",
    "price": "0.00001490",
    "origQty": "3.00000000",
    "executedQty": "3.00000000",
    "cummulativeQuoteQty": "0.00004470",
    "status": "FILLED",
    "timeInForce": "GTC",
    "type": "LIMIT",
    "side": "BUY",
    "time": 1550643800000,
    "updateTime": 1550644851000,
    "working": true
  }
]
```

### 账户期权历史委托 (USER_DATA)
```
GET /api/v1/option/historyOrders  (HMAC SHA256)
```
获取某交易对的成交历史

**权重:**
5

**参数:**

Name | Type | Mandatory | Description
------------ | ------------ | ------------ | ------------
symbol | STRING | YES |
startTime | LONG | NO |
endTime | LONG | NO |
orderId | LONG | NO |返回该orderId之后的成交，缺省返回最近的成交
limit | INT | NO | Default 500; max 500.
recvWindow | LONG | NO |
timestamp | LONG | YES |

**响应:**
```javascript
[
  {
    "symbol": "BTCCALLM",
    "orderId": "7",
    "price": "0.02",
    "origQty": "2.00",
    "executedQty": "0.00",
    "cummulativeQuoteQty": "0.00",
    "status": "CANCELED",
    "timeInForce": "GTC",
    "type": "LIMIT",
    "side": "BUY",
    "time": 1550564707000,
    "updateTime": 1551085162000,
    "working": true
  }
]
```

### 账户合约历史委托 (USER_DATA)
```
GET /api/v1/contract/historyOrders  (HMAC SHA256)
```
获取某交易对的成交历史

**权重:**
5

**参数:**

Name | Type | Mandatory | Description
------------ | ------------ | ------------ | ------------
symbol | STRING | YES |
startTime | LONG | NO |
endTime | LONG | NO |
orderId | LONG | NO |返回该orderId之后的成交，缺省返回最近的成交
limit | INT | NO | Default 500; max 500.
recvWindow | LONG | NO |
timestamp | LONG | YES |

**响应:**
```javascript
[
  {
    "symbol": "BTCUSDT",
    "orderId": "4613012029450485988",
    "updateTime": 1551246891000,
    "side": "BUY",
    "origQty": "2.0000",
    "executedQty": "2.0000",
    "price": "3800",
    "executedPrice": "3800",
    "status": "FILLED",
    "time": 1551185663000,
    "reject": null,
    "type": "LIMIT"
  }
]
```



### 用户发行期权 (TRADE)
```
POST /api/v1/option/release  (HMAC SHA256)
```
发行某个交易对的期权

**权重:**
1

**参数:**

Name | Type | Mandatory | Description
------------ | ------------ | ------------ | ------------
symbol | STRING | YES |
amount | INT | YES |
recvWindow | LONG | NO |
timestamp | LONG | YES |

**响应:**
```javascript
{
  "assetMargin": "BTC",
  "assetBase": "JEX",
  "assetOption": "BTCCALLM",
  "releaseAmount": "17780",
  "backAmount": "0",
  "marginAmount": "177.8000",
  "baseAmount": "17780.000",
  "availableMargin": "999616.1163",
  "availableBase": "267400.836",
  "availableOption": "17785",
  "createdDate": 1551428811520
}
```

### 用户赎回期权 (TRADE)
```
POST /api/v1/option/back  (HMAC SHA256)
```
赎回某个交易对的期权

**权重:**
1

**参数:**

Name | Type | Mandatory | Description
------------ | ------------ | ------------ | ------------
symbol | STRING | YES |
amount | INT | YES |
recvWindow | LONG | NO |
timestamp | LONG | YES |

**响应:**
```javascript
{
  "assetMargin": "BTC",
  "assetBase": "JEX",
  "assetOption": "BTCCALLM",
  "releaseAmount": "17780",
  "backAmount": "2",
  "marginAmount": "177.7800",
  "baseAmount": "17778.000",
  "availableMargin": "999616.1363",
  "availableBase": "267402.836",
  "availableOption": "17783",
  "createdDate": 1551668068964
}
```

### 账户期权持仓 (USER_DATA)
```
GET /api/v1/option/position  (HMAC SHA256)
```
获取某个期权交易对或全部的持仓

**权重:**
2

**参数:**

Name | Type | Mandatory | Description
------------ | ------------ | ------------ | ------------
symbol | STRING | NO |
recvWindow | LONG | NO |
timestamp | LONG | YES |

**响应:**
```javascript
[
  {
    "name": "BTCCALLM",
    "amount": "5",
    "availAmount": "5",
    "profit": "15.99999996",
    "profitRate": "1.14285714",
    "cost": "2.80",
    "type": "CALL",
    "execTime": 1553763600000,
    "execrPrice": "5.00",
    "orderIndex": 0
  }
]
```

### 账户期权发行赎回信息 (USER_DATA)
```
GET /api/v1/option/info  (HMAC SHA256)
```
获取账户某个期权的发行赎回信息

**权重:**
1

**参数:**

Name | Type | Mandatory | Description
------------ | ------------ | ------------ | ------------
symbol | STRING | YES |
recvWindow | LONG | NO |
timestamp | LONG | YES |

**响应:**
```javascript
{
  "assetMargin": "BTC",
  "assetBase": "JEX",
  "assetOption": "BTCCALLM",
  "releaseAmount": "17780",
  "backAmount": "2",
  "marginAmount": "177.78",
  "baseAmount": "17778.00",
  "availableMargin": "999616.1363",
  "availableBase": "267402.836",
  "availableOption": "17783"
}
```

### 账户期权发行赎回记录 (USER_DATA)
```
GET /api/v1/option/record  (HMAC SHA256)
```
获取账户某个期权或者全部的发行赎回交易记录

**权重:**
5

**参数:**

Name | Type | Mandatory | Description
------------ | ------------ | ------------ | ------------
symbol | STRING | YES |
startTime | LONG | NO |
endTime | LONG | NO |
fromId | LONG | NO |返回该orderId之后的成交，缺省返回最近的成交
limit | INT | NO | Default 500; max 500.
recvWindow | LONG | NO |
timestamp | LONG | YES |

**响应:**
```javascript
[
  {
    "createdDate": 1551668069000,
    "name": "BTCCALLM",
    "status": 2,
    "amount": "2",
    "convertRatio": "0.10000000",
    "rateMargin": "0.10",
    "baseAmount": "2.000",
    "marginAmount": "0.0200",
    "assetMargin": "BTC",
    "assetBase": "JEX",
    "id": 21821
  }
]
```



### 账户合约账单 (USER_DATA)
```
GET /api/v1/contract/bill  (HMAC SHA256)
```
获取账户的合约账单

**权重:**
1

**参数:**

Name | Type | Mandatory | Description
------------ | ------------ | ------------ | ------------
startTime | LONG | NO |
endTime | LONG | NO |
fromId | LONG | NO |返回该orderId之后的成交，Default -1(返回最近的成交)
limit | INT | NO | Default 100; max 100.
recvWindow | LONG | NO |
timestamp | LONG | YES |

**响应:**
```javascript
[
  {
    "id": 854797,
    "amount": 0.662,
    "createdDate": "2019-02-27 13:54:46",
    "symbol": "BTCUSDT",
    "type": "positionSize"
  },
  {
    "id": 854875,
    "amount": 1.338,
    "createdDate": "2019-02-27 13:55:32",
    "symbol": "BTCUSDT",
    "type": "positionSize"
  }
]
```

### 合约资金费率
```
GET /api/v1/contract/historyRate  
```
获取某合约的资金费率

**权重:**
1

**参数:**

Name | Type | Mandatory | Description
------------ | ------------ | ------------ | ------------
symbol | STRING | YES |
startTime | LONG | NO |
endTime | LONG | NO |
fromId | LONG | NO |返回该fromId之后的记录，Default -1(返回最近的记录)
limit | INT | NO | Default 100; max 100.
recvWindow | LONG | NO |
timestamp | LONG | YES |

**响应:**
```javascript
[
  {
    "id": 14415,
    "fundingRate": 0.0001,
    "fundingRateDay": 0.0003,
    "symbol": "EOSUSDT",
    "date": 1550833084000
  },
  {
    "id": 14447,
    "fundingRate": 0.0001,
    "fundingRateDay": 0.0003,
    "symbol": "EOSUSDT",
    "date": 1550851200000
  }
]
```


### 合约保护基金 
```
GET /api/v1/contract/protectionFund  
```
获取某合约的保护基金

**权重:**
1

**参数:**

Name | Type | Mandatory | Description
------------ | ------------ | ------------ | ------------
symbol | STRING | YES |
startTime | LONG | NO |
endTime | LONG | NO |
start | LONG | NO |返回该start之后的记录，Default -1(返回最近的记录)
size | INT | NO | Default 100; max 100.
recvWindow | LONG | NO |
timestamp | LONG | YES |

**响应:**
```javascript
[
  {
    "id": 1879095,
    "createdDate": 1551083545000,
    "symbol": "EOSUSDT",
    "insuranceFund": 218.47009356847536
  },
  {
    "id": 1879097,
    "createdDate": 1551083545000,
    "symbol": "EOSUSDT",
    "insuranceFund": 661.414109510817
  }
]
```


### 转移保证金 (USER_DATA)
```
POST /api/v1/contract/transferMargin  (HMAC SHA256)
```
币币余额转移到某合约的保证金

**权重:**
1

**参数:**

Name | Type | Mandatory | Description
------------ | ------------ | ------------ | ------------
symbol | STRING | YES |
amount | STRING | NO |
recvWindow | LONG | NO |
timestamp | LONG | YES |

**响应:**
```javascript
{
  "asset": "USDT",
  "balance": 889486,
  "lockBalance": 1.6
}
```

### 转出保证金 (USER_DATA)
```
POST /api/v1/contract/turnoutMargin  (HMAC SHA256)
```
某合约的保证金转移到币币余额

**权重:**
1

**参数:**

Name | Type | Mandatory | Description
------------ | ------------ | ------------ | ------------
symbol | STRING | YES |
amount | STRING | NO |
recvWindow | LONG | NO |
timestamp | LONG | YES |

**响应:**
```javascript
{}
```




## 用户数据流订阅接口
此处仅列出如何得到数据流名称及如何维持有效期的接口，具体订阅方式参考另一篇websocket接口文档

### 新建用户数据流 (USER_STREAM)
```
POST /api/v1/userDataStream
```
从创建起60分钟有效

**权重:**
1

**Parameters:**
NONE

**响应:**
```javascript
{
  "listenKey": "pqia91ma19a5s61cv6a81va65sdf19v8a65a1a5s61cv6a81va65sdf19v8a65a1" //用于订阅的数据流名
}
```

### Keepalive (USER_STREAM)
```
PUT /api/v1/userDataStream
```
延长用户数据流有效期到60分钟之后。 建议每30分钟调用一次

**权重:**
1

**参数:**

Name | Type | Mandatory | Description
------------ | ------------ | ------------ | ------------
listenKey | STRING | YES

**响应:**
```javascript
{}
```

### 关闭用户数据流 (USER_STREAM)
```
DELETE /api/v1/userDataStream
```

**权重:**
1

**参数:**

Name | Type | Mandatory | Description
------------ | ------------ | ------------ | ------------
listenKey | STRING | YES

**响应:**
```javascript
{}
```



### 用户资产详情 (USER_DATA)
```
GET /wapi/v1/assetDetail
```

**权重:**
1

**参数:**

Name | Type | Mandatory | Description
------------ | ------------ | ------------ | ------------
recvWindow |	LONG |	NO	
timestamp |	LONG |	YES	
**响应:**
```javascript
{
  "ADC": {
    "minWithdrawAmount": 0,
    "withdrawFee": "0",
    "depositStatus": true,
    "withdrawStatus": false
  },
  "ATT": {
    "minWithdrawAmount": 10,
    "withdrawFee": "[0.01]",
    "depositStatus": false,
    "withdrawStatus": false
  }
}
```


### 用户充值地址 (USER_DATA)
```
GET /wapi/v1/depositAddress
```

**权重:**
1

**参数:**

Name | Type | Mandatory | Description
------------ | ------------ | ------------ | ------------
recvWindow |	LONG |	NO	
timestamp |	LONG |	YES	
**响应:**
```javascript
{
  "address": "chongzhidizhichongzhidizhi",
  "asset": "BTC",
  "success": true
}
```



### 用户充值历史记录 (USER_DATA)
```
GET /wapi/v1/depositHistory
```

**权重:**
5

**参数:**

Name | Type | Mandatory | Description
------------ | ------------ | ------------ | ------------
asset |	STRING |	NO	|
status |	int |	NO	| 0:pending,1:success
startTime | LONG | NO |
endTime | LONG | NO |
recvWindow |	LONG |	NO	
timestamp |	LONG |	YES	
**响应:**
```javascript
[
  {
    "insertTime": 1530096910000,
    "amount": 1,
    "asset": "BTC",
    "address": "chongzhidizhichongzhidizhi",
    "txId": "g3jrglkelmrioqnfmkcmxcmkllfemfkmfmseme",
    "status": 2
  },
  {
    "insertTime": 1530096910000,
    "amount": 1,
    "asset": "BTC",
    "address": "chongzhidizhichongzhidizhi",
    "txId": "gjrglkel4mrioqnfmkcmxcmkllfemfkmfmseme",
    "status": 2
  }
]

```

### 用户交易手续费 (USER_DATA)
```
GET /wapi/v1/tradeFee
```

**权重:**
1

**参数:**

Name | Type | Mandatory | Description
------------ | ------------ | ------------ | ------------
symbol |	STRING |	NO	|
recvWindow |	LONG |	NO	
timestamp |	LONG |	YES	
**响应:**
```javascript
[
  {
    "symbol": "DASHUSDT",
    "maker": -0.001,
    "taker": -0.001
  },
  {
    "symbol": "BTCUSDT",
    "maker": -0.1,
    "taker": -0.1
  }
]
```

### 用户提现 (USER_DATA)
```
GET /wapi/v1/withdraw
```

**权重:**
1

**参数:**

Name | Type | Mandatory | Description
------------ | ------------ | ------------ | ------------
asset |	STRING |	YES	| 资产名称
address | STRING | YES | 提现地址
addressTag | STRING | NO | 地址标识符
amount | BIGDECIMAL | YES | 提现数量
name | STRING | NO | 地址别名
recvWindow |	LONG |	NO	
timestamp |	LONG |	YES	
**响应:**
```javascript
{
  "id": "660A0D4AEB32BE67"
}
```

### 用户提现历史记录 (USER_DATA)
```
GET /wapi/v1/withdrawHistory
```

**权重:**
5

**参数:**

Name | Type | Mandatory | Description
------------ | ------------ | ------------ | ------------
asset |	STRING |	NO	|
status |	INT |	NO	| 0:Email Sent,1:Cancelled 2:Awaiting Approval 3:Rejected 4:Processing 5:Failure 6Completed
startTime | LONG | NO |
endTime | LONG | NO |
recvWindow |	LONG |	NO	
timestamp |	LONG |	YES	
**响应:**
```javascript
{
  "withdrawRecordList": [
    {
      "id": "20F13F5291908F81",
      "amount": 1,
      "address": "swswsdgsdwesdfwsdw",
      "asset": "BTC",
      "txId": g3jrglkelmrioqnfmkcmxcmkllfemfkmfmseme,
      "applyTime": 1551339300000,
      "status": 4
    },
    {
      "id": "AAEAAECD5F890E47",
      "amount": 1,
      "address": "swswsdgsdwesdfwsdw",
      "asset": "BTC",
      "txId": g3jrglkelmrioqnfmkcmxcmkllfemfkmfmseme,
      "applyTime": 1551340680000,
      "status": 4
    }
  ],
  "success": true
}
```
