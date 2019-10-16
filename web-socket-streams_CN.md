# Websocket 行情接口

## 基本信息

* 本篇所列出的所有wss接口的baseurl为: wss://ws.jex.com 或者 wss://testnetws.jex.com
* 其中 wss://testnetws.jex.com 为模拟环境。
* 所有stream均可以直接访问，或者作为组合streams的一部分。
* 直接访问时URL格式为 /ws/\<streamName\>
* 组合streams的URL格式为 /stream?streams=\<streamName1\>/\<streamName2\>/\<streamName3\>
* 订阅组合streams时，事件payload会以这样的格式封装 {"stream":"\<streamName\>","data":\<rawPayload\>}
* stream名称中所有交易对均为大写
* 每个到stream.jex.com的链接有效期不超过24小时，请妥善处理断线重连。
* 每3分钟，服务端会发送ping帧，客户端应当在10分钟内回复pong帧，否则服务端会主动断开链接。允许客户端发送不成对的pong帧(即客户端可以以高于10分钟每次的频率发送pong帧保持链接)。

## 其他说明
-  wss 的 contract 订单的sell数量统一为负数

## Stream 详细定义

## 最近成交

> 每隔100 ms 推送一次，上一秒内所有的逐笔交易，一笔逐笔交易的定义是仅有一个吃单者与一个挂单者相互交易。

### Stream 名称: \<symbol\>@\<tradeType\>

### Playload

```javascript
[{
  "e": "spotTrade",     // 事件类型
  "E": 123456789,       // 事件时间
  "s": "JEXBTC",        // 交易对
  "t": 12345,           // 交易ID
  "p": "0.001",         // 成交价格
  "q": "100",           // 成交数量
  "T": 123456785,       // 成交时间
  "m": true,            // 买方是否是做市方。如true，则此次成交是一个主动卖出单，否则是一个主动买入单。
},{...},...
]
```

### tradeType类型说明

- 支持以下三个事件类型
  - spotTrade 现货类型
  - optionTrade 期权类型
  - contractTrade 合约类型

## K线

>K线stream逐秒推送所请求的K线种类(最新一根K线)的更新

- 订阅Kline需要提供间隔参数，最短为分钟线，最长为周线。支持以下间隔:

  - 1M 1分钟
  - 5M 5分钟
  - 15M 15分钟
  - 30M 30分钟
  - 1H 1小时
  - 4H 4小时
  - 1D 1天
  - 1W 1周

### Stream 名称: \<symbol\>@\<klineType\>_\<interval\>

### Playload

```javascript
{
  "e": "spotKline",     // 事件类型
  "E": 123456789,       // 事件时间
  "s": "JEXBTC",        // 交易对
  "k": {
    "t": 123400000,     // 这根K线的起始时间
    "T": 123460000,     // 这根K线的结束时间
    "s": "JEXBTC",      // 交易对
    "i": "1m",          // K线间隔
    "o": "0.0010",      // 这根K线期间第一笔成交价
    "c": "0.0020",      // 这根K线期间末一笔成交价
    "h": "0.0025",      // 这根K线期间最高成交价
    "l": "0.0015",      // 这根K线期间最低成交价
    "v": "1000",        // 这根K线期间成交量
    "q": "1.0000",      // 这根K线期间成交额
  }
}
```

### klineType类型说明

- klineType 支持的类型
  - spotKline
  - optionKline
  - contractKline

## 按Symbol的精简Ticker

> 按Symbol逐秒刷新的24小时精简ticker信息

### Stream 名称: \<symbol\>@\<miniTickerType\>

### Playload

```javascript
{
    "e": "24hrSpotMiniTicker",   // 事件类型
    "E": 123456789,              // 事件时间
    "s": "JEXBTC",               // 交易对
    "c": "0.0025",               // 最新成交价格
    "h": "0.0025",               // 24小时内最高成交价
    "l": "0.0010",               // 24小时内最低成交加
    "m": "10000",                // 标记价(只有contractMiniTicker 类型才存在字段)
    "i": "10000",                // 指数价(只有contractMiniTicker 类型才存在字段)
    "v": "10000",                // 成交量
    "q": "18"，                  // 成交额

}

```

### miniTickerType类型说明

- miniTickerType 支持的类型
  - spotMiniTicker
  - optionMiniTicker
  - contractMiniTicker

## 全市场所有Symbol的精简Ticker

> 按Symbol逐秒刷新的24小时精简全市场ticker信息

### Stream名称: !\<miniTickerType\>@arr

### Playload

