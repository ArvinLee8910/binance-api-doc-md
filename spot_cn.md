---
title: Binance API Documentation
language_tabs: # must be one of https://git.io/vQNgJ
  - shell
  #- javascript
  #- jason

toc_footers:
  - <a href='https://www.binance.com/'>Welcome to Binance</a>

includes:

search: true
---
#<center>敬请期待</center>

2019-09-03

Faster order book data with 100ms updates: <symbol>@depth@100ms and <symbol>@depth#@100ms
Added "Update Speed:" to Websocket Market Streams
Removed deprecated v1 endpoints as per previous announcement:
GET api/v1/order
GET api/v1/openOrders
POST api/v1/order
DELETE api/v1/order
GET api/v1/allOrders
GET api/v1/account
GET api/v1/myTrades
2019-08-16

GET api/v1/depth limit of 10000 has been temporarily removed

In Q4 2017, the following endpoints were deprecated and removed from the API documentation. They have been permanently removed from the API as of this version. We apologize for the omission from the original changelog:

GET api/v1/order
GET api/v1/openOrders
POST api/v1/order
DELETE api/v1/order
GET api/v1/allOrders
GET api/v1/account
GET api/v1/myTrades
Streams, endpoints, parameters, payloads, etc. described in the documents in this repository are considered official and supported. The use of any other streams, endpoints, parameters, or payloads, etc. is not supported; use them at your own risk and with no guarantees.

2019-09-15

Rest API

New order type: OCO ("One Cancels the Other")

An OCO has 2 orders: (also known as legs in financial terms)
STOP_LOSS or STOP_LOSS_LIMIT leg
LIMIT_MAKER leg
Price Restrictions:
SELL Orders : Limit Price > Last Price > Stop Price
BUY Orders : Limit Price < Last Price < Stop Price
As stated, the prices must "straddle" the last traded price on the symbol. EX: If the last price is 10:
A SELL OCO must have the limit price greater than 10, and the stop price less than 10.
A BUY OCO must have a limit price less than 10, and the stop price greater than 10.
Quantity Restrictions:
Both legs must have the same quantity.
ICEBERG quantities however, do not have to be the same.
Execution Order:
If the LIMIT_MAKER is touched, the limit maker leg will be executed first BEFORE cancelling the Stop Loss Leg.
if the Market Price moves such that the STOP_LOSS or STOP_LOSS_LIMIT will trigger, the Limit Maker leg will be cancelled BEFORE executing the STOP_LOSS Leg.
Cancelling an OCO
Cancelling either order leg will cancel the entire OCO.
The entire OCO can be canceled via the orderListId or the listClientOrderId.
New Enums for OCO:
ListStatusType
RESPONSE - used when ListStatus is responding to a failed action. (either order list placement or cancellation)
EXEC_STARTED - used when an order list has been placed or there is an update to a list's status.
ALL_DONE - used when an order list has finished executing and is no longer active.
ListOrderStatus
EXECUTING - used when an order list has been placed or there is an update to a list's status.
ALL_DONE - used when an order list has finished executing and is no longer active.
REJECT - used when ListStatus is responding to a failed action. (either order list placement or cancellation)
ContingencyType
OCO - specifies the type of order list.
New Endpoints:
POST api/v3/order/oco
DELETE api/v3/orderList
GET api/v3/orderList
recvWindow cannot exceed 60000.

New intervalLetter values for headers:

SECOND => S
MINUTE => M
HOUR => H
DAY => D
New Headers X-MBX-USED-WEIGHT-(intervalNum)(intervalLetter) will give your current used request weight for the (intervalNum)(intervalLetter) rate limiter. For example, if there is a one minute request rate weight limiter set, you will get a X-MBX-USED-WEIGHT-1M header in the response. The legacy header X-MBX-USED-WEIGHT will still be returned and will represent the current used weight for the one minute request rate weight limit.

New Header X-MBX-ORDER-COUNT-(intervalNum)(intervalLetter)that is updated on any valid order placement and tracks your current order count for the interval; rejected/unsuccessful orders are not guaranteed to have X-MBX-ORDER-COUNT-** headers in the response.

Eg. X-MBX-ORDER-COUNT-1S for "orders per 1 second" and X-MBX-ORDER-COUNT-1D for orders per "one day"
GET api/v1/depth now supports limit 5000 and 10000; weights are 50 and 100 respectively.

GET api/v1/exchangeInfo has a new parameter ocoAllowed.

USER DATA STREAM

executionReport event now contains "g" which has the orderListId; it will be set to -1 for non-OCO orders.
New Event Type listStatus; listStatus is sent on an update to any OCO order.
New Event Type outboundAccountPosition; outboundAccountPosition is sent any time an account's balance changes and contains the assets that could have changed by the event that generated the balance change (a deposit, withdrawal, trade, order placement, or cancelation).
NEW ERRORS

-1131 BAD_RECV_WINDOW
recvWindow must be less than 60000
-1099 Not found, authenticated, or authorized
This replaces error code -1999
NEW -2011 ERRORS

OCO_BAD_ORDER_PARAMS
A parameter for one of the orders is incorrect.
OCO_BAD_PRICES
The relationship of the prices for the orders is not correct.
UNSUPPORTED_ORD_OCO
OCO orders are not supported for this symbol.
2019-03-12

Rest API

X-MBX-USED-WEIGHT header added to Rest API responses.
Retry-After header added to Rest API 418 and 429 responses.
When canceling the Rest API can now return errorCode -1013 OR -2011 if the symbol's status isn't TRADING.
api/v1/depth no longer has the ignored and empty [].
api/v3/myTrades now returns quoteQty; the price * qty of for the trade.
Websocket streams

<symbol>@depth and <symbol>@depthX streams no longer have the ignored and empty [].
System improvements

Matching Engine stability/reliability improvements.
Rest API performance improvements.
2018-11-13

Rest API

Can now cancel orders through the Rest API during a trading ban.
New filters: PERCENT_PRICE, MARKET_LOT_SIZE, MAX_NUM_ICEBERG_ORDERS.
Added RAW_REQUST rate limit. Limits based on the number of requests over X minutes regardless of weight.
/api/v3/ticker/price increased to weight of 2 for a no symbol query.
/api/v3/ticker/bookTicker increased weight of 2 for a no symbol query.
DELETE /api/v3/order will now return an execution report of the final state of the order.
MIN_NOTIONAL filter has two new parameters: applyToMarket (whether or not the filter is applied to MARKET orders) and avgPriceMins (the number of minutes over which the price averaged for the notional estimation).
intervalNum added to /api/v1/exchangeInfo limits. intervalNum describes the amount of the interval. For example: intervalNum 5, with interval minute, means "every 5 minutes".
Explanation for the average price calculation:

(qty * price) of all trades / numTrades of the trades over previous 5 minutes.

If there is no trade in the last 5 minutes, it takes the first trade that happened outside of the 5min window. For example if the last trade was 20 minutes ago, that trade's price is the 5 min average.

If there is no trade on the symbol, there is no average price and market orders cannot be placed. On a new symbol with applyToMarket enabled on the MIN_NOTIONAL filter, market orders cannot be placed until there is at least 1 trade.

The current average price can be checked here: https://api.binance.com/api/v3/avgPrice?symbol=<symbol> For example: https://api.binance.com/api/v3/avgPrice?symbol=BNBUSDT

User data stream

Last quote asset transacted quantity (as variable Y) added to execution reports. Represents the lastPrice * lastQty (L * l).
2018-07-18

Rest API

New filter: ICEBERG_PARTS
POST api/v3/order new defaults for newOrderRespType. ACK, RESULT, or FULL; MARKET and LIMIT order types default to FULL, all other orders default to ACK.
POST api/v3/order RESULT and FULL responses now have cummulativeQuoteQty
GET api/v3/openOrders with no symbol weight reduced to 40.
GET api/v3/ticker/24hr with no symbol weight reduced to 40.
Max amount of trades from GET /api/v1/trades increased to 1000.
Max amount of trades from GET /api/v1/historicalTrades increased to 1000.
Max amount of aggregate trades from GET /api/v1/aggTrades increased to 1000.
Max amount of aggregate trades from GET /api/v1/klines increased to 1000.
Rest API Order lookups now return updateTime which represents the last time the order was updated; time is the order creation time.
Order lookup endpoints will now return cummulativeQuoteQty. If cummulativeQuoteQty is < 0, it means the data isn't available for this order at this time.
REQUESTS rate limit type changed to REQUEST_WEIGHT. This limit was always logically request weight and the previous name for it caused confusion.
User data stream

cummulativeQuoteQty field added to order responses and execution reports (as variable Z). Represents the cummulative amount of the quote that has been spent (with a BUY order) or received (with a SELL order). Historical orders will have a value < 0 in this field indicating the data is not available at this time. cummulativeQuoteQty divided by cummulativeQty will give the average price for an order.
O (order creation time) added to execution reports
2018-01-23 * GET /api/v1/historicalTrades weight decreased to 5 * GET /api/v1/aggTrades weight decreased to 1 * GET /api/v1/klines weight decreased to 1 * GET /api/v1/ticker/24hr all symbols weight decreased to number of trading symbols / 2 * GET /api/v3/allOrders weight decreased to 5 * GET /api/v3/myTrades weight decreased to 5 * GET /api/v3/account weight decreased to 5 * GET /api/v1/depth limit=500 weight decreased to 5 * GET /api/v1/depth limit=1000 weight decreased to 10 * -1003 error message updated to direct users to websockets

2018-01-20 * GET /api/v1/ticker/24hr single symbol weight decreased to 1 * GET /api/v3/openOrders all symbols weight decreased to number of trading symbols / 2 * GET /api/v3/allOrders weight decreased to 15 * GET /api/v3/myTrades weight decreased to 15 * GET /api/v3/order weight decreased to 1 * myTrades will now return both sides of a self-trade/wash-trade

2018-01-14 * GET /api/v1/aggTrades weight changed to 2 * GET /api/v1/klines weight changed to 2 * GET /api/v3/order weight changed to 2 * GET /api/v3/allOrders weight changed to 20 * GET /api/v3/account weight changed to 20 * GET /api/v3/myTrades weight changed to 20 * GET /api/v3/historicalTrades weight changed to 20

