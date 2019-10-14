# General API Information
* The base endpoint is:  https://www.jex.com or https://testnet.jex.com
* which the https://testnet.jex.com simulation environment。
* All endpoints return either a JSON object or array.
* Data is returned in ascending order. Oldest first, newest last。
* All time and timestamp related fields are in milliseconds
* HTTP `4XX` return codes are used for malformed requests; the issue is on the sender's side
* HTTP `429` return code is used when breaking a request rate limit.
* HTTP `418` return code is used when an IP has been auto-banned for continuing to send requests after receiving `429` codes.
* HTTP `5XX` return codes are used for internal errors; the issue is on
  jex's side.
  It is important to **NOT** treat this as a failure operation; the execution status is
  **UNKNOWN** and could have been a success.
* Any endpoint can return an ERROR; the error payload is as follows:
```javascript
{
  "code": -1121,
  "msg": "Invalid symbol."
}
```

* Specific error codes and messages defined in another document.
* For `GET` endpoints, parameters must be sent as a `query string`.
* For `POST`, `PUT`, and `DELETE` endpoints, the parameters may be sent as a
  `query string` or in the `request body` with content type
  `application/x-www-form-urlencoded`. You may mix parameters between both the
  `query string` and `request body` if you wish to do so.
* Parameters may be sent in any order.
* If a parameter sent in both the `query string` and `request body`, the
  `query string` parameter will be used.


# LIMITS
* New intervalLetter values for headers:
  - SECOND => S
  - MINUTE => M
  - HOUR => H
  - DAY => D
* The `/api/v1/exchangeInfo` `rateLimits` array contains objects related to the exchange's `RAW_REQUEST`, symbol accuracy,`REQUEST_WEIGHT`, and `ORDER` rate limits. These are further defined in the `ENUM definitions` section under `Rate limiters (rateLimitType)`.
* When a 429 is recieved, it's your obligation as an API to back off and not spam the API
* Repeatedly violating rate limits and/or failing to back off after receiving 429s will result in an automated IP ban (http status 418).
* IP bans are tracked and scale in duration for repeat offenders, from 2 minutes to 3 days
* New Headers  `X-MBX-USED-WEIGHT-(intervalNum)(intervalLetter) ` will give your current used request weight for the (intervalNum)(intervalLetter) rate limiter. For example, if there is a one minute request rate weight limiter set, you will get a  `X-MBX-USED-WEIGHT-1M ` header in the response. The legacy header  `X-MBX-USED-WEIGHT ` will still be returned and will represent the current used weight for the one minute request rate weight limit
* New Header `X-MBX-ORDER-COUNT-(intervalNum)(intervalLetter)`that is updated on any valid order placement and tracks your current order count for the interval; rejected/unsuccessful orders are not guaranteed to have `X-MBX-ORDER-COUNT-**` headers in the response。
  - Eg. `X-MBX-ORDER-COUNT-1S` for "orders per 1 second" and `X-MBX-ORDER-COUNT-1D` for orders per "one day"
* A `Retry-After` header is sent with a 418 or 429 responses and will give the number of seconds required to wait, in the case of a 418, to prevent a ban, or, in the case of a 429, until the ban is over.。
* Wrong order accuracy does not allow ordering

# Order Count Limits
* To keep an orderly market, jex imposes limits on the number of open orders per account. These limits are:
* Maximum 200 open orders per contract per account;
* Maximum 10 stop profit or stop  orders per contract per account;
* When placing a new order that causes these caps to be exceeded, it will be rejected with the message “has reach max order number [200|10]”.



# Other instructions
The number of contract orders sells for API is uniformly negative

# Endpoint security type
* Each endpoint has a security type that determines the how you will interact with it
* API-keys are passed into the Rest API via the `X-JEX-APIKEY` header
* API-keys and secret-keys are case sensitive
* API-keys can be configured to only access certain types of secure endpoints. For example, one API-key could be used for TRADE only, while another API-key can access everything except for TRADE routes.

Security Type | Description
------------ | ------------
NONE | Endpoint can be accessed freely.
TRADE | Endpoint requires sending a valid API-Key and signature
USER_DATA | Endpoint requires sending a valid API-Key and signature.
USER_STREAM | Endpoint requires sending a valid API-Key.
MARKET_DATA | Endpoint requires sending a valid API-Key.


# SIGNED (TRADE and USER_DATA) Endpoint security
* `SIGNED` endpoints require an additional parameter, `signature`, to be
  sent in the  `query string` or `request body`.
* Endpoints use `HMAC SHA256` signatures. The `HMAC SHA256 signature` is a keyed `HMAC SHA256` operation.
  Use your `secretKey` as the key and `totalParams` as the value for the HMAC operation.
* The `signature` is **not case sensitive**.
* `totalParams` is defined as the `query string` concatenated with the
  `request body`.
* The parameters of signature must be transmitted in the order of interface documents.

## Timing security
* A `SIGNED` endpoint also requires a parameter, `timestamp`, to be sent which
  should be the millisecond timestamp of when the request was created and sent.
* An additional parameter, `recvWindow`, may be sent to specify the number of
  milliseconds after `timestamp` the request is valid for. If `recvWindow`
  is not sent, **it defaults to 5000**.
* The logic is as follows:
  ```javascript
  if (timestamp < (serverTime + 1000) && (serverTime - timestamp) <= recvWindow) {
    // process request
  } else {
    // reject request
  }
  ```
**Serious trading is about timing.** Networks can be unstable and unreliable,
which can lead to requests taking varying amounts of time to reach the
servers. With `recvWindow`, you can specify that the request must be
processed within a certain number of milliseconds or be rejected by the
server.


**It recommended to use a small recvWindow of 5000 or less! The max cannot go beyond 60,000!**


## POST /api/v1/spot/order Examples for POST

Here is a step-by-step example of how to send a vaild signed payload from the
Linux command line using `echo`, `openssl`, and `curl`.

Key | Value
------------ | ------------
apiKey | vmPUZE6mv9SD5VNHk4HlWFsOr6aKE2zvsw0MuIgwCIPy6utIco14y7Ju91duEh8A
secretKey | NhqPtmdSJYdKjVHjA7PZj4Mge3R5YNiP1e3UZjInClVN65XAbvqqM6A7H5fATj0j


