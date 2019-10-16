# Web Socket Streams for JEX 

## General WSS information
* The base endpoint is: **wss://ws.jex.com** or **wss://testnetws.jex.com**
* which the **wss://testnetws.jex.com** simulation environment。
* Streams can be accessed either in a single raw stream or in a combined stream
* Raw streams are accessed at **/ws/\<streamName\>**
* Combined streams are accessed at **/stream?streams=\<streamName1\>/\<streamName2\>/\<streamName3\>**
* Combined stream events are wrapped as follows: **{"stream":"\<streamName\>","data":\<rawPayload\>}**
* All symbols for streams are **lowercase**
* A single connection to **stream.jex.com** is only valid for 24 hours; expect to be disconnected at the 24 hour mark
* The websocket server will send a `ping frame` every 3 minutes. If the websocket server does not receive a `pong frame` back from the connection within a 10 minute period, the connection will be disconnected. Unsolicited `pong frames` are allowed.

## 其他说明
- WSS contract order sells are uniformly negative

## Stream Detailed Stream information

## Trade Streams

> Every 100 ms push, every single transaction in the last second is defined as a transaction between only one order taker and one bill holder.

### Stream Name: \<symbol\>@\<tradeType\>

### Playload

```javascript
[{
  "e": "spotTrade",    //  Event type
  "E": 123456789,      // Event time
  "s": "JEXBTC",       // Symbol
  "t": 12345,          // Trade ID
  "p": "0.001",        // Price
  "q": "100",          // Quantity
  "T": 123456785,      // Trade time
  "m": true,           // Is the buyer the market maker?If true，this is a seller order，otherwise it is a buyer order.
},{...},...
]
```

### tradeType

- Support 3 types as below:
  - spotTrade 
  - optionTrade 
  - contractTrade 

## Kline/Candlestick Streams

>The Kline/Candlestick Stream push updates to the current klines/candlestick every second

- Booking Kline requires interval parameter, shortest one is minute line, longest one is week line. Support intervals as below:

  - 1M 
  - 5M 
  - 15M 
  - 30M
  - 1H 
  - 4H
  - 1D 
  - 1W

### Stream Name: \<symbol\>@\<klineType\>_\<interval\>

### Playload

```javascript
{
  "e": "spotKline",     // Event type
  "E": 123456789,       // Event time
  "s": "JEXBTC",        // Symbol
  "k": {
    "t": 123400000,     // Kline start time
    "T": 123460000,     // Kline close time
    "s": "JEXBTC",      // Symbol
    "i": "1m",          // Interval
    "o": "0.0010",      // Open price
    "c": "0.0020",      // Close price
    "h": "0.0025",      // High price
    "l": "0.0015",      // Low price
    "v": "1000",        // Base asset volume
    "q": "1.0000",      // Quote asset volume
  }
}
```

### klineType

-Support types
  - spotKline
  - optionKline
  - contractKline

## Individual Symbol Mini Ticker Stream

> 24hr rolling window mini-ticker statistics for a single symbol pushed every second.

### Stream Name: \<symbol\>@\<miniTickerType\>

### Playload

```javascript
{
    "e": "24hrSpotMiniTicker",   // Event type
    "E": 123456789,              // Event time
    "s": "JEXBTC",               // Symbol
    "c": "0.0025",               // Close price
    "h": "0.0025",               //  High  price
    "o": "0.0025",               // Open price
    "l": "0.0010",               // Low price
    "m": "10000",                // mark price(Only  contractMiniTicker have)
    "i": "10000",                // index price(Only contractMiniTicker have)
    "v": "10000",                // Base asset volume
    "q": "18"                    // Total traded quote asset volume
}

```

### miniTickerType

- Support types:
  - spotMiniTicker
  - optionMiniTicker
  - contractMiniTicker

## All Market Mini Tickers Stream

> 24hr rolling window mini-ticker statistics for all symbols that changed in an array pushed every second

### StreamName: !\<miniTickerType\>@arr

### Playload

```javascript

[
  {
     // Same as <symbol>@miniTicker payload
  }
]
```

### miniTickerType 