```javascript

[
  {
    //数组每一个元素对应一个交易对，内容与 按Symbol的精简Ticker 相同
  }
]
```

### miniTickerType 类型说明

- miniTickerType 支持的类型
  - spotMiniTicker
  - optionMiniTicker
  - contractMiniTicker

## 按Symbol的完整Ticker

> 按Symbol逐秒刷新的24小时完整ticker信息

### Stream 名称: \<symbol\>@\<tickerType\>

### Playload

```javascript
{
  "e": "24hrSpotTicker",  // 事件类型
  "E": 123456789,         // 事件时间
  "s": "JEXBTC",          // 交易对
  "p": "0.0015",          // 24小时价格变化
  "P": "250.00",          // 24小时价格变化（百分比）
  "c": "0.0025",          // 最新成交价格
  "h": "0.0025",          // 24小时内最高成交价
  "l": "0.0010",          // 24小时内最低成交价
  "v": "10000",           // 24小时内成交量
  "q": "18",              // 24小时内成交额
  "m": "10000",           // 标记价(只有contractMiniTicker 类型才存在字段)
  "i": "10000",           // 指数价(只有contractMiniTicker 类型才存在字段)
  "O": 1234567890,        // TODO 0  统计开始时间
  "C": 86400000,          // TODO 3600*24*1000，24小时的毫秒数  统计关闭时间
}
```
## 全市场所有交易对的完整Ticker

> 同上，只是推送所有的交易对

### Stream名称: !\<tickerType\>@arr

### Payload:

```javascript
[
  {
    //数组每一个元素对应一个交易对，内容与 按Symbol的完整Ticker 相同
  }
]
```
- tickerType 支持的类型
  - spotTicker
  - optionTicker
  - contractTicker


## 有限档深度信息

> 每秒推送有限档深度信息。levels表示几档买卖单信息, 可选 5/10/20档

### Stream 名称: \<symbol\>@\<depthType\>\<levels\>

### Playload:
```javascript
{
  "bids": [             // 买单
    [
      "0.0024",         // 价
      "10",             // 量
    ]
  ],
  "asks": [             // 卖单
    [
      "0.0026",         // 价
      "100",            // 量
    ]
  ]
}
```
- depthType 支持的类型
  - spotDepth
  - optionDepth
  - contractDepth
- levels 支持的类型
  - 5
  - 10
  - 20

## 增量深度信息stream
> 每秒推送orderbook的变化部分（如果有）

### Stream 名称: \<symbol\>@\<depthType\>

### Payload:
```javascript
{
  "e": "spotDepthUpdate", // 事件类型
  "s": "JEXBTC",          // 交易对
  "v": 157,               // 推送的版本号
  "b": [                  // 变动的买单深度
    [
      "0.0024",          // 价
      "10",              // 量
    ]
  ],
  "a": [                 // 变动的卖单深度
    [
      "0.0026",          // 价
      "100",             // 量
    ]
  ]
}
```
- depthType 支持的类型
  - spotDepth
  - optionDepth
  - contractDepth
- 增量深度推送的事件类型  
  - spotDepthUpdate
  - optionDepthUpdate
  - contractDepthUpdate


# 用户私有数据 User Data Streams

## 基本信息
* 本篇所列出REST接口的baseurl **https://www.jex.com** 或者 **https://testnet.jex.com**
* 其中 **https://testnet.jex.com** 为模拟环境。
* 用于订阅账户数据的 `listenKey` 从创建时刻起有效期为60分钟
* 可以通过`PUT`一个`listenKey`延长60分钟有效期
* `DELETE`一个 `listenKey` 立即关闭当前数据流
* 本篇所列出的websocket接口baseurl: **wss://ws.jex.com** 或者 **wss://testnetws.jex.com**
 * 其中 **wss://testnetws.jex.com** 为模拟环境。
* 订阅账户数据流的stream名称为 **/ws/\<listenKey\>**
* 每个到stream.jex.com的链接有效期不超过24小时，请妥善处理断线重连。
* 账户数据流的消息**不保证**严格时间序; **请使用 E 字段进行排序**


## 与Websocket账户接口相关的REST接口 请参照 REST文档的 **用户数据流订阅接口**

### 账户更新
> 账户更新事件的 event type 固定为 `accountSpotInfo` AND `accountOptionInfo` AND `accountContractInfo` 分别对应现货、期权、期货的资产事件
当对应资产信息有变动时，会推送相应的事件

