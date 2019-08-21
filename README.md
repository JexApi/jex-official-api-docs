## Official Documentation for the JEX APIs and Streams.
* Streams, endpoints, parameters, payloads, etc. described in the documents in this repository are considered **official** and **supported**.
* The use of any other streams, endpoints, parameters, or payloads, etc. is **not supported**; **use them at your own risk and with no guarantees.**

Name | Description
------------ | ------------ 
[rest-api.md](./rest-api.md) | Details on the Rest API (/api)
[errors.md](./errors_CN.md) | Descriptions of possible error messages from the Rest API
[web-socket-streams.md](./web-socket-streams_CN.md) | Details on available streams and payloads

## Public API for JEX Exchange
### General API
API | Description
-------------- | -------------- 
[GET /api/v1/ping](./rest-api.md#test-connectivity-ping) | Test server connectivity PING 
[GET /api/v1/time](./rest-api.md#Check server time) | Get server time
[GET /api/v1/exchangeInfo](./rest-api.md#%Exchange information) | Get restriction info and trading pair info 
[GET /api/v1/optionInfo](./rest-api.md#%E6%9C%9F%E6%9D%83%E4%BA%A4%E6%98%93%E5%AF%B9%E4%BF%A1%E6%81%AF) | Get trading pair information of options transaction
[GET /api/v1/contractInfo](./rest-api.md#%E5%90%88%E7%BA%A6%E4%BA%A4%E6%98%93%E5%AF%B9%E4%BF%A1%E6%81%AF) | Get trading pair information of contract transaction
[GET /api/v1/account (HMAC SHA256)](./rest-api.md#%E8%B4%A6%E6%88%B7%E4%BF%A1%E6%81%AF-user_data) | Asset information of the account 
[POST /api/v1/userDataStream](./rest-api.md#%E6%96%B0%E5%BB%BA%E7%94%A8%E6%88%B7%E6%95%B0%E6%8D%AE%E6%B5%81-user_stream) | Create user data stream
[PUT /api/v1/userDataStream](./rest-api.md#keepalive-user_stream) | Extend the period of validity of user data stream
[DELETE /api/v1/userDataStream](./rest-api.md#%E5%85%B3%E9%97%AD%E7%94%A8%E6%88%B7%E6%95%B0%E6%8D%AE%E6%B5%81-user_stream) | Close user data stream
[GET /wapi/v1/assetDetail](./rest-api.md#%E7%94%A8%E6%88%B7%E8%B5%84%E4%BA%A7%E8%AF%A6%E6%83%85-user_data) | Details of user assets
[GET /wapi/v1/depositAddress](./rest-api.md#%E7%94%A8%E6%88%B7%E5%85%85%E5%80%BC%E5%9C%B0%E5%9D%80-user_data) | User's deposit address
[GET /wapi/v1/depositHistory](./rest-api.md#%E7%94%A8%E6%88%B7%E5%85%85%E5%80%BC%E5%8E%86%E5%8F%B2%E8%AE%B0%E5%BD%95-user_data) | Deposit history 
[GET /wapi/v1/tradeFee](./rest-api.md#%E7%94%A8%E6%88%B7%E4%BA%A4%E6%98%93%E6%89%8B%E7%BB%AD%E8%B4%B9-user_data) | Transaction fee 
[GET /wapi/v1/withdraw](./rest-api.md#%E7%94%A8%E6%88%B7%E6%8F%90%E7%8E%B0-user_data) | Withdraw 
[GET /wapi/v1/withdrawHistory](./rest-api.md#%E7%94%A8%E6%88%B7%E6%8F%90%E7%8E%B0%E5%8E%86%E5%8F%B2%E8%AE%B0%E5%BD%95-user_data) | Withdraw History 

### Coins Transaction API
API | Description
-------------- | -------------- 
[GET /api/v1/spot/depth](./rest-api.md#%E5%B8%81%E5%B8%81%E6%B7%B1%E5%BA%A6%E4%BF%A1%E6%81%AF) | Depth
[GET /api/v1/spot/trades](./rest-api.md#%E5%B8%81%E5%B8%81%E8%BF%91%E6%9C%9F%E6%88%90%E4%BA%A4) | Recent trades
[GET /api/v1/spot/historicalTrades](./rest-api.md#%E6%9F%A5%E8%AF%A2%E5%B8%81%E5%B8%81%E5%8E%86%E5%8F%B2%E6%88%90%E4%BA%A4market_data) | Historical trades
[GET /api/v1/spot/klines](./rest-api.md#%E5%B8%81%E5%B8%81k%E7%BA%BF%E6%95%B0%E6%8D%AE) | K-line
[GET /api/v1/spot/avgPrice](./rest-api.md#%E5%B8%81%E5%B8%81%E5%BD%93%E5%89%8D%E5%B9%B3%E5%9D%87%E4%BB%B7%E6%A0%BC) | Average price now
[GET /api/v1/spot/ticker/24hr](./rest-api.md#%E5%B8%81%E5%B8%8124hr%E4%BB%B7%E6%A0%BC%E5%8F%98%E5%8A%A8%E6%83%85%E5%86%B5) | Price change in 24 hours
[GET /api/v1/spot/ticker/price](./rest-api.md#%E5%B8%81%E5%B8%81%E6%9C%80%E6%96%B0%E4%BB%B7%E6%A0%BC%E6%8E%A5%E5%8F%A3) | Latest price
[GET /api/v1/spot/ticker/bookTicker](./rest-api.md#%E5%B8%81%E5%B8%81%E6%9C%80%E4%BC%98%E6%8C%82%E5%8D%95%E6%8E%A5%E5%8F%A3) | Optimal entry order
[POST /api/v1/spot/order/test (HMAC SHA256)](./rest-api.md#%E5%B8%81%E5%B8%81%E6%B5%8B%E8%AF%95%E4%B8%8B%E5%8D%95%E6%8E%A5%E5%8F%A3-trade) | Test placing order
[POST /api/v1/spot/order  (HMAC SHA256)](./rest-api.md#%E5%B8%81%E5%B8%81%E4%B8%8B%E5%8D%95--trade) | Place order 
[GET /api/v1/spot/order (HMAC SHA256)](./rest-api.md#%E5%B8%81%E5%B8%81%E6%9F%A5%E8%AF%A2%E8%AE%A2%E5%8D%95-user_data) | Check orders
[DELETE /api/v1/spot/order  (HMAC SHA256)](./rest-api.md#%E5%B8%81%E5%B8%81%E6%92%A4%E9%94%80%E8%AE%A2%E5%8D%95-trade) | Cancel order 
[GET /api/v1/spot/openOrders  (HMAC SHA256)](./rest-api.md#%E6%9F%A5%E7%9C%8B%E8%B4%A6%E6%88%B7%E5%BD%93%E5%89%8D%E5%B8%81%E5%B8%81%E6%8C%82%E5%8D%95-user_data) | Check entry orders 
[GET /api/v1/spot/historyOrders  (HMAC SHA256)](./rest-api.md#%E8%B4%A6%E6%88%B7%E5%B8%81%E5%B8%81%E5%8E%86%E5%8F%B2%E5%A7%94%E6%89%98-user_data) | Historical entry orders
 


### Options Transaction API
API | Description
-------------- | -------------- 
[GET /api/v1/option/depth](./rest-api.md#%E6%9C%9F%E6%9D%83%E6%B7%B1%E5%BA%A6%E4%BF%A1%E6%81%AF) | Depth
[GET /api/v1/option/trades](./rest-api.md#%E6%9C%9F%E6%9D%83%E8%BF%91%E6%9C%9F%E6%88%90%E4%BA%A4) | Recent trades
[GET /api/v1/option/historicalTrades](./rest-api.md#%E6%9F%A5%E8%AF%A2%E6%9C%9F%E6%9D%83%E5%8E%86%E5%8F%B2%E6%88%90%E4%BA%A4market_data) | Historical trades
[GET /api/v1/option/klines](./rest-api.md#%E6%9F%A5%E8%AF%A2%E6%9C%9F%E6%9D%83k%E7%BA%BF%E6%95%B0%E6%8D%AE) | K-line
[GET /api/v1/option/avgPrice](./rest-api.md#%E6%9F%A5%E8%AF%A2%E6%9C%9F%E6%9D%83%E5%BD%93%E5%89%8D%E5%B9%B3%E5%9D%87%E4%BB%B7%E6%A0%BC) | Average price now
[GET /api/v1/option/ticker/24hr](./rest-api.md#%E6%9F%A5%E8%AF%A2%E6%9C%9F%E6%9D%8324hr%E4%BB%B7%E6%A0%BC%E5%8F%98%E5%8A%A8%E6%83%85%E5%86%B5) | Price change in 24 hours
[GET /api/v1/option/ticker/price](./rest-api.md#%E6%9F%A5%E8%AF%A2%E6%9C%9F%E6%9D%83%E6%9C%80%E6%96%B0%E4%BB%B7%E6%A0%BC%E6%8E%A5%E5%8F%A3) | Latest price
[GET /api/v1/option/ticker/bookTicker](./rest-api.md#%E6%9F%A5%E8%AF%A2%E6%9C%9F%E6%9D%83%E6%9C%80%E4%BC%98%E6%8C%82%E5%8D%95%E6%8E%A5%E5%8F%A3) | Optimal entry order
[POST /api/v1/option/order/test (HMAC SHA256)](./rest-api.md#%E6%9C%9F%E6%9D%83%E6%B5%8B%E8%AF%95%E4%B8%8B%E5%8D%95%E6%8E%A5%E5%8F%A3-trade) | Test placing order
[POST /api/v1/option/order  (HMAC SHA256)](./rest-api.md#%E6%9C%9F%E6%9D%83%E4%B8%8B%E5%8D%95--trade) | Place order 
[GET /api/v1/option/order (HMAC SHA256)](./rest-api.md#%E6%9C%9F%E6%9D%83%E6%9F%A5%E8%AF%A2%E8%AE%A2%E5%8D%95-user_data) | Check orders
[DELETE /api/v1/option/order  (HMAC SHA256)](./rest-api.md#%E6%9C%9F%E6%9D%83%E6%92%A4%E9%94%80%E8%AE%A2%E5%8D%95-trade) | Cancel order 
[GET /api/v1/option/openOrders  (HMAC SHA256)](./rest-api.md#%E6%9F%A5%E7%9C%8B%E8%B4%A6%E6%88%B7%E5%BD%93%E5%89%8D%E6%9C%9F%E6%9D%83%E6%8C%82%E5%8D%95-user_data) | Check entry orders
[GET /api/v1/option/historyOrders  (HMAC SHA256)](./rest-api.md#%E8%B4%A6%E6%88%B7%E6%9C%9F%E6%9D%83%E5%8E%86%E5%8F%B2%E5%A7%94%E6%89%98-user_data) | Historical entry orders 
[POST /api/v1/option/release  (HMAC SHA256)](./rest-api.md#%E7%94%A8%E6%88%B7%E5%8F%91%E8%A1%8C%E6%9C%9F%E6%9D%83-trade) | Users short options
[POST /api/v1/option/back  (HMAC SHA256)](./rest-api.md#%E7%94%A8%E6%88%B7%E8%B5%8E%E5%9B%9E%E6%9C%9F%E6%9D%83-trade) | Users call options
[GET /api/v1/option/position  (HMAC SHA256)](./rest-api.md#%E8%B4%A6%E6%88%B7%E6%9C%9F%E6%9D%83%E6%8C%81%E4%BB%93-user_data) | Options positions of the account
[GET /api/v1/option/info  (HMAC SHA256)](./rest-api.md#%E8%B4%A6%E6%88%B7%E6%9C%9F%E6%9D%83%E5%8F%91%E8%A1%8C%E8%B5%8E%E5%9B%9E%E4%BF%A1%E6%81%AF-user_data) | Short & call options info of the account
[GET /api/v1/option/record  (HMAC SHA256)](./rest-api.md#%E8%B4%A6%E6%88%B7%E6%9C%9F%E6%9D%83%E5%8F%91%E8%A1%8C%E8%B5%8E%E5%9B%9E%E8%AE%B0%E5%BD%95-user_data) | Short & call options records of the account 


### Contract Transaction API
API | Description
-------------- | -------------- 
[GET /api/v1/contract/depth](./rest-api.md#%E5%90%88%E7%BA%A6%E6%B7%B1%E5%BA%A6%E4%BF%A1%E6%81%AF) | Depth
[GET /api/v1/contract/trades](./rest-api.md#%E5%90%88%E7%BA%A6%E8%BF%91%E6%9C%9F%E6%88%90%E4%BA%A4) | Recent trades
[GET /api/v1/contract/historicalTrades](./rest-api.md#%E6%9F%A5%E8%AF%A2%E5%90%88%E7%BA%A6%E5%8E%86%E5%8F%B2%E6%88%90%E4%BA%A4market_data) | Historical trades
[GET /api/v1/contract/klines](./rest-api.md#%E6%9F%A5%E8%AF%A2%E5%90%88%E7%BA%A6k%E7%BA%BF%E6%95%B0%E6%8D%AE) | K-line
[GET /api/v1/contract/avgPrice](./rest-api.md#%E6%9F%A5%E8%AF%A2%E5%90%88%E7%BA%A6%E5%BD%93%E5%89%8D%E5%B9%B3%E5%9D%87%E4%BB%B7%E6%A0%BC) | Average price now
[GET /api/v1/contract/ticker/24hr](./rest-api.md#%E6%9F%A5%E8%AF%A2%E5%90%88%E7%BA%A624hr%E4%BB%B7%E6%A0%BC%E5%8F%98%E5%8A%A8%E6%83%85%E5%86%B5) | Price change in 24 hours
[GET /api/v1/contract/ticker/price](./rest-api.md#%E6%9F%A5%E8%AF%A2%E5%90%88%E7%BA%A6%E6%9C%80%E6%96%B0%E4%BB%B7%E6%A0%BC%E6%8E%A5%E5%8F%A3) | Latest price
[GET /api/v1/contract/ticker/bookTicker](./rest-api.md#%E6%9F%A5%E8%AF%A2%E5%90%88%E7%BA%A6%E6%9C%80%E4%BC%98%E6%8C%82%E5%8D%95%E6%8E%A5%E5%8F%A3) | Optimal entry order
[GET /api/v1/contract/ticker/indicesPrice](./rest-api.md#%E5%90%88%E7%BA%A6%E7%9A%84%E6%8C%87%E6%95%B0%E4%BB%B7%E6%A0%BC%E6%A0%87%E8%AE%B0%E4%BB%B7%E6%A0%BC) | Indices Price, marked price
[POST /api/v1/contract/order/test (HMAC SHA256)](./rest-api.md#%E5%90%88%E7%BA%A6%E6%B5%8B%E8%AF%95%E4%B8%8B%E5%8D%95%E6%8E%A5%E5%8F%A3-trade) | Test placing order
[POST /api/v1/contract/order  (HMAC SHA256)](./rest-api.md#%E5%90%88%E7%BA%A6%E4%B8%8B%E5%8D%95--trade) | Place order
[GET /api/v1/contract/order (HMAC SHA256)](./rest-api.md#%E5%90%88%E7%BA%A6%E6%9F%A5%E8%AF%A2%E8%AE%A2%E5%8D%95-user_data) | Cancel order
[DELETE /api/v1/contract/order  (HMAC SHA256)](./rest-api.md#%E5%90%88%E7%BA%A6%E6%92%A4%E9%94%80%E8%AE%A2%E5%8D%95-trade) | Check entry orders
[GET /api/v1/contract/openOrders  (HMAC SHA256)](./rest-api.md#%E6%9F%A5%E7%9C%8B%E8%B4%A6%E6%88%B7%E5%BD%93%E5%89%8D%E5%90%88%E7%BA%A6%E6%8C%82%E5%8D%95-user_data) | Contract liquidation
[POST /api/v1/contract/liquidation  (HMAC SHA256)](./rest-api.md#%E5%90%88%E7%BA%A6%E5%B9%B3%E4%BB%93-trade) | Contract liquidation
[GET /api/v1/contract/liquidationOrder](./rest-api.md#%E6%9F%A5%E7%9C%8B%E8%B4%A6%E6%88%B7%E5%90%88%E7%BA%A6%E5%B9%B3%E4%BB%93%E5%8D%95-market_data) | Check liquidation orders
[GET /api/v1/contract/position  (HMAC SHA256)](./rest-api.md#%E6%9F%A5%E7%9C%8B%E8%B4%A6%E6%88%B7%E5%90%88%E7%BA%A6%E4%BB%93%E4%BD%8D-user_data) | Check contract positions of the account
[GET /api/v1/contract/position/leverage  (HMAC SHA256)](./rest-api.md#%E8%B0%83%E6%95%B4%E8%B4%A6%E6%88%B7%E5%90%88%E7%BA%A6%E6%9D%A0%E6%9D%86-user_data) | Adjust contract leverage of the account
[GET /api/v1/contract/historyOrders  (HMAC SHA256)](./rest-api.md#%E8%B4%A6%E6%88%B7%E5%90%88%E7%BA%A6%E5%8E%86%E5%8F%B2%E5%A7%94%E6%89%98-user_data) | Historical entry orders of the account
[GET /api/v1/contract/bill  (HMAC SHA256)](./rest-api.md#%E8%B4%A6%E6%88%B7%E5%90%88%E7%BA%A6%E8%B4%A6%E5%8D%95-user_data) | Contract bill
[GET /api/v1/contract/historyRate](./rest-api.md#%E5%90%88%E7%BA%A6%E8%B5%84%E9%87%91%E8%B4%B9%E7%8E%87) | Contract capital fee
[GET /api/v1/contract/protectionFund](./rest-api.md#%E5%90%88%E7%BA%A6%E4%BF%9D%E6%8A%A4%E5%9F%BA%E9%87%91) | Contract protection fund
[POST /api/v1/contract/transferMargin  (HMAC SHA256)](./rest-api.md#%E8%BD%AC%E7%A7%BB%E4%BF%9D%E8%AF%81%E9%87%91-user_data) | Transfer margin 
[POST /api/v1/contract/turnoutMargin  (HMAC SHA256)](./rest-api.md#%E8%BD%AC%E5%87%BA%E4%BF%9D%E8%AF%81%E9%87%91-user_data) | Turn out margin


### WebSocket Common Data
Stream | Description
-------------- | -------------- 
[\<symbol\>@\<tradeType\>](./web-socket-streams.md#%E6%9C%80%E8%BF%91%E6%88%90%E4%BA%A4) | Recent trades
[\<symbol\>@\<klineType\>_\<interval\>](./web-socket-streams.md#K%E7%BA%BF) | K-line
[\<symbol\>@\<miniTickerType\>](./web-socket-streams.md#%E6%8C%89Symbol%E7%9A%84%E7%B2%BE%E7%AE%80Ticker) | Simplified Ticker
[\<symbol\>@\<tradeType\>](./web-socket-streams.md#%E6%8C%89Symbol%E7%9A%84%E5%AE%8C%E6%95%B4Ticker) | Complete Ticker
[\<symbol\>@\<depthType\>\<levels\>](./web-socket-streams.md#%E6%9C%89%E9%99%90%E6%A1%A3%E6%B7%B1%E5%BA%A6%E4%BF%A1%E6%81%AF) | Depth information
[\<symbol\>@\<depthType\>](./web-socket-streams.md#%E5%A2%9E%E9%87%8F%E6%B7%B1%E5%BA%A6%E4%BF%A1%E6%81%AFstream) | Depth information stream

### webSocket User Data
Stream | Description
-------------- | -------------- 
[accountSpotInfo](./web-socket-streams.md#%E8%B4%A6%E6%88%B7%E6%9B%B4%E6%96%B0) | Change of spot assets
[accountSpotInfo](./web-socket-streams.md#%E8%B4%A6%E6%88%B7%E6%9B%B4%E6%96%B0) | Change of options assets
[accountContractInfo](./web-socket-streams.md#%E8%B4%A6%E6%88%B7%E6%9B%B4%E6%96%B0) | Change of contract assets
[execSpotReport](./web-socket-streams.md#%E8%AE%A2%E5%8D%95%E4%BA%A4%E6%98%93%E6%9B%B4%E6%96%B0) | Update spot orders/ trades
[execOptionReport](./web-socket-streams.md#%E8%AE%A2%E5%8D%95%E4%BA%A4%E6%98%93%E6%9B%B4%E6%96%B0) | Update options orders/ trades
[execContractReport](./web-socket-streams.md#%E8%AE%A2%E5%8D%95%E4%BA%A4%E6%98%93%E6%9B%B4%E6%96%B0) | Update contract orders/ trades
[contractPositions](./web-socket-streams.md#%E5%90%88%E7%BA%A6%E6%8C%81%E4%BB%93) | Positions of contract



