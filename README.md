![image alt text](logo.svg)

**Blockchain.io API documentation v.0.1**

# **Summary**

[API Base URL](#api-base-url)

[Authentication](#authentication)

[Security](#security)

[Manual Refresh Token Workflow](#manual-refresh-token-workflow)

[API V2 Members Me](#apiv2membersme)

[API V2 Markets](#apiv2markets)

[API V2 Tickers Market](#apiv2tickersmarket)

[API V2 Tickers](#apiv2tickers)

[API V2 Accounts](#apiv2accounts)

[API V2 Accounts Currency](#apiv2accountscurrency)

[API V2 Deposits](#apiv2deposits)

[API V2 Deposit](#apiv2deposit)

[API V2 Deposit Address](#apiv2deposit_address)

[API V2 Orders Clear](#apiv2ordersclear)

[API V2 Orders](#apiv2orders)

[API V2 Orders multi](#apiv2ordersmulti)

[API V2 Order Delete](#apiv2orderdelete)

[API V2 Order](#apiv2order)

[API V2 Order Book](#apiv2order_book)

[API V2 Depth](#apiv2depth)

[API V2 Trades My](#apiv2tradesmy)

[API V2 Trades](#apiv2trades)

[API V2 K](#apiv2k)

[API V2 K with pending trades](#apiv2k_with_pending_trades)

[API V2 Timestamp](#apiv2timestamp)

[API V2 Fees Trading](#apiv2feestrading)

[API V2 Fees Deposit](#apiv2feesdeposit)

[API V2 Fees Withdraw](#apiv2feeswithdraw)

[API V2 Member Levels](#apiv2member_levels)

[API V2 Currency Trades](#apiv2currencytrades)

[API V2 Currencies](#apiv2currencies)

[API V2 Currencies Id](#apiv2currenciesid)


## **API Base URL**

**[https://api.blockchain.io](https://api.beta.blockchain.io)**

* * *


## **Authentication**

**/api/v1/sessions**

* * *


**_POST_**

**Description:**  POST request with email, password, application_id and otp_code to get the JWT token as a response for using it in others requests.

**Parameters**

|Name|Located In|Description|Required|Schema|
|----|----------|-----------|--------|------|
|email|formData|User e-mail|Yes|string|
|password|formData|User password (plein text)|Yes|string|
|application_id|formData|Blockchain.io Application ID : bc2d8526e1605579340d994ef0d6|Yes|string|
|otp_code|formData|2FA Google Authentificator code|Yes|string|

**Example**

**cURL :**

```bash
  curl --request POST \  --url 'https://auth.blockchain.io/api/v1/sessions?email={{user_email}}&password={{user_password}}&application_id=bc2d8526e1605579340d994ef0d6&otp_code={{otp_code}}' \  --header 'content-type: multipart/form-data; boundary=----WebKitFormBoundary7MA4YWxkTrZu0gW'
```

**Ruby :**

```ruby
require 'uri'require 'net/http'url = URI("https://auth.blockchain.io/api/v1/sessions?email={{user_email}}&password={{user_password}}&application_id=bc2d8526e1605579340d994ef0d6&otp_code={{otp_code}}")http = Net::HTTP.new(url.host, url.port)request = Net::HTTP::Post.new(url)response = http.request(request)puts response.read_body</td>
```

**Response**

| Example of a Authentication token (JWT token) response | Type |
|----------|----------|
|eyJhbGciOiJSUzI1NiJ9.eyJpYXQiOjE1MzYyNDE4NDMsImV4cCI6MTUzNjI1NjI0Mywic3ViIjoic2Vzc2lvbiIsImlzcyI6ImJhcm9uZyIsImF1ZCI6WyJwZWF0aW8iLCJiYXJvbmciXSwianRpIjoiQTZGOUQ2OUQzODExQzA1Rjg1REE3RDcwIiwidWlkIjoiSUQ2Mjc1NTAwNDE2IiwiZW1haWwiOiJzYW11ZWwuZ29tZXNAcGF5bWl1bS5jb20iLCJyb2xlIjoiYWRtaW4iLCJsZXZlbCI6NCwic3RhdGUiOiJhY3RpdmUifQ.jQoke1um5PIBiB_A1R5DNgnPhM2WO1Hgqa0bbOzMt8nKHcNBCL_KTDz9iVPJ1gVT0Z1_6Ak_oy_-SIbXLqdcn9tGiC3DR4eEXDqol84AIkNIgR9XIiQtwBESBDnK4ZLlBSGHKHu2bvUSsAm9r2eVwu0c-7L0YhxTt44cOJaHc9UKm00xL8iWsctTAFYX6YOZ265fi7wHj-JmMqtu7euU9cNtPBq-apv4gMJsTkGg-smsMCAhgp31gkCJpSvjyhDZCFXDMHCan7tT_GjVdGc66X6gllpBreghXxZD7sbBjcEdOI6fWxScYOCKaA4ppFudVGk0z5ASYNMWOpqrvYhMZw|string|

## **Security**

* * *

**Bearer**

<table>
  <tr>
    <td>apiKey</td>
    <td>API Key</td>
  </tr>
  <tr>
    <td>Name</td>
    <td>JWT</td>
  </tr>
  <tr>
    <td>In</td>
    <td>header</td>
  </tr>
</table>


## Manual refresh token workflow

**1 : Generate Private and Public keys using Ruby :**

```bash
ruby -e "require 'openssl'; require 'base64'; OpenSSL::PKey::RSA.generate(2048).tap { |p| puts '', 'PRIVATE RSA KEY (URL-safe Base64 encoded, PEM):', '', Base64.urlsafe_encode64(p.to_pem), '', 'PUBLIC RSA KEY (URL-safe Base64 encoded, PEM):', '', Base64.urlsafe_encode64(p.public_key.to_pem) }"</td>
```

**2 : Save both public and private keys**

**3 : Create a API_KEY (only once) :**

```bash
curl -X POST -H 'Content-Type: application/json' -H "Authorization: Bearer THE_EXPIRED_OR_CURRENT_JWT_TOKEN" -d "{\"public_key\":\"YOUR_PUBLIC_KEY\", \"scopes\":\"peatio\", \"totp_code\":\"YOUR_2FA_CODE\"}" https://auth.blockchain.io/api/v1/api_keys?totp_code=YOUR_2FA_CODE
```


Save the API_KEY

**4 : Use this ruby script to sign the private key :**

file : sign_script.rb

```ruby
require 'openssl'require 'base64'require 'json'require 'securerandom'require 'jwt'require 'active_support/time'secret_key = OpenSSL::PKey.read(Base64.urlsafe_decode64('YOUR_PRIVATE_KEY'))payload = {  iat: Time.current.to_i,  exp: 20.minutes.from_now.to_i,  sub: 'api_key_jwt',  iss: 'external',  jti: SecureRandom.hex(12).upcase}jwt_token = JWT.encode(payload, secret_key, 'RS256')puts jwt_token.to_s</td>
```

**5 : Save the script (ex : sign_script.rb)and Run it :**

```bash
  ruby sign_script.rb
```

It will output the JWT Token Request

**6 : Get the final JWT Token for using in[ blockchain.io](http://blockchain.io/) API calls :**

```bash
curl -X POST -H 'Content-Type: application/json' -d '{"kid":"YOUR_API_KEY", "jwt_token": "GENERATED_JWT_TOKEN_REQUEST"}' https://auth.blockchain.io/api/v1/sessions/generate_jwt
```

**Response**

|Example of a Authentication token (JWT token) response given by a token key| Type|
|-----|-----|
|`{"token":"eyJhbGciOiJSUzI1NiJ9.eyJpYXQiOjE1Mzc0NTgwMTgsImV4cCI6MTUzNzU0NDQxOCwic3ViIjoic2Vzc2lvbiIsImlzcyI6ImJhcm9uZyIsImF1ZCI6InRyYWRlIiwianRpIjoiQ0UxQUZEMEZEMDdFQUE1RDUyMjRFRkRDIiwidWlkIjoiSUQ2Mjc1NTAwNDE2IiwiZW1haWwiOiJzYW11ZWwuZ29tZXNAcGF5bWl1bS5jb20iLCJyb2xlIjoiYWRtaW4iLCJsZXZlbCI6Mywic3RhdGUiOiJhY3RpdmUiLCJhcGlfa2lkIjoiM2Q2ODVhN2ItYWFlNS00YTZiLWJkYWItNjRhNmUzYjFhM2I2In0.x7myykg1ikdE1G1sckYCWUG7OUMi_SwaQjmCDki_fSEmBumuNE8kLwuve_wrAjzaQamxIVM6-Yu7mVvhFN5ennP0PpK1pvf45FCnYRCQGbgqyggi4VgNuKkdzvvVRh4xQ2P3iNGrAXdr-Hs5VzLL2ZsSHSoSr8LQSaq9JOZsXL31_7eWjpR8dS_I7G5pCkc8y56Iurgu4hE0uzkS31Sa_Vc6DKFSIEfvJbTHIalexBZlMx7thiH6ojPdjGLu-Vq_G3tPo4rPZ61qWLRrC5FsBps14P9sr2Cnjr5lk2yf4FF5GuL6jxAcEdu1FiVmRO3_QFdogC7l0R8GtobK-qqCkw"}`|string|


## /api/v2/members/me

* * *


**_GET_**

**Description:** Get your profile and accounts info.

**Requirements:** Authorization Bearer Token

**Example**

**cURL:**

```bash
curl --request GET \  --url https://api.blockchain.io/api/v2/members/me \  --header 'Authorization: Bearer {{JWT_TOKEN}}'
```

**Ruby:**

```bash
require 'uri'require 'net/http'url = URI("https://api.blockchain.io/api/v2/members/me")http = Net::HTTP.new(url.host, url.port)request = Net::HTTP::Get.new(url)request["Authorization"] = 'Bearer {{JWT_TOKEN}}'response = http.request(request)puts response.read_body
```    


**Responses**

|Code|Description|
|-------|---------|
|200|Get your profile and accounts info.|


## **/api/v2/markets**

* * *

**_GET_**

**Description:** Get all available markets.

**Requirements:** None

**Example**

**cURL:**

```bash
curl --request GET \  --url https://api.blockchain.io/api/v2/markets
```

**Ruby:**

```ruby
require 'uri'require 'net/http'url = URI("https://api.blockchain.io/api/v2/markets")http = Net::HTTP.new(url.host, url.port)request = Net::HTTP::Get.new(url)response = http.request(request)puts response.read_body
```

**Responses**

|Code|Description|
|-------|---------|
|200|Get all available markets.|


## /api/v2/**tickers/{market}**

* * *

**_GET_**

**Description:** Get ticker of specific market.

**Requirements:** None

**Parameters**

|Name|Located In|Description|Required|Schema|
|----|----------|-----------|--------|------|
|market|path|Unique market id. It's always in the form of xxxyyy, where xxx is the base currency code, yyy is the quote currency code, e.g. 'ETHBTC'. All available markets can be found at /api/api/v2/markets.|Yes|String|


**Example**

**cURL:**

```bash
curl --request GET \  --url https://api.blockchain.io/api/v2/tickers/ethbtc
```

**Ruby:**
```ruby
require 'uri'require 'net/http'url = URI("https://api.blockchain.io/api/v2/tickers/ethbtc")http = Net::HTTP.new(url.host, url.port)request = Net::HTTP::Get.new(url)response = http.request(request)puts response.read_body
```

**Responses**

|Code|Description|
|-------|---------|
|200|Get ticker of specific market.|


## /api/v2/tickers

* * *


**_GET_**

**Description:** Get ticker of all markets.

**Requirements:** None

**Example**

**cURL:**

```bash
curl --request GET \  --url https://api.blockchain.io/api/v2/tickers
```

**Ruby:**

```ruby
require 'uri'require 'net/http'url = URI("https://api.blockchain.io/api/v2/tickers")http = Net::HTTP.new(url.host, url.port)request = Net::HTTP::Get.new(url)response = http.request(request)puts response.read_body
```


**Responses**

|Code|Description|
|-------|---------|
|200|Get ticker of all markets.|


## /api/v2/**accounts**

* * *


**_GET_**

**Description:** Get your accounts info.

**Requirements:** Authorization Bearer Token

**Example**

**cURL:**

```bash
curl --request GET \  --url https://api.blockchain.io/api/v2/accounts \  --header 'Authorization: Bearer {{JWT_TOKEN}}'
```

**Ruby:**

```ruby
require 'uri'require 'net/http'url = URI("https://api.blockchain.io/api/v2/accounts")http = Net::HTTP.new(url.host, url.port)request = Net::HTTP::Get.new(url)request["Authorization"] = 'Bearer {{JWT_TOKEN}}'response = http.request(request)puts response.read_body
```


**Responses**

|Code|Description|
|-------|---------|
|200|Get your profile and accounts info.|


## /api/v2/**accounts/{currency}**

* * *


**_GET_**

**Description:** Get your accounts info filtered by currency.

**Requirements:** Authorization Bearer Token

**Example**

**cURL:**

```bash
curl --request GET \  --url https://api.blockchain.io/api/v2/accounts/btc \  --header 'Authorization: Bearer {{JWT_TOKEN}}'
```


**Ruby:**

```ruby
require 'uri'require 'net/http'url = URI("https://api.blockchain.io/api/v2/accounts/btc")http = Net::HTTP.new(url.host, url.port)request = Net::HTTP::Get.new(url)request["Authorization"] = 'Bearer {{JWT_TOKEN}}'response = http.request(request)puts response.read_body
```


**Responses**

|Code|Description|
|-------|---------|
|200|Get your accounts info filtered by currency.|

## /api/v2/**deposits**

* * *


**_GET_**

**Description:** Get your deposits history.

**Requirements:** Authorization Bearer Token

**Parameters**

|Name|Located In|Decription|Required|Schema|
|----|----------|----------|--------|------|
|currency|query|Currency value contains BCH, BTC, ETH, LTC, XRP|No|string|
|limit|query|Set result limit.|No|integer|
|state|query|Filter by "submitted", "canceled", "rejected" or "accepted"|No|string|


**Example**

**cURL:**

```bash
curl --request GET \  --url 'https://api.blockchain.io/api/v2/deposits?currency=btc&limit=10&state=accepted' \  --header 'Authorization: Bearer {{JWT_TOKEN}}'
```


**Ruby:**

```ruby
require 'uri'require 'net/http'url = URI("https://api.blockchain.io/api/v2/deposits?currency=btc&limit=10&state=accepted")http = Net::HTTP.new(url.host, url.port)request = Net::HTTP::Get.new(url)request["Authorization"] = 'Bearer {{JWT_TOKEN}}'response = http.request(request)puts response.read_body
```


**Responses**

|Code|Description|
|-------|---------|
|200|Get your deposits history.|


## /api/v2/deposit

* * *


**_GET_**

**Description:** Get details of specific deposit.

**Requirements:** Authorization Bearer Token

**Parameters**

|Name|Located In|Decription|Required|Schema|
|----|----------|----------|--------|------|
|txid|query||Yes|string|


**Example**

**cURL:**

```bash
curl --request GET \  --url 'https://api.blockchain.io/api/v2/deposit?txid={{TXID}}' \  --header 'Authorization: Bearer {{JWT_TOKEN}}'
```


**Ruby:**

```ruby
require 'uri'require 'net/http'url = URI("https://api.blockchain.io/api/v2/deposit?txid={{TXID}}")http = Net::HTTP.new(url.host, url.port)request = Net::HTTP::Get.new(url)request["Authorization"] = 'Bearer {{JWT_TOKEN}}'response = http.request(request)puts response.read_body
```


**Responses**

|Code|Description|
|-------|---------|
|200|Get details of specific deposit.|


## /api/v2/**deposit_address**

* * *


**_GET_**

**Description:** Returns deposit address for account you want to deposit to. The address may be blank because address generation process is still in progress. If this case you should try again later.

**Requirements:** Authorization Bearer Token

**Parameters**

|Name|Located In|Decription|Required|Schema|
|----|----------|----------|--------|------|
|currency|query|The account you want to deposit to.|Yes|string|
|address_format|query|Address format legacy/cash|No|string|


**Example**

**cURL:**

```bash
curl --request GET \  --url 'https://api.blockchain.io/api/v2/deposit_address?currency=eth' \  --header 'Authorization: Bearer {{JWT_TOKEN}}'
```


**Ruby:**

```bash
require 'uri'require 'net/http'url = URI("https://api.blockchain.io/api/v2/deposit_address?currency=eth")http = Net::HTTP.new(url.host, url.port)request = Net::HTTP::Get.new(url)request["Authorization"] = 'Bearer {{JWT_TOKEN}}'response = http.request(request)puts response.read_body
```


**Responses**


|Code|Description|
|-------|---------|
|200|Returns deposit address for account you want to deposit to. The address may be blank because address generation process is still in progress. If this case you should try again later.|


## **/api/v2/orders/clear**

* * *


**_POST_**

**Description:** Cancel all my orders.

**Requirements:** Authorization Bearer Token

**Parameters**

|Name|Located In|Description|Required|Schema|
|----|----------|-----------|--------|------|
|side|formData|If present, only sell orders (asks) or buy orders (bids) will be cancelled.|No|String|


**Example**

**cURL:**

```bash
curl --request POST \  --url 'https://api.blockchain.io/api/v2/orders/clear?side=asks' \  --header 'Authorization: Bearer {{JWT_TOKEN}}'
```

**Ruby:**

```ruby
require 'uri'require 'net/http'url = URI("https://api.blockchain.io/api/v2/orders/clear?side=asks")http = Net::HTTP.new(url.host, url.port)request = Net::HTTP::Post.new(url)request["Authorization"] = 'Bearer {{JWT_TOKEN}}'response = http.request(request)puts response.read_body</td>
```

**Responses**

|Code|Description|
|-------|---------|
|201|Cancel all my orders.|


## **/api/v2/orders**

* * *


**_POST_**

**Description:** Create a Sell/Buy order.

**Requirements:** Authorization Bearer Token

**Parameters**


|Name|Located In|Description|Required|Schema|
|----|----------|-----------|--------|------|
|market|formData|Unique market id. It's always in the form of xxxyyy, where xxx is the base currency code, yyy is the quote currency code, e.g. 'ETHBTC'. All available markets can be found at /api/api/v2/markets.|Yes|String|
|side|formData|Either 'sell' or 'buy'.|Yes|string|
|volume|formData|The amount user want to sell/buy. An order could be partially executed, e.g. an order sell 5 btc can be matched with a buy 3 btc order, left 2 btc to be sold; in this case the order's volume would be '5.0', its remaining_volume would be '2.0', its executed volume is '3.0'.|Yes|float|
|order_type|formData|Either 'market' or 'limit' |No|string|
|price|formData|Price for each unit. e.g. If you want to sell/buy 1 btc at 3000 usd, the price is '3000.0'|Yes|float|


**Example**

**cURL:**

```bash
curl --request POST \  --url https://api.blockchain.io/api/v2/orders \  --header 'content-type: multipart/form-data; boundary=----WebKitFormBoundary7MA4YWxkTrZu0gW' \  --form market=xrpbtc \  --form side=sell \  --form volume=5 \  --form order_type=limit \  --form price=0.1
```


**Ruby:**

```ruby
require 'uri'require 'net/http'url = URI("https://api.blockchain.io/api/v2/orders")http = Net::HTTP.new(url.host, url.port)request = Net::HTTP::Post.new(url)request.body = "------WebKitFormBoundary7MA4YWxkTrZu0gW\r\nContent-Disposition: form-data; name=\"market\"\r\n\r\nxrpbtc\r\n------WebKitFormBoundary7MA4YWxkTrZu0gW\r\nContent-Disposition: form-data; name=\"side\"\r\n\r\nsell\r\n------WebKitFormBoundary7MA4YWxkTrZu0gW\r\nContent-Disposition: form-data; name=\"volume\"\r\n\r\n5\r\n------WebKitFormBoundary7MA4YWxkTrZu0gW\r\nContent-Disposition: form-data; name=\"order_type\"\r\n\r\nlimit\r\n------WebKitFormBoundary7MA4YWxkTrZu0gW\r\nContent-Disposition: form-data; name=\"price\"\r\n\r\n0.1\r\n------WebKitFormBoundary7MA4YWxkTrZu0gW--"response = http.request(request)puts response.read_bodyhttp = Net::HTTP.new(url.host, url.port)request = Net::HTTP::Get.new(url)request["Authorization"] = 'Bearer {{JWT_TOKEN}}'response = http.request(request)puts response.read_body
```


**Responses**


|Code|Description|
|-------|---------|
|201|Create a Sell/Buy order.|


**_GET_**

**Description:** Get your orders, results is paginated.

**Requirements:** Authorization Bearer Token

**Parameters**

|Name|Located In|Description|Required|Schema|
|----|----------|-----------|--------|------|
|market|query|Unique market id. It's always in the form of xxxyyy, where xxx is the base currency code, yyy is the quote currency code, e.g. 'ETHBTC'. All available markets can be found at /api/api/v2/markets.|Yes|string|
|state|query|Filter order by "wait", "done" or "cancel"|No|string|
|limit|query|Limit the number of returned orders, default to 100.|No|integer|
|page|query|Specify the page of paginated results.|No|integer|
|order_by|query|If set, returned orders will be sorted in specific order, default to 'asc'.|No|string|

**Example**

**cURL:**

```bash
curl --request GET \  --url 'https://api.blockchain.io/api/v2/orders?market=ethbtc&state=wait&limit=10 \  --header 'Authorization: Bearer {{JWT_TOKEN}}'
```


**Ruby:**

```ruby
require 'uri'require 'net/http'url = URI("https://api.blockchain.io/api/v2/orders?market=ethbtc&state=wait&limit=10")http = Net::HTTP.new(url.host, url.port)request = Net::HTTP::Get.new(url)request["Authorization"] = 'Bearer {{JWT_TOKEN}}'response = http.request(request)puts response.read_body
```


**Responses**

|Code|Description|
|-------|---------|
|200|Get your orders, results is paginated.|


## **/api/v2/orders/multi**

* * *


**_POST_**

**Description:** Create multiple sell/buy orders.

**Requirements:** Authorization Bearer Token

**Parameters**

|Name|Located In|Description|Required|Schema|
|----|----------|-----------|--------|------|
|market|formData|Unique market id. It's always in the form of xxxyyy, where xxx is the base currency code, yyy is the quote currency code, e.g. 'ETHBTC'. All available markets can be found at /api/api/v2/markets.|Yes|string|
|orders[side]|formData|Either 'sell' or 'buy'.|Yes|[ string ]|
|orders[volume]|formData|The amount user want to sell/buy. An order could be partially executed, e.g. an order sell 5 BTC can be matched with a buy 3 BTC order, left 2 BTC to be sold; in this case the order's volume would be '5.0', its remaining_volume would be '2.0', its executed volume is '3.0'.|Yes|[ float ]|
|orders[order_type]|formData||No|[ string ]|
|orders[price]|formData|Price for each unit. e.g. If you want to sell/buy 1 BTC at 10 ETH, the price is '10.0'|Yes|[ float ]|


**Example**

**cURL:**

```bash
curl --request POST \  --url https: /api.blockchain.io/api/v2/orders/multi \  --header 'Content-Type: application/json' \  --data '{ "market": "xrpbtc", "orders": [	 {		 "side": "sell",		 "volume": 0.1,		 "order_type": "limit",		 "price": 10	 } ]}
```


**Ruby:**

```ruby
require 'uri'require 'net/http'url = URI("https://api.blockchain.io/api/v2/orders/multi")http = Net::HTTP.new(url.host, url.port)request = Net::HTTP::Post.new(url)request["Content-Type"] = 'application/json'request.body = "{\n \"market\": \"xrpbtc\",\n \"orders\": [\n\t {\n\t\t \"side\": \"sell\",\n\t\t \"volume\": 0.1,\n\t\t \"order_type\": \"limit\",\n\t\t \"price\": 10\n\t }\n ]\n}\n "response = http.request(request)puts response.read_body
response = http.request(request)puts response.read_bodyhttp = Net::HTTP.new(url.host, url.port)request = Net::HTTP::Get.new(url)request["Authorization"] = 'Bearer {{JWT_TOKEN}}'response = http.request(request)puts response.read_body
```


**Responses**

|Code|Description|
|-------|---------|
|201|Create multiple sell/buy orders.|


## **/api/v2/order/delete**

* * *


**_POST_**

**Description:** Cancel an order.

**Requirements:** Authorization Bearer Token

**Parameters**

|Name|Located In|Description|Required|Schema|
|----|----------|-----------|--------|------|
|id|formData|Unique order id.|Yes|integer|


**Example**

**cURL:**

```bash
curl --request POST \  --url https://api.blockchain.io/api/v2/order/delete \  --header 'content-type: multipart/form-data; boundary=----WebKitFormBoundary7MA4YWxkTrZu0gW' \  --form id={{order_id}} \
```


**Ruby:**

```ruby
require 'uri'require 'net/http'url = URI("https://api.blockchain.io/api/v2/order/delete")http = Net::HTTP.new(url.host, url.port)request = Net::HTTP::Post.new(url)request.body = "------WebKitFormBoundary7MA4YWxkTrZu0gW\r\nContent-Disposition: form-data; name=\"id\"\r\n\r\n{{order_id}}\r\n------WebKitFormBoundary7MA4YWxkTrZu0gW\r\nContent-Disposition: form-data;
response = http.request(request)puts response.read_bodyhttp = Net::HTTP.new(url.host, url.port)request = Net::HTTP::Get.new(url)request["Authorization"] = 'Bearer {{JWT_TOKEN}}'response = http.request(request)puts response.read_body
```


**Responses**

|Code|Description|
|-------|---------|
|201|Cancel an order.|

## /api/v2/**order**

* * *


**_GET_**

**Description:** Get information of specified order.

**Requirements:** Authorization Bearer Token

**Parameters**

|Name|Located In|Description|Required|Schema|
|----|----------|-----------|--------|------|
|id|query|Unique order id.|Yes|integer|


**Example**

**cURL:**

```bash
curl --request GET \  --url 'https://api.blockchain.io/api/v2/order?id={{order_id}} \  
  --header 'Authorization: Bearer {{JWT_TOKEN}}'
```


**Ruby:**

```bash
require 'uri'require 'net/http'url = URI("https://api.blockchain.io/api/v2/order?id={{order_id}}")http = Net::HTTP.new(url.host, url.port)request = Net::HTTP::Get.new(url)request["Authorization"] = 'Bearer {{JWT_TOKEN}}'response = http.request(request)puts response.read_body
```


**Responses**

|Code|Description|
|-------|---------|
|200|Get information of specified order.|


## **/api/v2/order_book**

* * *


**_GET_**

**Description:** Get the order book of specified market.

**Requirements:** None

**Parameters**

|Name|Located In|Description|Required|Schema|
|----|----------|-----------|--------|------|
|market|query|Unique market id. It's always in the form of xxxyyy, where xxx is the base currency code, yyy is the quote currency code, e.g. 'ETHBTC'. All available markets can be found at /api/api/v2/markets.|Yes|string|
|asks_limit|query|Limit the number of returned sell orders. Default to 20.|No|integer|
|bids_limit|query|Limit the number of returned buy orders. Default to 20.|No|integer|


**Example**

**cURL:**

```bash
curl --request GET \  --url https://api.blockchain.io/api/v2/order_book?market=ethbtc&asks_limit=10
```


**Ruby:**

```ruby
require 'uri'require 'net/http'url = URI("https://api.blockchain.io/api/v2/order_book?market=ethbtc&asks_limit=10")http = Net::HTTP.new(url.host, url.port)request = Net::HTTP::Get.new(url)response = http.request(request)puts response.read_body
```


**Responses**

|Code|Description|
|-------|---------|
|200|Get the order book of specified market.|


## **/api/v2/depth**

* * *


**_GET_**

**Description:** Get depth or specified market. Both asks and bids are sorted from highest price to lowest.

**Requirements:** None

**Example**

**cURL:**

```bash
curl --request GET \  --url https://api.blockchain.io/api/v2/depth?market=ethbtc&limit=2
```


**Ruby:**

```ruby
require 'uri'require 'net/http'url = URI("https://api.blockchain.io/api/v2/depth?market=ethbtc&limit=2")http = Net::HTTP.new(url.host, url.port)request = Net::HTTP::Get.new(url)response = http.request(request)puts response.read_body
```


**Parameters**

|Name|Located In|Description|Required|Schema|
|----|----------|-----------|--------|------|
|market|query|Unique market id. It's always in the form of xxxyyy, where xxx is the base currency code, yyy is the quote currency code, e.g. 'ETHBTC'. All available markets can be found at /api/api/v2/markets.|Yes|string|
|limit|query|Limit the number of returned price levels. Default to 300.|No|integer|


**Responses**

|Code|Description|
|-------|---------|
|200|Get depth or specified market. Both asks and bids are sorted from highest price to lowest.|


## **/api/v2/trades/my**

* * *


**_GET_**

**Description:** Get your executed trades. Trades are sorted in reverse creation order.

**Requirements:** Authorization Bearer Token

**Parameters**

|Name|Located In|Description|Required|Schema|
|----|----------|-----------|--------|------|
|market|query|Unique market id. It's always in the form of xxxyyy, where xxx is the base currency code, yyy is the quote currency code, e.g. 'ETHBTC'. All available markets can be found at /api/api/v2/markets.|Yes|string|
|limit|query|Limit the number of returned trades. Default to 50.|No|integer|
|timestamp|query|An integer represents the seconds elapsed since Unix epoch. If set, only trades executed before the time will be returned.|No|integer|
|from|query|Trade id. If set, only trades created after the trade will be returned.|No|integer|
|to|query|Trade id. If set, only trades created before the trade will be returned.|No|integer|
|order_by|query|If set, returned trades will be sorted in specific order, default to 'desc'.|No|string|


**Example**

**cURL:**

```bash
curl --request GET \  --url 'https://api.blockchain.io/api/v2/trades/my?market=ethbtc&limit=100 \  
  --header 'Authorization: Bearer {{JWT_TOKEN}}'
```


**Ruby:**

```ruby
require 'uri'require 'net/http'url = URI("https://api.blockchain.io/api/v2/trades/my?market=ethbtc&limit=100")http = Net::HTTP.new(url.host, url.port)request = Net::HTTP::Get.new(url)request["Authorization"] = 'Bearer {{JWT_TOKEN}}'response = http.request(request)puts response.read_body
```


**Responses**

|Code|Description|
|-------|---------|
|200|Get your executed trades. Trades are sorted in reverse creation order.|


## **/api/v2/trades**

* * *


**_GET_**

**Description:** Get recent trades on market, each trade is included only once. Trades are sorted in reverse creation order.

**Requirements:** None

**Parameters**

|Name|Located In|Description|Required|Schema|
|----|----------|-----------|--------|------|
|market|query|Unique market id. It's always in the form of xxxyyy, where xxx is the base currency code, yyy is the quote currency code, e.g. 'ETHBTC'. All available markets can be found at v2/markets.|Yes|string|
|limit|query|Limit the number of returned trades. Default to 50.|No|integer|
|timestamp|query|An integer represents the seconds elapsed since Unix epoch. If set, only trades executed before the time will be returned.|No|integer|
|from|query|Trade id. If set, only trades created after the trade will be returned.|No|integer|
|to|query|Trade id. If set, only trades created before the trade will be returned.|No|integer|
|order_by|query|If set, returned trades will be sorted in specific order, default to 'desc'.|No|string|


**Example**

**cURL:**


```bash
curl --request GET \  --url https://api.blockchain.io/api/v2/trades?market=ethbtc&limit=50
```


**Ruby:**

```ruby
require 'uri'require 'net/http'url = URI("https://api.blockchain.io/api/v2/trades?market=ethbtc&limit=50")http = Net::HTTP.new(url.host, url.port)request = Net::HTTP::Get.new(url)response = http.request(request)puts response.read_body
```


**Responses**

|Code|Description|
|-------|---------|
|200|Get recent trades on market, each trade is included only once. Trades are sorted in reverse creation order.|


## **/api/v2/k**

* * *


**_GET_**

**Description:** Get OHLC(k line) of specific market.

**Requirements:** None

**Parameters**

|Name|Located In|Description|Required|Schema|
|----|----------|-----------|--------|------|
|market|query|Unique market id. It's always in the form of xxxyyy, where xxx is the base currency code, yyy is the quote currency code, e.g. 'ETHBTC'. All available markets can be found at /api/v2/markets.|Yes|string|
|limit|query|Limit the number of returned data points, default to 30.|No|integer|
|period|query|Time period of K line, default to 1. You can choose between 1, 5, 15, 30, 60, 120, 240, 360, 720, 1440, 4320, 10080|No|integer|
|time_from|query|An integer represents the seconds elapsed since Unix epoch. If set, only k-line data after that time will be returned.|No|integer|
|time_to|query|An integer represents the seconds elapsed since Unix epoch. If set, only k-line data till that time will be returned.|No|integer|


**Example**

**cURL:**

```bash
curl --request GET \  --url https://api.blockchain.io/api/v2/k?market=ethbtc&limit=50
```


**Ruby:**

```ruby
require 'uri'require 'net/http'url = URI("https://api.blockchain.io/api/v2/k?market=ethbtc&limit=50")http = Net::HTTP.new(url.host, url.port)request = Net::HTTP::Get.new(url)response = http.request(request)puts response.read_body
```


**Responses**

|Code|Description|
|-------|---------|
|200|Get OHLC(k line) of specific market.|


## /api/v2/**k_with_pending_trades**

* * *


**_GET_**

**Description:** Get K data with pending trades, which are the trades not included in K data yet, because there's delay between trade generated and processed by K data generator.

**Requirements:** None

**Parameters**


|Name|Located In|Description|Required|Schema|
|----|----------|-----------|--------|------|
|market|query|Unique market id. It's always in the form of xxxyyy, where xxx is the base currency code, yyy is the quote currency code, e.g. 'ETHBTC'. All available markets can be found at /api/api/v2/markets.|Yes|string|
|trade_id|query|The trade id of the first trade you received.|Yes|integer|
|limit|query|Limit the number of returned data points, default to 30.|No|integer|
|period|query|Time period of K line, default to 1. You can choose between 1, 5, 15, 30, 60, 120, 240, 360, 720, 1440, 4320, 10080|No|integer|
|time_from|query|An integer represents the seconds elapsed since Unix epoch. If set, only k-line data after that time will be returned.|No|integer|
|time_to|query|An integer represents the seconds elapsed since Unix epoch. If set, only k-line data till that time will be returned.|No|integer|


**Example**

**cURL:**

```bash
curl --request GET \  --url https://api.blockchain.io/api/v2/k_with_pending_trades?market=ethbtc&trade_id={{trade_id}}
```


**Ruby:**

```ruby
require 'uri'require 'net/http'url = URI("https://api.blockchain.io/api/v2/k_with_pending_trades?market=ethbtc&trade_id={{trade_id}}")http = Net::HTTP.new(url.host, url.port)request = Net::HTTP::Get.new(url)response = http.request(request)puts response.read_body
```


**Responses**

|Code|Description|
|-------|---------|
|200|Get K data with pending trades, which are the trades not included in K data yet, because there's delay between trade generated and processed by K data generator.|


## **/api/v2/timestamp**

* * *


**_GET_**

**Description:** Get server current time, in seconds since Unix epoch.

**Requirements:** None

**Example**

**cURL:**

```bash
curl --request GET \  --url https://api.blockchain.io/api/v2/timestamp
```


**Ruby:**

```ruby
require 'uri'require 'net/http'url = URI("https://api.blockchain.io/api/v2/timestamp")http = Net::HTTP.new(url.host, url.port)request = Net::HTTP::Get.new(url)response = http.request(request)puts response.read_body
```


**Responses**

|Code|Description|
|-------|---------|
|200|Get server current time, in seconds since Unix epoch.|


## **/api/v2/fees/trading**

* * *


**_GET_**

**Description:** Returns trading fees for markets.

**Requirements:** None

**Example**

**cURL:**

```bash
curl --request GET \  --url https://api.blockchain.io/api/v2/fees/trading
```


**Ruby:**

```ruby
require 'uri'require 'net/http'url = URI("https://api.blockchain.io/api/v2/fees/trading")http = Net::HTTP.new(url.host, url.port)request = Net::HTTP::Get.new(url)response = http.request(request)puts response.read_body
```


**Responses**

|Code|Description|
|-------|---------|
|200|Returns trading fees for markets.|


## **/api/v2/fees/deposit**

* * *


**_GET_**

**Description:** Returns deposit fees for currencies.

**Requirements:** None

**Example**

**cURL:**

```bash
curl --request GET \  --url https://api.blockchain.io/api/v2/fees/deposit
```


**Ruby:**

```ruby
require 'uri'require 'net/http'url = URI("https://api.blockchain.io/api/v2/fees/deposit")http = Net::HTTP.new(url.host, url.port)request = Net::HTTP::Get.new(url)response = http.request(request)puts response.read_body
```

**Responses**

|Code|Description|
|-------|---------|
|200|Returns deposit fees for currencies.|


## **/api/v2/fees/withdraw**

* * *


**_GET_**

**Description:** Returns withdraw fees for currencies.

**Requirements:** None

**Example**

**cURL:**

```bash
curl --request GET \  --url https://api.blockchain.io/api/v2/fees/withdraw
```


**Ruby:**

```ruby
require 'uri'require 'net/http'url = URI("https://api.blockchain.io/api/v2/fees/withdraw")http = Net::HTTP.new(url.host, url.port)request = Net::HTTP::Get.new(url)response = http.request(request)puts response.read_body
```


**Responses**

|Code|Description|
|-------|---------|
|200|Returns withdraw fees for currencies.|


## **/api/v2/member_levels**

* * *


**_GET_**

**Description:** Returns list of member levels and the privileges they provide.

**Requirements:** None

**Example**

**cURL:**

```bash
curl --request GET \  --url https://api.blockchain.io/api/v2/member_levels
```


**Ruby:**

```ruby
require 'uri'require 'net/http'url = URI("https://api.blockchain.io/api/v2/member_levels")http = Net::HTTP.new(url.host, url.port)request = Net::HTTP::Get.new(url)response = http.request(request)puts response.read_body
```


**Responses**

|Code|Description|
|-------|---------|
|200|Returns list of member levels and the privileges they provide.|


## **/api/v2/currency/trades**

* * *


**_GET_**

**Description:** Get currency trades at last 24h

**Requirements:** None

**Parameters**

|Name|Located In|Description|Required|Schema|
|----|----------|-----------|--------|------|
|currency|query||Yes|string|


**Example**

**cURL:**

```bash
curl --request GET \  --url https://api.blockchain.io/api/v2/currency/trades?currency=eth
```


**Ruby:**

```ruby
require 'uri'require 'net/http'url = URI("https://api.blockchain.io/api/v2/currency/trades?currency=eth")http = Net::HTTP.new(url.host, url.port)request = Net::HTTP::Get.new(url)response = http.request(request)puts response.read_body
```


**Responses**

|Code|Description|
|-------|---------|
|200|Get currency trades at last 24h|


## **/api/v2/currencies**

* * *


**_GET_**

**Description:** Get list of currencies

**Requirements:** None

**Parameters**

|Name|Located In|Description|Required|Schema|
|----|----------|-----------|--------|------|
|type|query|Currency type|No|string|


**Example**

**cURL:**


```bash
curl --request GET \  --url https://api.blockchain.io/api/v2/currencies
```


**Ruby:**

```ruby
require 'uri'require 'net/http'url = URI("https://api.blockchain.io/api/v2/currencies")http = Net::HTTP.new(url.host, url.port)request = Net::HTTP::Get.new(url)response = http.request(request)puts response.read_body
```


**Responses**

|Code|Description|Schema|
|----|-----------|------|
|200|Get list of currencies|[ Currency ]|


## **/api/v2/currencies/{id}**

* * *


**_GET_**

**Description:** Get a currency

**Requirements:** None

**Parameters**

|Name|Located In|Description|Required|Schema|
|----|----------|-----------|--------|------|
|path|Currency code.|Yes|string|


**Example**

**cURL:**

```bash
curl --request GET \  --url https://api.blockchain.io/api/v2/currencies/btc
```


**Ruby:**

```ruby
require 'uri'require 'net/http'url = URI("https://api.blockchain.io/api/v2/currencies/btc")http = Net::HTTP.new(url.host, url.port)request = Net::HTTP::Get.new(url)response = http.request(request)puts response.read_body
```


**Responses**

|Code|Description|Schema|
|----|-----------|------|
|200|Get a currency|Currency|


**Models**

* * *


**Currency**

Get a currency

|Name|Type|Description|Required|
|----|----|-----------|--------|
|id|string|Currency code.|No|
|symbol|string|Currency symbol|No|
|type|string|Currency type|No|
|deposit_fee|string|Currency deposit fee|No|
|withdraw_fee|string|Currency withdraw fee|No|
|quick_withdraw_limit|string|Currency quick withdraw limit|No|
|base_factor|string|Currency base factor|No|
|precision|string|Currency precision|No|
|icon_url|string|Currency icon|No|
