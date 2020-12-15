
# Public Rest API for WazirX

## General API Information
* The base endpoint is: **https://api.wazirx.com**
* All endpoints return either a JSON object or array.
* All time and timestamp related fields are in **milliseconds**.

## HTTP Return Codes

* HTTP `4XX` return codes are used for malformed requests;
  the issue is on the sender's side.
* HTTP `403` return code is used when the WAF Limit (Web Application Firewall) has been violated.
* HTTP `429` return code is used when breaking a request rate limit.
* HTTP `418` return code is used when an IP has been auto-banned for continuing to send requests after receiving `429` codes.
* HTTP `5XX` return codes are used for internal errors; the issue is on
  WazirX's side.
  It is important to **NOT** treat this as a failure operation; the execution status is
  **UNKNOWN** and could have been a success.


## Error Codes

> Sample Payload below:

```json-doc
{
  "code": -1121,
  "message": "Invalid symbol."
}
```

* Any endpoint can return an ERROR

## General Information on Endpoints
* For `GET` endpoints, parameters must be sent as a `query string`.
* For `POST`, `PUT`, and `DELETE` endpoints, the parameters may be sent as a
  `query string` or in the `request body` with content type
  `application/x-www-form-urlencoded`. You may mix parameters between both the
  `query string` and `request body` if you wish to do so.
* Parameters may be sent in any order.
* If a parameter sent in both the `query string` and `request body`, the
  `query string` parameter will be used.


# Endpoint security type
* Each endpoint has a security type that determines the how you will
  interact with it. This is stated next to the NAME of the endpoint.
    * If no security type is stated, assume the security type is NONE.
* API-keys are passed into the Rest API via the `X-WX-APIKEY`
  header.
* API-keys and secret-keys **are case sensitive**.
* API-keys can be configured to only access certain types of secure endpoints.
 For example, one API-key could be used for TRADE only, while another API-key
 can access everything except for TRADE routes.
* By default, API-keys can access all secure routes.

Security Type | Description
------------ | ------------
NONE | Endpoint can be accessed freely.
TRADE | Endpoint requires sending a valid API-Key and signature.
USER_DATA | Endpoint requires sending a valid API-Key and signature.
MARKET_DATA | Endpoint requires sending a valid API-Key.


* `TRADE` and `USER_DATA` endpoints are `SIGNED` endpoints.

## SIGNED (TRADE and USER_DATA) Endpoint security
* `SIGNED` endpoints require an additional parameter, `signature`, to be
  sent in the  `query string` or `request body`.
* Endpoints use `HMAC SHA256` signatures. The `HMAC SHA256 signature` is a keyed `HMAC SHA256` operation.
  Use your `secretKey` as the key and `totalParams` as the value for the HMAC operation.
* The `signature` is **not case sensitive**.
* `totalParams` is defined as the `query string` concatenated with the
  `request body`.

## Timing security
* A `SIGNED` endpoint also requires a parameter, `timestamp`, to be sent which
  should be the millisecond timestamp of when the request was created and sent.
* An additional parameter, `recvWindow`, may be sent to specify the number of
  milliseconds after `timestamp` the request is valid for. If `recvWindow`
  is not sent, **it defaults to 5000**.

> The logic is as follows:

