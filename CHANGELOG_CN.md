# 更新日志 

## 2019-12-23
### Rest API
[GET /api/v1/contract/batchOrder (HMAC SHA256)](./rest-api_CN.md#%E6%89%B9%E9%87%8F%E6%9F%A5%E8%AF%A2-user_data) |批量查询


## 2019-10-28
### websocket
* 优化websocket，让连接更稳定
* 推荐使用组合streams方式进行连接，每个组合streams可最多订阅10个stream


## 2019-10-22
### Rest API
* 合约下单增加新的 type 类型: `stopLimit`,`profitLimit`
* 用户预设止盈止损指令（`stopLimit`,`profitLimit`），提前设置 触发价格（`triggerPrice`）、委托价格(`price`)、委托数量(`quantity`)。当对应 “指标” 满足用户设置的 触发价（`triggerPrice`）格 条件时，止盈止损订单将被触发，系统将按照用户设置的 委托价格(`price`)，委托数量(`quantity`) 提交一笔限价委托订单。


#### 条件单的触发价格必须:
* 止盈

  * 多仓： 对未来价格看涨，设置触发价格高于最新价格（或标记价格、指数），可以设置卖出止盈指令；

  * 空仓： 对未来价格看跌，设置触发价格低于最新价格（或标记价格、指数），可以设置买入止盈指令；

* 止损

  * 多仓： 对未来价格看跌，设置触发价格低于最新价格（或标记价格、指数），可以设置卖出止损指令；

   * 空仓： 对未来价格看涨，设置触发价格高于最新价格（或标记价格、指数），可以设置买入止损指令；

* 当触发价格 = 最新价格（或标记价格、指数），或者无持仓时候，不可发起止盈止损委托指令；
## 2019-10-15
### 文档更新
* 增加文档更新的日志
* 增加常见问题的更新
## 2019-10-14
### Rest API
* 增加委托数量限制 每个账户每个合约最多 200 笔未执行普通交易委托数量
* 每个账户每个合约最多 10 笔止损或者止盈交易委托数量;
* 当发出超过这些上限的新交易委托时，该交易将被拒绝，并显示“has reach max order number 200或者10”
## 2019-10-11
### Rest API
* 增加批量撤单的接口，接口为 [DELETE /api/v1/contract/batchOrder (HMAC SHA256)](./rest-api_CN.md#%E6%89%B9%E9%87%8F%E6%92%A4%E5%8D%95user_data) |批量撤单
* 增加获取历史成交信息，接口为 [GET /api/v1/contract/userHistoricalTrades (HMAC SHA256)](./rest-api_CN.md#%E8%8E%B7%E5%8F%96%E5%8E%86%E5%8F%B2%E6%88%90%E4%BA%A4%E4%BF%A1%E6%81%AF-user_data) |获取历史成交信息 

## 2019-09-27
### Rest API
*  Headers `X-MBX-USED-WEIGHT-(intervalNum)(intervalLetter)` 会给你当前的请求权重 (intervalNum)(intervalLetter)频率的限制. 举个例子有分钟请求权重频率的设置, 在请求返回中含有  `X-MBX-USED-WEIGHT-1M `. 请求头header 中`X-MBX-USED-WEIGHT ` 仍然会返回并且已经使用的次数。
* Header 中 `X-MBX-ORDER-COUNT-(intervalNum)(intervalLetter)`在任何有效的订单上更新，并跟踪间隔的当前订单计数; 拒绝/不成功的订单不确保有`X-MBX-ORDER-COUNT-**` headers 在返回值中。
  - 例子. `X-MBX-ORDER-COUNT-1S` 是 "这秒有多少单" and` X-MBX-ORDER-COUNT-1D` 是 ”这一天有多少单“
* 消息头header上`Retry-After`会给出具体需要等待的时间,以秒为单位，直到解除封禁。