基本信息
本篇列出REST接口的baseurl https://api.binance.com
所有接口的响应都是JSON格式
响应中如有数组，数组元素以时间升序排列，越早的数据越提前。
所有时间、时间戳均为UNIX时间，单位为毫秒
HTTP 4XX 错误码用于指示错误的请求内容、行为、格式。
HTTP 429 错误码表示警告访问频次超限，即将被封IP
HTTP 418 表示收到429后继续访问，于是被封了。
HTTP 5XX 错误码用于指示Binance服务侧的问题。
HTTP 504 表示API服务端已经向业务核心提交了请求但未能获取响应，特别需要注意的是504代码不代表请求失败，而是未知。很可能已经得到了执行，也有可能执行失败，需要做进一步确认。
每个接口都有可能抛出异常，异常响应格式如下：
{
  "code": -1121,
  "msg": "Invalid symbol."
}
具体的错误码及其解释在错误代码汇总.md
GET方法的接口, 参数必须在query string中发送.
POST, PUT, 和 DELETE 方法的接口, 参数可以在 query string中发送，也可以在 request body中发送(content type application/x-www-form-urlencoded。允许混合这两种方式发送参数。但如果同一个参数名在query string和request body中都有，query string中的会被优先采用。
对参数的顺序不做要求。


访问限制
The following intervalLetter values for headers:
SECOND => S
MINUTE => M
HOUR => H
DAY => D
在 /api/v1/exchangeInfo接口中rateLimits数组里包含有REST接口(不限于本篇的REST接口)的访问限制。包括带权重的访问频次限制、下单速率限制。本篇枚举定义章节有限制类型的进一步说明。
The /wapi/v3 rateLimits array contains objects related to the exchange's REQUESTS and ORDER rate limits.
违反上述任何一个访问限制都会收到HTTP 429，这是一个警告
每一个接口均有一个相应的权重(weight)，有的接口根据参数不同可能拥有不同的权重。越消耗资源的接口权重就会越大
每个请求都会有X-MBX-USED-WEIGHT header， 它包含了现在该IP地址在这一分钟权重的使用量
当收到429告警时，调用者应当降低访问频率或者停止访问。
收到429后仍然继续违反访问限制，会被封禁IP，并收到418错误码
频繁违反限制，封禁时间会逐渐延长，从最短2分钟到最长3天.
A Retry-After header is sent with a 418 or 429 responses and will give the number of seconds required to wait, in the case of a 418, to prevent a ban, or, in the case of a 429, until the ban is over.

接口鉴权类型
每个接口都有自己的鉴权类型，鉴权类型决定了访问时应当进行何种鉴权
如果需要 API-key，应当在HTTP头中以X-MBX-APIKEY字段传递
API-key 与 API-secret 是大小写敏感的
API-keys can be configured to only access certain types of secure endpoints. For example, one API-key could be used for TRADE only, while another API-key can access everything except for TRADE routes.
API-keys默认可以访问所有鉴权类型can access all secure routes.
鉴权类型              描述
NONE	        不需要鉴权的接口
TRADE	        需要有效的API-KEY和签名
USER_DATA	    需要有效的API-KEY和签名
USER_STREAM	  需要有效的API-KEY
MARKET_DATA	  需要有效的API-KEY
TRADE, MARGIN 和 USER_DATA 是 需要签名的接口.

需要签名的接口(TRADE、USER_DATA AND MARGIN)
调用这些接口时，除了接口本身所需的参数外，还需要传递signature即签名参数。
签名使用HMAC SHA256算法. API-KEY所对应的API-Secret作为 HMAC SHA256 的密钥，其他所有参数作为HMAC SHA256的操作对象，得到的输出即为签名。
签名大小写不敏感。
所有参数包含参数与参数值

时间同步安全
签名接口均需要同时传递参数与时间戳, 时间戳的值应当是请求发送时刻的unix时间戳（毫秒）
服务器收到请求时会判断请求中的时间戳，如果是5000毫秒之前发出的，则请求会被认为无效。这个时间窗口值可以通过发送可选参数recvWindow来自定义。
另外，如果服务器计算得出客户端时间戳在服务器时间的‘未来’一秒以上，也会拒绝请求。
逻辑伪代码：
if (timestamp < (serverTime + 1000) && (serverTime - timestamp) <= recvWindow) {
  // process request
} else {
  // reject request
}

关于交易时效性 互联网状况并不100%可靠，不可完全依赖,因此你的程序本地到币安服务器的时延会有抖动. 这是我们设置recvWindow的目的所在，如果你从事高频交易，对交易时效性有较高的要求，可以灵活设置recvWindow以达到你的要求。 不推荐使用5秒以上的recvWindow

POST /api/v3/order签名接口示例
以下是如何从Linux命令行使用echo，openssl，和curl调用接口下单的示例：

Key	                    Value
apiKey	vmPUZE6mv9SD5VNHk4HlWFsOr6aKE2zvsw0MuIgwCIPy6utIco14y7Ju91duEh8A
secretKey	NhqPtmdSJYdKjVHjA7PZj4Mge3R5YNiP1e3UZjInClVN65XAbvqqM6A7H5fATj0j


参数   	值
symbol	LTCBTC
side	BUY
type	LIMIT
timeInForce	GTC
quantity	1
price	0.1
recvWindow	5000
timestamp	1499827319559

示例 1: 通过query string发送
HMAC SHA256 签名:
$ echo -n "symbol=LTCBTC&side=BUY&type=LIMIT&timeInForce=GTC&quantity=1&price=0.1&recvWindow=5000&timestamp=1499827319559" | openssl dgst -sha256 -hmac "NhqPtmdSJYdKjVHjA7PZj4Mge3R5YNiP1e3UZjInClVN65XAbvqqM6A7H5fATj0j"
(stdin)= c8db56825ae71d6d79447849e617115f4a920fa2acdcab2b053c4b2838bd6b71

curl 调用:
(HMAC SHA256)
$ curl -H "X-MBX-APIKEY: vmPUZE6mv9SD5VNHk4HlWFsOr6aKE2zvsw0MuIgwCIPy6utIco14y7Ju91duEh8A" -X POST 'https://api.binance.com/api/v3/order?symbol=LTCBTC&side=BUY&type=LIMIT&timeInForce=GTC&quantity=1&price=0.1&recvWindow=5000&timestamp=1499827319559&signature=c8db56825ae71d6d79447849e617115f4a920fa2acdcab2b053c4b2838bd6b71'

queryString:
symbol=LTCBTC&side=BUY&type=LIMIT&timeInForce=GTC&quantity=1&price=0.1&recvWindow=5000&timestamp=1499827319559

示例 2: 通过request body发送
HMAC SHA256 签名:
$ echo -n "symbol=LTCBTC&side=BUY&type=LIMIT&timeInForce=GTC&quantity=1&price=0.1&recvWindow=5000&timestamp=1499827319559" | openssl dgst -sha256 -hmac "NhqPtmdSJYdKjVHjA7PZj4Mge3R5YNiP1e3UZjInClVN65XAbvqqM6A7H5fATj0j"
(stdin)= c8db56825ae71d6d79447849e617115f4a920fa2acdcab2b053c4b2838bd6b71

curl 调用:
(HMAC SHA256)
$ curl -H "X-MBX-APIKEY: vmPUZE6mv9SD5VNHk4HlWFsOr6aKE2zvsw0MuIgwCIPy6utIco14y7Ju91duEh8A" -X POST 'https://api.binance.com/api/v3/order' -d 'symbol=LTCBTC&side=BUY&type=LIMIT&timeInForce=GTC&quantity=1&price=0.1&recvWindow=5000&timestamp=1499827319559&signature=c8db56825ae71d6d79447849e617115f4a920fa2acdcab2b053c4b2838bd6b71'

requestBody:
symbol=LTCBTC&side=BUY&type=LIMIT&timeInForce=GTC&quantity=1&price=0.1&recvWindow=5000&timestamp=1499827319559

示例 3: 混合使用 query string 与 request body
queryString: symbol=LTCBTC&side=BUY&type=LIMIT&timeInForce=GTC
requestBody: quantity=1&price=0.1&recvWindow=5000&timestamp=1499827319559

HMAC SHA256 签名:
$ echo -n "symbol=LTCBTC&side=BUY&type=LIMIT&timeInForce=GTCquantity=1&price=0.1&recvWindow=5000&timestamp=1499827319559" | openssl dgst -sha256 -hmac "NhqPtmdSJYdKjVHjA7PZj4Mge3R5YNiP1e3UZjInClVN65XAbvqqM6A7H5fATj0j"
(stdin)= 0fd168b8ddb4876a0358a8d14d0c9f3da0e9b20c5d52b2a00fcf7d1c602f9a77

curl command:
(HMAC SHA256)
$ curl -H "X-MBX-APIKEY: vmPUZE6mv9SD5VNHk4HlWFsOr6aKE2zvsw0MuIgwCIPy6utIco14y7Ju91duEh8A" -X POST 'https://api.binance.com/api/v3/order?symbol=LTCBTC&side=BUY&type=LIMIT&timeInForce=GTC' -d 'quantity=1&price=0.1&recvWindow=5000&timestamp=1499827319559&signature=0fd168b8ddb4876a0358a8d14d0c9f3da0e9b20c5d52b2a00fcf7d1c602f9a77'
注意在示例3中签名是不一样的. 在 "GTC" 和 "quantity=1"之间是没有‘&’ 符号的.

SIGNED Endpoint Examples for POST /wapi/v3/withdraw.html
Here is a step-by-step example of how to send a vaild signed payload from the Linux command line using echo, openssl, and curl.

Key	Value
apiKey	vmPUZE6mv9SD5VNHk4HlWFsOr6aKE2zvsw0MuIgwCIPy6utIco14y7Ju91duEh8A
secretKey	NhqPtmdSJYdKjVHjA7PZj4Mge3R5YNiP1e3UZjInClVN65XAbvqqM6A7H5fATj0j
Parameter	Value
asset	ETH
address	0x6915f16f8791d0a1cc2bf47c13a6b2a92000504b
addressTag	1 (Secondary address identifier for coins like XRP,XMR etc.)
amount	1
recvWindow	5000
name	addressName (Description of the address)
timestamp	1508396497000
signature	157fb937ec848b5f802daa4d9f62bea08becbf4f311203bda2bd34cd9853e320
Example 1: As a query string
HMAC SHA256 signature:

    $ echo -n "asset=ETH&address=0x6915f16f8791d0a1cc2bf47c13a6b2a92000504b&amount=1&recvWindow=5000&timestamp=1510903211000" | openssl dgst -sha256 -hmac "NhqPtmdSJYdKjVHjA7PZj4Mge3R5YNiP1e3UZjInClVN65XAbvqqM6A7H5fATj0j"
    (stdin)= 157fb937ec848b5f802daa4d9f62bea08becbf4f311203bda2bd34cd9853e320

curl command:

    (HMAC SHA256)
    $ curl -H "X-MBX-APIKEY: vmPUZE6mv9SD5VNHk4HlWFsOr6aKE2zvsw0MuIgwCIPy6utIco14y7Ju91duEh8A" -X POST 'https://www.binance.com/wapi/v3/withdraw.html?asset=ETH&address=0x6915f16f8791d0a1cc2bf47c13a6b2a92000504b&amount=1&recvWindow=5000&name=addressName&timestamp=1510903211000&signature=157fb937ec848b5f802daa4d9f62bea08becbf4f311203bda2bd34cd9853e320'

queryString:
asset=ETH
&address=0x6915f16f8791d0a1cc2bf47c13a6b2a92000504b &amount=1&recvWindow=5000&name=test&timestamp=1510903211000

Note that for wapi, parameters must be sent in query strings.

Public API Definitions
Terminology
base asset refers to the asset that is the quantity of a symbol.
quote asset refers to the asset that is the price of a symbol.
ENUM definitions
Symbol status (status):

PRE_TRADING
TRADING
POST_TRADING
END_OF_DAY
HALT
AUCTION_MATCH
BREAK
Symbol type:

SPOT
Order status (status):

NEW
PARTIALLY_FILLED
FILLED
CANCELED
PENDING_CANCEL (currently unused)
REJECTED
EXPIRED
OCO Status (listStatusType):

RESPONSE
EXEC_STARTED
ALL_DONE
OCO Order Status (listOrderStatus):

EXECUTING
ALL_DONE
REJECT
ContingencyType

OCO
Order types (orderTypes, type):

LIMIT
MARKET
STOP_LOSS
STOP_LOSS_LIMIT
TAKE_PROFIT
TAKE_PROFIT_LIMIT
LIMIT_MAKER
Order side (side):

BUY
SELL
Time in force (timeInForce):

GTC
IOC
FOK
Kline/Candlestick chart intervals:

m -> minutes; h -> hours; d -> days; w -> weeks; M -> months

1m
3m
5m
15m
30m
1h
2h
4h
6h
8h
12h
1d
3d
1w
1M
Rate limiters (rateLimitType)

REQUEST_WEIGHT

    {
      "rateLimitType": "REQUEST_WEIGHT",
      "interval": "MINUTE",
      "intervalNum": 1,
      "limit": 1200
    }
ORDERS

    {
      "rateLimitType": "ORDERS",
      "interval": "SECOND",
      "intervalNum": 1,
      "limit": 10
    }
RAW_REQUESTS

    {
      "rateLimitType": "RAW_REQUESTS",
      "interval": "MINUTE",
      "intervalNum": 5,
      "limit": 5000
    }
REQUEST_WEIGHT

ORDERS

RAW_REQUESTS

Rate limit intervals (interval)

SECOND
MINUTE
DAY
Filters
Filters define trading rules on a symbol or an exchange. Filters come in two forms: symbol filters and exchange filters.

Symbol Filters
PRICE_FILTER
ExchangeInfo format:

  {
    "filterType": "PRICE_FILTER",
    "minPrice": "0.00000100",
    "maxPrice": "100000.00000000",
    "tickSize": "0.00000100"
  }
The PRICE_FILTER defines the price rules for a symbol. There are 3 parts:

minPrice defines the minimum price/stopPrice allowed; disabled on minPrice == 0.
maxPrice defines the maximum price/stopPrice allowed; disabled on maxPrice == 0.
tickSize defines the intervals that a price/stopPrice can be increased/decreased by; disabled on tickSize == 0.
Any of the above variables can be set to 0, which disables that rule in the price filter. In order to pass the price filter, the following must be true for price/stopPrice of the enabled rules:

price >= minPrice
price <= maxPrice
(price-minPrice) % tickSize == 0
PERCENT_PRICE
ExchangeInfo format:

  {
    "filterType": "PERCENT_PRICE",
    "multiplierUp": "1.3000",
    "multiplierDown": "0.7000",
    "avgPriceMins": 5
  }
The PERCENT_PRICE filter defines valid range for a price based on the average of the previous trades. avgPriceMins is the number of minutes the average price is calculated over. 0 means the last price is used.

In order to pass the percent price, the following must be true for price: * price <= weightedAveragePrice * multiplierUp * price >= weightedAveragePrice * multiplierDown

LOT_SIZE
ExchangeInfo format:

  {
    "filterType": "LOT_SIZE",
    "minQty": "0.00100000",
    "maxQty": "100000.00000000",
    "stepSize": "0.00100000"
  }
The LOT_SIZE filter defines the quantity (aka "lots" in auction terms) rules for a symbol. There are 3 parts:

minQty defines the minimum quantity/icebergQty allowed.
maxQty defines the maximum quantity/icebergQty allowed.
stepSize defines the intervals that a quantity/icebergQty can be increased/decreased by.
In order to pass the lot size, the following must be true for quantity/icebergQty:

quantity >= minQty
quantity <= maxQty
(quantity-minQty) % stepSize == 0
MIN_NOTIONAL
ExchangeInfo format:

  {
    "filterType": "MIN_NOTIONAL",
    "minNotional": "0.00100000",
    "applyToMarket": true,
    "avgPriceMins": 5
  }
The MIN_NOTIONAL filter defines the minimum notional value allowed for an order on a symbol. An order's notional value is the price * quantity. applyToMarket determines whether or not the MIN_NOTIONAL filter will also be applied to MARKET orders. Since MARKET orders have no price, the average price is used over the last avgPriceMins minutes. avgPriceMins is the number of minutes the average price is calculated over. 0 means the last price is used.

ICEBERG_PARTS
ExchangeInfo format:

  {
    "filterType": "ICEBERG_PARTS",
    "limit": 10
  }
The ICEBERG_PARTS filter defines the maximum parts an iceberg order can have. The number of ICEBERG_PARTS is defined as CEIL(qty / icebergQty).

MARKET_LOT_SIZE
ExchangeInfo format:

  {
    "filterType": "MARKET_LOT_SIZE",
    "minQty": "0.00100000",
    "maxQty": "100000.00000000",
    "stepSize": "0.00100000"
  }
The MARKET_LOT_SIZE filter defines the quantity (aka "lots" in auction terms) rules for MARKET orders on a symbol. There are 3 parts:

minQty defines the minimum quantity allowed.
maxQty defines the maximum quantity allowed.
stepSize defines the intervals that a quantity can be increased/decreased by.
In order to pass the market lot size, the following must be true for quantity:

quantity >= minQty
quantity <= maxQty
(quantity-minQty) % stepSize == 0
MAX_NUM_ORDERS
ExchangeInfo format:

  {
    "filterType": "MAX_NUM_ORDERS",
    "limit": 25
  }
The MAX_NUM_ORDERS filter defines the maximum number of orders an account is allowed to have open on a symbol. Note that both "algo" orders and normal orders are counted for this filter.

MAX_NUM_ALGO_ORDERS
ExchangeInfo format:

  {
    "filterType": "MAX_NUM_ALGO_ORDERS",
    "maxNumAlgoOrders": 5
  }
The MAX_NUM_ALGO_ORDERS filter defines the maximum number of "algo" orders an account is allowed to have open on a symbol. "Algo" orders are STOP_LOSS, STOP_LOSS_LIMIT, TAKE_PROFIT, and TAKE_PROFIT_LIMIT orders.

MAX_NUM_ICEBERG_ORDERS
The MAX_NUM_ICEBERG_ORDERS filter defines the maximum number of ICEBERG orders an account is allowed to have open on a symbol. An ICEBERG order is any order where the icebergQty is > 0.

ExchangeInfo format:

  {
    "filterType": "MAX_NUM_ICEBERG_ORDERS",
    "maxNumIcebergOrders": 5
  }
Exchange Filters
EXCHANGE_MAX_NUM_ORDERS
ExchangeInfo format:

  {
    "filterType": "EXCHANGE_MAX_NUM_ORDERS",
    "maxNumOrders": 1000
  }
The MAX_NUM_ORDERS filter defines the maximum number of orders an account is allowed to have open on the exchange. Note that both "algo" orders and normal orders are counted for this filter.

EXCHANGE_MAX_NUM_ALGO_ORDERS
ExchangeInfo format:

  {
    "filterType": "EXCHANGE_MAX_ALGO_ORDERS",
    "maxNumAlgoOrders": 200
  }
The MAX_ALGO_ORDERS filter defines the maximum number of "algo" orders an account is allowed to have open on the exchange. "Algo" orders are STOP_LOSS, STOP_LOSS_LIMIT, TAKE_PROFIT, and TAKE_PROFIT_LIMIT orders.

Wallet Endpoints
System Status (System)
Response:

{ 
    "status": 0,              // 0: normal，1：system maintenance
    "msg": "normal"           // normal or system maintenance
}
GET /wapi/v3/systemStatus.html

Fetch system status.

Withdraw
Response:

{
    "msg": "success",
    "success": true,
    "id":"7213fea8e94b4a5593d507237e5a555b"
}
POST /wapi/v3/withdraw.html (HMAC SHA256)

Submit a withdraw request.

Weight: 1

Parameters:

Name	Type	Mandatory	Description
asset	STRING	YES	
address	STRING	YES	
addressTag	STRING	NO	Secondary address identifier for coins like XRP,XMR etc.
amount	DECIMAL	YES	
name	STRING	NO	Description of the address
recvWindow	LONG	NO	
timestamp	LONG	YES	
Deposit History (USER_DATA)
Response:

{
    "depositList": [
        {
            "insertTime": 1508198532000,
            "amount": 0.04670582,
            "asset": "ETH",
            "address": "0x6915f16f8791d0a1cc2bf47c13a6b2a92000504b",
            "txId": "0xdf33b22bdb2b28b1f75ccd201a4a4m6e7g83jy5fc5d5a9d1340961598cfcb0a1",
            "status": 1
        },
        {
            "insertTime": 1508298532000,
            "amount": 1000,
            "asset": "XMR",
            "address": "463tWEBn5XZJSxLU34r6g7h8jtxuNcDbjLSjkn3XAXHCbLrTTErJrBWYgHJQyrCwkNgYvyV3z8zctJLPCZy24jvb3NiTcTJ",
            "addressTag": "342341222",
            "txId": "b3c6219639c8ae3f9cf010cdc24fw7f7yt8j1e063f9b4bd1a05cb44c4b6e2509",
            "status": 1
        }
    ],
    "success": true
}
GET /wapi/v3/depositHistory.html (HMAC SHA256)

Fetch deposit history.

Weight: 1

Parameters:

Name	Type	Mandatory	Description
asset	STRING	NO	
status	INT	NO	0(0:pending,6: credited but cannot withdraw, 1:success)
startTime	LONG	NO	
endTime	LONG	NO	
recvWindow	LONG	NO	
timestamp	LONG	YES	
Withdraw History (USER_DATA)
Response:

{
    "withdrawList": [
        {
            "id":"7213fea8e94b4a5593d507237e5a555b",
            "amount": 1,
            "address": "0x6915f16f8791d0a1cc2bf47c13a6b2a92000504b",
            "asset": "ETH",
            "txId": "0xdf33b22bdb2b28b1f75ccd201a4a4m6e7g83jy5fc5d5a9d1340961598cfcb0a1",
            "applyTime": 1508198532000,
            "status": 4
        },
        {
            "id":"7213fea8e94b4a5534ggsd237e5a555b",
            "amount": 1000,
            "address": "463tWEBn5XZJSxLU34r6g7h8jtxuNcDbjLSjkn3XAXHCbLrTTErJrBWYgHJQyrCwkNgYvyV3z8zctJLPCZy24jvb3NiTcTJ",
            "addressTag": "342341222",
            "txId": "b3c6219639c8ae3f9cf010cdc24fw7f7yt8j1e063f9b4bd1a05cb44c4b6e2509",
            "asset": "XMR",
            "applyTime": 1508198532000,
            "status": 4
        }
    ],
    "success": true
}
GET /wapi/v3/withdrawHistory.html (HMAC SHA256)

Fetch withdraw history.

Weight: 1

Parameters:

Name	Type	Mandatory	Description
asset	STRING	NO	
status	INT	NO	0(0:Email Sent,1:Cancelled 2:Awaiting Approval 3:Rejected 4:Processing 5:Failure 6Completed)
startTime	LONG	NO	
endTime	LONG	NO	
recvWindow	LONG	NO	
timestamp	LONG	YES	
Deposit Address (USER_DATA)
Response:

{
    "address": "0x6915f16f8791d0a1cc2bf47c13a6b2a92000504b",
    "success": true,
    "addressTag": "1231212",
    "asset": "BNB"
}

GET /wapi/v3/depositAddress.html (HMAC SHA256)

Fetch deposit address.

Weight: 1

Parameters:

Name	Type	Mandatory	Description
asset	STRING	YES	
status	Boolean	NO	
recvWindow	LONG	NO	
timestamp	LONG	YES	
Account Status (USER_DATA)
Response:

{
    "msg": "Order failed:Low Order fill rate! Will be reactivated after 5 minutes.",
    "success": true,
    "objs": [
        "5"
    ]
}
GET /wapi/v3/accountStatus.html

Fetch account status detail.

Weight: 1

Parameters:

Name	Type	Mandatory	Description
recvWindow	LONG	NO	
timestamp	LONG	YES	
Account API Trading Status (USER_DATA)
Response:

{
    "success": true,     // Query result
    "status": {          // API trading status detail
        "isLocked": false,   // API trading function is locked or not
        "plannedRecoverTime": 0,  // If API trading function is locked, this is the planned recover time
        "triggerCondition": { 
            "GCR": 150,  // Number of GTC orders
            "IFER": 150, // Number of FOK/IOC orders
            "UFR": 300   // Number of orders
        },
        "indicators": {  // The indicators updated every 30 seconds
           "BTCUSDT": [  // The symbol
            {
            "i": "UFR",  // Unfilled Ratio (UFR)
            "c": 20,     // Count of all orders
            "v": 0.05,   // Current UFR value
            "t": 0.995   // Trigger UFR value
            },
            {
            "i": "IFER", // IOC/FOK Expiration Ratio (IFER)
            "c": 20,     // Count of FOK/IOC orders
            "v": 0.99,   // Current IFER value
            "t": 0.99    // Trigger IFER value
            },
            {
            "i": "GCR",  // GTC Cancellation Ratio (GCR)
            "c": 20,     // Count of GTC orders
            "v": 0.99,   // Current GCR value
            "t": 0.99    // Trigger GCR value
            }
            ],
            "ETHUSDT": [ 
            {
            "i": "UFR",
            "c": 20,
            "v": 0.05,
            "t": 0.995
            },
            {
            "i": "IFER",
            "c": 20,
            "v": 0.99,
            "t": 0.99
            },
            {
            "i": "GCR",
            "c": 20,
            "v": 0.99,
            "t": 0.99
            }
            ]
        },
        "updateTime": 1547630471725   // The query result return time
    }
}
GET /wapi/v3/apiTradingStatus.html

Fetch account api trading status detail.

For more details about our api trading rules, please refer to the link:https://support.binance.com/hc/en-us/articles/115003235691

Weight: 1

Parameters:

Name	Type	Mandatory	Description
recvWindow	LONG	NO	
timestamp	LONG	YES	
DustLog (USER_DATA)
Response:

{
    "success": true, 
    "results": {
        "total": 2,   //Total counts of exchange
        "rows": [
            {
                "transfered_total": "0.00132256",//Total transfered BNB amount for this exchange.
                "service_charge_total": "0.00002699",   //Total service charge amount for this exchange.
                "tran_id": 4359321,
                "logs": [           //Details of  this exchange.
                    {
                        "tranId": 4359321,
                        "serviceChargeAmount": "0.000009",
                        "uid": "10000015",
                        "amount": "0.0009",
                        "operateTime": "2018-05-03 17:07:04",
                        "transferedAmount": "0.000441",
                        "fromAsset": "USDT"
                    },
                    {
                        "tranId": 4359321,
                        "serviceChargeAmount": "0.00001799",
                        "uid": "10000015",
                        "amount": "0.0009",
                        "operateTime": "2018-05-03 17:07:04",
                        "transferedAmount": "0.00088156",
                        "fromAsset": "ETH"
                    }
                ],
                "operate_time": "2018-05-03 17:07:04" //The time of this exchange.
            },
            {
                "transfered_total": "0.00058795",
                "service_charge_total": "0.000012",
                "tran_id": 4357015,
                "logs": [       // Details of  this exchange.
                    {
                        "tranId": 4357015,
                        "serviceChargeAmount": "0.00001",
                        "uid": "10000015",
                        "amount": "0.001",
                        "operateTime": "2018-05-02 13:52:24",
                        "transferedAmount": "0.00049",
                        "fromAsset": "USDT"
                    },
                    {
                        "tranId": 4357015,
                        "serviceChargeAmount": "0.000002",
                        "uid": "10000015",
                        "amount": "0.0001",
                        "operateTime": "2018-05-02 13:51:11",
                        "transferedAmount": "0.00009795",
                        "fromAsset": "ETH"
                    }
                ],
                "operate_time": "2018-05-02 13:51:11"
            }
        ]
    }
}
GET /wapi/v3/userAssetDribbletLog.html (HMAC SHA256)

Fetch small amounts of assets exchanged BNB records.

Weight: 1

Parameters:

Name	Type	Mandatory	Description
recvWindow	LONG	NO	
timestamp	LONG	YES	
Trade Fee (USER_DATA)
Response:

{
    "tradeFee": [{
        "symbol": "ADABNB",
        "maker": 0.9000,
        "taker": 1.0000
    }, {
        "symbol": "BNBBTC",
        "maker": 0.3000,
        "taker": 0.3000
    }],
    "success": true
}
GET /wapi/v3/tradeFee.html (HMAC SHA256)

Fetch trade fee.

Weight: 1

Parameters:

Name	Type	Mandatory	Description
recvWindow	LONG	NO	
timestamp	LONG	YES	
symbol	STRING	NO	
Asset Detail (USER_DATA)
Response:

{
    "success": true,
    "assetDetail": {
        "CTR": {
            "minWithdrawAmount": "70.00000000", //min withdraw amount
            "depositStatus": false,//deposit status
            "withdrawFee": 35, // withdraw fee
            "withdrawStatus": true, //withdraw status
            "depositTip": "Delisted, Deposit Suspended" //reason
        },
        "SKY": {
            "minWithdrawAmount": "0.02000000",
            "depositStatus": true,
            "withdrawFee": 0.01,
            "withdrawStatus": true
        }   
    }
}
GET /wapi/v3/assetDetail.html (HMAC SHA256)

Fetch asset detail.

Weight: 1

Parameters:

Name	Type	Mandatory	Description
recvWindow	LONG	NO	
timestamp	LONG	YES	
Query Sub-account List(For Master Account)
Response:

{
    "success":true,
    "subAccounts":[
        {
            "email":"123@test.com",
            "status":"enabled",
            "activated":true,
            "mobile":"91605290",
            "gAuth":true,
            "createTime":1544433328000
        },
        {
            "email":"321@test.com",
            "status":"disabled",
            "activated":true,
            "mobile":"22501238",
            "gAuth":true,
            "createTime":1544433328000
        }
    ]
}
GET /wapi/v3/sub-account/list.html (HMAC SHA256)

Fetch sub account list.

Weight: 1

Parameters:

Name	Type	Mandatory	Description
email	STRING	NO	Sub-account email
status	STRING	NO	Sub-account status: enabled or disabled
page	INT	NO	Default value: 1
limit	INT	NO	Default value: 500
recvWindow	LONG	NO	
timestamp	LONG	YES	
Query Sub-account Transfer History(For Master Account)
Response:

{
    "success":true,
    "transfers":[
        {
            "from":"aaa@test.com",
            "to":"bbb@test.com",
            "asset":"BTC",
            "qty":"1",
            "time":1544433328000
        },
        {
            "from":"bbb@test.com",
            "to":"ccc@test.com",
            "asset":"ETH",
            "qty":"2",
            "time":1544433328000
        }
    ]
}
GET /wapi/v3/sub-account/transfer/history.html (HMAC SHA256)

Fetch transfer history list

Weight: 1

Parameters:

Name	Type	Mandatory	Description
email	STRING	YES	Sub-account email
startTime	LONG	NO	Default return the history with in 100 days
endTime	LONG	NO	Default return the history with in 100 days
page	INT	NO	Default value: 1
limit	INT	NO	Default value: 500
recvWindow	LONG	NO	
timestamp	LONG	YES	
Sub-account Transfer(For Master Account)
Response:

{
    "success":true,
    "txnId":"2966662589"
}

POST /wapi/v3/sub-account/transfer.html (HMAC SHA256)

Execute sub-account transfer

Weight: 1

Parameters:

Name	Type	Mandatory	Description
fromEmail	STRING	YES	Sender email
toEmail	STRING	YES	Recipient email
asset	STRING	YES	
amount	DECIMAL	YES	
recvWindow	LONG	NO	
timestamp	LONG	YES	
Query Sub-account Assets(For Master Account)
Response:

{

    "success":true,
    "balances":[
        {
            "asset":"ADA",
            "free":10000,
            "locked":0
        },
        {
            "asset":"BNB",
            "free":10003,
            "locked":0
        },
        {
            "asset":"BTC",
            "free":11467.6399,
            "locked":0
        },
        {
            "asset":"ETH",
            "free":10004.995,
            "locked":0
        },
        {
            "asset":"USDT",
            "free":11652.14213,
            "locked":0
        }
    ]
}
GET /wapi/v3/sub-account/assets.html (HMAC SHA256)

Fetch sub-account assets

Weight: 1

Parameters:

Name	Type	Mandatory	Description
email	STRING	YES	Sub account email
recvWindow	LONG	NO	
timestamp	LONG	YES	
symbol	STRING	NO	
Dust Transfer (USER_DATA)
Response:

{
    "totalServiceCharge":"0.02102542",
    "totalTransfered":"1.05127099",
    "transferResult":[
        {
            "amount":"0.03000000",
            "fromAsset":"ETH",
            "operateTime":1563368549307,
            "serviceChargeAmount":"0.00500000",
            "tranId":2970932918,
            "transferedAmount":"0.25000000"
        },
        {
            "amount":"0.09000000",
            "fromAsset":"LTC",
            "operateTime":1563368549404,
            "serviceChargeAmount":"0.01548000",
            "tranId":2970932918,
            "transferedAmount":"0.77400000"
        },
        {
            "amount":"248.61878453",
            "fromAsset":"TRX",
            "operateTime":1563368549489,
            "serviceChargeAmount":"0.00054542",
            "tranId":2970932918,
            "transferedAmount":"0.02727099"
        }
    ]
}
Post /sapi/v1/asset/dust (HMAC SHA256)

Convert dust assets to BNB.

Weight: 1

Parameters:

Name	Type	Mandatory	Description
asset	ARRAY	YES	The asset being converted. For example: asset=BTC&asset=USDT
recvWindow	LONG	NO	
timestamp	LONG	YES	
Asset Dividend Record (USER_DATA)
Response:

{
    "rows":[
        {
            "amount":"10.00000000",
            "asset":"BHFT",
            "divTime":1563189166000,
            "enInfo":"BHFT distribution",
            "tranId":2968885920
        },
        {
            "amount":"10.00000000",
            "asset":"BHFT",
            "divTime":1563189165000,
            "enInfo":"BHFT distribution",
            "tranId":2968885920
        }
    ],
    "total":2
}
Get /sapi/v1/asset/assetDividend (HMAC SHA256)

Query asset dividend record.

Weight: 1

Parameters:

Name	Type	Mandatory	Description
asset	STRING	NO	
startTime	LONG	NO	
endTime	LONG	NO	
recvWindow	LONG	NO	
timestamp	LONG	YES	
Market Data Endpoints
Test Connectivity
Response:

{}
GET /api/v1/ping

Test connectivity to the Rest API.

Weight:

1

Parameters:

NONE

Check Server Time
Response:

{
  "serverTime": 1499827319559
}
GET /api/v1/time

Test connectivity to the Rest API and get the current server time.

Weight: 1

Parameters:

NONE

Exchange Information
Response:

{
    "timezone": "UTC",
    "serverTime": 1565246363776,
    "rateLimits": [
        {
            //These are defined in the `ENUM definitions` section under `Rate Limiters (rateLimitType)`.
            //All limits are optional
        }
    ],
    "exchangeFilters": [
            //These are the defirned filters in the `Filters` section.
            //All filters are optional.
    ],
    "symbols": [
        {
            "symbol": "ETHBTC",
            "status": "TRADING",
            "baseAsset": "ETH",
            "baseAssetPrecision": 8,
            "quoteAsset": "BTC",
            "quotePrecision": 8,
            "orderTypes": [
                "LIMIT",
                "LIMIT_MAKER",
                "MARKET",
                "STOP_LOSS",
                "STOP_LOSS_LIMIT",
                "TAKE_PROFIT",
                "TAKE_PROFIT_LIMIT"
            ],
            "icebergAllowed": true,
            "ocoAllowed": true,
            "isSpotTradingAllowed": true,
            "isMarginTradingAllowed": false,
            "filters": [
            //These are defined in the Filters section.
            //All filters are optional
            ]
        }
    ]
}
GET /api/v1/exchangeInfo

Current exchange trading rules and symbol information

Weight: 1

Parameters:

NONE

Order Book
Response:

{
  "lastUpdateId": 1027024,
  "bids": [
    [
      "4.00000000",     // PRICE
      "431.00000000"    // QTY
    ]
  ],
  "asks": [
    [
      "4.00000200",
      "12.00000000"
    ]
  ]
}
GET /api/v1/depth

Weight:

Adjusted based on the limit:

Limit	Weight
5, 10, 20, 50, 100	1
500	5
1000	10
5000	50
10000	100
Parameters:

Name	Type	Mandatory	Description
symbol	STRING	YES	
limit	INT	NO	Default 100; max 1000. Valid limits:[5, 10, 20, 50, 100, 500, 1000, 5000, 10000]
Caution: setting limit=0 can return a lot of data.

Recent Trades List
Response:

[
  {
    "id": 28457,
    "price": "4.00000100",
    "qty": "12.00000000",
    "time": 1499865549590,
    "isBuyerMaker": true,
    "isBestMatch": true
  }
]
GET /api/v1/trades

Get recent trades (up to last 500).

Weight: 1

Parameters:

Name	Type	Mandatory	Description
symbol	STRING	YES	
limit	INT	NO	Default 500; max 1000.
Old Trade Lookup
Response:

[
  {
    "id": 28457,
    "price": "4.00000100",
    "qty": "12.00000000",
    "time": 1499865549590,
    "isBuyerMaker": true,
    "isBestMatch": true
  }
]
GET /api/v1/historicalTrades

Get older market trades.

Weight: 5

Parameters:

Name	Type	Mandatory	Description
symbol	STRING	YES	
limit	INT	NO	Default 500; max 1000.
fromId	LONG	NO	TradeId to fetch from. Default gets most recent trades.
X-MBX-APIKEY required
Compressed/Aggregate Trades List
Response:

[
  {
    "a": 26129,         // Aggregate tradeId
    "p": "0.01633102",  // Price
    "q": "4.70443515",  // Quantity
    "f": 27781,         // First tradeId
    "l": 27781,         // Last tradeId
    "T": 1498793709153, // Timestamp
    "m": true,          // Was the buyer the maker?
    "M": true           // Was the trade the best price match?
  }
]
GET /api/v1/aggTrades

Get compressed, aggregate trades. Trades that fill at the time, from the same order, with the same price will have the quantity aggregated.

Weight: 1

Parameters:

Name	Type	Mandatory	Description
symbol	STRING	YES	
fromId	LONG	NO	ID to get aggregate trades from INCLUSIVE.
startTime	LONG	NO	Timestamp in ms to get aggregate trades from INCLUSIVE.
endTime	LONG	NO	Timestamp in ms to get aggregate trades until INCLUSIVE.
limit	INT	NO	Default 500; max 1000.
If both startTime and endTime are sent, time between startTime and endTime must be less than 1 hour.
If fromId, startTime, and endTime are not sent, the most recent aggregate trades will be returned.
Kline/Candlestick Data
Response:

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
    "17928899.62484339" // Ignore.
  ]
]
GET /api/v1/klines