```json-doc
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


**It is recommended to use a small recvWindow of 5000 or less! The max cannot go beyond 60,000!**


## SIGNED Endpoint Examples for POST /uapi/v1/order
Here is a step-by-step example of how to send a vaild signed payload from the
Linux command line using `echo`, `openssl`, and `curl`.

Key | Value
------------ | ------------
apiKey | vmPUZE6mv9SD5VNHk4HlWFsOr6aKE2zvsw0MuIgwCIPy6utIco14y7Ju91duEh8A
secretKey | NhqPtmdSJYdKjVHjA7PZj4Mge3R5YNiP1e3UZjInClVN65XAbvqqM6A7H5fATj0j


Parameter | Value
------------ | ------------
symbol | ltcbtc
side | buy
type | limit
quantity | 1
price | 0.1
recvWindow | 5000
timestamp | 1499827319559

## Example 1: As a request body 
> Request Body:

```shell
symbol=ltcbtc&side=buy&type=limit&quantity=1&price=0.1&recvWindow=5000&timestamp=1499827319559
```

> HMAC SHA256 signature:

```shell
[linux]$ echo -n "symbol=ltcbtc&side=buy&type=limit&quantity=1&price=0.1&recvWindow=5000&timestamp=1499827319559" | openssl dgst -sha256 -hmac "NhqPtmdSJYdKjVHjA7PZj4Mge3R5YNiP1e3UZjInClVN65XAbvqqM6A7H5fATj0j"
(stdin)= a03b8ba3ae3bad7b78fcec42224967e8cc19faec1a9d05c1f46200b9c5cab360
```

> (HMAC SHA256)

```shell
[linux]$ curl -H "X-WX-APIKEY: vmPUZE6mv9SD5VNHk4HlWFsOr6aKE2zvsw0MuIgwCIPy6utIco14y7Ju91duEh8A" -X POST 'https://api.wazirx.com/uapi/v1/order' -d 'symbol=ltcbtc&side=buy&type=limit&quantity=1&price=0.1&recvWindow=5000&timestamp=1499827319559&signature=a03b8ba3ae3bad7b78fcec42224967e8cc19faec1a9d05c1f46200b9c5cab360'
```

## Example 2: As a query string
> Query String:

```shell
symbol=ltcbtc&side=buy&type=limit&quantity=1&price=0.1&recvWindow=5000&timestamp=1499827319559
```

> HMAC SHA256 signature:

```shell
[linux]$ echo -n "symbol=ltcbtc&side=buy&type=limit&quantity=1&price=0.1&recvWindow=5000&timestamp=1499827319559" | openssl dgst -sha256 -hmac "NhqPtmdSJYdKjVHjA7PZj4Mge3R5YNiP1e3UZjInClVN65XAbvqqM6A7H5fATj0j"
(stdin)= a03b8ba3ae3bad7b78fcec42224967e8cc19faec1a9d05c1f46200b9c5cab360
```


> Curl Command:

```shell
[linux]$ curl -H "X-WX-APIKEY: vmPUZE6mv9SD5VNHk4HlWFsOr6aKE2zvsw0MuIgwCIPy6utIco14y7Ju91duEh8A" -X POST 'https://api.wazirx.com/uapi/v1/order?symbol=ltcbtc&side=buy&type=limit&quantity=1&price=0.1&recvWindow=5000&timestamp=1499827319559&signature=a03b8ba3ae3bad7b78fcec42224967e8cc19faec1a9d05c1f46200b9c5cab360'
```

## Example 3: Mixed query string and request body
> Query String:

```shell
symbol=ltcbtc&side=buy&type=limit
```

> Request Body:

```shell
quantity=1&price=0.1&recvWindow=5000&timestamp=1499827319559
```

> HMAC SHA256 signature:

```shell
[linux]$ echo -n "symbol=ltcbtc&side=buy&type=limitquantity=1&price=0.1&recvWindow=5000&timestamp=1499827319559" | openssl dgst -sha256 -hmac "NhqPtmdSJYdKjVHjA7PZj4Mge3R5YNiP1e3UZjInClVN65XAbvqqM6A7H5fATj0j"
(stdin)= e8dc96bc41383d42f5dca9af18fdec5017555ba53256b55408c4e7cbbea79225
```


> CURL command (HMAC SHA256):

```shell
[linux]$ curl -H "X-WX-APIKEY: vmPUZE6mv9SD5VNHk4HlWFsOr6aKE2zvsw0MuIgwCIPy6utIco14y7Ju91duEh8A" -X POST 'https://api.wazirx.com/uapi/v1/order?symbol=ltcbtc&side=buy&type=limit' -d 'quantity=1&price=0.1&recvWindow=5000&timestamp=1499827319559&signature=e8dc96bc41383d42f5dca9af18fdec5017555ba53256b55408c4e7cbbea79225'
```

Note that the signature is different in example 3.
There is no & between "type=limit" and "quantity=1".

# Public API Endpoints

## ENUM definitions

**Order status (status):**

* idle - The order is idle not yet triggered.
* wait - The order is still open and waiting to be filled completely.
* done - The order has been completely filled.
* cancelled - The order has been canceled by the user.

**Order types (orderTypes, type):**

* limit
* stop_limit

**Order side (side):**

* buy
* sell


# General endpoints

## Test connectivity

```
GET /uapi/v1/ping
```

> Response:

```json-doc
{}
```

Test connectivity to the Rest API.

**Parameters:**
NONE

<br/>
<br/>

## System status

```
GET /uapi/v1/systemStatus
```

> Response:

```json-doc
{ 
    "status": 0,              // 0: normal，1：system maintenance
    "message": "normal"       // normal or system maintenance
}
```

Fetch system status. 

**Parameters:**
NONE

<br/>
<br/>
<br/>
<br/>
<br/>

## Check server time

```
GET /uapi/v1/time
```

> Response:

```json-doc
{
  "serverTime": 1499827319559
}
```


Test connectivity to the Rest API and get the current server time.

**Parameters:**
NONE


# Market Data endpoints
## 24hr tickers price change statistics

```
GET /uapi/v1/tickers/24hr
```

> Response:

```json-doc
[
  {
    "symbol": "btcinr",
    "baseAsset": "btc",
    "quoteAsset": "inr",
    "openPrice": "704999.0",
    "lowPrice": "702603.0",    
    "highPrice": "730001.0",
    "lastPrice": "720101.0",
    "volume": "891.8329",
    "bidPrice": "720102.0",
    "askPrice": "722999.0",
    "at": 1588829734
  }
]
```
24 hour rolling window price change statistics. 


## 24hr ticker price change statistics

```
GET /uapi/v1/ticker/24hr
```

> Response:

```json-doc
{
  "symbol": "btcinr",
  "baseAsset": "btc",
  "quoteAsset": "inr",
  "openPrice": "704999.0",
  "lowPrice": "702603.0",    
  "highPrice": "730001.0",
  "lastPrice": "720101.0",
  "volume": "891.8329",
  "bidPrice": "720102.0",
  "askPrice": "722999.0",
  "at": 1588829734
}
```

24 hour rolling window price change statistics. 

**Parameters:**

Name | Type | Mandatory | Description
------------ | ------------ | ------------ | ------------
symbol | STRING | YES |

* If the symbol is not sent, tickers for all symbols will be returned in an array.

<br />

## Order book

```
GET /uapi/v1/depth
```

> Response:

```json-doc
{
   "at": 1588831243,
   "asks":[
      [
         "9291.0",   // PRICE
         "0.0119"    // QTY
      ]
   ],
   "bids":[
      [
         "9253.0",   // PRICE
         "1.0456"    // QTY
      ]
   ]
}
```

**Parameters:**

Name | Type | Mandatory | Description
------------ | ------------ | ------------ | ------------
symbol | STRING | YES |
limit | INT | NO | Default 20; max 1000. Valid limits:[1, 5, 10, 20, 50, 100, 500, 1000]

<br />

## Recent trades list

```
GET /uapi/v1/trades
```

> Response:

```json-doc
[
  {
    "id": 28457,
    "price": "4.00000100",
    "qty": "12.00000000",
    "quoteQty": "48.000012",
    "time": 1499865549590,
    "isBuyerMaker": true
  }
]
```

Get recent trades.

**Parameters:**

Name | Type | Mandatory | Description
------------ | ------------ | ------------ | ------------
symbol | STRING | YES |
limit | INT | NO | Default 500; max 1000.

<br />

## Old trade lookup (MARKET_DATA)

```
GET /uapi/v1/historicalTrades
```

> Response:

```json-doc
[
  {
    "id": 28457,
    "price": "4.00000100",
    "qty": "12.00000000",
    "quoteQty": "48.000012",
    "time": 1499865549590,
    "isBuyerMaker": true
  }
]
```

Get older trades.

**Parameters:**

Name | Type | Mandatory | Description
------------ | ------------ | ------------ | ------------
symbol | STRING | YES |
limit | INT | NO | Default 500; max 1000.
fromId | LONG | NO | TradeId to fetch from. Default gets most recent trades.


# Account endpoints

## New order (TRADE)

```
POST /uapi/v1/order  (HMAC SHA256)
```

> Response:

```json-doc
{
  "id": 28,
  "symbol": "btcusdt",
  "price": "9293.0",
  "origQty": "10.0",
  "executedQty": "8.2",
  "status": "wait",
  "type": "limit",
  "side": "sell",
  "createdTime": 1499827319559,
  "updatedTime": 1499827319559
}
```

Send in a new order.

**Parameters:**

Name | Type | Mandatory | Description
------------ | ------------ | ------------ | ------------
symbol | STRING | YES |
side | ENUM | YES |
type | ENUM | YES | `limit` or `stop_limit`
quantity | DECIMAL | NO |
price | DECIMAL | NO |
stopPrice | DECIMAL | NO |
recvWindow | LONG | NO | The value cannot be greater than ```60000```
timestamp | LONG | YES |

Additional mandatory parameters based on `type`:

Type | Additional mandatory parameters
------------ | ------------
`limit` | `quantity`, `price`
`stop_limit` | `quantity`,  `price`, `stopPrice`


## Test new order (TRADE)

```
POST /uapi/v1/order/test (HMAC SHA256)
```

> Response:

```json-doc
{}
```

Test new order creation and signature/recvWindow long.
Validates a new order but does not send it into the matching engine.

**Parameters:**
Same as `POST /uapi/v1/order`


## Query order (USER_DATA)

```
GET /uapi/v1/order (HMAC SHA256)
```

> Response:

```json-doc
{
  "id": 30,
  "symbol": "btcusdt",
  "price": "9293.0",
  "stopPrice": "9200.0",
  "origQty": "10.0",
  "executedQty": "0.0",
  "status": "idle",
  "type": "stop_limit",
  "side": "sell",
  "createdTime": 1499827319559,
  "updatedTime": 1507725176595
}
```

Check an order's status.

**Parameters:**

Name | Type | Mandatory | Description
------------ | ------------ | ------------ | ------------
symbol | STRING | YES |
orderId | LONG | YES |
recvWindow | LONG | NO | The value cannot be greater than ```60000```
timestamp | LONG | YES |


<br />

## Current open orders (USER_DATA)

```
GET /uapi/v1/openOrders  (HMAC SHA256)
```

> Response:

```json-doc
[
  {
    "id": 28,
    "symbol": "btcusdt",
    "price": "9293.0",
    "origQty": "10.0",
    "executedQty": "8.2",
    "status": "wait",
    "type": "limit",
    "side": "sell",
    "createdTime": 1499827319559,
    "updatedTime": 1499827319559
  },
  {
    "id": 30,
    "symbol": "btcusdt",
    "price": "9293.0",
    "stopPrice": "9200.0",
    "origQty": "10.0",
    "executedQty": "0.0",
    "status": "idle",
    "type": "stop_limit",
    "side": "sell",
    "createdTime": 1499827319559,
    "updatedTime": 1507725176595
  }
]
```

Get all open orders on a symbol.

**Parameters:**

Name | Type | Mandatory | Description
------------ | ------------ | ------------ | ------------
symbol | STRING | NO |
orderId | LONG | NO |
recvWindow | LONG | NO | The value cannot be greater than ```60000```
timestamp | LONG | YES |

**Notes:**
* If the symbol is not sent, orders for all symbols will be returned in an array.
* If `orderId` is set, it will get orders >= that `orderId`. Otherwise most recent orders are returned.

<br />

## All orders (USER_DATA)

```
GET /uapi/v1/allOrders (HMAC SHA256)
```

> Response:

```json-doc
[
  {
    "id": 28,
    "symbol": "btcusdt",
    "price": "9293.0",
    "origQty": "10.0",
    "executedQty": "8.2",
    "status": "cancel",
    "type": "limit",
    "side": "sell",
    "createdTime": 1499827319559,
    "updatedTime": 1499827319559
  },
  {
    "id": 30,
    "symbol": "btcusdt",
    "price": "9293.0",
    "stopPrice": "9200.0",
    "origQty": "10.0",
    "executedQty": "0.0",
    "status": "cancel",
    "type": "stop_limit",
    "side": "sell",
    "createdTime": 1499827319559,
    "updatedTime": 1507725176595
  }
]
```

Get all account orders; "idle", "wait", "cancel" or "done".

**Parameters:**

Name | Type | Mandatory | Description
------------ | ------------ | ------------ | ------------
symbol | STRING | YES |
orderId | LONG | NO |
startTime | LONG | NO |
endTime | LONG | NO |
limit | INT | NO | Default 500; max 1000.
recvWindow | LONG | NO | The value cannot be greater than ```60000```
timestamp | LONG | YES |

**Notes:**
* If `orderId` is set, it will get orders >= that `orderId`. Otherwise most recent orders are returned.

<br />

## Cancel order (TRADE)

```
DELETE /uapi/v1/order  (HMAC SHA256)
```

> Response:

```json-doc
{
  "id": 30,
  "symbol": "btcusdt",
  "price": "9293.0",
  "stopPrice": "9200.0",
  "origQty": "10.0",
  "executedQty": "0.0",
  "status": "cancel",
  "type": "stop_limit",
  "side": "sell",
  "createdTime": 1499827319559,
  "updatedTime": 1507725176595
}
```

Cancel an active order.

**Parameters:**

Name | Type | Mandatory | Description
------------ | ------------ | ------------ | ------------
symbol | STRING | YES |
orderId | LONG | YES |
recvWindow | LONG | NO | The value cannot be greater than ```60000```
timestamp | LONG | YES |


<br />

## Cancel All Open Orders on a Symbol (TRADE)

```
DELETE /uapi/v1/openOrders (HMAC SHA256)
```

> Response:

```json-doc
[
  {
      "id": 28,
      "symbol": "btcusdt",
      "price": "9293.0",
      "origQty": "10.0",
      "executedQty": "8.2",
      "status": "cancel",
      "type": "limit",
      "side": "sell",
      "createdTime": 1499827319559,
      "updatedTime": 1499827319559
    },
    {
      "id": 30,
      "symbol": "btcusdt",
      "price": "9293.0",
      "stopPrice": "9200.0",
      "origQty": "10.0",
      "executedQty": "0.0",
      "status": "cancel",
      "type": "stop_limit",
      "side": "sell",
      "createdTime": 1499827319559,
      "updatedTime": 1507725176595
    }
]
```

Cancels all active orders on a symbol.

Name | Type | Mandatory | Description
------------ | ------------ | ------------ | ------------
symbol | STRING | YES |
recvWindow | LONG | NO | The value cannot be greater than ```60000```
timestamp | LONG | YES |


<br />

## Account information (USER_DATA)

```
GET /uapi/v1/account (HMAC SHA256)
```

> Response:

```json-doc
{
  "accountType": "default",
  "canTrade": true,
  "canWithdraw": true,
  "updateTime": 123456789
}
```

Get current account information.

**Parameters:**

Name | Type | Mandatory | Description
------------ | ------------ | ------------ | ------------
recvWindow | LONG | NO | The value cannot be greater than ```60000```
timestamp | LONG | YES |

<br />

## Fund details (USER_DATA)

```
GET /uapi/v1/funds (HMAC SHA256)
```

> Response:

```json-doc
[
  {
    "asset": "btc",
    "free": "4723846.89208129",
    "locked": "0.0"
  },
  {
    "asset": "wrx",
    "free": "4763368.68006011",
    "locked": "0.0",
    "reservedFee": "12.5"
  }
]
```

Get fund details for current account.

**Parameters:**

Name | Type | Mandatory | Description
------------ | ------------ | ------------ | ------------
recvWindow | LONG | NO | The value cannot be greater than ```60000```
timestamp | LONG | YES |

