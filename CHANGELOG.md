# CHANGELOG for jex's API  (2019-10-15)
## 2019-10-15
### Document update
* Increase the log of document updates
* Add updates to frequently asked questions
## 2019-10-14
### Rest API
* Maximum 200 open orders per contract per account;
* Maximum 10 stop profit or stop orders per contract per account;
* When placing a new order that causes these caps to be exceeded, it will be rejected with the message “has reach max order number [200|10]
## 2019-10-11
### Rest API
* Add interface cancel all orders [DELETE /api/v1/contract/batchOrder (HMAC SHA256)](./rest-api.md#cancel-all-orders) | cancel all orders
* Add interface Contract bill of obtained account [GET /api/v1/contract/userHistoricalTrades (HMAC SHA256)](./rest-api.md#contract-bill-of-the-accountuser_data-1) | Contract bill of obtained account

## 2019-09-27
### Rest API
* New `intervalLetter` values for headers:
    * SECOND => S
    * MINUTE => M
    * HOUR => H
    * DAY => D
* New Headers `X-MBX-USED-WEIGHT-(intervalNum)(intervalLetter)` will give your current used request weight for the (intervalNum)(intervalLetter) rate limiter. For example, if there is a one minute request rate weight limiter set, you will get a `X-MBX-USED-WEIGHT-1M` header in the response. The legacy header `X-MBX-USED-WEIGHT` will still be returned and will represent the current used weight for the one minute request rate weight limit.
* New Header `X-MBX-ORDER-COUNT-(intervalNum)(intervalLetter)`that is updated on any valid order placement and tracks your current order count for the interval; rejected/unsuccessful orders are not guaranteed to have `X-MBX-ORDER-COUNT-**` headers in the response.
    * Eg. `X-MBX-ORDER-COUNT-1S` for "orders per 1 second" and `X-MBX-ORDER-COUNT-1D` for orders per "one day"
* GET api/v1/depth now supports `limit` 5000 and 10000; weights are 50 and 100 respectively.
* GET api/v1/exchangeInfo has a new parameter `ocoAllowed`.