- Support types: 
  - spotMiniTicker
  - optionMiniTicker
  - contractMiniTicker

## Individual Symbol Ticker Streams

> 24hr rollwing window ticker statistics for a single symbol pushed every second. 

### Stream Name: \<symbol\>@\<tickerType\>

### Playload

```javascript
{
  "e": "24hrSpotTicker",  // Event type
  "E": 123456789,         // Event time
  "s": "JEXBTC",          // Symbol
  "p": "0.0015",          // Price change in 24h
  "P": "250.00",          // Price change in 24h percent
  "c": "0.0025",          // Last price
  "h": "0.0025",          // Highest price in 24h
  "l": "0.0010",          // Lowest price in 24h
  "v": "10000",           // Trade volume in 24h
  "q": "18",              // Trade asset in 24h
  "m": "10000",           // mark price(Only  contractMiniTicker have)
  "i": "10000",           // index price(Only contractMiniTicker have)
  "O": 1234567890,        // TODO 0  Starting time
  "C": 86400000,          // TODO 3600*24*1000，milliseconds in 24h  ending time
}
```
## All Market Tickers Stream

> 24hr rolling window ticker statistics for all symbols that changed in an array pushed every second.

### StreamName: !\<tickerType\>@arr

### Payload:

```javascript
[
  {
     //Same as <symbol>@ticker payload
  }
]
```
- tickerType 
  - spotTicker
  - optionTicker
  - contractTicker


## Partial Book Depth Streams

> Top <levels> bids and asks, pushed every second. Valid <levels> are 5, 10, or 20.

### Stream Name: \<symbol\>@\<depthType\>\<levels\>

### Playload:
```javascript
{
  "bids": [             // Buy
    [
      "0.0024",         // Price
      "10",             // Quantity
    ]
  ],
  "asks": [             // Sell
    [
      "0.0026",         // Price
      "100",            // Quantity
    ]
  ]
}
```


- depthType 
  - spotDepth
  - optionDepth
  - contractDepth
- levels 
  - 5
  - 10
  - 20

## Diff. Depth Stream
> Order book price and quantity depth updates used to locally manage an order book pushed every second.

### Stream Name: \<symbol\>@\<depthType\>

### Payload:
```javascript
{
  "e": "spotDepthUpdate", // Event type
  "s": "JEXBTC",          // Symbol
  "v": 157,               // Version to be updated
  "b": [                  // Changing depth for buyer
    [
      "0.0024",          // Price
      "10",              // Quantity
    ]
  ],
  "a": [                 // Changing depth for seller
    [
      "0.0026",          // Price
      "100",             // Quantity
    ]
  ]
}
```
- depthType Support types
  - spotDepth
  - optionDepth
  - contractDepth
- Types for Depth Streams 
  - spotDepthUpdate
  - optionDepthUpdate
  - contractDepthUpdate


# User Data Streams for JEX

## 基本信息 
* The base API endpoint is: **https://www.jex.com** or **https://testnet.jex.com**
* which the **https://testnet.jex.com** Simulation environment。
* A User Data Stream `listenKey` is valid for 60 minutes after creation.
* Doing a `PUT` on a `listenKey` will extend its validity for 60 minutes.
* Doing a `DELETE` on a `listenKey` will close the stream.
* The base websocket endpoint is: **wss://ws.jex.com**   or **wss://testnetws.jex.com**
* which the **wss://testnetws.jex.com** Simulation environment。
* User Data Streams are accessed at **/ws/\<listenKey\>**
* A single connection to **stream.jex.com** is only valid for 24 hours; expect to be disconnected at the 24 hour mark
* User data stream payloads are **not guaranteed** to be in order during heavy periods; **make sure to order your updates using E**


## REST API related to Websocket account API Please refer to user data subscription API of REST document

### Account updates
> Event type of account updates is fixed to `accountSpotInfo` AND `accountOptionInfo` AND `accountContractInfo` They respectively refers to the asset related event of spot, options and futures.
When there is a change of asset, relevant event will be pushed