Kline/candlestick bars for a symbol.
Klines are uniquely identified by their open time.

Weight: 1

Parameters:

Name	Type	Mandatory	Description
symbol	STRING	YES	
interval	ENUM	YES	
startTime	LONG	NO	
endTime	LONG	NO	
limit	INT	NO	Default 500; max 1000.
If startTime and endTime are not sent, the most recent klines are returned.
Current Average Price
Response:

{
  "mins": 5,
  "price": "9.35751834"
}
GET /api/v3/avgPrice

Current average price for a symbol.

Weight: 1

Parameters:

Name	Type	Mandatory	Description
symbol	STRING	YES	
24hr Ticker Price Change Statistics
Response:

{
  "symbol": "BNBBTC",
  "priceChange": "-94.99999800",
  "priceChangePercent": "-95.960",
  "weightedAvgPrice": "0.29628482",
  "prevClosePrice": "0.10002000",
  "lastPrice": "4.00000200",
  "lastQty": "200.00000000",
  "bidPrice": "4.00000000",
  "askPrice": "4.00000200",
  "openPrice": "99.00000000",
  "highPrice": "100.00000000",
  "lowPrice": "0.10000000",
  "volume": "8913.30000000",
  "quoteVolume": "15.30000000",
  "openTime": 1499783499040,
  "closeTime": 1499869899040,
  "firstId": 28385,   // First tradeId
  "lastId": 28460,    // Last tradeId
  "count": 76         // Trade count
}
OR

