# Question answer
## Error response
### -1022 Signature for this request is not valid or user does not exist 
 * Key or Secret error
 * The parameters must be in the order of the interface document
 * Please ensure that the parameters are signed using the HMACSHA256 method
 * The method of get or post in the Request Method Note that different request methods correspond to different interfaces.
 * Content-type: application/x-www-form-urlencoded 

### -1021 Timestamp for this request is outside of the recvWindow
* First get the interface of the server time `/api/v1/time`
* Prevent local time and server time from being inconsistent, get the difference between server time and local time, calculate correction parameter timestamp `timestamp`
* Time synchronization security on the document address is[Timing security](./rest-api.md#timing-security) 
### -1121 Invalid symbol 
* Make sure the symbol is obtained from the `/api/v1/exchangeInfo` interface。
* The spelling of the trading pair is `symbol`. The uppercase letter is `SYMBOL`

### -1003 Too many requests; current limit is %d requests %s.
* The frequency limit of this interface has been exceeded. This interface is temporarily unavailable。
* Try to connect to the market with websocket 
* Reduce the frequency of use of the interface and request according to the restrictions of the interface
* Access restricted documents are in  [LIMITS](./rest-api.md#limits)

## question

### How to get a symbol
* Get all the symbols（spot，option，contract）[GET  /api/v1/exchangeInfo ](./rest-api.md#exchange-information) | symbol information:
  - `optionSymbols` Attribute is the trading pair of options
  - `contractSymbols` Attribute is a contract transaction pair
  - `spotSymbols` Attribute is a spot transaction pair


### How to connect websocket
* websocket the base endpoint is: **wss://ws.jex.com**， Raw streams are accessed at **/ws/\<streamName\>** or Combined streams are accessed at **/stream?streams=\<streamName1\>/\<streamName2\>/\<streamName3\>**
* Example: In the url format of direct access, javascript is a btcusdt in the web browser for currency trading.：
    - Trade Streams `new WebSocket('wss://ws.jex.com/ws/btcusdt@spotTrade')`
    - Kline/Candlestick Streams `new WebSocket('wss://ws.jex.com/ws/btcusdt@spotKline_1M')`
    - Individual Symbol Mini Ticker Stream `new WebSocket('wss://ws.jex.com/ws/btcusdt@spotMiniTicker')`
    - Individual Symbol Ticker Streams `new WebSocket('wss://ws.jex.com/ws/btcusdt@spotTicker')`
    - Partial Book Depth Streams `new WebSocket('wss://ws.jex.com/ws/btcusdt@spotDepth5')`

### The order status returned when the contract is placed is  entrusting. The order is queried in the current order by orderId. The return does not exist
* entrusting Not yet in the current commission，May wait for it to become a successful order。 It may also be that after a while, it may become a failure to place an order due to certain leverage or margin.You can view the order with the inquiry order information  [GET /api/v1/contract/order (HMAC SHA256)](./rest-api.md#place-order-in-contract-transactiontrade) | Place order in contract transaction (USER_DATA)

 


