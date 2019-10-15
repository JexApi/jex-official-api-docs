# 更新日志 (2019-10-15)
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
* 增加批量撤单的接口，接口为 [DELETE /api/v1/contract/batchOrder (HMAC SHA256)](./rest-api_CN.md#%E6%89%B9%E9%87%8F%E6%92%A4%E5%8D%95) |批量撤单
* 增加获取历史成交信息，接口为 [GET /api/v1/contract/userHistoricalTrades (HMAC SHA256)](./rest-api_CN.md#%E8%8E%B7%E5%8F%96%E5%8E%86%E5%8F%B2%E6%88%90%E4%BA%A4%E4%BF%A1%E6%81%AF-user_data) |获取历史成交信息 

## 2019-09-27
### Rest API
* * Headers `X-MBX-USED-WEIGHT-(intervalNum)(intervalLetter)` 会给你当前的请求权重 (intervalNum)(intervalLetter)频率的限制. 举个例子有分钟请求权重频率的设置, 在请求返回中含有  `X-MBX-USED-WEIGHT-1M `. 请求头header 中`X-MBX-USED-WEIGHT ` 仍然会返回并且已经使用的次数。
* Header 中 `X-MBX-ORDER-COUNT-(intervalNum)(intervalLetter)`在任何有效的订单上更新，并跟踪间隔的当前订单计数; 拒绝/不成功的订单不确保有`X-MBX-ORDER-COUNT-**` headers 在返回值中。
  - 例子. `X-MBX-ORDER-COUNT-1S` 是 "这秒有多少单" and` X-MBX-ORDER-COUNT-1D` 是 ”这一天有多少单“
* 消息头header上`Retry-After`会给出具体需要等待的时间,以秒为单位，直到解除封禁。