[
  {
    "symbol": "BNBBTC",
    "priceChange": "-94.99999800",
    "priceChangePercent": "-95.960",
    "weightedAvgPrice": "0.29628482",
    "prevClosePrice": "0.10002000",
    "lastPrice": "4.00000200",
    "lastQty": "200.00000000",
    "bidPrice": "4.00000000",
    "askPrice": "4.00000200",
    "openPrice": "99.00000000",
    "highPrice": "100.00000000",
    "lowPrice": "0.10000000",
    "volume": "8913.30000000",
    "quoteVolume": "15.30000000",
    "openTime": 1499783499040,
    "closeTime": 1499869899040,
    "firstId": 28385,   // First tradeId
    "lastId": 28460,    // Last tradeId
    "count": 76         // Trade count
  }
]
GET /api/v1/ticker/24hr

24 hour rolling window price change statistics. Careful when accessing this with no symbol.

Weight:

1 for a single symbol;
40 when the symbol parameter is omitted

Parameters:

Name	Type	Mandatory	Description
symbol	STRING	NO	
If the symbol is not sent, tickers for all symbols will be returned in an array.
Symbol Price Ticker
Response:

{
  "symbol": "LTCBTC",
  "price": "4.00000200"
}
OR

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
GET /api/v3/ticker/price

Latest price for a symbol or symbols.

Weight:

1 for a single symbol;
2 when the symbol parameter is omitted

Parameters:

Name	Type	Mandatory	Description
symbol	STRING	NO	
If the symbol is not sent, prices for all symbols will be returned in an array.
Symbol Order Book Ticker
Response:

{
  "symbol": "LTCBTC",
  "bidPrice": "4.00000000",
  "bidQty": "431.00000000",
  "askPrice": "4.00000200",
  "askQty": "9.00000000"
}
OR

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
GET /api/v3/ticker/bookTicker

Best price/qty on the order book for a symbol or symbols.

Weight:

1 for a single symbol;
2 when the symbol parameter is omitted

Parameters:

Name	Type	Mandatory	Description
symbol	STRING	NO	
If the symbol is not sent, bookTickers for all symbols will be returned in an array.
Websocket Market Streams
The base endpoint is: wss://stream.binance.com:9443
Streams can be accessed either in a single raw stream or in a combined stream
Raw streams are accessed at /ws/<streamName>
Combined streams are accessed at /stream?streams=<streamName1>/<streamName2>/<streamName3>
Combined stream events are wrapped as follows: {"stream":"<streamName>","data":<rawPayload>}
All symbols for streams are lowercase
A single connection to stream.binance.com is only valid for 24 hours; expect to be disconnected at the 24 hour mark
The websocket server will send a ping frame every 3 minutes. If the websocket server does not receive a pong frame back from the connection within a 10 minute period, the connection will be disconnected. Unsolicited pong frames are allowed.
Aggregate Trade Streams
Payload:

{
  "e": "aggTrade",  // Event type
  "E": 123456789,   // Event time
  "s": "BNBBTC",    // Symbol
  "a": 12345,       // Aggregate trade ID
  "p": "0.001",     // Price
  "q": "100",       // Quantity
  "f": 100,         // First trade ID
  "l": 105,         // Last trade ID
  "T": 123456785,   // Trade time
  "m": true,        // Is the buyer the market maker?
  "M": true         // Ignore
}
The Aggregate Trade Streams push trade information that is aggregated for a single taker order.

Stream Name: <symbol>@aggTrade

Update Speed: Real-time

Trade Streams
Payload:

{
  "e": "trade",     // Event type
  "E": 123456789,   // Event time
  "s": "BNBBTC",    // Symbol
  "t": 12345,       // Trade ID
  "p": "0.001",     // Price
  "q": "100",       // Quantity
  "b": 88,          // Buyer order ID
  "a": 50,          // Seller order ID
  "T": 123456785,   // Trade time
  "m": true,        // Is the buyer the market maker?
  "M": true         // Ignore
}
The Trade Streams push raw trade information; each trade has a unique buyer and seller.

Stream Name: <symbol>@trade

Update Speed: Real-time

Kline/Candlestick Streams
Payload:

{
  "e": "kline",     // Event type
  "E": 123456789,   // Event time
  "s": "BNBBTC",    // Symbol
  "k": {
    "t": 123400000, // Kline start time
    "T": 123460000, // Kline close time
    "s": "BNBBTC",  // Symbol
    "i": "1m",      // Interval
    "f": 100,       // First trade ID
    "L": 200,       // Last trade ID
    "o": "0.0010",  // Open price
    "c": "0.0020",  // Close price
    "h": "0.0025",  // High price
    "l": "0.0015",  // Low price
    "v": "1000",    // Base asset volume
    "n": 100,       // Number of trades
    "x": false,     // Is this kline closed?
    "q": "1.0000",  // Quote asset volume
    "V": "500",     // Taker buy base asset volume
    "Q": "0.500",   // Taker buy quote asset volume
    "B": "123456"   // Ignore
  }
}
The Kline/Candlestick Stream push updates to the current klines/candlestick every second.

Stream Name: <symbol>@kline_<interval>

Update Speed: 2000ms

Kline/Candlestick chart intervals:

m -> minutes; h -> hours; d -> days; w -> weeks; M -> months

1m
3m
5m
15m
30m
1h
2h
4h
6h
8h
12h
1d
3d
1w
1M
Individual Symbol Mini Ticker Stream
Payload:

  {
    "e": "24hrMiniTicker",  // Event type
    "E": 123456789,         // Event time
    "s": "BNBBTC",          // Symbol
    "c": "0.0025",          // Close price
    "o": "0.0010",          // Open price
    "h": "0.0025",          // High price
    "l": "0.0010",          // Low price
    "v": "10000",           // Total traded base asset volume
    "q": "18"               // Total traded quote asset volume
  }
24hr rolling window mini-ticker statistics. These are NOT the statistics of the UTC day, but a 24hr rolling window for the previous 24hrs.

Stream Name: <symbol>@miniTicker

Update Speed: 1000ms

All Market Mini Tickers Stream
Payload:

[
  {
    // Same as <symbol>@miniTicker payload
  }
]
24hr rolling window mini-ticker statistics for all symbols that changed in an array. These are NOT the statistics of the UTC day, but a 24hr rolling window for the previous 24hrs. Note that only tickers that have changed will be present in the array.

Stream Name: !miniTicker@arr

Update Speed: 1000ms

Individual Symbol Ticker Streams
Payload:

{
  "e": "24hrTicker",  // Event type
  "E": 123456789,     // Event time
  "s": "BNBBTC",      // Symbol
  "p": "0.0015",      // Price change
  "P": "250.00",      // Price change percent
  "w": "0.0018",      // Weighted average price
  "x": "0.0009",      // First trade(F)-1 price (first trade before the 24hr rolling window)
  "c": "0.0025",      // Last price
  "Q": "10",          // Last quantity
  "b": "0.0024",      // Best bid price
  "B": "10",          // Best bid quantity
  "a": "0.0026",      // Best ask price
  "A": "100",         // Best ask quantity
  "o": "0.0010",      // Open price
  "h": "0.0025",      // High price
  "l": "0.0010",      // Low price
  "v": "10000",       // Total traded base asset volume
  "q": "18",          // Total traded quote asset volume
  "O": 0,             // Statistics open time
  "C": 86400000,      // Statistics close time
  "F": 0,             // First trade ID
  "L": 18150,         // Last trade Id
  "n": 18151          // Total number of trades
}
24hr rollwing window ticker statistics for a single symbol. These are NOT the statistics of the UTC day, but a 24hr rolling window for the previous 24hrs.