Parameters | Value
------------ | ------------
symbol | LTCBTC
side | BUY
type | LIMIT
timeInForce | GTC
quantity | 1
price | 0.1
recvWindow | 5000
timestamp | 1499827319559


### Example 1: As a query string
* **queryString:** symbol=LTCBTC&side=BUY&type=LIMIT&timeInForce=GTC&quantity=1&price=0.1&recvWindow=5000&timestamp=1499827319559
* **HMAC SHA256 signature:**

    ```
    [linux]$ echo -n "symbol=LTCBTC&side=BUY&type=LIMIT&timeInForce=GTC&quantity=1&price=0.1&recvWindow=5000&timestamp=1499827319559" | openssl dgst -sha256 -hmac "NhqPtmdSJYdKjVHjA7PZj4Mge3R5YNiP1e3UZjInClVN65XAbvqqM6A7H5fATj0j"
    (stdin)= c8db56825ae71d6d79447849e617115f4a920fa2acdcab2b053c4b2838bd6b71
    ```


* **curl command:**

    ```
    (HMAC SHA256)
    [linux]$ curl -H "X-JEX-APIKEY: vmPUZE6mv9SD5VNHk4HlWFsOr6aKE2zvsw0MuIgwCIPy6utIco14y7Ju91duEh8A" -X POST 'https://www.jex.com/api/v1/order?symbol=LTCBTC&side=BUY&type=LIMIT&timeInForce=GTC&quantity=1&price=0.1&recvWindow=5000&timestamp=1499827319559&signature=c8db56825ae71d6d79447849e617115f4a920fa2acdcab2b053c4b2838bd6b71'
    ```

### Example 2: As a request body
* **requestBody:** symbol=LTCBTC&side=BUY&type=LIMIT&timeInForce=GTC&quantity=1&price=0.1&recvWindow=5000&timestamp=1499827319559
* **HMAC SHA256 signature:**

    ```
    [linux]$ echo -n "symbol=LTCBTC&side=BUY&type=LIMIT&timeInForce=GTC&quantity=1&price=0.1&recvWindow=5000&timestamp=1499827319559" | openssl dgst -sha256 -hmac "NhqPtmdSJYdKjVHjA7PZj4Mge3R5YNiP1e3UZjInClVN65XAbvqqM6A7H5fATj0j"
    (stdin)= c8db56825ae71d6d79447849e617115f4a920fa2acdcab2b053c4b2838bd6b71
    ```


* **curl command:**

    ```
    (HMAC SHA256)
    [linux]$ curl -H "X-JEX-APIKEY: vmPUZE6mv9SD5VNHk4HlWFsOr6aKE2zvsw0MuIgwCIPy6utIco14y7Ju91duEh8A" -X POST 'https://www.jex.com/api/v1/order' -d 'symbol=LTCBTC&side=BUY&type=LIMIT&timeInForce=GTC&quantity=1&price=0.1&recvWindow=5000&timestamp=1499827319559&signature=c8db56825ae71d6d79447849e617115f4a920fa2acdcab2b053c4b2838bd6b71'
    ```

### Example 3: Mixed query string and request body
* **queryString:** symbol=LTCBTC&side=BUY&type=LIMIT&timeInForce=GTC
* **requestBody:** quantity=1&price=0.1&recvWindow=5000&timestamp=1499827319559
* **HMAC SHA256 signature:**

    ```
    [linux]$ echo -n "symbol=LTCBTC&side=BUY&type=LIMIT&timeInForce=GTCquantity=1&price=0.1&recvWindow=5000&timestamp=1499827319559" | openssl dgst -sha256 -hmac "NhqPtmdSJYdKjVHjA7PZj4Mge3R5YNiP1e3UZjInClVN65XAbvqqM6A7H5fATj0j"
    (stdin)= 0fd168b8ddb4876a0358a8d14d0c9f3da0e9b20c5d52b2a00fcf7d1c602f9a77
    ```


* **curl command:**

    ```
    (HMAC SHA256)
    [linux]$ curl -H "X-JEX-APIKEY: vmPUZE6mv9SD5VNHk4HlWFsOr6aKE2zvsw0MuIgwCIPy6utIco14y7Ju91duEh8A" -X POST 'https://www.jex.com/api/v1/order?symbol=LTCBTC&side=BUY&type=LIMIT&timeInForce=GTC' -d 'quantity=1&price=0.1&recvWindow=5000&timestamp=1499827319559&signature=0fd168b8ddb4876a0358a8d14d0c9f3da0e9b20c5d52b2a00fcf7d1c602f9a77'
    ```

Note that the signature is different in example 3.
There is no & between "GTC" and "quantity=1".




# Public API Endpoints
## Terminology
* `base asset` refers to the asset that is the `quantity` of a symbol.
* `quote asset` refers to the asset that is the `price` of a symbol.



## ENUM definitions
**Symbol status (status):**

* PRE_TRADING 
* TRADING 
* POST_TRADING 
* END_OF_DAY 
* HALT 
* AUCTION_MATCH 
* BREAK 

**Symbol type:**

* SPOT 
* OPTION 
* CONTRACT 

**spot and option Order status (status):**

* NEW 
* PARTIALLY_FILLED  
* FILLED  
* CANCELED  
* PENDING_CANCEL (currently unused))
* FAIL 
* CANCLEFILLED
* REJECTED  (currently unused)
* EXPIRED  (currently unused)


**contract status (status):**
 
* ENTRUSTED 
* ENTRUSTING 
* FAIL 
* PARTFILLED 
* FILLED 
* CANCEL 

**Order types (orderTypes, type):**

* LIMIT 
* MARKET  
* STOP_LOSS 
* STOP_LOSS_LIMIT 
* TAKE_PROFIT 
* TAKE_PROFIT_LIMIT 
* LIMIT_MAKER 

**Order side (side):**

* BUY 
* SELL 

**Time in force:**

* GTC - Good Till Cancel 
* IOC - Immediate or Cancel 
* FOK - Fill or Kill 

**Kline/Candlestick chart intervals:**

m -> minutes; h -> hours; d -> days; w -> weeks; M -> months

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

**Rate limiters (rateLimitType)**

