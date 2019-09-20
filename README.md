![image alt text](logo.svg)

# Blockchain.io HMAC API
Version: 1.0  
Last updated: September 20th 2019

# Table of contents

- [HMAC Authentication : how does it work ?](#hmac-authentication--how-does-it-work-)
- [Security notice](#security-notice)
- [How do I format request ?](#how-do-i-format-request-)
- [Public endpoints](#public-endpoints)
  - [GET /ping](#test-api-connectivity)
  - [GET /time](#check-server-time)
  - [GET /exchangeInfo](#exchange-trading-rules-and-symbols-informations)
  - [GET /depth](#order-book-depth)
  - [GET /trades](#recent-trades-list)
  - [GET /historicalTrades](#old-trades-lookup)
  - [GET /aggTrades](#compressedaggregate-trades-list)
  - [GET /klines](#klinecandlestick-data)
  - [GET /avgPrice](#current-average-price)
  - [GET /ticker/24hr](#24hr-ticker-price-change-statistics)
  - [GET /ticker/price](#symbol-price-ticker)
  - [GET /ticker/bookTicker](#symbol-order-book-ticker)
- [Private endpoints](#private-endpoints)
  - [POST /order](#create-order)
  - [POST /order/test](#create-order--test)
  - [GET /order](#query-order)
  - [DEL /order](#cancel-order)
  - [GET /openOrders](#current-open-orders)
  - [GET /allOrders](#all-orders)
  - [GET /account](#account-information)
  - [GET /myTrades](#account-trade-list)
- [Timing security](#timing-security)
- [Error codes](#error-codes)

## HMAC authentication : how does it work ?

Blockchain.io uses HMAC as authentication method for our API endpoints. The purpose is to sign your requests in a simple and secure fashion in order to assure maximum security.

*Concept is simple :*

- Each user can generate **a pair of keys**, also known as **API token**.
- An **API token** is composed of a **public key** and a **secret key**.
- **Public key** is passed into your request headers.
- **Secret key** is used to sign your payload.

Once request has been sent, BCIO API will use your public key as a mean to retrieve your secret key from the database. Then, using fetched secret key, the API will determine the expected signature and compare it against the signature you sent. 
If signatures match, **congratulations** : authentication has been successful !

*cf. ‘[How do I format request ?](#how-do-i-format-request-)’

## Security notice

Whereas public key can be revealed to everyone without consequences, **secret key must be kept secretly and shouldn’t be given to anyone under any circumstances**. Otherwise, you expose yourself to **security issues regarding your BCIO account**.

## How do I format request ?

*Since public endpoints don’t require any authentication, there is no need for API keys and therefore, no need to sign your request.* ***This section is useful for private endpoints only.***

For security purposes, every private endpoint has two mandatory and one optional parameters :

- **timestamp** (mandatory)**:** *millisecond timestamp of when the request was created and sent*
- **recvWindow** (optional) **:** *number of milliseconds after* `timestamp` *the request is valid for*
- **signature** (mandatory)**:** *signed payload using* *your secret key*

[You can find more informations about the importance of these parameters here.](#timing-security)

Your public key has to be passed in `X-BCIO-APIKEY` header field.

Here is a step-by-step example of how to send a valid signed payload from the Linux command line using `echo`, `openssl`, and `curl`.

**POST** `https://api.blockchain.io/v1/order`

| **Key**    | **Value**                                                        |
| ---------- | ---------------------------------------------------------------- |
| public_key | vmPUZE6mv9SD5VNHk4HlWFsOr6aKE2zvsw0MuIgwCIPy6utIco14y7Ju91duEh8A |
| secret_key | NhqPtmdSJYdKjVHjA7PZj4Mge3R5YNiP1e3UZjInClVN65XAbvqqM6A7H5fATj0j |

| **Parameter** | **Value**     |
| ------------- | ------------- |
| symbol        | LTCBTC        |
| side          | BUY           |
| type          | LIMIT         |
| timeInForce   | GTC           |
| quantity      | 1             |
| price         | 0.1           |
| recvWindow    | 5000          |
| timestamp     | 1499827319559 |


**Example 1: As a query string**


  - **Query string**
  `symbol=LTCBTC&side=BUY&type=LIMIT&timeInForce=GTC&quantity=1&price=0.1&recvWindow=5000&timestamp=1499827319559`   

  - **HMAC SHA256 signature**
    ```bash
    $ echo -n "symbol=LTCBTC&side=BUY&type=LIMIT&timeInForce=GTC&quantity=1&price=0.1&recvWindow=5000&timestamp=1499827319559" | openssl dgst -sha256 -hmac "NhqPtmdSJYdKjVHjA7PZj4Mge3R5YNiP1e3UZjInClVN65XAbvqqM6A7H5fATj0j"
    ```
    
    `c8db56825ae71d6d79447849e617115f4a920fa2acdcab2b053c4b2838bd6b71`

  - **cURL command**
  ```bash
  $ curl -H "X-BCIO-APIKEY: vmPUZE6mv9SD5VNHk4HlWFsOr6aKE2zvsw0MuIgwCIPy6utIco14y7Ju91duEh8A" -X POST 'https://api.blockchain.io/v1/order?symbol=LTCBTC&side=BUY&type=LIMIT&timeInForce=GTC&quantity=1&price=0.1&recvWindow=5000&timestamp=1499827319559&signature=c8db56825ae71d6d79447849e617115f4a920fa2acdcab2b053c4b2838bd6b71'
  ```

**Example 2: As a request body**


  - **Request body**
    `symbol=LTCBTC&side=BUY&type=LIMIT&timeInForce=GTC&quantity=1&price=0.1&recvWindow=5000&timestamp=1499827319559`


  - **HMAC SHA256 signature**
    ```bash
    $ echo -n "symbol=LTCBTC&side=BUY&type=LIMIT&timeInForce=GTC&quantity=1&price=0.1&recvWindow=5000&timestamp=1499827319559" | openssl dgst -sha256 -hmac "NhqPtmdSJYdKjVHjA7PZj4Mge3R5YNiP1e3UZjInClVN65XAbvqqM6A7H5fATj0j"
    ```
    
    `c8db56825ae71d6d79447849e617115f4a920fa2acdcab2b053c4b2838bd6b71`


  - **cURL command**
    ```bash
    $ curl -H "X-BCIO-APIKEY: vmPUZE6mv9SD5VNHk4HlWFsOr6aKE2zvsw0MuIgwCIPy6utIco14y7Ju91duEh8A" -X POST 'https://api.blockchain.io/v1/order' -d 'symbol=LTCBTC&side=BUY&type=LIMIT&timeInForce=GTC&quantity=1&price=0.1&recvWindow=5000&timestamp=1499827319559&signature=c8db56825ae71d6d79447849e617115f4a920fa2acdcab2b053c4b2838bd6b71'
    ```

**Example 3: Mixed query string and request body**


  - **Query string**
  `symbol=LTCBTC&side=BUY&type=LIMIT&timeInForce=GTC`

  - **Request body**
  `quantity=1&price=0.1&recvWindow=5000&timestamp=1499827319559`
  
  - **HMAC SHA256 signature**
    ```bash
    $ echo -n "symbol=LTCBTC&side=BUY&type=LIMIT&timeInForce=GTCquantity=1&price=0.1&recvWindow=5000&timestamp=1499827319559" | openssl dgst -sha256 -hmac "NhqPtmdSJYdKjVHjA7PZj4Mge3R5YNiP1e3UZjInClVN65XAbvqqM6A7H5fATj0j"
    ```
    
    `0fd168b8ddb4876a0358a8d14d0c9f3da0e9b20c5d52b2a00fcf7d1c602f9a77`


  - **cURL command**
    ```bash
    $ curl -H "X-BCIO-APIKEY: vmPUZE6mv9SD5VNHk4HlWFsOr6aKE2zvsw0MuIgwCIPy6utIco14y7Ju91duEh8A" -X POST 'https://api.blockchain.io/v1/order?symbol=LTCBTC&side=BUY&type=LIMIT&timeInForce=GTC' -d 'quantity=1&price=0.1&recvWindow=5000&timestamp=1499827319559&signature=0fd168b8ddb4876a0358a8d14d0c9f3da0e9b20c5d52b2a00fcf7d1c602f9a77'
    ```

*Note that the signature is different in example 3. There is no ‘&' between "GTC" and "quantity=1".*

## Public endpoints

#### Test API connectivity

    GET https://api.blockchain.io/v1/ping


- **Parameters :** NONE
- **Response :** 
    ```javascript
    {}
    ```
---
#### Check server time

    GET https://api.blockchain.io/v1/time


- **Parameters :** NONE
- **Response :** 
    ```javascript
    {
      "serverTime": 1499827319559
    }
    ```
---
#### Exchange trading rules and symbols informations

    GET https://api.blockchain.io/v1/exchangeInfo


- **Parameters :** NONE
- **Response :** 
    ```javascript
    {
      "timezone": "UTC",
      "serverTime": 1508631584636,
      "rateLimits": [
      ],
      "exchangeFilters": [
      ],
      "symbols": [{
        "symbol": "ETHBTC",
        "status": "TRADING",
        "baseAsset": "ETH",
        "baseAssetPrecision": 8,
        "quoteAsset": "BTC",
        "quotePrecision": 8,
        "orderTypes": [
          // These are defined in the `ENUM definitions` section under `Order types (orderTypes)`.
          // All orderTypes are optional.
        ],
        "icebergAllowed": false,
        "filters": [
        ]
      }]
    }
    ```
---
#### Order book depth

    GET https://api.blockchain.io/v1/depth


- **Parameters :**

| **Name** | **Type** | **Mandatory** | **Description**                                                     |
| -------- | -------- | ------------- | ------------------------------------------------------------------- |
| symbol   | STRING   | YES           |                                                                     |
| limit    | INT      | NO            | Default 100; max 1000. Valid limits:[5, 10, 20, 50, 100, 500, 1000] |

- **Response :**
    ```javascript
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
    ```
---
#### Recent trades list

    GET https://api.blockchain.io/v1/trades


- **Parameters** :

| **Name** | **Type** | **Mandatory** | **Description**        |
| -------- | -------- | ------------- | ---------------------- |
| symbol   | STRING   | YES           |                        |
| limit    | INT      | NO            | Default 500; max 1000. |



- **Response** :
    ```javascript
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
    ```
---
#### Old trades lookup

    GET https://api.blockchain.io/v1/historicalTrades


- **Parameters** :

| **Name** | **Type** | **Mandatory** | **Description**        |
| -------- | -------- | ------------- | ---------------------- |
| symbol   | STRING   | YES           |                        |
| limit    | INT      | NO            | Default 500; max 1000. |
| fromId    | INT      | NO            | TradeId to fetch from. Default gets most recent trades |



- **Response** :
    ```javascript
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
    ```
---
#### Compressed/Aggregate trades list

    GET https://api.blockchain.io/v1/aggTrades

Get compressed, aggregate trades. Trades that fill at the time, from the same order, with the same price will have the quantity aggregated.


- **Parameters :**

| **Name**  | **Type** | **Mandatory** | **Description**                                          |
| --------- | -------- | ------------- | -------------------------------------------------------- |
| symbol    | STRING   | YES           |                                                          |
| fromId    | LONG     | NO            | ID to get aggregate trades from INCLUSIVE.               |
| startTime | LONG     | NO            | Timestamp in ms to get aggregate trades from INCLUSIVE.  |
| endTime   | LONG     | NO            | Timestamp in ms to get aggregate trades until INCLUSIVE. |
| limit     | INT      | NO            | Default 500; max 1000.                                   |

    - If both startTime and endTime are sent, time between startTime and endTime must be less than 1 hour.
    - If fromId, startTime, and endTime are not sent, the most recent aggregate trades will be returned.


- **Response :**
    ```javascript
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
    ```
---
#### Kline/Candlestick data

    GET https://api.blockchain.io/v1/klines


- **Parameters :**

| **Name**  | **Type** | **Mandatory** | **Description**        |
| --------- | -------- | ------------- | ---------------------- |
| symbol    | STRING   | YES           |                        |
| interval  | ENUM     | YES           |                        |
| startTime | LONG     | NO            |                        |
| endTime   | LONG     | NO            |                        |
| limit     | INT      | NO            | Default 500; max 1000. |

- **Kline/Candlestick chart intervals:**

m -> minutes; h -> hours; d -> days; w -> weeks; M -> months

    - 1m
    - 3m
    - 5m
    - 15m
    - 30m
    - 1h
    - 2h
    - 4h
    - 6h
    - 8h
    - 12h
    - 1d
    - 3d
    - 1w
    - 1M


- **Response :**
    ```javascript
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
    ```
---
#### Current average price

    GET https://api.blockchain.io/v1/avgPrice


- **Parameters :**

| **Name** | **Type** | **Mandatory** | **Description** |
| -------- | -------- | ------------- | --------------- |
| symbol   | STRING   | YES           |                 |



- **Response :**
    ```javascript
    {
      "mins": 5,
      "price": "9.35751834"
    }
    ```
---
#### 24hr ticker price change statistics

    GET https://api.blockchain.io/v1/ticker/24hr


- **Parameters :**

| **Name** | **Type** | **Mandatory** | **Description** |
| -------- | -------- | ------------- | --------------- |
| symbol   | STRING   | NO            |                 |

If the symbol is not sent, tickers for all symbols will be returned in an array.


- **Response :**
    ```javascript
    {
      "symbol": "LTCBTC",
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
    ```
---
#### Symbol price ticker

    GET https://api.blockchain.io/v1/ticker/price


- **Parameters :**

| **Name** | **Type** | **Mandatory** | **Description** |
| -------- | -------- | ------------- | --------------- |
| symbol   | STRING   | NO            |                 |

If the symbol is not sent, prices for all symbols will be returned in an array.


- **Response :**
    ```javascript
    {
      "symbol": "LTCBTC",
      "price": "4.00000200"
    }
    ```
---
#### Symbol order book ticker

    GET https://api.blockchain.io/v1/ticker/bookTicker


- **Parameters :**

| **Name** | **Type** | **Mandatory** | **Description** |
| -------- | -------- | ------------- | --------------- |
| symbol   | STRING   | NO            |                 |

If the symbol is not sent, bookTickers for all symbols will be returned in an array.


- **Response :**
    ```javascript
    {
      "symbol": "LTCBTC",
      "bidPrice": "4.00000000",
      "bidQty": "431.00000000",
      "askPrice": "4.00000200",
      "askQty": "9.00000000"
    }
    ```

## Private endpoints

#### Create order

    POST https://api.blockchain.io/v1/order


- **Parameters :**

| **Name**         | **Type** | **Mandatory** | **Description**                                                                                                                           |
| ---------------- | -------- | ------------- | ----------------------------------------------------------------------------------------------------------------------------------------- |
| symbol           | STRING   | YES           |                                                                                                                                           |
| side             | ENUM     | YES           | `BUY` or `SELL`                                                                                                                                         |
| type             | ENUM     | YES           | `MARKET` or `LIMIT`                                                                                                                                         |
| timeInForce      | ENUM     | NO            | Only GTC available                                                                                                                        |
| quantity         | DECIMAL  | YES           |                                                                                                                                           |
| price            | DECIMAL  | NO            |                                                                                                                                           |
| newClientOrderId | STRING   | NO            | A unique id for the order. Automatically generated if not sent.                                                                           |
| newOrderRespType | ENUM     | NO            | Set the response JSON. `ACK`, `RESULT`, or `FULL`; `MARKET` and `LIMIT` order types default to `FULL`, all other orders default to `ACK`. |
| recvWindow       | LONG     | NO            |                                                                                                                                           |
| timestamp        | LONG     | YES           |                                                                                                                                           |

- Additional mandatory parameters based on `type`:

  | **Type** | **Additional mandatory parameters** |
  | -------- | ----------------------------------- |
  | `LIMIT`  | `timeInForce`, `price`              |

- Order `status`:

  | **Type** | **Description** |
  | -------- | ----------------------------------- |
  | `NEW`  | Order has been newly created and hasn't been consumed yet              |
  | `PARTIALLY_FILLED` | Order has been partially consumed and still available |
  | `FILLED` | Order has been fully consumed and is not available anymore |
  | `CANCELLED` | Order has been cancelled |


- **Response ACK :**
    ```javascript
    {
      "symbol": "LTCBTC",
      "orderId": 28,
      "clientOrderId": "6gCrw2kRUAF9CvJDGP16IP",
      "transactTime": 1507725176595
    }
    ```


- **Response RESULT :**
    ```javascript
    {
      "symbol": "LTCBTC",
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
    ```


- **Response FULL :**
    ```javascript
    {
      "symbol": "LTCBTC",
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
          "commissionAsset": "BTC"
        },
        {
          "price": "3999.00000000",
          "qty": "5.00000000",
          "commission": "19.99500000",
          "commissionAsset": "BTC"
        },
        {
          "price": "3998.00000000",
          "qty": "2.00000000",
          "commission": "7.99600000",
          "commissionAsset": "BTC"
        },
        {
          "price": "3997.00000000",
          "qty": "1.00000000",
          "commission": "3.99700000",
          "commissionAsset": "BTC"
        },
        {
          "price": "3995.00000000",
          "qty": "1.00000000",
          "commission": "3.99500000",
          "commissionAsset": "BTC"
        }
      ]
    }
    ```
---
#### Create order : test

    POST https://api.blockchain.io/v1/order/test

Test new order creation and signature/recvWindow long. Creates and validates a new order but does not send it into the matching engine.


- **Parameters :**


Same as `POST https://api.blockchain.io/v1/order`


- **Response :**
    ```javascript
    {}
    ```
---
#### Query order

    GET https://api.blockchain.io/v1/order


- **Parameters :**

| **Name**          | **Type** | **Mandatory** | **Description** |
| ----------------- | -------- | ------------- | --------------- |
| symbol            | STRING   | YES           |                 |
| orderId           | LONG     | NO            |                 |
| origClientOrderId | STRING   | NO            |                 |
| recvWindow        | LONG     | NO            |                 |
| timestamp         | LONG     | YES           |                 |

    - Either `orderId` or `origClientOrderId` must be sent.
    - For some historical orders `cummulativeQuoteQty` will be < 0, meaning the data is not available at this time.


- **Response :**
    ```javascript
    {
      "symbol": "LTCBTC",
      "orderId": 1,
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
    ```
---
#### Cancel order

    DELETE https://api.blockchain.io/v1/order


- **Parameters :**

| **Name**          | **Type** | **Mandatory** | **Description**                                                            |
| ----------------- | -------- | ------------- | -------------------------------------------------------------------------- |
| symbol            | STRING   | YES           |                                                                            |
| orderId           | LONG     | NO            |                                                                            |
| origClientOrderId | STRING   | NO            |                                                                            |
| newClientOrderId  | STRING   | NO            | Used to uniquely identify this cancel. Automatically generated by default. |
| recvWindow        | LONG     | NO            |                                                                            |
| timestamp         | LONG     | YES           |                                                                            |

    - Either `orderId` or `origClientOrderId` must be sent.


- **Response :**
    ```javascript
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
    ```
---
#### Current open orders

    GET https://api.blockchain.io/v1/openOrders  (HMAC SHA256)


- **Parameters :**

| **Name**   | **Type** | **Mandatory** | **Description** |
| ---------- | -------- | ------------- | --------------- |
| symbol     | STRING   | NO            |                 |
| recvWindow | LONG     | NO            |                 |
| timestamp  | LONG     | YES           |                 |

    - If the symbol is not sent, orders for all symbols will be returned in an array.


- **Response :**
    ```javascript
    [
      {
        "symbol": "LTCBTC",
        "orderId": 1,
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
    ```
---
#### All orders

    GET https://api.blockchain.io/v1/allOrders

Get all account orders; active, canceled, or filled.

- **Parameters :**

| **Name**   | **Type** | **Mandatory** | **Description**        |
| ---------- | -------- | ------------- | ---------------------- |
| symbol     | STRING   | YES           |                        |
| orderId    | LONG     | NO            |                        |
| startTime  | LONG     | NO            |                        |
| endTime    | LONG     | NO            |                        |
| limit      | INT      | NO            | Default 500; max 1000. |
| recvWindow | LONG     | NO            |                        |
| timestamp  | LONG     | YES           |                        |

    - If `orderId` is set, it will get orders >= that `orderId`. Otherwise most recent orders are returned.
    - For some historical orders `cummulativeQuoteQty` will be < 0, meaning the data is not available at this time.


- **Response :**
    ```javascript
    [
      {
        "symbol": "LTCBTC",
        "orderId": 1,
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
    ```
---
#### Account information

    GET https://api.blockchain.io/v1/account


- **Parameters :**

| **Name**   | **Type** | **Mandatory** | **Description** |
| ---------- | -------- | ------------- | --------------- |
| recvWindow | LONG     | NO            |                 |
| timestamp  | LONG     | YES           |                 |



- **Response :**
    ```javascript
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
    ```
---
#### Account trade list

    GET https://api.blockchain.io/v1/myTrades


- **Parameters :**

| **Name**   | **Type** | **Mandatory** | **Description**                                         |
| ---------- | -------- | ------------- | ------------------------------------------------------- |
| symbol     | STRING   | YES           |                                                         |
| startTime  | LONG     | NO            |                                                         |
| endTime    | LONG     | NO            |                                                         |
| fromId     | LONG     | NO            | TradeId to fetch from. Default gets most recent trades. |
| limit      | INT      | NO            | Default 500; max 1000.                                  |
| recvWindow | LONG     | NO            |                                                         |
| timestamp  | LONG     | YES           |                                                         |

    - If `fromId` is set, it will get orders >= that `fromId`. Otherwise most recent orders are returned.


- **Response :**
    ```javascript
    [
      {
        "symbol": "LTCBTC",
        "id": 28457,
        "orderId": 100234,
        "price": "4.00000100",
        "qty": "12.00000000",
        "quoteQty": "48.000012",
        "commission": "10.10000000",
        "commissionAsset": "LTC",
        "time": 1499865549590,
        "isBuyer": true,
        "isMaker": false,
        "isBestMatch": true
      }
    ]
    ```
---
## Timing security

A private endpoint requires a parameter, `timestamp`, to be sent which should be the millisecond timestamp when the request was created and sent.
An additional parameter, `recvWindow`, may be sent to specify the number of milliseconds after `timestamp` the request is valid for. If `recvWindow` is not sent, **it defaults to 5000**.

Logic is as follows:

    if (timestamp < (serverTime + 1000) && (serverTime - timestamp) <= recvWindow) {
      // process request
    } else {
      // reject request
    }

**Serious trading is about timing.** Networks can be unstable and unreliable, which can lead to requests taking varying amounts of time to reach the servers. With `recvWindow`, you can specify that the request must be processed within a certain number of milliseconds or be rejected by the server.
It recommended to use **a small recvWindow of 5000 or less!**

## Error codes

**-1022 INVALID_SIGNATURE**

- Signature for this request is not valid.

**-1102 MANDATORY_PARAM_EMPTY_OR_MALFORMED**

- A mandatory parameter was not sent, was empty/null, or malformed.
- Mandatory parameter '%s' was not sent, was empty/null, or malformed.

**-1104 UNREAD_PARAMETERS**

- Not all sent parameters were read.

**-1112 NO_DEPTH**

- No orders on book for symbol.

**-1115 INVALID_TIF**

- Invalid timeInForce.

**-1116 INVALID_ORDER_TYPE**

- Invalid orderType.

**-1117 INVALID_SIDE**

- Invalid side.

**-1120 BAD_INTERVAL**

- Invalid interval.

**-1121 BAD_SYMBOL**

- Invalid symbol.

**-2013 NO_SUCH_ORDER**

- Order does not exist.