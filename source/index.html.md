
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
* Any endpoint can return an ERROR

Sample Payload below:
```json5
{
  "code": -1121,
  "message": "Invalid symbol."
}
```
* Specific error codes and messages are defined in [Errors Codes](./errors.md).

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

# SIGNED (TRADE and USER_DATA) Endpoint security
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


## General endpoints

```
GET /uapi/v1/ping
```

> Response:

```json-doc
{}
```
### Test connectivity

Test connectivity to the Rest API.

**Parameters:**
NONE

<br/>
<br/>

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

### System status

Fetch system status. 

**Parameters:**
NONE

<br/>
<br/>
<br/>
<br/>
<br/>

```
GET /uapi/v1/time
```

> Response:

```json-doc
{
  "serverTime": 1499827319559
}
```

### Check server time
Test connectivity to the Rest API and get the current server time.

**Parameters:**
NONE



## Market Data endpoints

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
### 24hr tickers price change statistics

24 hour rolling window price change statistics. 

<br />
<br />
<br />
<br />
<br />
<br />
<br />
<br />
<br />
<br />
<br />
<br />
<br />
<br />
<br />


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

### 24hr ticker price change statistics
24 hour rolling window price change statistics. 

**Parameters:**

Name | Type | Mandatory | Description
------------ | ------------ | ------------ | ------------
symbol | STRING | YES |

* If the symbol is not sent, tickers for all symbols will be returned in an array.

<br />
<br />
<br />
<br />
<br />
<br />
<br />

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
### Order book

**Parameters:**

Name | Type | Mandatory | Description
------------ | ------------ | ------------ | ------------
symbol | STRING | YES |
limit | INT | NO | Default 20; max 1000. Valid limits:[1, 5, 10, 20, 50, 100, 500, 1000]

<br />
<br />
<br />
<br />
<br />
<br />
<br />
<br />
<br />


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



### Recent trades list
Get recent trades (up to last 500).

**Parameters:**

Name | Type | Mandatory | Description
------------ | ------------ | ------------ | ------------
symbol | STRING | YES |
limit | INT | NO | Default 500; max 1000.

<br />
<br />
<br />
<br />

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

### Old trade lookup (MARKET_DATA)

Get older trades.

**Parameters:**

Name | Type | Mandatory | Description
------------ | ------------ | ------------ | ------------
symbol | STRING | YES |
limit | INT | NO | Default 500; max 1000.
fromId | LONG | NO | TradeId to fetch from. Default gets most recent trades.




## Account endpoints

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
### New order (TRADE)

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



```
POST /uapi/v1/order/test (HMAC SHA256)
```

> Response:

```json-doc
{}
```
### Test new order (TRADE)

Test new order creation and signature/recvWindow long.
Validates a new order but does not send it into the matching engine.

**Parameters:**
Same as `POST /uapi/v1/order`


<br />
<br />

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
### Query order (USER_DATA)

Check an order's status.

**Parameters:**

Name | Type | Mandatory | Description
------------ | ------------ | ------------ | ------------
symbol | STRING | YES |
orderId | LONG | YES |
recvWindow | LONG | NO | The value cannot be greater than ```60000```
timestamp | LONG | YES |


<br />
<br />
<br />

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

### Current open orders (USER_DATA)

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
<br />
<br />
<br />
<br />
<br />
<br />
<br />
<br />
<br />
<br />

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

### All orders (USER_DATA)

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
<br />
<br />
<br />
<br />
<br />


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

### Cancel order (TRADE)

Cancel an active order.

**Parameters:**

Name | Type | Mandatory | Description
------------ | ------------ | ------------ | ------------
symbol | STRING | YES |
orderId | LONG | YES |
recvWindow | LONG | NO | The value cannot be greater than ```60000```
timestamp | LONG | YES |


<br />
<br />
<br />

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

### Cancel All Open Orders on a Symbol (TRADE)

Cancels all active orders on a symbol.

Name | Type | Mandatory | Description
------------ | ------------ | ------------ | ------------
symbol | STRING | YES |
recvWindow | LONG | NO | The value cannot be greater than ```60000```
timestamp | LONG | YES |


<br />
<br />
<br />
<br />
<br />
<br />
<br />
<br />
<br />
<br />
<br />
<br />
<br />
<br />
<br />
<br />
<br />
<br />

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
### Account information (USER_DATA)

Get current account information.

**Parameters:**

Name | Type | Mandatory | Description
------------ | ------------ | ------------ | ------------
recvWindow | LONG | NO | The value cannot be greater than ```60000```
timestamp | LONG | YES |

<br />

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
### Fund details (USER_DATA)

Get fund details for current account.

**Parameters:**

Name | Type | Mandatory | Description
------------ | ------------ | ------------ | ------------
recvWindow | LONG | NO | The value cannot be greater than ```60000```
timestamp | LONG | YES |

<br />
<br />
<br />
<br />
<br />
<br />
<br />