* REQUESTS_WEIGHT  
* ORDERS    
* RAW_REQUESTS 

**Rate limit intervals (interval)**

* SECOND
* MINUTE
* DAY

**Endpoints for futures account**
* transfer       
* fee            
* funding        
* closPosition   
* liquidation    
* adl            
* autoMargin     
* positionSize   



## General endpoints
### Test connectivity PING
```
GET /api/v1/ping
```
Test connectivity 

**Weight:**
1

**Parameters:**
NONE

**Response:**
```javascript
{}
```

### Check server time
```
GET /api/v1/time
```
Get the current server time.

**Weight:**
1

**Parameters:**
NONE

**Response:**
```javascript
{
  "serverTime": 1499827319595
}
```

### Exchange information
```
GET /api/v1/exchangeInfo
```
Get the current trading rules and symbol information

**Weight:**
1

**Parameters:**
NONE

**Response:**
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

### Exchange information for options
```
GET /api/v1/optionInfo
```
Get the current trading rules and symbol information

**Weight:**
1

**Parameters:**
NONE

**Response:**
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

### Exchange information for futures
```
GET /api/v1/contractInfo
```
Get the current trading rules and symbol information

**Weight:**
1

**Parameters:**
NONE

**Response:**
```javascript
[
  {
    "fundingRate": 0.000002,
    "fundingInterval": 600000,
    "nextFundingTime": 1551337200000,
    "predictedFundingRate": 0.000002,
    "turnover24": 212921.712,
    "markMethod": 2,
    "markPrice": 2300.0046,
    "lastPrice": 2300,
    "autoDeleveraging": "DISABLE",
    "symbol": "BTCUSDT"
  }
]
```




## Market Data endpoints
### Depth information for spot
```
GET /api/v1/spot/depth
```

**Weight:1**


**Parameters:**

Name | Type   | Mandatory  | Description
------------ | ------------ | ------------ | ------------
symbol | STRING | YES |
limit | INT | NO | Default  60; Max 60. Available:[5, 10, 20, 50, 60]


