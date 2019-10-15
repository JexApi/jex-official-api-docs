## Official Documentation for the JEX APIs and Streams.
* Streams, endpoints, parameters, payloads, etc. described in the documents in this repository are considered **official** and **supported**.
* The use of any other streams, endpoints, parameters, or payloads, etc. is **not supported**; **use them at your own risk and with no guarantees.**

Name | Description
------------ | ------------ 
[rest-api.md](./rest-api.md) | Details on the Rest API (/api)
[errors.md](./errors_CN.md) | Descriptions of possible error messages from the Rest API
[web-socket-streams.md](./web-socket-streams.md) | Details on available streams and payloads
[problem-solving.md](./problem-solving.md) | Common Questions and Answers
[CHANGELOG.md](./CHANGELOG.md) | Update log

## Public API for JEX Exchange
### General API
API | Description
-------------- | -------------- 
[GET /api/v1/ping](./rest-api.md#test-connectivity-ping) | Test server connectivity PING 
[GET /api/v1/time](./rest-api.md#Check%20server%20time) | Get server time
[GET /api/v1/exchangeInfo](./rest-api.md#exchange-information) | Get restriction info and trading pair info 
[GET /api/v1/optionInfo](./rest-api.md#exchange-information-for-options) | Get trading pair information of options transaction
[GET /api/v1/contractInfo](./rest-api.md#exchange-information-for-futures) | Get trading pair information of contract transaction
[GET /api/v1/account (HMAC SHA256)](./rest-api.md#account-informationuser_data) | Asset information of the account 
[POST /api/v1/userDataStream](./rest-api.md#start-user-data-stream-user_stream) | Create user data stream
[PUT /api/v1/userDataStream](./rest-api.md#keepalive-user_stream) | Extend the period of validity of user data stream
[DELETE /api/v1/userDataStream](./rest-api.md#close-user-data-stream-user_stream) | Close user data stream
[GET /wapi/v1/assetDetail](./rest-api.md#users-assets-user_data) | Details of user assets
[GET /wapi/v1/depositAddress](./rest-api.md#depositing-address-of-the-useruser_data) | User's deposit address
[GET /wapi/v1/depositHistory](./rest-api.md#historical-depositing-records-of-the-user-user_data) | Deposit history 
[GET /wapi/v1/tradeFee](./rest-api.md#service-fee-of-the-trading-user_data) | Transaction fee 
[GET /wapi/v1/withdrawHistory](./rest-api.md#historical-withdrawal-records-of-the-useruser_data) | Withdraw History 

### Coins Transaction API
API | Description
-------------- | -------------- 
[GET /api/v1/spot/depth](./rest-api.md#depth-information-for-spot) | Depth
[GET /api/v1/spot/trades](./rest-api.md#recent-trades-for-spot) | Recent trades
[GET /api/v1/spot/historicalTrades](./rest-api.md#old-trade-lookup-market_data) | Historical trades
[GET /api/v1/spot/klines](./rest-api.md#klinecandlestick-data) | K-line
[GET /api/v1/spot/avgPrice](./rest-api.md#current-average-price) | Average price now
[GET /api/v1/spot/ticker/24hr](./rest-api.md#24hr-ticker-price-change-statistics) | Price change in 24 hours
[GET /api/v1/spot/ticker/price](./rest-api.md#price-ticker-for-spot) | Latest price
[GET /api/v1/spot/ticker/bookTicker](./rest-api.md#symbol-order-book-ticker-for-spot) | Optimal entry order
[POST /api/v1/spot/order/test (HMAC SHA256)](./rest-api.md#test-placing-order-api-of-coins-transactiontrade) | Test placing order
[POST /api/v1/spot/order  (HMAC SHA256)](./rest-api.md#place-order-in-coins-transactiontrade) | Place order 
[GET /api/v1/spot/order (HMAC SHA256)](./rest-api.md#check-orders-of-coins-transactionuser_data) | Check orders
[DELETE /api/v1/spot/order  (HMAC SHA256)](./rest-api.md#cancel-order-for-coins-transactiontrade) | Cancel order 
[GET /api/v1/spot/openOrders  (HMAC SHA256)](./rest-api.md#check-entry-orders-of-coins-transaction-of-this-accountuser_data) | Check entry orders 
[GET /api/v1/spot/historyOrders  (HMAC SHA256)](./rest-api.md#historical-coins-entry-order-of-the-account-user_data) | Historical entry orders
 


### Options Transaction API
API | Description
-------------- | -------------- 
[GET /api/v1/option/depth](./rest-api.md#depth-information-for-options) | Depth
[GET /api/v1/option/trades](./rest-api.md#recent-trades-for-options) | Recent trades
[GET /api/v1/option/historicalTrades](./rest-api.md#old-trade-lookup-for-options-market_data) | Historical trades
[GET /api/v1/option/klines](./rest-api.md#kline-data-lookup-for-options) | K-line
[GET /api/v1/option/avgPrice](./rest-api.md#look-up-current-average-price-for-options) | Average price now
[GET /api/v1/option/ticker/24hr](./rest-api.md#look-up-24hr-ticker-price-change-statistics-for-options) | Price change in 24 hours
[GET /api/v1/option/ticker/price](./rest-api.md#look-up-price-ticker-for-options) | Latest price
[GET /api/v1/option/ticker/bookTicker](./rest-api.md#look-up-symbol-order-book-ticker-for-options) | Optimal entry order
[POST /api/v1/option/order/test (HMAC SHA256)](./rest-api.md#test-placing-order-api-of-options-transactiontrade) | Test placing order
[POST /api/v1/option/order  (HMAC SHA256)](./rest-api.md#place-order-in-contract-transactiontrade) | Place order 
[GET /api/v1/option/order (HMAC SHA256)](./rest-api.md#check-orders-of-contract-transactionuser_data) | Check orders
[DELETE /api/v1/option/order  (HMAC SHA256)](./rest-api.md#cancel-order-for-options-transactiontrade) | Cancel order 
[GET /api/v1/option/openOrders  (HMAC SHA256)](./rest-api.md#check-entry-orders-of-options-transaction-of-this-account-user_data) | Check entry orders
[GET /api/v1/option/historyOrders  (HMAC SHA256)](./rest-api.md#historical-options-entry-order-of-the-account-user_data) | Historical entry orders 
[POST /api/v1/option/release  (HMAC SHA256)](./rest-api.md#users-short-optionstrade) | Users short options
[POST /api/v1/option/back  (HMAC SHA256)](./rest-api.md#users-call-optionstrade) | Users call options
[GET /api/v1/option/position  (HMAC SHA256)](./rest-api.md#options-positions-of-the-accountuser_data) | Options positions of the account
[GET /api/v1/option/info  (HMAC SHA256)](./rest-api.md#short--call-options-information-of-the-accountuser_data) | Short & call options info of the account
[GET /api/v1/option/record  (HMAC SHA256)](./rest-api.md#historical-records-of-short--call-options-of-the-account-user_data) | Short & call options records of the account 


### Contract Transaction API
API | Description
-------------- | -------------- 
[GET /api/v1/contract/depth](./rest-api.md#depth-information-for-futures) | Depth
[GET /api/v1/contract/trades](./rest-api.md#recent-trades-for-futures) | Recent trades
[GET /api/v1/contract/historicalTrades](./rest-api.md#old-trade-lookup-for-futures-market_data) | Historical trades
[GET /api/v1/contract/klines](./rest-api.md#kline-data-lookup-for-futures) | K-line
[GET /api/v1/contract/avgPrice](./rest-api.md#look-up-current-average-price-for-futures) | Average price now
[GET /api/v1/contract/ticker/24hr](./rest-api.md#look-up-24hr-ticker-price-change-statistics-for-futrues) | Price change in 24 hours
[GET /api/v1/contract/ticker/price](./rest-api.md#look-up-price-ticker-for-futures) | Latest price
[GET /api/v1/contract/ticker/bookTicker](./rest-api.md#look-up-symbol-order-book-ticker-for-futures) | Optimal entry order
[GET /api/v1/contract/ticker/indicesPrice](./rest-api.md#index-price-and-mark-price-for-futures) | Indices Price, marked price
[POST /api/v1/contract/order/test (HMAC SHA256)](./rest-api.md#test-placing-order-api-of-contract-transactiontrade) | Test placing order
[POST /api/v1/contract/order  (HMAC SHA256)](./rest-api.md#place-order-in-contract-transactiontrade) | Place order
[GET /api/v1/contract/order (HMAC SHA256)](./rest-api.md#check-orders-of-contract-transactionuser_data) | Check orders
[DELETE /api/v1/contract/order  (HMAC SHA256)](./rest-api.md#cancel-order-for-contract-transaction-trade) | Cancel order
[GET /api/v1/contract/openOrders  (HMAC SHA256)](./rest-api.md#check-entry-orders-of-contract-transaction-of-this-accountuser_data) | Check entry orders
[POST /api/v1/contract/liquidation  (HMAC SHA256)](./rest-api.md#close-positions-for-contracttrade) | Contract liquidation
[GET /api/v1/contract/liquidationOrder](./rest-api.md#check-orders-of-closed-positions-of-the-account-market_data-) | Check liquidation orders
[GET /api/v1/contract/position  (HMAC SHA256)](./rest-api.md#check-contract-positions-of-the-accountuser_data) | Check contract positions of the account
[GET /api/v1/contract/position/leverage  (HMAC SHA256)](./rest-api.md#adjust-contract-leverage-of-the-accountuser_data) | Adjust contract leverage of the account
[GET /api/v1/contract/historyOrders  (HMAC SHA256)](./rest-api.md#historical-contract-entry-order-of-the-account-user_data) | Historical entry orders of the account
[GET /api/v1/contract/bill  (HMAC SHA256)](./rest-api.md#contract-bill-of-the-accountuser_data) | Contract bill
[GET /api/v1/contract/historyRate](./rest-api.md#capital-fee-rate-of-contract) | Contract capital fee
[GET /api/v1/contract/protectionFund](./rest-api.md#contract-protection-fund) | Contract protection fund
[POST /api/v1/contract/transferMargin  (HMAC SHA256)](./rest-api.md#transfer-marginuser_data) | Transfer margin 
[GET /api/v1/contract/userHistoricalTrades (HMAC SHA256)](./rest-api.md#contract-bill-of-the-accountuser_data-1) | Contract bill of obtained account
[DELETE /api/v1/contract/batchOrder (HMAC SHA256)](./rest-api.md#cancel-all-orders) | cancel all orders


### WebSocket Common Data
Stream | Description
-------------- | -------------- 
[\<symbol\>@\<tradeType\>](./web-socket-streams.md#stream-name-symboltradetype) | Recent trades
[\<symbol\>@\<klineType\>_\<interval\>](./web-socket-streams.md#klinecandlestick-streams) | K-line
[\<symbol\>@\<miniTickerType\>](./web-socket-streams.md#stream-name-symbolminitickertype) | Simplified Ticker
[\<symbol\>@\<tradeType\>](./web-socket-streams.md#all-market-tickers-stream) | Complete Ticker
[\<symbol\>@\<depthType\>\<levels\>](./web-socket-streams.md#partial-book-depth-streams) | Depth information
[\<symbol\>@\<depthType\>](./web-socket-streams.md#stream-name-symboldepthtypelevels) | Depth information stream

### webSocket User Data
Stream | Description
-------------- | -------------- 
[accountSpotInfo](./web-socket-streams.md#account-updates) | Change of spot assets
[accountOptionInfo](./web-socket-streams.md#account-updates) | Change of options assets
[accountContractInfo](./web-socket-streams.md#account-updates) | Change of contract assets
[execSpotReport](./web-socket-streams.md#update-of-orders) | Update spot orders/ trades
[execOptionReport](./web-socket-streams.md#update-of-orders) | Update options orders/ trades
[execContractReport](./web-socket-streams.md#update-of-orders) | Update contract orders/ trades
[contractPositions](./web-socket-streams.md#contract-positions) | Positions of contract