```
POST /uapi/v1/withdraw  (HMAC SHA256)
```

> Response:

```json-doc
{
    "message": "success",
    "success": true,
    "id": "2342"
}
```

### Withdraw funds

Submit a withdraw request.

**Parameters:**

Name | Type | Mandatory | Description
------------ | ------------ | ------------ | ------------
asset    |  STRING |    YES
address     | STRING | YES    
addressTag | STRING | NO | Secondary address identifier for coins like XRP,XMR etc.
amount | DECIMAL | YES    
note | STRING | NO | Description of the address
recvWindow | LONG | NO    
timestamp | LONG | YES    

<br />

```
GET /uapi/v1/withdrawHistory (HMAC SHA256)
```

> Response:

```json-doc
{
    "withdrawList": [
        {
            "id": "8345",
            "amount": "0.99",
            "transactionFee": "0.01",
            "address": "0x6915f16f8791d0a1cc2bf47c13a6b2a92000504b",
            "asset": "btc",
            "txId": null,
            "applyTime": 1508198532000,
            "status": "submitted"
        },
        {
            "id": "2342",
            "amount": "999.9999",
            "transactionFee": "0.0001",
            "address": "463tWEBn5XZJSxLU34r6g7h8jtxuNcDbjLSjkn3XAXHCbLrTTErJrBWYgHJQyrCwkNgYvyV3z8zctJLPCZy24jvb3NiTcTJ",
            "addressTag": "342341222",
            "txId": "b3c6219639c8ae3f9cf010cdc24fw7f7yt8j1e063f9b4bd1a05cb44c4b6e2509",
            "asset": "xrp",
            "applyTime": 1509832749300,
            "status": "done"
        }
    ],
    "success": true
}
```

### Withdraw history (USER_DATA)

Fetch withdraw history.

**Parameters:**

Name | Type | Mandatory | Description
------------ | ------------ | ------------ | ------------
asset | STRING | NO    
status | STRING | NO | submitting, verification_pending, canceled, submitted, rejected, accepted, processing, done, failed
startTime | LONG | NO    
endTime | LONG | NO    
recvWindow | LONG | NO    
timestamp | LONG | YES    

<br />
<br />
<br />
<br />
<br />
<br />
<br />
<br />

```
GET  /uapi/v1/depositAddress (HMAC SHA256)
```

> Response:

```json-doc
{
    "asset": "btc",
    "address": "0xhc24f045834hff34fh705394855g5h49835734fb"
}
```

> OR

```json-doc
{
    "asset": "xrp",
    "address": "0x6915f16f8791d0a1cc2bf47c13a6b2a92000504b",
    "addressTag": "1231212"
}
```
### Deposit address (USER_DATA)

Fetch deposit address.

**Parameters:**

Name | Type | Mandatory | Description
------------ | ------------ | ------------ | ------------
asset | STRING | YES    
status | Boolean | NO
recvWindow | LONG | NO    
timestamp | LONG | YES    

<br />
<br />
<br />
<br />

```
GET /uapi/v1/depositHistory (HMAC SHA256)
```

> Response:

```json-doc
{
    "depositList": [
        {
            "insertTime": 1508198532000,
            "amount": "0.04670582",
            "asset": "btc",
            "address": "0x6915f16f8791d0a1cc2bf47c13a6b2a92000504b",
            "txId": "0xdf33b22bdb2b28b1f75ccd201a4a4m6e7g83jy5fc5d5a9d1340961598cfcb0a1",
            "status": "submitted"
        },
        {
            "insertTime": 1508298532000,
            "amount": "1000",
            "asset": "xrp",
            "address": "463tWEBn5XZJSxLU34r6g7h8jtxuNcDbjLSjkn3XAXHCbLrTTErJrBWYgHJQyrCwkNgYvyV3z8zctJLPCZy24jvb3NiTcTJ",
            "addressTag": "342341222",
            "txId": "b3c6219639c8ae3f9cf010cdc24fw7f7yt8j1e063f9b4bd1a05cb44c4b6e2509",
            "status": "accepted"
        }
    ],
    "success": true
}
```
### Deposit history (USER_DATA)

Fetch deposit history.

**Parameters:**

Name | Type | Mandatory | Description
------------ | ------------ | ------------ | ------------
asset | STRING | NO    
status | STRING | NO | submitted, rejected or accepted
startTime | LONG | NO    
endTime | LONG | NO    
recvWindow | LONG | NO    
timestamp | LONG | YES    


# Filters

Filters define trading rules on a symbol or an exchange.
Filters come in two forms: `symbol filters` and `exchange filters`.

## Symbol filters
### PRICE_FILTER

> /exchangeInfo format:

```json-doc
{
  "filterType": "PRICE_FILTER",
  "minPrice": "0.00000100",
  "maxPrice": "100000.00000000",
  "tickSize": "0.00000100"
}
```

The `PRICE_FILTER` defines the `price` rules for a symbol. There are 3 parts:

