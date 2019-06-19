## 接口简介
### 通用接口
接口 | 说明
-------------- | -------------- 
[GET /api/v1/ping](https://github.com/openjex/jex-official-api-docs/blob/master/rest-api.md#%E6%B5%8B%E8%AF%95%E6%9C%8D%E5%8A%A1%E5%99%A8%E8%BF%9E%E9%80%9A%E6%80%A7-ping) | 测试服务器连通性 PING  
[GET /api/v1/time](https://github.com/JexApi/jex-official-api-docs/blob/master/rest-api.md#%E8%8E%B7%E5%8F%96%E6%9C%8D%E5%8A%A1%E5%99%A8%E6%97%B6%E9%97%B4) | 获取服务器时间
[GET /api/v1/exchangeInfo](https://github.com/JexApi/jex-official-api-docs/blob/master/rest-api.md#%E4%BA%A4%E6%98%93%E5%AF%B9%E4%BF%A1%E6%81%AF) | 获取此时的限制信息和交易对信息 
[GET /api/v1/optionInfo](https://github.com/JexApi/jex-official-api-docs/blob/master/rest-api.md#%E6%9C%9F%E6%9D%83%E4%BA%A4%E6%98%93%E5%AF%B9%E4%BF%A1%E6%81%AF) | 获取此时期权的交易对信息
[GET /api/v1/contractInfo](https://github.com/JexApi/jex-official-api-docs/blob/master/rest-api.md#%E5%90%88%E7%BA%A6%E4%BA%A4%E6%98%93%E5%AF%B9%E4%BF%A1%E6%81%AF) | 获取此时合约的交易对信息
[GET /api/v1/account (HMAC SHA256)](https://github.com/JexApi/jex-official-api-docs/blob/master/rest-api.md#%E8%B4%A6%E6%88%B7%E4%BF%A1%E6%81%AF-user_data) | 账户资产信息 
[POST /api/v1/userDataStream](https://github.com/JexApi/jex-official-api-docs/blob/master/rest-api.md#%E6%96%B0%E5%BB%BA%E7%94%A8%E6%88%B7%E6%95%B0%E6%8D%AE%E6%B5%81-user_stream) | 新建用户数据流
[PUT /api/v1/userDataStream](https://github.com/JexApi/jex-official-api-docs/blob/master/rest-api.md#keepalive-user_stream) | 延长用户数据流有效期
[DELETE /api/v1/userDataStream](https://github.com/JexApi/jex-official-api-docs/blob/master/rest-api.md#%E5%85%B3%E9%97%AD%E7%94%A8%E6%88%B7%E6%95%B0%E6%8D%AE%E6%B5%81-user_stream) | 关闭用户数据流
[GET /wapi/v1/assetDetail](https://github.com/JexApi/jex-official-api-docs/blob/master/rest-api.md#%E7%94%A8%E6%88%B7%E8%B5%84%E4%BA%A7%E8%AF%A6%E6%83%85-user_data) | 用户资产详情
[GET /wapi/v1/depositAddress](https://github.com/JexApi/jex-official-api-docs/blob/master/rest-api.md#%E7%94%A8%E6%88%B7%E5%85%85%E5%80%BC%E5%9C%B0%E5%9D%80-user_data) | 用户充值地址
[GET /wapi/v1/depositHistory](https://github.com/JexApi/jex-official-api-docs/blob/master/rest-api.md#%E7%94%A8%E6%88%B7%E5%85%85%E5%80%BC%E5%8E%86%E5%8F%B2%E8%AE%B0%E5%BD%95-user_data) | 充值历史记录 
[GET /wapi/v1/tradeFee](https://github.com/JexApi/jex-official-api-docs/blob/master/rest-api.md#%E7%94%A8%E6%88%B7%E4%BA%A4%E6%98%93%E6%89%8B%E7%BB%AD%E8%B4%B9-user_data) | 交易手续费 
[GET /wapi/v1/withdraw](https://github.com/JexApi/jex-official-api-docs/blob/master/rest-api.md#%E7%94%A8%E6%88%B7%E6%8F%90%E7%8E%B0-user_data) | 提现 
[GET /wapi/v1/withdrawHistory](https://github.com/JexApi/jex-official-api-docs/blob/master/rest-api.md#%E7%94%A8%E6%88%B7%E6%8F%90%E7%8E%B0%E5%8E%86%E5%8F%B2%E8%AE%B0%E5%BD%95-user_data) | 提现历史记录 

### 币币接口
接口 | 说明
-------------- | -------------- 
[GET /api/v1/spot/depth](https://github.com/JexApi/jex-official-api-docs/blob/master/rest-api.md#%E5%B8%81%E5%B8%81%E6%B7%B1%E5%BA%A6%E4%BF%A1%E6%81%AF) | 币币深度信息
[GET /api/v1/spot/trades](https://github.com/JexApi/jex-official-api-docs/blob/master/rest-api.md#%E5%B8%81%E5%B8%81%E8%BF%91%E6%9C%9F%E6%88%90%E4%BA%A4) | 币币近期成交
[GET /api/v1/spot/historicalTrades](https://github.com/JexApi/jex-official-api-docs/blob/master/rest-api.md#%E6%9F%A5%E8%AF%A2%E5%B8%81%E5%B8%81%E5%8E%86%E5%8F%B2%E6%88%90%E4%BA%A4market_data) | 
[GET /api/v1/spot/klines](https://github.com/JexApi/jex-official-api-docs/blob/master/rest-api.md#%E5%B8%81%E5%B8%81k%E7%BA%BF%E6%95%B0%E6%8D%AE) | 币币K线数据
[GET /api/v1/spot/avgPrice](https://github.com/JexApi/jex-official-api-docs/blob/master/rest-api.md#%E5%B8%81%E5%B8%81%E5%BD%93%E5%89%8D%E5%B9%B3%E5%9D%87%E4%BB%B7%E6%A0%BC) | 币币当前平均价格
[GET /api/v1/spot/ticker/24hr](https://github.com/JexApi/jex-official-api-docs/blob/master/rest-api.md#%E5%B8%81%E5%B8%8124hr%E4%BB%B7%E6%A0%BC%E5%8F%98%E5%8A%A8%E6%83%85%E5%86%B5) | 币币24hr价格变动情况
[GET /api/v1/spot/ticker/price](https://github.com/JexApi/jex-official-api-docs/blob/master/rest-api.md#%E5%B8%81%E5%B8%81%E6%9C%80%E6%96%B0%E4%BB%B7%E6%A0%BC%E6%8E%A5%E5%8F%A3) | 币币最新价格
[GET /api/v1/spot/ticker/bookTicker](https://github.com/JexApi/jex-official-api-docs/blob/master/rest-api.md#%E5%B8%81%E5%B8%81%E6%9C%80%E4%BC%98%E6%8C%82%E5%8D%95%E6%8E%A5%E5%8F%A3) | 币币最优挂单
[POST /api/v1/spot/order/test (HMAC SHA256)](https://github.com/JexApi/jex-official-api-docs/blob/master/rest-api.md#%E5%B8%81%E5%B8%81%E6%B5%8B%E8%AF%95%E4%B8%8B%E5%8D%95%E6%8E%A5%E5%8F%A3-trade) | 币币测试下单
[POST /api/v1/spot/order  (HMAC SHA256)](https://github.com/JexApi/jex-official-api-docs/blob/master/rest-api.md#%E5%B8%81%E5%B8%81%E4%B8%8B%E5%8D%95--trade) | 币币下单 
[GET /api/v1/spot/order (HMAC SHA256)](https://github.com/JexApi/jex-official-api-docs/blob/master/rest-api.md#%E5%B8%81%E5%B8%81%E6%9F%A5%E8%AF%A2%E8%AE%A2%E5%8D%95-user_data) | 币币查询订单
[DELETE /api/v1/spot/order  (HMAC SHA256)](https://github.com/JexApi/jex-official-api-docs/blob/master/rest-api.md#%E5%B8%81%E5%B8%81%E6%92%A4%E9%94%80%E8%AE%A2%E5%8D%95-trade) | 币币撤销订单 
[GET /api/v1/spot/openOrders  (HMAC SHA256)](https://github.com/JexApi/jex-official-api-docs/blob/master/rest-api.md#%E6%9F%A5%E7%9C%8B%E8%B4%A6%E6%88%B7%E5%BD%93%E5%89%8D%E5%B8%81%E5%B8%81%E6%8C%82%E5%8D%95-user_data) | 查看账户当前币币挂单 
[GET /api/v1/spot/historyOrders  (HMAC SHA256)](https://github.com/JexApi/jex-official-api-docs/blob/master/rest-api.md#%E8%B4%A6%E6%88%B7%E5%B8%81%E5%B8%81%E5%8E%86%E5%8F%B2%E5%A7%94%E6%89%98-user_data) | 币币历史委托
 


### 期权接口
接口 | 说明
-------------- | -------------- 
[GET /api/v1/option/depth](https://github.com/JexApi/jex-official-api-docs/blob/master/rest-api.md#%E6%9C%9F%E6%9D%83%E6%B7%B1%E5%BA%A6%E4%BF%A1%E6%81%AF) | 期权深度信息
[GET /api/v1/option/trades](https://github.com/JexApi/jex-official-api-docs/blob/master/rest-api.md#%E6%9C%9F%E6%9D%83%E8%BF%91%E6%9C%9F%E6%88%90%E4%BA%A4) | 期权近期成交
[GET /api/v1/option/historicalTrades](https://github.com/JexApi/jex-official-api-docs/blob/master/rest-api.md#%E6%9F%A5%E8%AF%A2%E6%9C%9F%E6%9D%83%E5%8E%86%E5%8F%B2%E6%88%90%E4%BA%A4market_data) | 查询期权历史成交
[GET /api/v1/option/klines](https://github.com/JexApi/jex-official-api-docs/blob/master/rest-api.md#%E6%9F%A5%E8%AF%A2%E6%9C%9F%E6%9D%83k%E7%BA%BF%E6%95%B0%E6%8D%AE) | 查询期权K线数据
[GET /api/v1/option/avgPrice](https://github.com/JexApi/jex-official-api-docs/blob/master/rest-api.md#%E6%9F%A5%E8%AF%A2%E6%9C%9F%E6%9D%83%E5%BD%93%E5%89%8D%E5%B9%B3%E5%9D%87%E4%BB%B7%E6%A0%BC) | 查询期权当前平均价格
[GET /api/v1/option/ticker/24hr](https://github.com/JexApi/jex-official-api-docs/blob/master/rest-api.md#%E6%9F%A5%E8%AF%A2%E6%9C%9F%E6%9D%8324hr%E4%BB%B7%E6%A0%BC%E5%8F%98%E5%8A%A8%E6%83%85%E5%86%B5) | 查询期权24hr价格变动情况
[GET /api/v1/option/ticker/price](https://github.com/JexApi/jex-official-api-docs/blob/master/rest-api.md#%E6%9F%A5%E8%AF%A2%E6%9C%9F%E6%9D%83%E6%9C%80%E6%96%B0%E4%BB%B7%E6%A0%BC%E6%8E%A5%E5%8F%A3) | 查询期权最新价格接口
[GET /api/v1/option/ticker/bookTicker](https://github.com/JexApi/jex-official-api-docs/blob/master/rest-api.md#%E6%9F%A5%E8%AF%A2%E6%9C%9F%E6%9D%83%E6%9C%80%E4%BC%98%E6%8C%82%E5%8D%95%E6%8E%A5%E5%8F%A3) | 查询期权最优挂单
[POST /api/v1/option/order/test (HMAC SHA256)](https://github.com/JexApi/jex-official-api-docs/blob/master/rest-api.md#%E6%9C%9F%E6%9D%83%E6%B5%8B%E8%AF%95%E4%B8%8B%E5%8D%95%E6%8E%A5%E5%8F%A3-trade) | 期权测试下单
[POST /api/v1/option/order  (HMAC SHA256)](https://github.com/JexApi/jex-official-api-docs/blob/master/rest-api.md#%E6%9C%9F%E6%9D%83%E4%B8%8B%E5%8D%95--trade) | 期权下单 
[GET /api/v1/option/order (HMAC SHA256)](https://github.com/JexApi/jex-official-api-docs/blob/master/rest-api.md#%E6%9C%9F%E6%9D%83%E6%9F%A5%E8%AF%A2%E8%AE%A2%E5%8D%95-user_data) | 期权查询订单
[DELETE /api/v1/option/order  (HMAC SHA256)](https://github.com/JexApi/jex-official-api-docs/blob/master/rest-api.md#%E6%9C%9F%E6%9D%83%E6%92%A4%E9%94%80%E8%AE%A2%E5%8D%95-trade) | 期权撤销订单 
[GET /api/v1/option/openOrders  (HMAC SHA256)](https://github.com/JexApi/jex-official-api-docs/blob/master/rest-api.md#%E6%9F%A5%E7%9C%8B%E8%B4%A6%E6%88%B7%E5%BD%93%E5%89%8D%E6%9C%9F%E6%9D%83%E6%8C%82%E5%8D%95-user_data) | 查看账户当前期权挂单
[GET /api/v1/option/historyOrders  (HMAC SHA256)](https://github.com/JexApi/jex-official-api-docs/blob/master/rest-api.md#%E8%B4%A6%E6%88%B7%E6%9C%9F%E6%9D%83%E5%8E%86%E5%8F%B2%E5%A7%94%E6%89%98-user_data) | 账户期权历史委托 
[POST /api/v1/option/release  (HMAC SHA256)](https://github.com/JexApi/jex-official-api-docs/blob/master/rest-api.md#%E7%94%A8%E6%88%B7%E5%8F%91%E8%A1%8C%E6%9C%9F%E6%9D%83-trade) | 用户发行期权
[POST /api/v1/option/back  (HMAC SHA256)](https://github.com/JexApi/jex-official-api-docs/blob/master/rest-api.md#%E7%94%A8%E6%88%B7%E8%B5%8E%E5%9B%9E%E6%9C%9F%E6%9D%83-trade) | 用户赎回期权
[GET /api/v1/option/position  (HMAC SHA256)](https://github.com/JexApi/jex-official-api-docs/blob/master/rest-api.md#%E8%B4%A6%E6%88%B7%E6%9C%9F%E6%9D%83%E6%8C%81%E4%BB%93-user_data) | 账户期权持仓
[GET /api/v1/option/info  (HMAC SHA256)](https://github.com/JexApi/jex-official-api-docs/blob/master/rest-api.md#%E8%B4%A6%E6%88%B7%E6%9C%9F%E6%9D%83%E5%8F%91%E8%A1%8C%E8%B5%8E%E5%9B%9E%E4%BF%A1%E6%81%AF-user_data) | 账户期权发行赎回信息
[GET /api/v1/option/record  (HMAC SHA256)](https://github.com/JexApi/jex-official-api-docs/blob/master/rest-api.md#%E8%B4%A6%E6%88%B7%E6%9C%9F%E6%9D%83%E5%8F%91%E8%A1%8C%E8%B5%8E%E5%9B%9E%E8%AE%B0%E5%BD%95-user_data) | 账户期权发行赎回记录 


### 合约接口
接口 | 说明
-------------- | -------------- 
[GET /api/v1/contract/depth](https://github.com/JexApi/jex-official-api-docs/blob/master/rest-api.md#%E5%90%88%E7%BA%A6%E6%B7%B1%E5%BA%A6%E4%BF%A1%E6%81%AF) | 合约深度信息
[GET /api/v1/contract/trades](https://github.com/JexApi/jex-official-api-docs/blob/master/rest-api.md#%E5%90%88%E7%BA%A6%E8%BF%91%E6%9C%9F%E6%88%90%E4%BA%A4) | 合约近期成交
[GET /api/v1/contract/historicalTrades](https://github.com/JexApi/jex-official-api-docs/blob/master/rest-api.md#%E6%9F%A5%E8%AF%A2%E5%90%88%E7%BA%A6%E5%8E%86%E5%8F%B2%E6%88%90%E4%BA%A4market_data) | 查询合约历史成交
[GET /api/v1/contract/klines](https://github.com/JexApi/jex-official-api-docs/blob/master/rest-api.md#%E6%9F%A5%E8%AF%A2%E5%90%88%E7%BA%A6k%E7%BA%BF%E6%95%B0%E6%8D%AE) | 查询合约K线数据
[GET /api/v1/contract/avgPrice](https://github.com/JexApi/jex-official-api-docs/blob/master/rest-api.md#%E6%9F%A5%E8%AF%A2%E5%90%88%E7%BA%A6%E5%BD%93%E5%89%8D%E5%B9%B3%E5%9D%87%E4%BB%B7%E6%A0%BC) | 查询合约当前平均价格
[GET /api/v1/contract/ticker/24hr](https://github.com/JexApi/jex-official-api-docs/blob/master/rest-api.md#%E6%9F%A5%E8%AF%A2%E5%90%88%E7%BA%A624hr%E4%BB%B7%E6%A0%BC%E5%8F%98%E5%8A%A8%E6%83%85%E5%86%B5) | 查询合约24hr价格变动情况
[GET /api/v1/contract/ticker/price](https://github.com/JexApi/jex-official-api-docs/blob/master/rest-api.md#%E6%9F%A5%E8%AF%A2%E5%90%88%E7%BA%A6%E6%9C%80%E6%96%B0%E4%BB%B7%E6%A0%BC%E6%8E%A5%E5%8F%A3) | 查询合约最新价格
[GET /api/v1/contract/ticker/bookTicker](https://github.com/JexApi/jex-official-api-docs/blob/master/rest-api.md#%E6%9F%A5%E8%AF%A2%E5%90%88%E7%BA%A6%E6%9C%80%E4%BC%98%E6%8C%82%E5%8D%95%E6%8E%A5%E5%8F%A3) | 查询合约最优挂单
[GET /api/v1/contract/ticker/indicesPrice](https://github.com/JexApi/jex-official-api-docs/blob/master/rest-api.md#%E5%90%88%E7%BA%A6%E7%9A%84%E6%8C%87%E6%95%B0%E4%BB%B7%E6%A0%BC%E6%A0%87%E8%AE%B0%E4%BB%B7%E6%A0%BC) | 合约的指数价格，标记价格
[POST /api/v1/contract/order/test (HMAC SHA256)](https://github.com/JexApi/jex-official-api-docs/blob/master/rest-api.md#%E5%90%88%E7%BA%A6%E6%B5%8B%E8%AF%95%E4%B8%8B%E5%8D%95%E6%8E%A5%E5%8F%A3-trade) | 合约测试下单
[POST /api/v1/contract/order  (HMAC SHA256)](https://github.com/JexApi/jex-official-api-docs/blob/master/rest-api.md#%E5%90%88%E7%BA%A6%E4%B8%8B%E5%8D%95--trade) | 合约下单
[GET /api/v1/contract/order (HMAC SHA256)](https://github.com/JexApi/jex-official-api-docs/blob/master/rest-api.md#%E5%90%88%E7%BA%A6%E6%9F%A5%E8%AF%A2%E8%AE%A2%E5%8D%95-user_data) | 合约查询订单
[DELETE /api/v1/contract/order  (HMAC SHA256)](https://github.com/JexApi/jex-official-api-docs/blob/master/rest-api.md#%E5%90%88%E7%BA%A6%E6%92%A4%E9%94%80%E8%AE%A2%E5%8D%95-trade) | 合约撤销订单
[GET /api/v1/contract/openOrders  (HMAC SHA256)](https://github.com/JexApi/jex-official-api-docs/blob/master/rest-api.md#%E6%9F%A5%E7%9C%8B%E8%B4%A6%E6%88%B7%E5%BD%93%E5%89%8D%E5%90%88%E7%BA%A6%E6%8C%82%E5%8D%95-user_data) | 查看账户当前合约挂单
[POST /api/v1/contract/liquidation  (HMAC SHA256)](https://github.com/JexApi/jex-official-api-docs/blob/master/rest-api.md#%E5%90%88%E7%BA%A6%E5%B9%B3%E4%BB%93-trade) | 合约平仓
[GET /api/v1/contract/liquidationOrder](https://github.com/JexApi/jex-official-api-docs/blob/master/rest-api.md#%E6%9F%A5%E7%9C%8B%E8%B4%A6%E6%88%B7%E5%90%88%E7%BA%A6%E5%B9%B3%E4%BB%93%E5%8D%95-market_data) | 查看账户合约平仓单
[GET /api/v1/contract/position  (HMAC SHA256)](https://github.com/JexApi/jex-official-api-docs/blob/master/rest-api.md#%E6%9F%A5%E7%9C%8B%E8%B4%A6%E6%88%B7%E5%90%88%E7%BA%A6%E4%BB%93%E4%BD%8D-user_data) | 查看账户合约仓位
[GET /api/v1/contract/position/leverage  (HMAC SHA256)](https://github.com/JexApi/jex-official-api-docs/blob/master/rest-api.md#%E8%B0%83%E6%95%B4%E8%B4%A6%E6%88%B7%E5%90%88%E7%BA%A6%E6%9D%A0%E6%9D%86-user_data) | 调整账户合约杠杆
[GET /api/v1/contract/historyOrders  (HMAC SHA256)](https://github.com/JexApi/jex-official-api-docs/blob/master/rest-api.md#%E8%B4%A6%E6%88%B7%E5%90%88%E7%BA%A6%E5%8E%86%E5%8F%B2%E5%A7%94%E6%89%98-user_data) | 账户合约历史委托
[GET /api/v1/contract/bill  (HMAC SHA256)](https://github.com/JexApi/jex-official-api-docs/blob/master/rest-api.md#%E8%B4%A6%E6%88%B7%E5%90%88%E7%BA%A6%E8%B4%A6%E5%8D%95-user_data) | 合约账单
[GET /api/v1/contract/historyRate](https://github.com/JexApi/jex-official-api-docs/blob/master/rest-api.md#%E5%90%88%E7%BA%A6%E8%B5%84%E9%87%91%E8%B4%B9%E7%8E%87) | 合约资金费率
[GET /api/v1/contract/protectionFund](https://github.com/JexApi/jex-official-api-docs/blob/master/rest-api.md#%E5%90%88%E7%BA%A6%E4%BF%9D%E6%8A%A4%E5%9F%BA%E9%87%91) | 合约保护基金
[POST /api/v1/contract/transferMargin  (HMAC SHA256)](https://github.com/JexApi/jex-official-api-docs/blob/master/rest-api.md#%E8%BD%AC%E7%A7%BB%E4%BF%9D%E8%AF%81%E9%87%91-user_data) | 转移保证金 
[POST /api/v1/contract/turnoutMargin  (HMAC SHA256)](https://github.com/JexApi/jex-official-api-docs/blob/master/rest-api.md#%E8%BD%AC%E5%87%BA%E4%BF%9D%E8%AF%81%E9%87%91-user_data) | 转出保证金