### Playload:
```javascript
{
  "e": "accountSpotInfo",       // 事件类型
  "E": 1499405658849,           // 事件时间
  "m": 0,                       // 挂单费率
  "t": 0,                       // 吃单费率
  "u": 1499405658848,           // 账户末次更新时间戳
  "B": [                        // 余额
    {
      "a": "LTC",               // 资产名称
      "o": "1.57",              // 委托保证金（仅合约资产存在）
      "p":  "0"  ，             // 持仓保证金（仅合约资产存在）
      "f": "17366.18538083",    // 可用余额
      "l": "0.00000000",        // 冻结余额  
      "T": true,                // 是否允许交易
      "W": true,                // 是否允许提现
      "D": true,                // 是否允许充值
    },
    {
      "a": "BTC",
      "f": "10537.85314051",
      "l": "2.19464093",
      "T": true,                // 是否允许交易
      "W": true,                // 是否允许提现
       "D": true,               // 是否允许充值
 },
    {
    "T": true,                  // 是否允许交易
    "W": true,                  // 是否允许提现
    "D": true,                  // 是否允许充值
    "a": "ETH",
    "f": "17902.35190619",
    "l": "0.00000000"
    }
  ]
}
```

### 订单交易更新

> 当有新订单创建、订单有新成交或者新的状态变化时会推送此类事件

event type统一为 `execSpotReport` AND `execOptionReport` AND `execContractReport` 分别对应现货、期权、合约的事件类型

具体内容需要读取 `x`字段 判断执行类型

### 现货和期权的Playload:
```javascript
{
  "E": 1499405658849,            // 事件时间
  "e": "execSpotReport",         // 事件类型
  "s": "JEXBTC",                 // 交易对
  "S": "BUY",                    // 订单方向
  "q": "1.00000000",             // 订单原始数量
  "p": "0.10264410",             // 订单原始价格
  "X": "NEW",                    // 订单的当前状态
  "i": 4293153,                  // 订单ID
  "z": "0.00000000",             // 订单累计已成交数量
  "O": 1499405658657,            // 订单创建时间
  "Z": "0.00000000",             // 订单累计已成交金额
}
```
#### 现货和期权可能的执行类型(X字段):


* NEW 新建订单
* PARTIALLY_FILLED  部分成交
* FILLED  全部成交
* CANCELED  已撤销
* FAIL 下单失败
* CANCLEFILLED 撤单完成



### 合约的Playload:
```javascript
{
  "E": 1499405658849,            // 事件时间
  "e": "execContractReport",     // 事件类型
  "s": "ETHBTC",                 // 交易对           (被拒绝时不存在)
  "X": "NEW",                    // 订单的当前状态   
  "r": "NONE",                   // 订单被拒绝的原因  (被拒绝才会存在)
  "i": 4293153,                  // 订单ID
  "z": "0.00000000",             // 订单累计已成交数量 (被拒绝时不存在)
  "Z": "0.00000000",             // 订单累计已成交金额 (被拒绝时不存在)
}
```

#### 合约可能的执行类型(X字段):

* ENTRUSTED 下单完成
* ENTRUSTING 下单中
* FAIL 下单失败
* PARTFILLED 部分成交
* FILLED 订单完成
* CANCEL 订单取消

#### 被拒绝的例子:
``` javascript
{
  E: 1571225843289
  X: "FAIL"
  e: "execContractReport"
  i: "4612616205276108403"
  r: "insufficient Available"  
}
```
####  失败的原因(R字段)
  * insufficient Available
  * too many pending requests
  * position is being liquidated
  * position is being closed
  * closing empty position
  * closing order is enough
  * contract is disabled
  * reach liquidation price
  * reach risk limit



### 合约持仓

> 当合约持仓有变化时，信息推送
> event type 为 contractPositions

#### playload

```javascript
{
	"E": 1553050064078,
	"e": "contractPositions",
	"p": [{
		"M": "100", // 最大可用杠杆
		"R": "3903.99060000", // 已用风险限额
		"S": "0", // sell委托价值
		"a": "20", // 强减顺序
		"b": "0", // buy委托价值
		"c": "1301.33020000", //开仓价格
		"d": "longs", //方向
		"i": "0.05", //初始保证金
		"l": "-49.9759", //平仓价格
		"m": "0.005", //维持保证金
		"n": "0", //当前委托价值
		"o": "4088.0779300000", //保证金
		"q": "3.0000",//持仓数量
		"r": "900000",//当前风险值
		"s": "EOSUSDT",//交易对
		"t": "3903.99060000", //持仓成本价值
		"v": "20.00"//杠杆
	}]
}

```