* `minPrice` defines the minimum `price`/`stopPrice` allowed; disabled on `minPrice` == 0.
* `maxPrice` defines the maximum `price`/`stopPrice` allowed; disabled on `maxPrice` == 0.
* `tickSize` defines the intervals that a `price`/`stopPrice` can be increased/decreased by; disabled on `tickSize` == 0.

Any of the above variables can be set to 0, which disables that rule in the `price filter`. In order to pass the `price filter`, the following must be true for `price`/`stopPrice` of the enabled rules:

* `price` >= `minPrice`
* `price` <= `maxPrice`
* (`price`-`minPrice`) % `tickSize` == 0


### PERCENT_PRICE

> /exchangeInfo format:

```json-doc
{
  "filterType": "PERCENT_PRICE",
  "multiplierUp": "1.3000",
  "multiplierDown": "0.7000",
  "avgPriceMins": 5
}
```

The `PERCENT_PRICE` filter defines valid range for a price based on the average of the previous trades.
`avgPriceMins` is the number of minutes the average price is calculated over. 0 means the last price is used.

In order to pass the `percent price`, the following must be true for `price`:
* `price` <= `weightedAveragePrice` * `multiplierUp`
* `price` >= `weightedAveragePrice` * `multiplierDown`

<br />
<br />

### LOT_SIZE

> /exchangeInfo format:

```json-doc
{
  "filterType": "LOT_SIZE",
  "minQty": "0.00100000",
  "maxQty": "100000.00000000",
  "stepSize": "0.00100000"
}
```

The `LOT_SIZE` filter defines the `quantity` (aka "lots" in auction terms) rules for a symbol. There are 3 parts:

* `minQty` defines the minimum `quantity`/`icebergQty` allowed.
* `maxQty` defines the maximum `quantity`/`icebergQty` allowed.
* `stepSize` defines the intervals that a `quantity`/`icebergQty` can be increased/decreased by.

In order to pass the `lot size`, the following must be true for `quantity`/`icebergQty`:

* `quantity` >= `minQty`
* `quantity` <= `maxQty`
* (`quantity`-`minQty`) % `stepSize` == 0


### MIN_NOTIONAL

> /exchangeInfo format:

```json-doc
{
  "filterType": "MIN_NOTIONAL",
  "minNotional": "0.00100000",
  "applyToMarket": true,
  "avgPriceMins": 5
}
```

The `MIN_NOTIONAL` filter defines the minimum notional value allowed for an order on a symbol.
An order's notional value is the `price` * `quantity`.
`applyToMarket` determines whether or not the `MIN_NOTIONAL` filter will also be applied to `MARKET` orders.
Since `MARKET` orders have no price, the average price is used over the last `avgPriceMins` minutes.
`avgPriceMins` is the number of minutes the average price is calculated over. 0 means the last price is used.


<br />
<br />
<br />
<br />

### MARKET_LOT_SIZE

> /exchangeInfo format:

```json-doc
{
  "filterType": "MARKET_LOT_SIZE",
  "minQty": "0.00100000",
  "maxQty": "100000.00000000",
  "stepSize": "0.00100000"
}
```


The `MARKET_LOT_SIZE` filter defines the `quantity` (aka "lots" in auction terms) rules for `MARKET` orders on a symbol. There are 3 parts:

* `minQty` defines the minimum `quantity` allowed.
* `maxQty` defines the maximum `quantity` allowed.
* `stepSize` defines the intervals that a `quantity` can be increased/decreased by.

In order to pass the `market lot size`, the following must be true for `quantity`:

* `quantity` >= `minQty`
* `quantity` <= `maxQty`
* (`quantity`-`minQty`) % `stepSize` == 0


### MAX_NUM_ORDERS

> /exchangeInfo format:

```json-doc
{
  "filterType": "MAX_NUM_ORDERS",
  "limit": 25
}
```

The `MAX_NUM_ORDERS` filter defines the maximum number of orders an account is allowed to have open on a symbol.
Note that both "algo" orders and normal orders are counted for this filter.

<br />
<br />
<br />
<br />

### MAX_POSITION FILTER

> /exchangeInfo format:

```json-doc
{
  "filterType":"MAX_POSITION",
  "maxPosition":"10.00000000"
}
```

The `MAX_POSITION` filter defines the allowed maximum position an account can have on the base asset of a symbol. An account's position defined as the sum of the account's:
1. free balance of the base asset
1. locked balance of the base asset
1. sum of the qty of all open BUY orders

`BUY` orders will be rejected if the account's position is greater than the maximum position allowed.



## Exchange Filters

> /exchangeInfo format:

```json-doc
{
  "filterType": "EXCHANGE_MAX_NUM_ORDERS",
  "maxNumOrders": 1000
}
```
### EXCHANGE_MAX_NUM_ORDERS

The `MAX_NUM_ORDERS` filter defines the maximum number of orders an account is allowed to have open on the exchange.