Stream Name: <symbol>@ticker

Update Speed: 1000ms

All Market Tickers Stream
Payload:

[
  {
    // Same as <symbol>@ticker payload
  }
]
24hr rolling window ticker statistics for all symbols that changed in an array. These are NOT the statistics of the UTC day, but a 24hr rolling window for the previous 24hrs. Note that only tickers that have changed will be present in the array.

Stream Name: !ticker@arr

Update Speed: 1000ms

Individual Symbol Book Ticker Streams
Payload: javascript { "u":400900217, // order book updateId "s":"BNBUSDT", // symbol "b":"25.35190000", // best bid price "B":"31.21000000", // best bid qty "a":"25.36520000", // best ask price "A":"40.66000000" // best ask qty }

Pushes any update to the best bid or ask's price or quantity in real-time for a specified symbol.

Stream Name: @bookTicker

Update Speed: Real-time

All Book Tickers Stream
Payload: javascript { // Same as <symbol>@bookTicker payload }

Pushes any update to the best bid or ask's price or quantity in real-time for all symbols.

Stream Name: !bookTicker

Update Speed: Real-time

Partial Book Depth Streams
Payload:

{
  "lastUpdateId": 160,  // Last update ID
  "bids": [             // Bids to be updated
    [
      "0.0024",         // Price level to be updated
      "10"              // Quantity
    ]
  ],
  "asks": [             // Asks to be updated
    [
      "0.0026",         // Price level to be updated
      "100"            // Quantity
    ]
  ]
}
Top bids and asks, Valid are 5, 10, or 20.

Stream Names: @depth OR @depth@100ms.

Update Speed: 1000ms or 100ms

Diff. Depth Stream
Payload:

{
  "e": "depthUpdate", // Event type
  "E": 123456789,     // Event time
  "s": "BNBBTC",      // Symbol
  "U": 157,           // First update ID in event
  "u": 160,           // Final update ID in event
  "b": [              // Bids to be updated
    [
      "0.0024",       // Price level to be updated
      "10"            // Quantity
    ]
  ],
  "a": [              // Asks to be updated
    [
      "0.0026",       // Price level to be updated
      "100"           // Quantity
    ]
  ]
}
Stream Name: @depth OR @depth@100ms

Update Speed: 1000ms or 100ms

Order book price and quantity depth updates used to locally manage an order book.

How to manage a local order book correctly
Open a stream to wss://stream.binance.com:9443/ws/bnbbtc@depth
Buffer the events you receive from the stream
Get a depth snapshot from https://www.binance.com/api/v3/depth?symbol=BNBBTC&limit=1000
Drop any event where u is <= lastUpdateId in the snapshot
The first processed should have U <= lastUpdateId+1 AND u >= lastUpdateId+1
While listening to the stream, each new event's U should be equal to the previous event's u+1
The data in each event is the absolute quantity for a price level
If the quantity is 0, remove the price level
Receiving an event that removes a price level that is not in your local order book can happen and is normal.
Account/Trades Endpoints
Test New Order (TRADE)
Response:

{}
POST /api/v3/order/test (HMAC SHA256)

Test new order creation and signature/recvWindow long. Creates and validates a new order but does not send it into the matching engine.

Weight: 1

Parameters:

Same as POST /api/v3/order

New Order (TRADE)
Response ACK:

{
  "symbol": "BTCUSDT",
  "orderId": 28,
  "orderListId": -1, //Unless OCO, value will be -1
  "clientOrderId": "6gCrw2kRUAF9CvJDGP16IP",
  "transactTime": 1507725176595
}
Response RESULT:

{
  "symbol": "BTCUSDT",
  "orderId": 28,
  "orderListId": -1, //Unless OCO, value will be -1
  "clientOrderId": "6gCrw2kRUAF9CvJDGP16IP",
  "transactTime": 1507725176595,
  "price": "1.00000000",
  "origQty": "10.00000000",
  "executedQty": "10.00000000",
  "cummulativeQuoteQty": "10.00000000",
  "status": "FILLED",
  "timeInForce": "GTC",
  "type": "MARKET",
  "side": "SELL"
}
Response FULL:

{
  "symbol": "BTCUSDT",
  "orderId": 28,
  "orderListId": -1, //Unless OCO, value will be -1
  "clientOrderId": "6gCrw2kRUAF9CvJDGP16IP",
  "transactTime": 1507725176595,
  "price": "1.00000000",
  "origQty": "10.00000000",
  "executedQty": "10.00000000",
  "cummulativeQuoteQty": "10.00000000",
  "status": "FILLED",
  "timeInForce": "GTC",
  "type": "MARKET",
  "side": "SELL",
  "fills": [
    {
      "price": "4000.00000000",
      "qty": "1.00000000",
      "commission": "4.00000000",
      "commissionAsset": "USDT"
    },
    {
      "price": "3999.00000000",
      "qty": "5.00000000",
      "commission": "19.99500000",
      "commissionAsset": "USDT"
    },
    {
      "price": "3998.00000000",
      "qty": "2.00000000",
      "commission": "7.99600000",
      "commissionAsset": "USDT"
    },
    {
      "price": "3997.00000000",
      "qty": "1.00000000",
      "commission": "3.99700000",
      "commissionAsset": "USDT"
    },
    {
      "price": "3995.00000000",
      "qty": "1.00000000",
      "commission": "3.99500000",
      "commissionAsset": "USDT"
    }
  ]
}
POST /api/v3/order (HMAC SHA256)

Send in a new order.

Weight: 1

Parameters:

Name	Type	Mandatory	Description
symbol	STRING	YES	
side	ENUM	YES	
type	ENUM	YES	
timeInForce	ENUM	NO	
quantity	DECIMAL	YES	
price	DECIMAL	NO	
newClientOrderId	STRING	NO	A unique id for the order. Automatically generated if not sent.
stopPrice	DECIMAL	NO	Used with STOP_LOSS, STOP_LOSS_LIMIT, TAKE_PROFIT, and TAKE_PROFIT_LIMIT orders.
icebergQty	DECIMAL	NO	Used with LIMIT, STOP_LOSS_LIMIT, and TAKE_PROFIT_LIMIT to create an iceberg order.
newOrderRespType	ENUM	NO	Set the response JSON. ACK, RESULT, or FULL; MARKET and LIMIT order types default to FULL, all other orders default to ACK.
recvWindow	LONG	NO	The value cannot be greater than 60000
timestamp	LONG	YES	
Additional mandatory parameters based on type:

Type	Additional mandatory parameters
LIMIT	timeInForce, quantity, price
MARKET	quantity
STOP_LOSS	quantity, stopPrice
STOP_LOSS_LIMIT	timeInForce, quantity, price, stopPrice
TAKE_PROFIT	quantity, stopPrice
TAKE_PROFIT_LIMIT	timeInForce, quantity, price, stopPrice
LIMIT_MAKER	quantity, price
Other info:

LIMIT_MAKER are LIMIT orders that will be rejected if they would immediately match and trade as a taker.
STOP_LOSS and TAKE_PROFIT will execute a MARKET order when the stopPrice is reached.
Any LIMIT or LIMIT_MAKER type order can be made an iceberg order by sending an icebergQty.
Any order with an icebergQty MUST have timeInForce set to GTC.
Trigger order price rules against market price for both MARKET and LIMIT versions:

Price above market price: STOP_LOSS BUY, TAKE_PROFIT SELL
Price below market price: STOP_LOSS SELL, TAKE_PROFIT BUY
Cancel Order (TRADE)
Response:

{
    "symbol": "LTCBTC",
    "origClientOrderId": "myOrder1",
    "orderId": 4,
    "orderListId": -1, //Unless part of an OCO, the value will always be -1.
    "clientOrderId": "cancelMyOrder1",
    "price": "2.00000000",
    "origQty": "1.00000000",
    "executedQty": "0.00000000",
    "cummulativeQuoteQty": "0.00000000",
    "status": "CANCELED",
    "timeInForce": "GTC",
    "type": "LIMIT",
    "side": "BUY"
}
DELETE /api/v3/order (HMAC SHA256)

Cancel an active order.

Weight: 1

Parameters:

Name	Type	Mandatory	Description
symbol	STRING	YES	
orderId	LONG	NO	
origClientOrderId	STRING	NO	
newClientOrderId	STRING	NO	Used to uniquely identify this cancel. Automatically generated by default.
recvWindow	LONG	NO	The value cannot be greater than 60000
timestamp	LONG	YES	
Either orderId or origClientOrderId must be sent.

Query Order (USER_DATA)
Response:

{
  "symbol": "LTCBTC",
  "orderId": 1,
  "orderListId": -1, //Unless OCO, value will be -1
  "clientOrderId": "myOrder1",
  "price": "0.1",
  "origQty": "1.0",
  "executedQty": "0.0",
  "cummulativeQuoteQty": "0.0",
  "status": "NEW",
  "timeInForce": "GTC",
  "type": "LIMIT",
  "side": "BUY",
  "stopPrice": "0.0",
  "icebergQty": "0.0",
  "time": 1499827319559,
  "updateTime": 1499827319559,
  "isWorking": true
}
GET /api/v3/order (HMAC SHA256)

Check an order's status.

Weight: 1

Parameters:

Name	Type	Mandatory	Description
symbol	STRING	YES	
orderId	LONG	NO	
origClientOrderId	STRING	NO	
recvWindow	LONG	NO	The value cannot be greater than 60000
timestamp	LONG	YES	
Notes:

Either orderId or origClientOrderId must be sent.
For some historical orders cummulativeQuoteQty will be < 0, meaning the data is not available at this time.
Current Open Orders (USER_DATA)
Response:

[
  {
    "symbol": "LTCBTC",
    "orderId": 1,
    "orderListId": -1, //Unless OCO, the value will always be -1
    "clientOrderId": "myOrder1",
    "price": "0.1",
    "origQty": "1.0",
    "executedQty": "0.0",
    "cummulativeQuoteQty": "0.0",
    "status": "NEW",
    "timeInForce": "GTC",
    "type": "LIMIT",
    "side": "BUY",
    "stopPrice": "0.0",
    "icebergQty": "0.0",
    "time": 1499827319559,
    "updateTime": 1499827319559,
    "isWorking": true
  }
]
GET /api/v3/openOrders (HMAC SHA256)

Get all open orders on a symbol. Careful when accessing this with no symbol.

Weight: 1 for a single symbol; 40 when the symbol parameter is omitted

Parameters:

Name	Type	Mandatory	Description
symbol	STRING	NO	
recvWindow	LONG	NO	The value cannot be greater than 60000
timestamp	LONG	YES	
If the symbol is not sent, orders for all symbols will be returned in an array.
All Orders (USER_DATA)
Response:

[
  {
    "symbol": "LTCBTC",
    "orderId": 1,
    "orderListId": -1, //Unless OCO, the value will always be -1
    "clientOrderId": "myOrder1",
    "price": "0.1",
    "origQty": "1.0",
    "executedQty": "0.0",
    "cummulativeQuoteQty": "0.0",
    "status": "NEW",
    "timeInForce": "GTC",
    "type": "LIMIT",
    "side": "BUY",
    "stopPrice": "0.0",
    "icebergQty": "0.0",
    "time": 1499827319559,
    "updateTime": 1499827319559,
    "isWorking": true
  }
]
GET /api/v3/allOrders (HMAC SHA256)

Get all account orders; active, canceled, or filled.

Weight: 5 with symbol

Parameters:

Name	Type	Mandatory	Description
symbol	STRING	YES	
orderId	LONG	NO	
startTime	LONG	NO	
endTime	LONG	NO	
limit	INT	NO	Default 500; max 1000.
recvWindow	LONG	NO	The value cannot be greater than 60000
timestamp	LONG	YES	
Notes:

If orderId is set, it will get orders >= that orderId. Otherwise most recent orders are returned.
For some historical orders cummulativeQuoteQty will be < 0, meaning the data is not available at this time.
New OCO (TRADE)
Response:

{
  "orderListId": 0,
  "contingencyType": "OCO",
  "listStatusType": "EXEC_STARTED",
  "listOrderStatus": "EXECUTING",
  "listClientOrderId": "JYVpp3F0f5CAG15DhtrqLp",
  "transactionTime": 1563417480525,
  "symbol": "LTCBTC",
  "orders": [
    {
      "symbol": "LTCBTC",
      "orderId": 2,
      "clientOrderId": "Kk7sqHb9J6mJWTMDVW7Vos"
    },
    {
      "symbol": "LTCBTC",
      "orderId": 3,
      "clientOrderId": "xTXKaGYd4bluPVp78IVRvl"
    }
  ],
  "orderReports": [
    {
      "symbol": "LTCBTC",
      "orderId": 2,
      "orderListId": 0,
      "clientOrderId": "Kk7sqHb9J6mJWTMDVW7Vos",
      "transactTime": 1563417480525,
      "price": "0.000000",
      "origQty": "0.624363",
      "executedQty": "0.000000",
      "cummulativeQuoteQty": "0.000000",
      "status": "NEW",
      "timeInForce": "GTC",
      "type": "STOP_LOSS",
      "side": "BUY",
      "stopPrice": "0.960664"
    },
    {
      "symbol": "LTCBTC",
      "orderId": 3,
      "orderListId": 0,
      "clientOrderId": "xTXKaGYd4bluPVp78IVRvl",
      "transactTime": 1563417480525,
      "price": "0.036435",
      "origQty": "0.624363",
      "executedQty": "0.000000",
      "cummulativeQuoteQty": "0.000000",
      "status": "NEW",
      "timeInForce": "GTC",
      "type": "LIMIT_MAKER",
      "side": "BUY"
    }
  ]
}
POST /api/v3/order/oco (HMAC SHA256)

Weight: 1

Send in a new OCO

Parameters:

Name	Type	Mandatory	Descirption
symbol	STRING	YES	
listClientOrderId	STRING	NO	A unique Id for the entire orderList
side	ENUM	YES	
quantity	DECIMAL	YES	
limitClientOrderId	STRING	NO	A unique Id for the limit order
price	DECIMAL	YES	
limitIcebergQty	DECIMAL	NO	
stopClientOrderId	STRING	NO	A unique Id for the stop loss/stop loss limit leg
stopPrice	DECIMAL	YES	
stopLimitPrice	DECIMAL	NO	
stopIcebergQty	DECIMAL	NO	
stopLimitTimeInForce	ENUM	NO	Valid values are GTC/FOK/IOC
newOrderRespType	ENUM	NO	Set the response JSON.
recvWindow	LONG	NO	The value cannot be greater than 60000
timestamp	LONG	YES	
Other Info: * Price Restrictions: * SELL: Limit Price > Last Price > Stop Price * BUY: Limit Price < Last Price < Stop Price * Quantity Restrictions: * Both legs must have the same quantity * ICEBERG quantities however do not have to be the same

Cancel OCO (TRADE)
DELETE /api/v3/orderList (HMAC SHA256)

Weight: 1

Cancel an entire Order List

Parameters:

Name	Type	Mandatory	Description
symbol	STRING	YES	
orderListId	LONG	NO	Either orderListId or listClientOrderId must be provided
listClientOrderId	STRING	NO	Either orderListId or listClientOrderId must be provided
newClientOrderId	STRING	NO	Used to uniquely identify this cancel. Automatically generated by default
recvWindow	LONG	NO	The value cannot be greater than 60000
timestamp	LONG	YES	
Additional notes:

Cancelling an individual leg will cancel the entire OCO
Query OCO (USER_DATA)
Response:

{
    "orderListId": 27,
    "contingencyType": "OCO",
    "listStatusType": "EXEC_STARTED",
    "listOrderStatus": "EXECUTING",
    "listClientOrderId": "h2USkA5YQpaXHPIrkd96xE",
    "transactionTime": 1565245656253,
    "symbol": "LTCBTC",
    "orders": [
        {
            "symbol": "LTCBTC",
            "orderId": 4,
            "clientOrderId": "qD1gy3kc3Gx0rihm9Y3xwS"
        },
        {
            "symbol": "LTCBTC",
            "orderId": 5,
            "clientOrderId": "ARzZ9I00CPM8i3NhmU9Ega"
        }
    ]
}
GET /api/v3/orderList (HMAC SHA256)

Weight: 1

Retrieves a specific OCO based on provided optional parameters

Parameters:

Name	Type	Mandatory	Description
orderListId	LONG	NO	Either orderListId or listClientOrderId must be provided
origClientOrderId	STRING	NO	Either orderListId or listClientOrderId must be provided
recvWindow	LONG	NO	The value cannot be greater than 60000
timestamp	LONG	YES	
Query all OCO (USER_DATA)
Response:

[
    {
        "orderListId": 29,
        "contingencyType": "OCO",
        "listStatusType": "EXEC_STARTED",
        "listOrderStatus": "EXECUTING",
        "listClientOrderId": "amEEAXryFzFwYF1FeRpUoZ",
        "transactionTime": 1565245913483,
        "symbol": "LTCBTC",
        "orders": [
            {
                "symbol": "LTCBTC",
                "orderId": 4,
                "clientOrderId": "oD7aesZqjEGlZrbtRpy5zB"
            },
            {
                "symbol": "LTCBTC",
                "orderId": 5,
                "clientOrderId": "Jr1h6xirOxgeJOUuYQS7V3"
            }
        ]
    },
    {
        "orderListId": 28,
        "contingencyType": "OCO",
        "listStatusType": "EXEC_STARTED",
        "listOrderStatus": "EXECUTING",
        "listClientOrderId": "hG7hFNxJV6cZy3Ze4AUT4d",
        "transactionTime": 1565245913407,
        "symbol": "LTCBTC",
        "orders": [
            {
                "symbol": "LTCBTC",
                "orderId": 2,
                "clientOrderId": "j6lFOfbmFMRjTYA7rRJ0LP"
            },
            {
                "symbol": "LTCBTC",
                "orderId": 3,
                "clientOrderId": "z0KCjOdditiLS5ekAFtK81"
            }
        ]
    }
]
GET /api/v3/allOrderList (HMAC SHA256)

Weight: 10

Retrieves all OCO based on provided optional parameters

Parameters

Name	Type	Mandatory	Description
fromId	LONG	NO	If supplied, neither startTime or endTime can be provided
startTime	LONG	NO	
endTime	LONG	NO	
limit	INT	NO	Default Value: 500; Max Value: 1000
recvWindow	LONG	NO	The value cannot be greater than 60000
timestamp	LONG	YES	
Query Open OCO (USER_DATA)
Response:

[
    {
        "orderListId": 31,
        "contingencyType": "OCO",
        "listStatusType": "EXEC_STARTED",
        "listOrderStatus": "EXECUTING",
        "listClientOrderId": "wuB13fmulKj3YjdqWEcsnp",
        "transactionTime": 1565246080644,
        "symbol": "1565246079109",
        "orders": [
            {
                "symbol": "LTCBTC",
                "orderId": 4,
                "clientOrderId": "r3EH2N76dHfLoSZWIUw1bT"
            },
            {
                "symbol": "LTCBTC",
                "orderId": 5,
                "clientOrderId": "Cv1SnyPD3qhqpbjpYEHbd2"
            }
        ]
    }
]
GET /api/v3/openOrderList (HMAC SHA256)

Weight: 2

Parameters

Name	Type	Mandatory	Description
recvWindow	LONG	NO	The value cannot be greater than 60000
timestamp	LONG	YES	
Account Information (USER_DATA)
Response:

{
  "makerCommission": 15,
  "takerCommission": 15,
  "buyerCommission": 0,
  "sellerCommission": 0,
  "canTrade": true,
  "canWithdraw": true,
  "canDeposit": true,
  "updateTime": 123456789,
  "balances": [
    {
      "asset": "BTC",
      "free": "4723846.89208129",
      "locked": "0.00000000"
    },
    {
      "asset": "LTC",
      "free": "4763368.68006011",
      "locked": "0.00000000"
    }
  ]
}
GET /api/v3/account (HMAC SHA256)

Get current account information.

Weight: 5

Parameters:

Name	Type	Mandatory	Description
recvWindow	LONG	NO	The value cannot be greater than 60000
timestamp	LONG	YES	
Account Trade List (USER_DATA)
Response:

[
  {
    "symbol": "BNBBTC",
    "id": 28457,
    "orderId": 100234,
    "orderListId": -1, //Unless OCO, the value will always be -1
    "price": "4.00000100",
    "qty": "12.00000000",
    "quoteQty": "48.000012",
    "commission": "10.10000000",
    "commissionAsset": "BNB",
    "time": 1499865549590,
    "isBuyer": true,
    "isMaker": false,
    "isBestMatch": true
  }
]
GET /api/v3/myTrades (HMAC SHA256)

Get trades for a specific account and symbol.

Weight: 5 with symbol

Parameters:

Name	Type	Mandatory	Description
symbol	STRING	YES	
startTime	LONG	NO	
endTime	LONG	NO	
fromId	LONG	NO	TradeId to fetch from. Default gets most recent trades.
limit	INT	NO	Default 500; max 1000.
recvWindow	LONG	NO	The value cannot be greater than 60000
timestamp	LONG	YES	
Notes: * If fromId is set, it will get orders >= that fromId. Otherwise most recent orders are returned.

Margin Account/Trade
Margin Account Transfer (MARGIN)
Response:

{
    //transaction id
    "tranId": 100000001
}
Post /sapi/v1/margin/transfer Execute transfer between spot account and margin account.

Weight: 1

Parameters:

Name	Type	Mandatory	Description
asset	STRING	YES	The asset being transferred, e.g., BTC
amount	DECIMAL	YES	The amount to be transferred
type	INT	YES	1: transfer from main account to margin account 2: transfer from margin account to main account
recvWindow	LONG	NO	The value cannot be greater than 60000
timestamp	LONG	YES	
Margin Account Borrow (MARGIN)
Response:

{
    //transaction id
    "tranId": 100000001
}
Post /sapi/v1/margin/loan Apply for a loan.

Weight: 1

Parameters:

Name	Type	Mandatory	Description
asset	STRING	YES	
amount	DECIMAL	YES	
recvWindow	LONG	NO	The value cannot be greater than 60000
timestamp	LONG	YES	
Margin Account repay (MARGIN)
Response:

{
    //transaction id
    "tranId": 100000001
}
Post /sapi/v1/margin/repay Repay loan for margin account.

Weight: 1

Parameters:

Name	Type	Mandatory	Description
asset	STRING	YES	
amount	DECIMAL	YES	
recvWindow	LONG	NO	The value cannot be greater than 60000
timestamp	LONG	YES	
Query Margin Asset (MARKET_DATA)
Response:

{
      "assetFullName": "Binance Coin",
      "assetName": "BNB",
      "isBorrowable": false,
      "isMortgageable": true,
      "userMinBorrow": "0.00000000",
      "userMinRepay": "0.00000000"
}
Get /sapi/v1/margin/asset

Weight: 5

Parameters:

Name	Type	Mandatory	Description
asset	STRING	YES	
X-MBX-APIY required
Query Margin Pair (MARKET_DATA)
Response:

{
   "id":323355778339572400,
   "symbol":"BTCUSDT",
   "base":"BTC",
   "quote":"USDT",
   "isMarginTrade":true,
   "isBuyAllowed":true,
   "isSellAllowed":true
}
Get /sapi/v1/margin/pair

Weight: 5

Parameters:

Name	Type	Mandatory	Description
symbol	STRING	YES	
X-MBX-APIY required
Get All Margin Assets (MARKET_DATA)
Response:

  [
      {
          "assetFullName": "USD coin",
          "assetName": "USDC",
          "isBorrowable": true,
          "isMortgageable": true,
          "userMinBorrow": "0.00000000",
          "userMinRepay": "0.00000000"
      },
      {
          "assetFullName": "BNB-coin",
          "assetName": "BNB",
          "isBorrowable": true,
          "isMortgageable": true,
          "userMinBorrow": "1.00000000",
          "userMinRepay": "0.00000000"
      },
      {
          "assetFullName": "Tether",
          "assetName": "USDT",
          "isBorrowable": true,
          "isMortgageable": true,
          "userMinBorrow": "1.00000000",
          "userMinRepay": "0.00000000"
      },
      {
          "assetFullName": "etherum",
          "assetName": "ETH",
          "isBorrowable": true,
          "isMortgageable": true,
          "userMinBorrow": "0.00000000",
          "userMinRepay": "0.00000000"
      },
      {
          "assetFullName": "Bitcoin",
          "assetName": "BTC",
          "isBorrowable": true,
          "isMortgageable": true,
          "userMinBorrow": "0.00000000",
          "userMinRepay": "0.00000000"
      }
  ]
Get /sapi/v1/margin/allAssets

Weight: 5

Parameters:

None

X-MBX-APIY required
Get All Margin Pairs (MARKET_DATA)
Response:

[
    {
        "base": "BNB",
        "id": 351637150141315861,
        "isBuyAllowed": True,
        "isMarginTrade": True,
        "isSellAllowed": True,
        "quote": "BTC",
        "symbol": "BNBBTC"
    },
    {
        "base": "TRX",
        "id": 351637923235429141,
        "isBuyAllowed": True,
        "isMarginTrade": True,
        "isSellAllowed": True,
        "quote": "BTC",
        "symbol": "TRXBTC"
    },
    {
        "base": "XRP",
        "id": 351638112213990165,
        "isBuyAllowed": True,
        "isMarginTrade": True,
        "isSellAllowed": True,
        "quote": "BTC",
        "symbol": "XRPBTC"
    },
    {
        "base": "ETH",
        "id": 351638524530850581,
        "isBuyAllowed": True,
        "isMarginTrade": True,
        "isSellAllowed": True,
        "quote": "BTC",
        "symbol": "ETHBTC"
    },
    {
        "base": "BNB",
        "id": 376870400832855109,
        "isBuyAllowed": True,
        "isMarginTrade": True,
        "isSellAllowed": True,
        "quote": "USDT",
        "symbol": "BNBUSDT"
  },
]
Get /sapi/v1/margin/allPairs

Weight: 5

Parameters:

None

X-MBX-APIY required
Query Margin PriceIndex (MARKET_DATA)
Response:

{
   "calcTime": 1562046418000,
   "price": "0.00333930",
   "symbol": "BNBBTC"
}
Get /sapi/v1/margin/priceIndex

Weight: 5

Parameters:

Name	Type	Mandatory	Description
symbol	STRING	YES	
X-MBX-APIY required
Margin Account New Order (TRADE)
Response ACK:

{
  "symbol": "BTCUSDT",
  "orderId": 28,
  "clientOrderId": "6gCrw2kRUAF9CvJDGP16IP",
  "transactTime": 1507725176595
}
Response RESULT:

{
  "symbol": "BTCUSDT",
  "orderId": 28,
  "clientOrderId": "6gCrw2kRUAF9CvJDGP16IP",
  "transactTime": 1507725176595,
  "price": "1.00000000",
  "origQty": "10.00000000",
  "executedQty": "10.00000000",
  "cummulativeQuoteQty": "10.00000000",
  "status": "FILLED",
  "timeInForce": "GTC",
  "type": "MARKET",
  "side": "SELL"
}
Response FULL:

{
  "symbol": "BTCUSDT",
  "orderId": 28,
  "clientOrderId": "6gCrw2kRUAF9CvJDGP16IP",
  "transactTime": 1507725176595,
  "price": "1.00000000",
  "origQty": "10.00000000",
  "executedQty": "10.00000000",
  "cummulativeQuoteQty": "10.00000000",
  "status": "FILLED",
  "timeInForce": "GTC",
  "type": "MARKET",
  "side": "SELL",
  "fills": [
    {
      "price": "4000.00000000",
      "qty": "1.00000000",
      "commission": "4.00000000",
      "commissionAsset": "USDT"
    },
    {
      "price": "3999.00000000",
      "qty": "5.00000000",
      "commission": "19.99500000",
      "commissionAsset": "USDT"
    },
    {
      "price": "3998.00000000",
      "qty": "2.00000000",
      "commission": "7.99600000",
      "commissionAsset": "USDT"
    },
    {
      "price": "3997.00000000",
      "qty": "1.00000000",
      "commission": "3.99700000",
      "commissionAsset": "USDT"
    },
    {
      "price": "3995.00000000",
      "qty": "1.00000000",
      "commission": "3.99500000",
      "commissionAsset": "USDT"
    }
  ]
}
Post /sapi/v1/margin/order Post a new order for margin account.

Weight: 1

Parameters:

Name	Type	Mandatory	Description
symbol	STRING	YES	
side	ENUM	YES	BUY
SELL
type	ENUM	YES	
quantity	DECIMAL	YES	
price	DECIMAL	NO	
stopPrice	DECIMAL	NO	Used with STOP_LOSS, STOP_LOSS_LIMIT, TAKE_PROFIT, and TAKE_PROFIT_LIMIT orders.
newClientOrderId	STRING	NO	A unique id for the order. Automatically generated if not sent.
icebergQty	DECIMAL	NO	Used with LIMIT, STOP_LOSS_LIMIT, and TAKE_PROFIT_LIMIT to create an iceberg order.
newOrderRespType	ENUM	NO	Set the response JSON. ACK, RESULT, or FULL; MARKET and LIMIT order types default to FULL, all other orders default to ACK.
timeInForce	ENUM	NO	GTC,IOC,FOK
recvWindow	LONG	NO	The value cannot be greater than 60000
timestamp	LONG	YES	
Margin Account Cancel Order (TRADE)
Response:

{
  "symbol": "LTCBTC",
  "orderId": 28,
  "origClientOrderId": "myOrder1",
  "clientOrderId": "cancelMyOrder1",
  "transactTime": 1507725176595,
  "price": "1.00000000",
  "origQty": "10.00000000",
  "executedQty": "8.00000000",
  "cummulativeQuoteQty": "8.00000000",
  "status": "CANCELED",
  "timeInForce": "GTC",
  "type": "LIMIT",
  "side": "SELL"
}
Delete /sapi/v1/margin/order Cancel an active order for margin account.

Weight: 1

Parameters:

Name	Type	Mandatory	Description
symbol	STRING	YES	
orderId	LONG	NO	
origClientOrderId	STRING	NO	
newClientOrderId	STRING	NO	Used to uniquely identify this cancel. Automatically generated by default.
recvWindow	LONG	NO	The value cannot be greater than 60000
timestamp	LONG	YES	
Either orderId or origClientOrderId must be sent.

Get Transfer History (USER_DATA)
Response:

{
    "rows": [
    {
        "amount: "0.10000000",
        "asset": "BNB",
        "status": "CONFIRMED",
        "timestamp": 1566898617,
        "txId": 5240372201,
        "type": "ROLL_IN"
    },
    {
        "amount": "5.00000000",
        "asset": "USDT",
        "status": "CONFIRMED",
        "timestamp": 1566888436,
        "txId": 5239810406,
        "type": "ROLL_OUT"
    },
    {
        "amount": "1.00000000",
        "asset": "EOS,
        "status": "CONFIRMED",
        "timestamp": 1566888403,
        "txId": 5239808703,
        "type": "ROLL_IN"
    }
    "total": 3
} 
Get /sapi/v1/margin/transfer

Weight:s 5

Parameters:

Name	Type	Mandatory	Description
asset	STRING	No	
type	STRING	YES	Tranfer Type: ROLL_IN, ROLL_OUT
startTime	LONG	NO	
endTime	LONG	NO	
current	LONG	NO	Currently querying page. Start from 1. Default:1
size	LONG	NO	Default:10 Max:100
recvWindow	LONG	NO	The value cannot be greater than 60000
timestamp	LONG	YES	
Response in descending order
Query Loan Record (USER_DATA)
Response:

{
  "rows": [
    {
      "asset": "BNB",
      "principal": "0.84624403",
      "timestamp": 1555056425000,
      //one of PENDING (pending to execution), CONFIRMED (successfully loaned), FAILED (execution failed, nothing happened to your account);
      "status": "CONFIRMED"
    }
  ],
  "total": 1
}
Get /sapi/v1/margin/loan

Weight:s 5

Parameters:

Name	Type	Mandatory	Description
asset	STRING	YES	
txId	LONG	NO	the tranId in POST /sapi/v1/margin/loan
startTime	LONG	NO	
endTime	LONG	NO	
current	LONG	NO	Currently querying page. Start from 1. Default:1
size	LONG	NO	Default:10 Max:100
recvWindow	LONG	NO	The value cannot be greater than 60000
timestamp	LONG	YES	
txId or startTime must be sent. txId takes precedence.

Response in descending order

Query Repay Record (USER_DATA)
Response:

{
     "rows": [
         {
             //Total amount repaid
             "amount": "14.00000000",
             "asset": "BNB",
             //Interest repaid
             "interest": "0.01866667",
             //Principal repaid
             "principal": "13.98133333",
             //one of PENDING (pending to execution), CONFIRMED (successfully loaned), FAILED (execution failed, nothing happened to your account);
             "status": "CONFIRMED",
             "timestamp": 1563438204000,
             "txId": 2970933056
         }
     ],
     "total": 1
}
Get /sapi/v1/margin/repay

Weight: 5

Parameters:

Name	Type	Mandatory	Description
asset	STRING	YES	
txId	LONG	NO	return of /sapi/v1/margin/repay
startTime	LONG	NO	
endTime	LONG	NO	
current	LONG	NO	Currently querying page. Start from 1. Default:1
size	LONG	NO	Default:10 Max:100
recvWindow	LONG	NO	The value cannot be greater than 60000
timestamp	LONG	YES	
txId or startTime must be sent. txId takes precedence.

Response in descending order

Get Interest History (USER_DATA)
Response:

  {
      "rows": [
          {
              "asset": "BNB",
              "interest": "0.02414667",
              "interestAccuredTime": 1566813600,
              "interestRate": "0.01600000",
              "principal": "36.22000000",
              "type": "ON_BORROW"
          },
          {
              "asset": "BNB",
              "interest": "0.02019334",
              "interestAccuredTime": 1566813600,
              "interestRate": "0.01600000",
              "principal": "30.29000000",
              "type": "ON_BORROW"
          },
          {
              "asset": "BNB",
              "interest": "0.02098667",
              "interestAccuredTime": 1566813600,
              "interestRate": "0.01600000",
              "principal": "31.48000000",
              "type": "ON_BORROW"
          },
          {
              "asset": "BNB",
              "interest": "0.02483334",
              "interestAccuredTime": 1566806400,
              "interestRate": "0.01600000",
              "principal": "37.25000000",
              "type": "ON_BORROW"
          },
          {
              "asset": "BNB",
              "interest": "0.02165334",
              "interestAccuredTime": 1566806400,
              "interestRate": "0.01600000",
              "principal": "32.48000000",
              "type": "ON_BORROW"
          }
      ],
      "total": 5
  }
Get /sapi/v1/margin/interestHistory

Weight:s 5

Parameters:

Name	Type	Mandatory	Description
asset	STRING	No	
startTime	LONG	NO	
endTime	LONG	NO	
current	LONG	NO	Currently querying page. Start from 1. Default:1
size	LONG	NO	Default:10 Max:100
recvWindow	LONG	NO	The value cannot be greater than 60000
timestamp	LONG	YES	
Response in descending order
Get Force Liquidation Record (USER_DATA)
Response:

  {
      "rows": [
          {
              "avgPrice": "0.00388359",
              "executedQty": "31.39000000",
              "orderId": 180015097,
              "price": "0.00388110",
              "qty": "31.39000000",
              "side": "SELL",
              "symbol": "BNBBTC",
              "timeInForce": "GTC",
              "updatedTime": 1558941374745
          }
      ],
      "total": 1
  }
Get /sapi/v1/margin/forceLiquidationRec

Weight:s 5

Parameters:

Name	Type	Mandatory	Description
startTime	LONG	NO	
endTime	LONG	NO	
current	LONG	NO	Currently querying page. Start from 1. Default:1
size	LONG	NO	Default:10 Max:100
recvWindow	LONG	NO	The value cannot be greater than 60000
timestamp	LONG	YES	
Response in descending order
Query Margin Account Details (USER_DATA)
Response:

{
      "borrowEnabled": true,
      "marginLevel": "11.64405625",
      "totalAssetOfBtc": "6.82728457",
      "totalLiabilityOfBtc": "0.58633215",
      "totalNetAssetOfBtc": "6.24095242",
      "tradeEnabled": true,
      "transferEnabled": true,
      "userAssets": [
          {
              "asset": "BTC",
              "borrowed": "0.00000000",
              "free": "0.00499500",
              "interest": "0.00000000",
              "locked": "0.00000000",
              "netAsset": "0.00499500"
          },
          {
              "asset": "BNB",
              "borrowed": "201.66666672",
              "free": "2346.50000000",
              "interest": "0.00000000",
              "locked": "0.00000000",
              "netAsset": "2144.83333328"
          },
          {
              "asset": "ETH",
              "borrowed": "0.00000000",
              "free": "0.00000000",
              "interest": "0.00000000",
              "locked": "0.00000000",
              "netAsset": "0.00000000"
          },
          {
              "asset": "USDT",
              "borrowed": "0.00000000",
              "free": "0.00000000",
              "interest": "0.00000000",
              "locked": "0.00000000",
              "netAsset": "0.00000000"
          }
      ]
}
Get /sapi/v1/margin/account

Weight: 5

Parameters:

Name	Type	Mandatory	Description
recvWindow	LONG	NO	The value cannot be greater than 60000
timestamp	LONG	YES	
Query Margin Account's Order (USER_DATA)
Response:

{
   "clientOrderId": "ZwfQzuDIGpceVhKW5DvCmO",
   "cummulativeQuoteQty": "0.00000000",
   "executedQty": "0.00000000",
   "icebergQty": "0.00000000",
   "isWorking": true,
   "orderId": 213205622,
   "origQty": "0.30000000",
   "price": "0.00493630",
   "side": "SELL",
   "status": "NEW",
   "stopPrice": "0.00000000",
   "symbol": "BNBBTC",
   "time": 1562133008725,
   "timeInForce": "GTC",
   "type": "LIMIT",
   "updateTime": 1562133008725
}
Get /sapi/v1/margin/order

Weight: 5

Parameters:

Name	Type	Mandatory	Description
symbol	STRING	YES	
orderId	STRING	NO	
origClientOrderId	STRING	NO	
recvWindow	LONG	NO	The value cannot be greater than 60000
timestamp	LONG	YES	
Either orderId or origClientOrderId must be sent.
For some historical orders cummulativeQuoteQty will be < 0, meaning the data is not available at this time.
Query Margin Account's Open Order (USER_DATA)
Response:

[
   {
       "clientOrderId": "qhcZw71gAkCCTv0t0k8LUK",
       "cummulativeQuoteQty": "0.00000000",
       "executedQty": "0.00000000",
       "icebergQty": "0.00000000",
       "isWorking": true,
       "orderId": 211842552,
       "origQty": "0.30000000",
       "price": "0.00475010",
       "side": "SELL",
       "status": "NEW",
       "stopPrice": "0.00000000",
       "symbol": "BNBBTC",
       "time": 1562040170089,
       "timeInForce": "GTC",
       "type": "LIMIT",
       "updateTime": 1562040170089
      }
]
Get /sapi/v1/margin/openOrders

Weight: 10

Parameters:

Name	Type	Mandatory	Description
symbol	STRING	NO	
recvWindow	LONG	NO	The value cannot be greater than 60000
timestamp	LONG	YES	
If the symbol is not sent, orders for all symbols will be returned in an array.
When all symbols are returned, the number of requests counted against the rate limiter is equal to the number of symbols currently trading on the exchange.
Query Margin Account's All Order (USER_DATA)
Response:

[
      {
          "clientOrderId": "D2KDy4DIeS56PvkM13f8cP",
          "cummulativeQuoteQty": "0.00000000",
          "executedQty": "0.00000000",
          "icebergQty": "0.00000000",
          "isWorking": false,
          "orderId": 41295,
          "origQty": "5.31000000",
          "price": "0.22500000",
          "side": "SELL",
          "status": "CANCELED",
          "stopPrice": "0.18000000",
          "symbol": "BNBBTC",
          "time": 1565769338806,
          "timeInForce": "GTC",
          "type": "TAKE_PROFIT_LIMIT",
          "updateTime": 1565769342148
      },
      {
          "clientOrderId": "gXYtqhcEAs2Rn9SUD9nRKx",
          "cummulativeQuoteQty": "0.00000000",
          "executedQty": "0.00000000",
          "icebergQty": "1.00000000",
          "isWorking": true,
          "orderId": 41296,
          "origQty": "6.65000000",
          "price": "0.18000000",
          "side": "SELL",
          "status": "CANCELED",
          "stopPrice": "0.00000000",
          "symbol": "BNBBTC",
          "time": 1565769348687,
          "timeInForce": "GTC",
          "type": "LIMIT",
          "updateTime": 1565769352226
      },
      {
          "clientOrderId": "duDq1BqohhcMmdMs9FSuDy",
          "cummulativeQuoteQty": "0.39450000",
          "executedQty": "2.63000000",
          "icebergQty": "0.00000000",
          "isWorking": true,
          "orderId": 41297,
          "origQty": "2.63000000",
          "price": "0.00000000",
          "side": "SELL",
          "status": "FILLED",
          "stopPrice": "0.00000000",
          "symbol": "BNBBTC",
          "time": 1565769358139,
          "timeInForce": "GTC",
          "type": "MARKET",
          "updateTime": 1565769358139
      }

]
Get /sapi/v1/margin/allOrders

Weight: 5

Parameters:

Name	Type	Mandatory	Description
symbol	STRING	YES	
orderId	LONG	NO	
startTime	LONG	NO	
endTime	LONG	NO	
limit	INT	NO	Default 500; max 1000.
recvWindow	LONG	NO	The value cannot be greater than 60000
timestamp	LONG	YES	
If orderId is set, it will get orders >= that orderId. Otherwise most recent orders are returned.
For some historical orders cummulativeQuoteQty will be < 0, meaning the data is not available at this time.
Query Margin Account's Trade List (USER_DATA)
Response:

[
    {
        "commission": "0.00006000",
        "commissionAsset": "BTC",
        "id": 34,
        "isBestMatch": true,
        "isBuyer": false,
        "isMaker": false,
        "orderId": 39324,
        "price": "0.02000000",
        "qty": "3.00000000",
        "symbol": "BNBBTC",
        "time": 1561973357171
    }
]
Get /sapi/v1/margin/myTrades

Weight: 5

Parameters:

Name	Type	Mandatory	Description
symbol	STRING	YES	
startTime	LONG	NO	
endTime	LONG	NO	
fromId	LONG	NO	TradeId to fetch from. Default gets most recent trades.
limit	INT	NO	Default 500; max 1000.
recvWindow	LONG	NO	The value cannot be greater than 60000
timestamp	LONG	YES	
If fromId is set, it will get orders >= that fromId. Otherwise most recent orders are returned.
Query Max Borrow (USER_DATA)
Response:

{
    "amount": "1.69248805"
}
Get /sapi/v1/margin/maxBorrowable

Weight: 5

Parameters:

Name	Type	Mandatory	Description
asset	STRING	YES	
recvWindow	LONG	NO	The value cannot be greater than 60000
timestamp	LONG	YES	
Query Max Transfer-Out Amount (USER_DATA)
Response:

 {
      "amount": "3.59498107"
 }
Get /sapi/v1/margin/maxTransferable

Weight: 5

Parameters:

Name	Type	Mandatory	Description
asset	STRING	YES	
recvWindow	LONG	NO	The value cannot be greater than 60000
timestamp	LONG	YES	
User Data Streams
The base API endpoint is: https://api.binance.com
A User Data Stream listenKey is valid for 60 minutes after creation.
Doing a PUT on a listenKey will extend its validity for 60 minutes.
Doing a DELETE on a listenKey will close the stream.
The base websocket endpoint is: wss://stream.binance.com:9443
User Data Streams are accessed at /ws/<listenKey>
A single connection to stream.binance.com is only valid for 24 hours; expect to be disconnected at the 24 hour mark
User data stream payloads are not guaranteed to be in order during heavy periods; make sure to order your updates using E
LISTEN KEY (SPOT)
Create a ListenKey
Response:

{
  "listenKey": "pqia91ma19a5s61cv6a81va65sdf19v8a65a1a5s61cv6a81va65sdf19v8a65a1"
}
POST /api/v1/userDataStream

Start a new user data stream. The stream will close after 60 minutes unless a keepalive is sent.

Weight: 1

Parameters:

Name	Type	Mandatory	Description
recvWindow	LONG	NO	The value cannot be greater than 60000
timestamp	LONG	YES	
Ping/Keep-alive a ListenKey
Response:

{}
PUT /api/v1/userDataStream

Keepalive a user data stream to prevent a time out. User data streams will close after 60 minutes. It's recommended to send a ping about every 30 minutes.

Weight: 1

Parameters:

Name	Type	Mandatory	Description
listenKey	STRING	YES	
Close a ListenKey
Response:

{}
DELETE /api/v1/userDataStream

Close out a user data stream.

Weight: 1

Parameters:

Name	Type	Mandatory	Description
listenKey	STRING	YES	
LISTEN KEY (MARGIN)
Create a ListenKey
Response:

{"listenKey":  "T3ee22BIYuWqmvne0HNq2A2WsFlEtLhvWCtItw6ffhhdmjifQ2tRbuKkTHhr"}
POST /sapi/v1/userDataStream

Weight: 1

Parameters:

Name	Type	Mandatory	Description
recvWindow	LONG	NO	The value cannot be greater than 60000
timestamp	LONG	YES	
Ping/Keep-alive a ListenKey
Response:

{}
PUT /sapi/v1/userDataStream

Weight: 1

Parameters:

Name	Type	Mandatory	Description
listenKey	STRING	YES	
Close a ListenKey
Response:

{}
DELETE /sapi/v1/userDataStream

Weight: 1

Parameters:

Name	Type	Mandatory	Description
listenKey	STRING	YES	
Payload: Account Update
Account state is updated with the outboundAccountInfo event.

Payload:

{
  "e": "outboundAccountInfo",   // Event type
  "E": 1499405658849,           // Event time
  "m": 0,                       // Maker commission rate (bips)
  "t": 0,                       // Taker commission rate (bips)
  "b": 0,                       // Buyer commission rate (bips)
  "s": 0,                       // Seller commission rate (bips)
  "T": true,                    // Can trade?
  "W": true,                    // Can withdraw?
  "D": true,                    // Can deposit?
  "u": 1499405658848,           // Time of last account update
  "B": [                        // Balances array
    {
      "a": "LTC",               // Asset
      "f": "17366.18538083",    // Free amount
      "l": "0.00000000"         // Locked amount
    },
    {
      "a": "BTC",
      "f": "10537.85314051",
      "l": "2.19464093"
    },
    {
      "a": "ETH",
      "f": "17902.35190619",
      "l": "0.00000000"
    },
    {
      "a": "BNC",
      "f": "1114503.29769312",
      "l": "0.00000000"
    },
    {
      "a": "NEO",
      "f": "0.00000000",
      "l": "0.00000000"
    }
  ]
}
An additional event outboundAccountPosition is sent any time an account balance has changed and contains the assets that were possibly changed by the event that generated the balance change.

Payload:

{
  "e": "outboundAccountPosition", //Event type
  "E": 1564034571105,             //Event Time
  "u": 1564034571073,             //Time of last account update
  "B": [                          //Balances Array
    {
      "a": "ETH",                 //Asset
      "f": "10000.000000",        //Free
      "l": "0.000000"             //Locked
    }
  ]
}
Payload: Order Update
Orders are updated with the executionReport event. Check the API documentation and below for relevant enum definitions. Average price can be found by doing Z divided by z.

Payload:

{
  "e": "executionReport",        // Event type
  "E": 1499405658658,            // Event time
  "s": "ETHBTC",                 // Symbol
  "c": "mUvoqJxFIILMdfAW5iGSOW", // Client order ID
  "S": "BUY",                    // Side
  "o": "LIMIT",                  // Order type
  "f": "GTC",                    // Time in force
  "q": "1.00000000",             // Order quantity
  "p": "0.10264410",             // Order price
  "P": "0.00000000",             // Stop price
  "F": "0.00000000",             // Iceberg quantity
  "g": -1,                       // OrderListId
  "C": "null",                   // Original client order ID; This is the ID of the order being canceled
  "x": "NEW",                    // Current execution type
  "X": "NEW",                    // Current order status
  "r": "NONE",                   // Order reject reason; will be an error code.
  "i": 4293153,                  // Order ID
  "l": "0.00000000",             // Last executed quantity
  "z": "0.00000000",             // Cumulative filled quantity
  "L": "0.00000000",             // Last executed price
  "n": "0",                      // Commission amount
  "N": null,                     // Commission asset
  "T": 1499405658657,            // Transaction time
  "t": -1,                       // Trade ID
  "I": 8641984,                  // Ignore
  "w": true,                     // Is the order working? Stops will have
  "m": false,                    // Is this trade the maker side?
  "M": false,                    // Ignore
  "O": 1499405658657,            // Order creation time
  "Z": "0.00000000",             // Cumulative quote asset transacted quantity
  "Y": "0.00000000"              // Last quote asset transacted quantity (i.e. lastPrice * lastQty)
}
Execution types:

NEW
CANCELED
REPLACED (currently unused)
REJECTED
TRADE
EXPIRED
If the order is an OCO, an event will be displayed named ListStatus in addition to the executionReport event.

Payload

{
  "e": "listStatus",                //Event Type
  "E": 1564035303637,               //Event Time
  "s": "ETHBTC",                    //Symbol
  "g": 2,                           //OrderListId
  "c": "OCO",                       //Contingency Type
  "l": "EXEC_STARTED",              //List Status Type
  "L": "EXECUTING",                 //List Order Status
  "r": "NONE",                      //List Reject Reason
  "C": "F4QN4G8DlFATFlIUQ0cjdD",    //List Client Order ID
  "T": 1564035303625,               //Transaction Time
  "O": [                            //An array of objects
    {
      "s": "ETHBTC",                //Symbol
      "i": 17,                      // orderId
      "c": "AJYsMjErWJesZvqlJCTUgL" //ClientOrderId
    },
    {
      "s": "ETHBTC",
      "i": 18,
      "c": "bfYPSQdLoqAJeNrOr9adzq"
    }
  ]
}
Error Code
The error JSON payload:

{
  "code":-1121,
  "msg":"Invalid symbol."
}
Errors consist of two parts: an error code and a message. Codes are universal, but messages can vary.

10xx - General Server or Network issues
-1000 UNKNOWN
An unknown error occured while processing the request.
-1001 DISCONNECTED
Internal error; unable to process your request. Please try again.
-1002 UNAUTHORIZED
You are not authorized to execute this request.
-1003 TOO_MANY_REQUESTS
Too many requests queued.
Too many requests; please use the websocket for live updates.
Too many requests; current limit is %s requests per minute. Please use the websocket for live updates to avoid polling the API.
Way too many requests; IP banned until %s. Please use the websocket for live updates to avoid bans.
-1006 UNEXPECTED_RESP
An unexpected response was received from the message bus. Execution status unknown.
-1007 TIMEOUT
Timeout waiting for response from backend server. Send status unknown; execution status unknown.
-1014 UNKNOWN_ORDER_COMPOSITION
Unsupported order combination.
-1015 TOO_MANY_ORDERS
Too many new orders.
Too many new orders; current limit is %s orders per %s.
-1016 SERVICE_SHUTTING_DOWN
This service is no longer available.
-1020 UNSUPPORTED_OPERATION
This operation is not supported.
-1021 INVALID_TIMESTAMP
Timestamp for this request is outside of the recvWindow.
Timestamp for this request was 1000ms ahead of the server's time.
-1022 INVALID_SIGNATURE
Signature for this request is not valid.
-1099 Not found, authenticated, or authorized
This replaces error code -1999
11xx - Request issues
-1100 ILLEGAL_CHARS
Illegal characters found in a parameter.
Illegal characters found in parameter '%s'; legal range is '%s'.
-1101 TOO_MANY_PARAMETERS
Too many parameters sent for this endpoint.
Too many parameters; expected '%s' and received '%s'.
Duplicate values for a parameter detected.
-1102 MANDATORY_PARAM_EMPTY_OR_MALFORMED
A mandatory parameter was not sent, was empty/null, or malformed.
Mandatory parameter '%s' was not sent, was empty/null, or malformed.
Param '%s' or '%s' must be sent, but both were empty/null!
-1103 UNKNOWN_PARAM
An unknown parameter was sent.
-1104 UNREAD_PARAMETERS
Not all sent parameters were read.
Not all sent parameters were read; read '%s' parameter(s) but was sent '%s'.
-1105 PARAM_EMPTY
A parameter was empty.
Parameter '%s' was empty.
-1106 PARAM_NOT_REQUIRED
A parameter was sent when not required.
Parameter '%s' sent when not required.
-1111 BAD_PRECISION
Precision is over the maximum defined for this asset.
-1112 NO_DEPTH
No orders on book for symbol.
-1114 TIF_NOT_REQUIRED
TimeInForce parameter sent when not required.
-1115 INVALID_TIF
Invalid timeInForce.
-1116 INVALID_ORDER_TYPE
Invalid orderType.
-1117 INVALID_SIDE
Invalid side.
-1118 EMPTY_NEW_CL_ORD_ID
New client order ID was empty.
-1119 EMPTY_ORG_CL_ORD_ID
Original client order ID was empty.
-1120 BAD_INTERVAL
Invalid interval.
-1121 BAD_SYMBOL
Invalid symbol.
-1125 INVALID_LISTEN_KEY
This listenKey does not exist.
-1127 MORE_THAN_XX_HOURS
Lookup interval is too big.
More than %s hours between startTime and endTime.
-1128 OPTIONAL_PARAMS_BAD_COMBO
Combination of optional parameters invalid.
-1130 INVALID_PARAMETER
Invalid data sent for a parameter.
Data sent for paramter '%s' is not valid.
-1131 BAD_RECV_WINDOW
recvWindow must be less than 60000
-2010 NEW_ORDER_REJECTED
NEW_ORDER_REJECTED
-2011 CANCEL_REJECTED
CANCEL_REJECTED
-2013 NO_SUCH_ORDER
Order does not exist.
-2014 BAD_API_KEY_FMT
API-key format invalid.
-2015 REJECTED_MBX_KEY
Invalid API-key, IP, or permissions for action.
-2016 NO_TRADING_WINDOW
No trading window could be found for the symbol. Try ticker/24hrs instead.
Messages for -1010 ERROR_MSG_RECEIVED, -2010 NEW_ORDER_REJECTED, and -2011 CANCEL_REJECTED
This code is sent when an error has been returned by the matching engine. The following messages which will indicate the specific error:

Error message	Description
"Unknown order sent."	The order (by either orderId, clientOrderId, origClientOrderId) could not be found
"Duplicate order sent."	The clientOrderId is already in use
"Market is closed."	The symbol is not trading
"Account has insufficient balance for requested action."	Not enough funds to complete the action
"Market orders are not supported for this symbol."	MARKET is not enabled on the symbol
"Iceberg orders are not supported for this symbol."	icebergQty is not enabled on the symbol
"Stop loss orders are not supported for this symbol."	STOP_LOSS is not enabled on the symbol
"Stop loss limit orders are not supported for this symbol."	STOP_LOSS_LIMIT is not enabled on the symbol
"Take profit orders are not supported for this symbol."	TAKE_PROFIT is not enabled on the symbol
"Take profit limit orders are not supported for this symbol."	TAKE_PROFIT_LIMIT is not enabled on the symbol
"Price * QTY is zero or less."	price * quantity is too low
"IcebergQty exceeds QTY."	icebergQty must be less than the order quantity
"This action disabled is on this account."	Contact customer support; some actions have been disabled on the account.
"Unsupported order combination"	The orderType, timeInForce, stopPrice, and/or icebergQty combination isn't allowed.
"Order would trigger immediately."	The order's stop price is not valid when compared to the last traded price.
"Cancel order is invalid. Check origClientOrderId and orderId."	No origClientOrderId or orderId was sent in.
"Order would immediately match and take."	LIMIT_MAKER order type would immediately match and trade, and not be a pure maker order.
-9xxx Filter failures
Error message	Description
"Filter failure: PRICE_FILTER"	price is too high, too low, and/or not following the tick size rule for the symbol.
"Filter failure: PERCENT_PRICE"	price is X% too high or X% too low from the average weighted price over the last Y minutes.
"Filter failure: LOT_SIZE"	quantity is too high, too low, and/or not following the step size rule for the symbol.
"Filter failure: MIN_NOTIONAL"	price * quantity is too low to be a valid order for the symbol.
"Filter failure: ICEBERG_PARTS"	ICEBERG order would break into too many parts; icebergQty is too small.
"Filter failure: MARKET_LOT_SIZE"	MARKET order's quantity is too high, too low, and/or not following the step size rule for the symbol.
"Filter failure: MAX_NUM_ORDERS"	Account has too many open orders on the symbol.
"Filter failure: MAX_ALGO_ORDERS"	Account has too many open stop loss and/or take profit orders on the symbol.
"Filter failure: MAX_NUM_ICEBERG_ORDERS"	Account has too many open iceberg orders on the symbol.
"Filter failure: EXCHANGE_MAX_NUM_ORDERS"	Account has too many open orders on the exchange.
"Filter failure: EXCHANGE_MAX_ALGO_ORDERS"	Account has too many open stop loss and/or take profit orders on the excha


