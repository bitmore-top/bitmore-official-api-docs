BitMore-official-api-docs
============================================
Official Documentation for the [BitMore][] APIs and Streams([简体中文版文档][])

<!-- TOC -->

- [Introduction](#introduction)
- [Getting Started](#getting-started)
- [Encrypted Verification of API](#encrypted-verification-of-api)
    - [Generate an API Key](#generate-an-api-key)
    - [Initiate a Request](#initiate-a-request)
    - [Signature](#signature)
    - [Select timestamp](#select-timestamp)
    - [Request Process](#request-process)
        - [Request](#request)
        - [Pagination](#pagination)
    - [Standards and Specification](#standards-and-specification)
        - [Timestamp](#timestamp)
        - [For example,](#for-example)
        - [Numbers](#numbers)
        - [Rate Limits](#rate-limits)
            - [REST API](#rest-api)
- [Spot API Reference](#spot-api-reference)
    - [Spot Market API](#spot-market-api)
        - [1. Access the list of all trading pairs](#1-access-the-list-of-all-trading-pairs)
        - [2. Access the depth table of trading pairs](#2-access-the-depth-table-of-trading-pairs)
        - [3. Access the ticker of a trading pair](#3-access-the-ticker-of-a-trading-pair)
        - [4. Access the market trading records of a trading pair](#4-access-the-market-trading-records-of-a-trading-pair)
        - [5. Access Candlestick chart](#5-access-candlestick-chart)
        - [6. Access Server Time](#6-access-server-time)
    - [Spot Account API](#spot-account-api)
        - [1. Access account information](#1-access-account-information)
        - [2. Order Placement](#2-order-placement)
        - [3. Cancel all orders](#3-cancel-all-orders)
        - [4. Cancel a specified order](#4-cancel-a-specified-order)
        - [5. Search orders](#5-search-orders)
        - [6. Order inquiry by Order ID](#6-order-inquiry-by-order-id)
        - [7. Access the account statement](#7-access-the-account-statement)
        - [8. Withdrawal](#8-withdrawal)

<!-- /TOC -->
# Introduction
Welcome to [BitMore][] API document for developers.

This document provides instructions on how to use APIs related to account management, market information, trading functions among others in spot trading.

Market API provides market data that are accessible to the public. Account APIs and trading APIs, which provide functions such as order placement, order cancellation, order inquiry and account information, need identity authentication.

# Getting Started

REST, a.k.a Respresntational State Transfer, is an architectural style that defines a set of constraints and properties based on HTTP. REST is known for its clear structure, readability, standardization and scalability. Its advantages are as follows:

+ Each URL represents one web resource in RESTful architecture;
+ Acting as a representation of resources between client and server;
+ Client is enabled to operate server-side resources with 4 HTTP requests - representational state transfer.

Developers are recommended to use REST API to proceed spot trading and withdrawals.

# Encrypted Verification of API
## Generate an API Key

Before signing any request, you must generate an API key via BitMore’s official website 【User Center】- 【API】. After generating the key, there are three things you must bear in 
mind:
* API Key

* Secret

* Passphrase

API Key and Secret are randomly generated and provided.

## Initiate a Request

All REST requests must include the following headings:

* ACCESS-KEY api key as a string.
* ACCESS-SIGN uses base64-encoded signatures (see Signed Messages).
* ACCESS-TIMESTAMP is the timestamp of your request.
* ACCESS-PASSPHRASE is the password you specified when you generate the API key. 
* All requests should contain content like application/json and be valid JSON.

## Signature

The ACCESS-SIGN header is the output generated by using HMAC SHA256 to create the HMAC SHA256 using the BASE64 decoding secret key in the prehash string to generate timestamp + 
method + requestPath + "?" + queryString + body (where ‘+’ represents the string concatenation) and BASE64 encoded output. The timestamp value is the same as the ACCESS-TIMESTAMP 
header. This 
body is the request body string or omitted if there is no request body (usually the GET request). **This method should be capitalized.**

Remember that before using it as the key to HMAC, base64 decoding (the result is 64 bytes) is first performed on the 64-bit alphanumeric password string. In addition, the digest output is base64 encoded before sending the header.

User submitted parameters must be signed except for sign. First, the string to be signed is ordered according to the parameter name (first compare the first letter of all parameter names, in alphabetic order, if you encounter the same first letter, then you move to the second letter, and so on).

**For example, if we sign the following parameters**

```bash
curl "https://www.bitmore.top/api/v1/spot/ccex/orders?limit=100"       
```

```java
Timestamp = 1590000000.281
Method = "GET"
requestPath = "/api/v1/spot/ccex/orders"
queryString= "?limit=100"
body = ""
```

Generate the string to be signed   
```
Message = '1590000000.281/GET/api/v1/spot/ccex/orders?limit=100'
```
Then, the character to be signed is added with the private key parameters to generate the final character string to be signed.

For example:
```
hmac = hmac(secretkey, Message, SHA256)
Signature = base64.encode(hmac.digest())
```

## Select timestamp

The ACCESS-TIMESTAMP header must be the number of seconds since UTC's time Unix Epoch. Decimal values are allowed. Your timestamp must be within 30 seconds of the API service time, otherwise your request will be considered expired and rejected. If you think there is a large time difference between your server and the API server, then we recommend that you use the time point to check the API server time.

## Request Process 
  
The root URL for REST access：`https://www.bitmore.top`

### Request
All requests are based on Https protocol, contentType in the request header must be uniformly set as: ‘application/json’.

**Request Process Descriptions**

1. Request parameter: parameter encapsulation based on the port request.

2. Submitting request parameter: submit the encapsulated parameter request to the server via POST/GET/PUT/DELETE or other methods.

3. Server response: the server will first perform a security validation, then send back the requested data to the client in JSON format.

4. Data processing: processing server response data.

**Success**

HTTP status code 200 indicates a successful response and may contain content. If the response contains content, it will appear in the corresponding returned content.

**Common Error Code**

* 400 Bad Request – Invalid request format

* 401 Unauthorized – Invalid API Key

* 403 Forbidden – You do not have access to the requested resource

* 404 Not Found

* 429 Too Many Requests

* 500 Internal Server Error – We had a problem with our server

### Pagination
All REST requests that return datasets use cursor-based pagination.

Cursor-based pagination allows results to be obtained before and after the current page of the result and is well suited for real-time data. The subsequent requests can specify the direction of the requested data based on the current returned results, before and/or after it. The before and after cursors can be used by the response headers CB-BEFORE and CB-AFTER. 

**For example:**

`GET /orders?before=2&limit=30`

## Standards and Specification

### Timestamp

Unless otherwise specified, all timestamps in APIs are returned in microseconds.

### For example,

1524801032573

### Numbers

In order to maintain the accuracy of cross-platform, decimal numbers are returned as strings. We suggest that you might be better to convert the number to string when issuing the request to avoid truncation and precision errors. Integers (such as transaction number and sequence) do not need quotation marks.

### Rate Limits

When a rate limit is exceeded, a status of 429 Too Many Requests will be returned.

#### REST API

* Public interface: We limit the invocation of public interface via IP: up to 6 requests every 2s.

* Private interface: We limit the invocation of private interface via user ID: up to 6 requests every 2s.

* Special restrictions on specified interfaces are specified.

# Spot API Reference

## Spot Market API

### 1. Access the list of all trading pairs

**HTTP Request**

```http
    # Request
    GET /api/v1/spot/public/products
```
```javascript
    # Response
    [
        {
            "baseCurrency":"LTC",
            "baseMaxSize":"100000.00",
            "baseMinSize":"0.001",
            "code":"LTC_BTC",
            "quoteCurrency":"BTC",
            "quoteIncrement":"0"
        },
        {	"baseCurrency":"ETH",
            "baseMaxSize":"100000.00",
            "baseMinSize":"0.001",
            "code":"ETH_BTC",
            "quoteCurrency":"BTC",
            "quoteIncrement":"0"
        },
        ...
    ]
```

**Response Details**

| Field | Descirption |
| ----------|:-------:|
| code        | Trading pair code |
| base_currency   | Base currency |
| quote_currency  | Quote currency |
| base_min_size   | Minimum Transaction Volume |
| base_max_size   | Maximum Transaction Volume |
| quote_increment | Minimum Precision |

### 2. Access the depth table of trading pairs

**HTTP Request**

```http
    # Request
    GET /api/v1/spot/public/products/<code>/orderbook
```
```javascript
    # Response
    {
        "asks":[
            [
                "10463.3399",
                "0.0025"
            ],
            ...
        ],
        "bids":[
            [
                "7300.2456",
                "0.0022"
            ],
            ...
        ]
    }
```
**Response Details**  


|Field|Description|  
|---- |------------|
| asks | depth of sellers |
| bids | depth of buyers |

**Request Paramters**

| Name | Type  | Requited | Description |
| ------------- |-----|-----|-----|
| Code | String | Y | Trading Pair, e.g. ltc_btc |

### 3. Access the ticker of a trading pair

**HTTP Request**

    The snapshot of the latest price, the highest bid price, the lowest ask price and 24-hour trading volume.

```http
    # Request
    GET /api/v1/spot/public/products/<code>/ticker
```

```javascript
    # Response
    [
        1526268156264,
        "8823.352",
        "8121.9873",
        "8749.9604",
        "8260",
        "8481",
        "487.8924"
    ]
```

**Response Details (from the top down)**

|Field|Description|
|--------| :-------: |
|timestamp | 1526268156264 |
| 24hr Highest|8823.352|
| 24hr Lowest|8121.9873|
| ask | 8749 |
| bid |8260|
|latest price|8481|
|24h Vol|487.8924|

**Request Parameter**

|Name|Type|Required|Description| 
|------|-----|-----|-----|
|code|String|Y|Trading Pair, e.g. btc-usdt|

### 4. Access the market trading records of a trading pair

    The request supports pagination.

**HTTP Request**
```http
    # Request
    GET /api/v1/spot/public/products/<code>/fills
```
```javascript
    # Response
    [
        [
            "0.00329999",
            "10.99999999",
            "buy",
            1524801032573,
            64
        ],
        [
            "10",
            "0.02521534",
            "sell",
            1524801032573,
            62
        ]
    ]
```
**Response Description (In order)**

|Field|Description|
|--------|-----|
|Execution Price |0.00329999|
|Volume |10.99999999|
|Maker Side|Buy|
|Timestamp| 1524801032573|
|Transaction ID| 62|

**Request Paramters**

|Name|Type|Required|Description| 
|-----|-----|-----|-----| 
|code|String|Y|Trading pair, e.g. btc-usdt|

    **Explanation**

    + Side indicates that the direction of the order the maker places. Maker refers to a trader who places orders in the market, a marker is a passive transaction party

    + Buy suggests price fall, because the maker places a buy order and the order is executed, the price falls; in contrary, sell suggests price rise, because the maker places a sell order and the order is executed, the price rises.

### 5. Access Candlestick chart

**HTTP Request**

```http
    # Request
    GET  /api/v1/spot/public/products/<code>/candles?type=1min&start=start_time&end=end_time
```
```javascript
    # Response
    {
        [ 1415398768, 0.32, 0.42, 0.36, 0.41, 12.3 ]
        ...
    }
```

**Response Details (in order)**
    
|Field|Description|
|-----|-----|
|Start timestamp|1415398768|
|The lowest price|0.32|
|The highest price|0.42|
|Opening price|0.36|
|Closing price|0.41|

**Request parameters**
    
|Name|Type|Required|Description|
|-----|-----|-----|-----|
|code|String|Y|Trading pair, e.g.btc-usdt|
|type|String|Y|Candlestick chart period type, e.g.1min/1hour/day/week/month|
|start|String|Y|Opening time based on ISO 8601|
|end|String|Y|Closing time based on ISO 8601|

### 6. Access Server Time

    Access API server time. This interface does not require ID authentication.

**HTTP Request**
```http
    # Request
    
    GET /api/v1/spot/time
```
    
```javascript
    # Reponse

    {
        "iso": "2015-01-07T23:47:25.201Z",
        "epoch": 1524801032573
    }
```
    
**Response Description**
    
|Field|Description|  
|------|-----|  
|iso|server time expressed in time string by ISO 8061|
|epoch|server time expressed in timestamp|

       iso: Response is returned in time string by ISO 8061
       epoch: Response is retured in timestamp

    It is an API for accessing all the available trading pairs and their trading parameters.

## Spot Account API

### 1. Access account information

    Access the list of balance, inquiry of coin balances, freezing status and available fund in spot account.

**HTTP Request**
```
    # Request
    GET /api/v1/spot/ccex/account/assets
```
```
    # Response
    [
        {
            "available":"0.1",
            "balance":"0.1",
            "currencyCode":"ETH",
            "frozen":"0",
            "id":1
        },
        {
            "available":"1",
            "balance":"1",
            "currencyCode":"USDT",
            "frozen":"0",
            "id":1
        }
    ]
```

**Response Details**

|Field|Description|
|-----|-----|
|available|Avaliable Fund|
|balance|Number of coins in balance|
|currencyCode|Coin symbol|
|frozen|Frozen fund|
|id|Account ID|

### 2. Order Placement

    There are two categrories of orders that can be placed on BitMore -- limit order and market order.

**HTTP Request**
```
    # Request

    POST /api/v1/spot/ccex/orders
```

```javascript
    # Response

    {
        "result": true,
        "order_id": 123456
    }
```
    
    **Response Details**

    + orderId: Order ID
    + result: the result of the order placed

**Request Paramters**

|Name| Type | Required | Description |
|----|----|-----|-----|
|code|String|Y|Trading pair, e.g.btc-usdt|
|side|String|N|buy or sell|
|type|String|Y|limit order or market order|
|size|String|N|delivered when a limit order or selling market order if placed,representing the number of coins for trading|
|price|String|N|delivered when a limit order is placed, representing the price of the pair
|funds|String|N|delievered then a market order is placed, representing the number of quote currencies


### 3. Cancel all orders

    Cancel all unfilled orders of the target trading pair.

**HTTP Request**
```
    # Request
    DELETE /api/v1/spot/ccex/orders
```
```javascript
    # Response

    { ...}
```

**Request Paramters**

|Name|Paramters|Type|Description|
|----|-----| -----| -----|
|code|String|Y|Trading pairs, e.g. btc-usdt|

### 4. Cancel a specified order

    Cancel a specified order by order ID

**HTTP Request**

```http
    # Request
    DELETE /api/v1/spot/ccex/orders/orderId
```
```javascript
    # Response
    {...}
```

**Request Paramters**

|Name|Type|Required|Description|
|-----|-----|-----|-----|
|code|String|Y|Trading Pair, e.g. btc-usdt|
|orderId|String|Y|The ID of an unfilled order specified need to be cancelled|

### 5. Search orders

    Check all the orders by order status.
    
**HTTP Request**

```http   
    # Request
    POST /api/v1/spot/ccex/orders?code=eth_btc&status=open
```
```javascript
    # Response
    {
        "averagePrice": "0",
        "code": "chp-eth",
        "createdDate": 1526299182000,
        "filledVolume": "0",
        "funds": "0",
        "orderId": 9865872,
        "orderType": "limit",
        "price": "0.00001",
        "side": "buy",
        "status": "canceled",
        "volume": "1"
    }
```

**Response Details**

|Field|Description|
|-----|-----|
|averagePrice|average price for the filled orders; 0 for the unfilled orders|
|code|Trading pair, e.g.btc-usdt|
|createDate|Timestamp upon the placement of the order|
|filledVolume|the volume of the filled orders|
|funds|the amount of the filled|
|orderId|Order ID|
|price|Price set for the order|
|side|Order direction|
|status|Order Status|
|volume|Volume of coins in the order placed|

**请求参数**

|Name | Type | Required | Description |
|------|-----|-----|-----|
|code|String|Y|Trading pair, e.g.btc-usdt|
|status|String|Y| Order Status:open,filled,canceled,cancel,partially-filled|

### 6. Order inquiry by Order ID

    Inquiry of a specified order by order ID

**HTTP Request**
```http
    # Request
    POST /api/v1/spot/ccex/orders/9887828?code=chp-eth
```
```javascript
    # Response 
    {
        "averagePrice":"0",
        "code":"chp-eth",
        "createdDate":9887828,
        "filledVolume":"0",
        "funds":"0",
        "orderId":9865872,
        "orderType":"limit",
        "price":"0.00001",
        "side":"buy",
        "status":"canceled",
        "volume":"1"
    }
```

**Response Details**
    
|Field|Description|
|------|-----|
|averagePrice|average price for the filled orders; 0 for the unfilled orders|
|code|Trading pair, e.g.btc-usdt|
|createDate|Timestamp upon the placement of the order|
|filledVolume|the volume of the filled orders|
|funds|the amount of the filled|
|orderId|Order ID|
|price|Price set for the order|
|side|Order direction|
|status|Order Status|
|volume|Volume of coins in the order placed|

**Request Paramters**
    
|Name|Type|Required|Description
|-----|-----|-----|-----
|code|String|Y|Trading pair, e.g.btc-usdt|
|orderId|String|Y|Order Id|

### 7. Access the account statement

    Access the statement of a spot account

**HTTP Request**
```http
    # Request
    GET /api/v1/spot/ccex/account/assets/eth/ledger
```
```javascript
    # Response
    {
        "amount": "0.00106415",
        "balance": "0.65106415",
        "createdDate": 1526290483000,
        "details": {
            "orderId":9772566,
            "productId":"ETH_BTC"
        },
        "id": 27826010,
        "type": "buy"
    }
```

**Response Details**

|Field | Description |
|-----|-----|
|amount|Volume of coins traded on the statement|
|balance|Statement balance|
|createdDate|Timestamp on the statement taking place|
|details|Statement Details|
|orderId|Order ID|
|productId|Product ID|
|id|Statement ID|
|type|Transaction Type|

**Request Paramters**

|Name|Type|Required|Description|
|----|-----|-----|-----|
|code|String|Y| Trading pair, e.g.btc-usdt|

### 8. Withdrawal

    Withdraw to your wallet address.

**HTTP Request**

```http
    # Request
    POST /api/v1/spot/ccex/account/withdraw
```
```javascript
    # Response
    { ... }
```

**Request Parameters**

|Name|Type|Required|Description
|-----|-----|-----|-----|
|currencyCode|String|Y|Name of coin to be withdrawn, e.g.BTC|
|amount|String|Y|Withdraw amount|
|address|String|Y| Withdraw address|


    
[BitMore]: https://www.bitmore.top 
[简体中文版文档]: https://github.com/bitmore-top/bitmore-official-api-docs/blob/master/README_ZH_CN.md