### Playload:
```javascript
{
  "e": "accountSpotInfo",       // Event type
  "E": 1499405658849,           // Event time
  "m": 0,                       // Fee rate of entry order 
  "t": 0,                       // Fee rate of taking order
  "u": 1499405658848,           // Time stamp of
  "B": [                        // Balance
    {
      "a": "LTC",               // Asset name
      "o": "1.57",              // Order Margin（Only Furtures）
      "p":  "0"  ，             // Position Margin（Only Furtures ）
      "f": "17366.18538083",    // Free amount
      "l": "0.00000000",        // Locked amount  
      "T": true,                // Can trade
      "W": true,                // Can withdraw
      "D": true,                // Can deposit
    },
    {
      "a": "BTC",
      "f": "10537.85314051",
      "l": "2.19464093",
      "T": true,                // Can trade
      "W": true,                // Can withdraw
       "D": true,               // Can deposit
 },
    {
    "T": true,                  // Can trade
    "W": true,                  // Can withdraw
    "D": true,                  // Can deposit
    "a": "ETH",
    "f": "17902.35190619",
    "l": "0.00000000"
    }
  ]
}
```

### Update of orders

> Account state is updated with the outboundAccountInfo event.

event type is `execSpotReport` AND `execOptionReport` AND `execContractReport` They respectively refers to the asset related event of spot、options、futures Event type

Possible executive type of contract(X field)

### Playload of spot and options:
```javascript
{
  "E": 1499405658849,            // Event time
  "e": "execSpotReport",         // Event type
  "s": "JEXBTC",                 // Symbol
  "S": "BUY",                    // Order direction
  "q": "1.00000000",             //  Order quantity
  "p": "0.10264410",             //  Order price
  "X": "NEW",                    // Current order state
  "i": 4293153,                  // ID Order ID
  "z": "0.00000000",             //  Cumulative filled quantity
  "O": 1499405658657,            // Transaction time
  "Z": "0.00000000",             // Total transaction sum
}
```
#### Possible executive type of spot and options (X field):

* NEW 
* PARTIALLY_FILLED  
* FILLED  
* CANCELED  
* FAIL 
* CANCLEFILLED


###  contract Playload:
```javascript
{
  "E": 1499405658849,            // Event time
  "e": "execContractReport",     // Event type
  "s": "ETHBTC",                 // Symbol   (Not exist if refused)   
  "X": "NEW",                    // Current state of the order   
  "r": "NONE",                   // Refused reason of the order ( exist if refused)
  "i": 4293153,                  // Order ID
  "z": "0.00000000",             // Cumulative filled quantity (Not exist if refused)
  "Z": "0.00000000",             // Cumulative filled sum(Not exist if refused)

}
```



#### Possible executive type of contract(X field):

* ENTRUSTED 
* ENTRUSTING 
* FAIL 
* PARTFILLED 
* FILLED 
* CANCEL    


#### example if refused:
``` javascript
{
  E: 1571225843289
  X: "FAIL"
  e: "execContractReport"
  i: "4612616205276108403"
  r: "insufficient Available"  
}
```
#### if refused(R field) 
  *  insufficient Available
  *  too many pending requests
  *  position is being liquidated
  *  position is being closed
  *  closing empty position
  *  closing order is enough
  *  contract is disabled
  *  reach liquidation price
  *  reach risk limit

### Contract positions

> Where there is a change in contract position
> event type is  contractPositions

#### playload

```javascript
{
	"E": 1553050064078,
	"e": "contractPositions",
	"p": [{
		"M": "100", // Largest leverage available 
		"R": "3903.99060000", // Used risk limits
		"S": "0", // sell Entrusted value
		"a": "20", // Auto-deleverage sequence
		"b": "0", // buy Entrusted value
		"c": "1301.33020000", //Open price
		"d": "longs", //Direction 
		"i": "0.05", //Initial margin
		"l": "-49.9759", //Close price
		"m": "0.005", //Maintenance Margin
		"n": "0", //Current entrusted value 
		"o": "4088.0779300000", //Margin
		"q": "3.0000",//Holding positions 
		"r": "900000",//Current risk value
		"s": "EOSUSDT",//Symbol
		"t": "3903.99060000", //Holding positions costs value 
		"v": "20.00"//Leverage
	}]
}

```