**Response:**
```javascript
{
  "lastUpdateId": 1027024,
  "bids": [
    [
      "4.00000000",     // PRICE
      "431.00000000",   // QTY
      []                // IGNORE.
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
### Depth information for options
```
GET /api/v1/option/depth
```
**Weight:1**  
**Parameters:**  

Name | Type   | Mandatory  | Description  
------------ | ------------ | ------------ | ------------  
symbol | STRING | YES |  
limit | INT | NO | Default  60; Max 60. Available:[5, 10, 20, 50, 60]  
**Response:**  
```javascript
{
  "lastUpdateId": 1027024,
  "bids": [
    [
      "4.00000000",     // PRICE
      "431.00000000",   // QTY
      []                // IGNORE.
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
### Depth information for futures
```
GET /api/v1/contract/depth
```
**Weight:1**  
**Parameters:**  

Name | Type   | Mandatory  | Description
------------ | ------------ | ------------ | ------------
symbol | STRING | YES |
limit | INT | NO | Default  60; Max 60. Available:[5, 10, 20, 50, 60]  
**Response:**  
```javascript
{
  "lastUpdateId": 1027024,
  "bids": [
    [
      "4.00000000",     // PRICE
      "431.00000000",   // QTY
      []                // IGNORE.
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


### Recent trades for spot
```
GET /api/v1/spot/trades
```
Get recent trades

**Weight:**
1

**Parameters:**

Name | Type | Mandatory | Description
------------ | ------------ | ------------ | ------------
symbol | STRING | YES |
limit | INT | NO | Default 60; max 60.

**Response:**
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
### Recent trades for options
```
GET /api/v1/option/trades
```
**Weight:1**  
**Parameters:**  

Name | Type | Mandatory | Description
------------ | ------------ | ------------ | ------------
symbol | STRING | YES |
limit | INT | NO | Default 60; max 60.
  
**Response:**  
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
### Recent trades for futures
```
GET /api/v1/contract/trades
```
**Weight:1**  
**Parameters:**  

Name | Type | Mandatory | Description
------------ | ------------ | ------------ | ------------
symbol | STRING | YES |
limit | INT | NO | Default 60; max 60.  
**Response:**  
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



### Old trade lookup (MARKET_DATA)
```
GET /api/v1/spot/historicalTrades
```

**Weight:**
5

**Parameters:**

Name | Type | Mandatory | Description
------------ | ------------ | ------------ | ------------
symbol | STRING | YES |
limit | INT | NO | Default 200; max 500.
fromId | LONG | NO |TradeId to fetch from. Default gets most recent trades.

**Response:**
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
### Old trade lookup for options (MARKET_DATA)
```
GET /api/v1/option/historicalTrades
```
**Weight:5**  
**Parameters:**  

Name | Type | Mandatory | Description
------------ | ------------ | ------------ | ------------
symbol | STRING | YES |
limit | INT | NO | Default 200; max 500.
fromId | LONG | NO | TradeId to fetch from. Default gets most recent trades
  
**Response:**  
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
### Old trade lookup for futures (MARKET_DATA)
```
GET /api/v1/contract/historicalTrades
```
**Weight:5**  
**Parameters:**  

Name | Type | Mandatory | Description
------------ | ------------ | ------------ | ------------
symbol | STRING | YES |
limit | INT | NO | Default 200; max 500.
fromId | LONG | NO | TradeId to fetch from. Default gets most recent trades  
**Response:**  
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



### Kline/Candlestick data
```
GET /api/v1/spot/klines
```
Kline/candlestick bars for a symbol. Klines are uniquely identified by their open time.

**Weight:**
1

**Parameters:**

Name | Type | Mandatory | Description
------------ | ------------ | ------------ | ------------
symbol | STRING | YES |
interval | ENUM | YES | "1m":minute"3m": minute"5m": minute"15m":minute"30m":minute"1h":hour"2h":hour"4h":hour"6h":"12h":hour"1d":day"3d":day"1w":week
startTime | LONG | NO |
endTime | LONG | NO |
limit | INT | NO | Default 500; max 1000.

* If startTime and endTime are not sent, the most recent klines are returned.

**Response:**
```javascript
[
  [
    1499040000000,      // Open time
    "0.01634790",       // Open
    "0.80000000",       // High
    "0.01575800",       // Low
    "0.01577100",       // Close
    "148976.11427815",  // Volume
    1499644799999,      // Close time
    "2434.19055334",    // Quote asset volume
    308,                // Number of trades
    "1756.87402397",    // Taker buy base asset volume
    "28.46694368",      // Taker buy quote asset volume
    "17928899.62484339" // Ignore
  ]
]
```
### Kline data lookup for options
```
GET /api/v1/option/klines
```
**Weight:1**  
**Parameters:**  

Name | Type | Mandatory | Description
------------ | ------------ | ------------ | ------------
symbol | STRING | YES |
interval | ENUM | YES | "1m":minute"3m": minute"5m": minute"15m":minute"30m":minute"1h":hour"2h":hour"4h":hour"6h":"12h":hour"1d":day"3d":day"1w":week
startTime | LONG | NO |
endTime | LONG | NO |
limit | INT | NO | Default 500; max 1000.

* 缺省返回最近的数据
  
**Response:**  
```javascript
[
  [
    1499040000000,      // Open time
    "0.01634790",       // Open
    "0.80000000",       // High
    "0.01575800",       // Low
    "0.01577100",       // Close
    "148976.11427815",  // Volume
    1499644799999,      // Close time
    "2434.19055334",    // Quote asset volume
    308,                // Number of trades
    "1756.87402397",    // Taker buy base asset volume
    "28.46694368",      // Taker buy quote asset volume
    "17928899.62484339" // Ignore
  ]
]
```
### Kline data lookup for futures
```
GET /api/v1/contract/klines
```
**Weight:1**  
**Parameters:**  

Name | Type | Mandatory | Description
------------ | ------------ | ------------ | ------------
symbol | STRING | YES |
interval | ENUM | YES | "1m":minute"3m": minute"5m": minute"15m":minute"30m":minute"1h":hour"2h":hour"4h":hour"6h":"12h":hour"1d":day"3d":day"1w":week
startTime | LONG | NO |
endTime | LONG | NO |
limit | INT | NO | Default 500; max 1000.

* If startTime and endTime are not sent, the most recent klines are returned.
  
**Response:**  
```javascript
[
  [
    1499040000000,      // Open time
    "0.01634790",       // Open
    "0.80000000",       // High
    "0.01575800",       // Low
    "0.01577100",       // Close
    "148976.11427815",  // Volume
    1499644799999,      // Close time
    "2434.19055334",    // Quote asset volume
    308,                // Number of trades
    "1756.87402397",    // Taker buy base asset volume
    "28.46694368",      // Taker buy quote asset volume
    "17928899.62484339" // Ignore
  ]
]
```





### Current average price
```
GET /api/v1/spot/avgPrice
```
**Weight:**
1
**Parameters:**

Name | Type | Mandatory | Description
------------ | ------------ | ------------ | ------------
symbol | STRING | YES |

**Response:**
```javascript
{
  "mins": 5,
  "price": "9.35751834"
}
```
### Look up current average price for options
```
GET /api/v1/option/avgPrice
```
**Weight:1**  
**Parameters:**  

Name | Type | Mandatory | Description
------------ | ------------ | ------------ | ------------
symbol | STRING | YES | 

**Response:**  
```javascript
{
  "mins": 5,
  "price": "9.35751834"
}
```
### Look up current average price for futures
```
GET /api/v1/contract/avgPrice
```
**Weight:1**  
**Parameters:**  

Name | Type | Mandatory | Description
------------ | ------------ | ------------ | ------------
symbol | STRING | YES |  
**Response:**  
```javascript
{
  "mins": 5,
  "price": "9.35751834"
}
```







### 24hr ticker price change statistics
```
GET /api/v1/spot/ticker/24hr
```
24 hour rolling window price change statistics. Careful when accessing this with no symbol.

**Weight:**
1 for a single symbol; **40** when the symbol parameter is omitted


**Parameters:**

Name | Type | Mandatory | Description
------------ | ------------ | ------------ | ------------
symbol | STRING | NO |


**Response:**
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
### Look up 24hr ticker price change statistics for options
```
GET /api/v1/option/ticker/24hr
```
**Weight:**  
1 for a single symbol; **40** when the symbol parameter is omitted

**Parameters:**  

Name | Type | Mandatory | Description
------------ | ------------ | ------------ | ------------
symbol | STRING | NO |
  
**Response:**  
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
### Look up 24hr ticker price change statistics for futrues
```
GET /api/v1/contract/ticker/24hr
```
**Weight:**  
1 for a single symbol; **40** when the symbol parameter is omitted

**Parameters:**  

Name | Type | Mandatory | Description
------------ | ------------ | ------------ | ------------
symbol | STRING | NO |
  
**Response:**  
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









### Price ticker for spot
```
GET /api/v1/spot/ticker/price
```
Back to newest price

**Weight:**
1 for a single symbol; **2** when the symbol parameter is omitted


**Parameters:**

Name | Type | Mandatory | Description
------------ | ------------ | ------------ | ------------
symbol | STRING | NO |

* If the symbol is not sent, prices for all symbols will be returned in an array.

**Response:**
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
### Look up price ticker for options
```
GET /api/v1/option/ticker/price
```
**Weight:**  
1 for a single symbol; **2** when the symbol parameter is omitted
**Parameters:**  

Name | Type | Mandatory | Description
------------ | ------------ | ------------ | ------------
symbol | STRING | NO |

* If the symbol is not sent, prices for all symbols will be returned in an array.
**Response:**  
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
### Look up price ticker for futures
```
GET /api/v1/contract/ticker/price
```
**Weight:**  
1 for a single symbol; **2** when the symbol parameter is omitted

**Parameters:**  

Name | Type | Mandatory | Description
------------ | ------------ | ------------ | ------------
symbol | STRING | NO |

* If the symbol is not sent, prices for all symbols will be returned in an array.
  
**Response:**  
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







### Symbol order book ticker for spot
```
GET /api/v1/spot/ticker/bookTicker
```
Best price/qty on the order book for a symbol or symbols.

**Weight:**
1 for a single symbol; **2** when the symbol parameter is omitted

**Parameters:**

Name | Type | Mandatory | Description
------------ | ------------ | ------------ | ------------
symbol | STRING | NO |

* If the symbol is not sent, prices for all symbols will be returned in an array.

**Response:**
```javascript
{
  "symbol": "LTCBTC",
  "bidPrice": "4.00000000",//Best price for buy
  "bidQty": "431.00000000",//QTY
  "askPrice": "4.00000200",//Best price for sell
  "askQty": "9.00000000"//QTY
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
### Look up Symbol order book ticker for options
```
GET /api/v1/option/ticker/bookTicker
```
Best price/qty on the order book for a symbol or symbols.
**Weight:**  
1 for a single symbol; **2** when the symbol parameter is omitted 
**Parameters:**  

Name | Type | Mandatory | Description
------------ | ------------ | ------------ | ------------
symbol | STRING | NO |

* If the symbol is not sent, prices for all symbols will be returned in an array.  
**Response:**  
```javascript
{
  "symbol": "LTCBTC",
  "bidPrice": "4.00000000",//Best price for buy
  "bidQty": "431.00000000",//QTY
  "askPrice": "4.00000200",//Best price for sell
  "askQty": "9.00000000"//QTY
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
### Look up Symbol order book ticker for futures
```
GET /api/v1/contract/ticker/bookTicker
```
Best price/qty on the order book for a symbol or symbols.
**Weight:**  
1 for a single symbol; **2** when the symbol parameter is omitted 
**Parameters:**  

Name | Type | Mandatory | Description
------------ | ------------ | ------------ | ------------
symbol | STRING | NO |

* If the symbol is not sent, prices for all symbols will be returned in an array.  
**Response:**  
```javascript
{
  "bidPrice": "4.00000000",//Best price for buy
  "bidQty": "431.00000000",//QTY
  "askPrice": "4.00000200",//Best price for sell
  "askQty": "9.00000000"//QTY
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

### Index price and mark price for futures
```
GET /api/v1/contract/ticker/indicesPrice
```
Best price/qty on the order book for a symbol or symbols.

**Weight:**
1


**Parameters:**

Name | Type | Mandatory | Description
------------ | ------------ | ------------ | ------------
symbol | STRING | NO |

* If the symbol is not sent, prices for all symbols will be returned in an array.

**Response:**
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




## Account endpoints
### Place order in coins transaction（TRADE）
```
POST /api/v1/spot/order  (HMAC SHA256)
```

**Weight:**
1

**Parameters:**

Name | Type | Mandatory | Description
------------ | ------------ | ------------ | ------------
symbol | STRING | YES |
side | ENUM | YES |
type | ENUM | YES |  `LIMIT`
quantity | DECIMAL | YES |
price | DECIMAL | YES |
newOrderRespType | ENUM | NO | Specify response type   `ACK`, `RESULT`; Default is `ACK`. 
recvWindow | LONG | NO |
timestamp | LONG | YES |



Two choices for newOrderRespType

**Response ACK:**
Returning speed is fast, trading information not included, less information 
```javascript
{
  "symbol": "JEXBTC",
  "orderId": 28,
  "transactTime": 1507725176595
}
```

**Response RESULT:**
Returning speed is slow, returning some information on taking order transaction
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
### Place order in options transaction（TRADE）  (TRADE)
```
POST /api/v1/option/order  (HMAC SHA256)
```

**Weight:**  
1  
**Parameters:**

Name | Type | Mandatory | Description
------------ | ------------ | ------------ | ------------
symbol | STRING | YES |
side | ENUM | YES |
type | ENUM | YES | `LIMIT`
quantity | DECIMAL | YES |
price | DECIMAL | YES |
newOrderRespType | ENUM | NO | Specify response type   `ACK`, `RESULT`; Default is `ACK`. 
recvWindow | LONG | NO |
timestamp | LONG | YES |
**Response:**  




Two choices for newOrderRespType

**Response ACK:**
Returning speed is fast, trading information not included, less information 
```javascript
{
  "symbol": "JEXBTC",
  "orderId": 28,
  "transactTime": 1507725176595
}
```

**Response RESULT:**
Returning speed is slow, returning some information on taking order transaction
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


### Place order in contract transaction(TRADE)
```
POST /api/v1/contract/order  (HMAC SHA256)
```

**Weight:**
1
**Parameters:**

Name | Type | Mandatory | Description
------------ | ------------ | ------------ | ------------
symbol | STRING | YES |
side | ENUM | YES |
type | ENUM | YES | `LIMIT`
quantity | DECIMAL | YES |
price | DECIMAL | YES |
newOrderRespType | ENUM | NO | Specify response type   `ACK`, `RESULT`; Default is `ACK`. 
recvWindow | LONG | NO |
timestamp | LONG | YES |


**Response ACK:**
Returning speed is fast, trading information not included, less information 
```javascript
{
  "symbol": "BTCUSDT",
  "orderId": "4613019726031880200",
}
```

**Response RESULT:**
Returning speed is fast, returning some information on taking order transaction
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






### Test placing order API of coins transaction(TRADE)
```
POST /api/v1/spot/order/test (HMAC SHA256)
```
Used for test placing order request, won’t be submitted to matchmaking trading engine

**Weight:**
1

**Parameters:**

Reference   `POST /api/v1/spot/order`


**Response:**
```javascript
{}
```

### Test placing order API of options transaction(TRADE) 
```
POST /api/v1/option/order/test (HMAC SHA256)
```
Used for test placing order request, won’t be submitted to matchmaking trading engine

**Weight:**
1

**Parameters:**

Reference   `POST /api/v1/option/order`


**Response:**
```javascript
{}
```
### Test placing order API of contract transaction(TRADE)
```
POST /api/v1/contract/order/test (HMAC SHA256)
```
Used for test placing order request, won’t be submitted to matchmaking trading engine

**Weight:**
1

**Parameters:**

Reference   `POST /api/v1/contract/order`


**Response:**
```javascript
{}
```





### Check orders of coins transaction(USER_DATA)
```
GET /api/v1/spot/order (HMAC SHA256)
```
Check order status

**Weight:**
1

**Parameters:**

Name | Type | Mandatory | Description
------------ | ------------ | ------------ | ------------
symbol | STRING | YES |
orderId | LONG | YES |
recvWindow | LONG | NO |
timestamp | LONG | YES |



**Response:**
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

### Check orders of options transaction(USER_DATA) 
```
GET /api/v1/option/order (HMAC SHA256)
```
Check order status

**Weight:**
1

**Parameters:**

Name | Type | Mandatory | Description
------------ | ------------ | ------------ | ------------
symbol | STRING | YES |
orderId | LONG | YES |

recvWindow | LONG | NO |
timestamp | LONG | YES |




**Response:**
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

### Check orders of contract transaction(USER_DATA) 
```
GET /api/v1/contract/order (HMAC SHA256)
```
Check order status

**Weight:**
1

**Parameters:**

Name | Type | Mandatory | Description
------------ | ------------ | ------------ | ------------
symbol | STRING | YES |
orderId | LONG | YES |
recvWindow | LONG | NO |
timestamp | LONG | YES |




**Response:**
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





### Cancel order for coins transaction(TRADE) 
```
DELETE /api/v1/spot/order  (HMAC SHA256)
```

**Weight:**
1

**Parameters:**

Name | Type | Mandatory | Description
------------ | ------------ | ------------ | ------------
symbol | STRING | YES |
orderId | LONG | YES |
recvWindow | LONG | NO |
timestamp | LONG | YES |

**Response:**
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

### Cancel order for options transaction(TRADE) 
```
DELETE /api/v1/option/order  (HMAC SHA256)
```

**Weight:**
1

**Parameters:**  

Name | Type | Mandatory | Description
------------ | ------------ | ------------ | ------------
symbol | STRING | YES |
orderId | LONG | YES |
recvWindow | LONG | NO |
timestamp | LONG | YES |



**Response:**
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

### Cancel order for contract transaction (TRADE) 
```
DELETE /api/v1/contract/order  (HMAC SHA256)
```

**Weight:**
1

**Parameters:**

Name | Type | Mandatory | Description
------------ | ------------ | ------------ | ------------
symbol | STRING | YES |
orderId | LONG | YES |
recvWindow | LONG | NO |
timestamp | LONG | YES |




**Response:**
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








### Check entry orders of coins transaction of this account(USER_DATA)
```
GET /api/v1/spot/openOrders  (HMAC SHA256)
```

**Weight:**  
 5


**Parameters:**

Name | Type | Mandatory | Description
------------ | ------------ | ------------ | ------------
symbol | STRING | YES |
orderId | LONG | NO | Only orders after this orderID will be returned. Only partial recent orders will be returned
startTime | LONG | NO |
endTime | LONG | NO |
limit | INT | NO | Default 500; max 500.
recvWindow | LONG | NO |
timestamp | LONG | YES |


**Response:**
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

### Check entry orders of options transaction of this account (USER_DATA) 
```
GET /api/v1/option/openOrders  (HMAC SHA256)
```

**Weight:**
5

**Parameters:**

Name | Type | Mandatory | Description
------------ | ------------ | ------------ | ------------
symbol | STRING | YES |
orderId | LONG | NO | Only orders after this orderID will be returned. Only partial recent orders will be returned
startTime | LONG | NO |
endTime | LONG | NO |
limit | INT | NO | Default 500; max 500.
recvWindow | LONG | NO |
timestamp | LONG | YES |


**Response:**
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

### Check entry orders of contract transaction of this account(USER_DATA) 
```
GET /api/v1/contract/openOrders  (HMAC SHA256)
```

**Weight:**
5

**Parameters:**

Name | Type | Mandatory | Description
------------ | ------------ | ------------ | ------------
symbol | STRING | YES |
orderId | LONG | NO | Only orders after this orderID will be returned. Only partial recent orders will be returned
startTime | LONG | NO |
endTime | LONG | NO |
limit | INT | NO | Default 500; max 500.
recvWindow | LONG | NO |
timestamp | LONG | YES |


**Response:**
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

### Close positions for contract(TRADE)
```
POST /api/v1/contract/liquidation  (HMAC SHA256)
```

**Weight:**
1

**Parameters:**

Name | Type | Mandatory | Description
------------ | ------------ | ------------ | ------------
symbol | STRING | YES |
recvWindow | LONG | NO |
timestamp | LONG | YES |


**Response:**
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

### Check orders of closed positions of the account (MARKET_DATA )
```
GET /api/v1/contract/liquidationOrder  
```

**Weight:**
5

**Parameters:**

Name | Type | Mandatory | Description
------------ | ------------ | ------------ | ------------
symbol | STRING | YES |
orderId | LONG | NO | Only orders after this orderID will be returned. Only partial recent orders will be returned
startTime | LONG | NO |
endTime | LONG | NO |
limit | INT | NO | Default 500; max 500.


**Response:**
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

### Check contract positions of the account(USER_DATA)
```
GET /api/v1/contract/position  (HMAC SHA256)
```

**Weight:**
2

**Parameters:**

Name | Type | Mandatory | Description
------------ | ------------ | ------------ | ------------
symbol | STRING | NO |
recvWindow | LONG | NO |
timestamp | LONG | YES |


**Response:**
```javascript
[
  {
    "symbol": "EOSUSDT",
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

### Adjust contract leverage of the account(USER_DATA)
```
POST /api/v1/contract/position/leverage  (HMAC SHA256)
```

**Weight:**
1

**Parameters:**

Name | Type | Mandatory | Description
------------ | ------------ | ------------ | ------------
symbol | STRING | YES |
leverage | STRING | YES | 
recvWindow | LONG | NO |
timestamp | LONG | YES |


**Response:**
```javascript
{}
```



### Account information(USER_DATA)
```
GET /api/v1/account (HMAC SHA256)
```

**Weight:**
1

**Parameters:**

Name | Type | Mandatory | Description
------------ | ------------ | ------------ | ------------
recvWindow | LONG | NO |
timestamp | LONG | YES |

**Response:**
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


### Historical coins entry order of the account (USER_DATA) 
```
GET /api/v1/spot/historyOrders  (HMAC SHA256)
```
Obtain trading history of specified trading pair

**Weight:**
5

**Parameters:**

Name | Type | Mandatory | Description
------------ | ------------ | ------------ | ------------
symbol | STRING | YES |
orderId | LONG | NO |Only orders after this orderID will be returned. Only partial recent orders will be returned
startTime | LONG | NO |
endTime | LONG | NO |
limit | INT | NO | Default 500; max 500.
recvWindow | LONG | NO |
timestamp | LONG | YES |

**Response:**
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

### Historical options entry order of the account (USER_DATA)
```
GET /api/v1/option/historyOrders  (HMAC SHA256)
```
Obtain trading history of specified trading pair

**Weight:**
5

**Parameters:**

Name | Type | Mandatory | Description
------------ | ------------ | ------------ | ------------
symbol | STRING | YES |
orderId | LONG | NO |Only orders after this orderID will be returned. Only partial recent orders will be returned
startTime | LONG | NO |
endTime | LONG | NO |
limit | INT | NO | Default 500; max 500.
recvWindow | LONG | NO |
timestamp | LONG | YES |

**Response:**
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

### Historical contract entry order of the account (USER_DATA)
```
GET /api/v1/contract/historyOrders  (HMAC SHA256)
```
Obtain trading history of specified trading pair

**Weight:**
5

**Parameters:**

Name | Type | Mandatory | Description
------------ | ------------ | ------------ | ------------
symbol | STRING | YES |
orderId | LONG | NO |Only orders after this orderID will be returned. Only partial recent orders will be returned
startTime | LONG | NO |
endTime | LONG | NO |
limit | INT | NO | Default 500; max 500.
recvWindow | LONG | NO |
timestamp | LONG | YES |

**Response:**
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



### Users short options(TRADE)
```
POST /api/v1/option/release  (HMAC SHA256)
```
Short options of a specified trading pair

**Weight:**
1

**Parameters:**

Name | Type | Mandatory | Description
------------ | ------------ | ------------ | ------------
symbol | STRING | YES |
amount | INT | YES |
recvWindow | LONG | NO |
timestamp | LONG | YES |

**Response:**
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

### Users call options(TRADE) 
```
POST /api/v1/option/back  (HMAC SHA256)
```
Users call options of a specified trading pair

**Weight:**
1

**Parameters:**

Name | Type | Mandatory | Description
------------ | ------------ | ------------ | ------------
symbol | STRING | YES |
amount | INT | YES |
recvWindow | LONG | NO |
timestamp | LONG | YES |

**Response:**
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

### Options positions of the account(USER_DATA)
```
GET /api/v1/option/position  (HMAC SHA256)
```
Get all the positions of one specified options trading pair

**Weight:**
2

**Parameters:**

Name | Type | Mandatory | Description
------------ | ------------ | ------------ | ------------
symbol | STRING | NO |
recvWindow | LONG | NO |
timestamp | LONG | YES |

**Response:**
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

### Short & Call options information of the account(USER_DATA)
```
GET /api/v1/option/info  (HMAC SHA256)
```
Get the the short & call information of one option of the account 

**Weight:**
1

**Parameters:**

Name | Type | Mandatory | Description
------------ | ------------ | ------------ | ------------
symbol | STRING | YES |
recvWindow | LONG | NO |
timestamp | LONG | YES |

**Response:**
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

### Historical records of Short & Call options of the account (USER_DATA)
```
GET /api/v1/option/record  (HMAC SHA256)
```
Short & Call Records of one specified options or all options of the account

**Weight:**
5

**Parameters:**

Name | Type | Mandatory | Description
------------ | ------------ | ------------ | ------------
symbol | STRING | YES |
startTime | LONG | NO |
endTime | LONG | NO |
fromId | LONG | NO |Only orders after this orderID will be returned. Only partial recent orders will be returned
limit | INT | NO | Default 500; max 500.
recvWindow | LONG | NO |
timestamp | LONG | YES |

**Response:**
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



### Contract bill of the account(USER_DATA)
```
GET /api/v1/contract/bill  (HMAC SHA256)
```
Contract bill of obtained account

**Weight:**
1

**Parameters:**

Name | Type | Mandatory | Description
------------ | ------------ | ------------ | ------------
fromId | LONG | NO |Only orders after this orderID will be returned. Only partial recent orders will be returned
limit | INT | NO | Default 100; max 100.
startTime | LONG | NO |
endTime | LONG | NO |
recvWindow | LONG | NO |
timestamp | LONG | YES |

**Response:**
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


### Contract bill of the account(USER_DATA)
```
GET /api/v1/contract/userHistoricalTrades  (HMAC SHA256)
```
Contract bill of obtained account

**Weight:**
1

**Parameters:**

Name | Type | Mandatory | Description
------------ | ------------ | ------------ | ------------
symbol | STRING | YES |
endId | LONG | NO |Only orders after this orderID will be returned
limit | INT | NO | Default 1000; max 1000.
recvWindow | LONG | NO |
timestamp | LONG | YES |

**Response:**
```javascript
[
  {
    "id": 854797,
    "orderId":"4612604110637448048",
    "price":"10000.0",
    "qty":"-0.1000",
    "time":1569575228000
    "feeRate":"0.00000",
    "buyerMaker":false

  },
  {
    "id":"461732",
    "orderId":"4612604110637448048",
    "price":"10000.0",
    "qty":"-1.0000",
    "time":1569575147000,
    "feeRate":"0.00000",
    "buyerMaker":false
  }
]
```


### cancel all orders

```
DELETE /api/v1/contract/batchOrder
```

**Parameters:**

Name | Type | Mandatory | Description
------------ | ------------ | ------------ | ------------
ordersJsonArray | String | YES | json String (Request 1 time in 1 second, up to 10 orders in 1 time)
recvWindow | LONG | NO |
timestamp | LONG | YES |

### ordersJsonArray json String rule example：

``` javascript
[{"symbol":"btcusdt","orderId":"4612172002566865072"},{"symbol":"btcusdt","orderId":"4612170903055237327"},{"symbol":"EOSUSDT","orderId":"4612170903055237327"},{"symbol":"ETHUSDT","orderId":"4612168704031981750"}]
```

**Response:**

``` javascript
[{
	"symbol": "BTCUSDT",
	"orderId": "4612172002566865072",
	"updateTime": 1570696952000,
	"side": "buy",
	"origQty": "1.0000",
	"executedQty": "0.0000",
	"price": "7548.7",
	"executedPrice": "0.0",
	"status": "entrusted",
	"time": 1570696952000,
	"type": "limit"
}, {
	"msg": "Order does not exist.",
	"code": -2013
}, {
	"symbol": "EOSUSDT",
	"orderId": "4612170903055237327",
	"updateTime": 1570697170000,
	"side": "buy",
	"origQty": "1.0",
	"executedQty": "0.0",
	"price": "0.136",
	"executedPrice": "0.000",
	"status": "entrusted",
	"time": 1570697170000,
	"type": "limit"
}, {
	"symbol": "ETHUSDT",
	"orderId": "4612168704031981750",
	"updateTime": 1570697224000,
	"side": "buy",
	"origQty": "1.00",
	"executedQty": "0.00",
	"price": "90.96",
	"executedPrice": "0.00",
	"status": "entrusted",
	"time": 1570697224000,
	"type": "limit"
}]
```
**Precautions and instructions:**

- Signature and request notes:
  
  1. First Signature the parameters according to the general rules
  1. urlEncode on the parameters of ordersJsonArray
  1. Send a delete request

- Restrictions：
  1. Request 1 time at most 1 second
  1. 1 time up to 10 orders



### Capital fee rate of contract
```
GET /api/v1/contract/historyRate  
```
Get capital fee of one specified contract

**Weight:**
1

**Parameters:**

Name | Type | Mandatory | Description
------------ | ------------ | ------------ | ------------
symbol | STRING | YES |
fromId | LONG | NO |Only orders after this orderID will be returned. Only partial recent orders will be returned
limit | INT | NO | Default 100; max 100.
startTime | LONG | NO |
endTime | LONG | NO |

**Response:**
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

### Contract protection fund
```
GET /api/v1/contract/protectionFund  
```
Get contract protection fund of one specified contract

**Weight:**
1

**Parameters:**

Name | Type | Mandatory | Description
------------ | ------------ | ------------ | ------------
symbol | STRING | YES |
start | LONG | NO |Only orders after this orderID will be returned. Only partial recent orders will be returned
size | INT | NO | Default 100; max 100.
startTime | LONG | NO |
endTime | LONG | NO |

**Response:**
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

### Transfer Margin(USER_DATA)
```
POST /api/v1/contract/transferMargin  (HMAC SHA256)
```
The margin transferred to one specified contract from coins account balance

**Weight:**
1

**Parameters:**

Name | Type | Mandatory | Description
------------ | ------------ | ------------ | ------------
symbol | STRING | YES |
amount | STRING | NO |
recvWindow | LONG | NO |
timestamp | LONG | YES |

**Response:**
```javascript
{
  "asset": "USDT",
  "balance": 889486,
  "lockBalance": 1.6
}
```


## User Data Stream subscription API
Specifics on how user data streams work.
For detailed subscription method, please refer to another websocket document.


### Start user data stream (USER_STREAM)
```
POST /api/v1/userDataStream
```
Start a new user data stream. The stream will close after 60 minutes unless a keepalive is sent.

**Weight:**
1

**Parameters:**
NONE

**Response:**
```javascript
{
  "listenKey": "pqia91ma19a5s61cv6a81va65sdf19v8a65a1a5s61cv6a81va65sdf19v8a65a1" //用于订阅的数据流名
}
```

### Keepalive (USER_STREAM)
```
PUT /api/v1/userDataStream
```
Keepalive a user data stream to prevent a time out. User data streams will close after 60 minutes. It's recommended to send a ping about every 30 minutes.

**Weight:**
1

**Parameters:**

Name | Type | Mandatory | Description
------------ | ------------ | ------------ | ------------
listenKey | STRING | YES

**Response:**
```javascript
{}
```

### Close user data stream (USER_STREAM)
```
DELETE /api/v1/userDataStream
```

**Weight:**
1

**Parameters:**

Name | Type | Mandatory | Description
------------ | ------------ | ------------ | ------------
listenKey | STRING | YES

**Response:**
```javascript
{}
```



### User’s assets (USER_DATA)
```
GET /wapi/v1/assetDetail
```

**Weight:**
1

**Parameters:**

Name | Type | Mandatory | Description
------------ | ------------ | ------------ | ------------
recvWindow |	LONG |	NO	
timestamp |	LONG |	YES	
**Response:**
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


### Depositing address of the user(USER_DATA)
```
GET /wapi/v1/depositAddress
```

**Weight:**
1

**Parameters:**

Name | Type | Mandatory | Description
------------ | ------------ | ------------ | ------------
asset |	STRING |	YES	
recvWindow |	LONG |	NO	
timestamp |	LONG |	YES	
**Response:**
```javascript
{
  "address": "chongzhidizhichongzhidizhi",
  "asset": "BTC",
  "success": true
}
```



### Historical depositing records of the user (USER_DATA)
```
GET /wapi/v1/depositHistory
```

**Weight:**
5

**Parameters:**

Name | Type | Mandatory | Description
------------ | ------------ | ------------ | ------------
asset |	STRING |	NO	|
status |	int |	NO	| 0:pending,1:success
startTime | LONG | NO |
endTime | LONG | NO |
recvWindow |	LONG |	NO	
timestamp |	LONG |	YES	
**Response:**
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

### Service fee of the trading (USER_DATA)
```
GET /wapi/v1/tradeFee
```

**Weight:**
1

**Parameters:**

Name | Type | Mandatory | Description
------------ | ------------ | ------------ | ------------
symbol |	STRING |	NO	|
recvWindow |	LONG |	NO	
timestamp |	LONG |	YES	
**Response:**
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

### Historical withdrawal records of the user(USER_DATA)
```
GET /wapi/v1/withdrawHistory
```

**Weight:**
5

**Parameters:**

Name | Type | Mandatory | Description
------------ | ------------ | ------------ | ------------
asset |	STRING |	NO	|
status |	INT |	NO	| 0:Email Sent,1:Cancelled 2:Awaiting Approval 3:Rejected 4:Processing 5:Failure 6Completed
startTime | LONG | NO |
endTime | LONG | NO |
recvWindow |	LONG |	NO	
timestamp |	LONG |	YES	
**Response:**
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
