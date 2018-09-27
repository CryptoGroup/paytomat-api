# API authentication

## Basic authorization

API endpoint: `https://app.paytomat.com/api/v1/`.

Authorization method: `https://en.wikipedia.org/wiki/Basic_access_authentication`.

HTTP header example: `Authorization: Basic cGVwZTp0aGVfcGln`, where cGVwZTp0aGVfcGln is Base64 of login and password concatenated by a colon.

This authorization can be specified in URL directly. E.g.: `https://username:password@app.paytomat.com/api/v1`.

## JWT authorization

You can acquire a JWT token for your account at control panel or through a request with basic authorization method (see below). Read more about this standard: `https://jwt.io/`.

HTTP header example: `Authorization: Bearer eyJhbGciOiJIUzI1NiJ9.eyJpZCI6NiwicmVhbG0iOiJtZXJjaGFudF9rZXkifQ.hTdblIJI6mcPCyZ5u9eiYI8e6tRJ5VRtU0AbLQ8WlAw`, where string after Bearer is the token.

### Usage example for curl

`curl 'https://app.paytomat.com/api/v1/balance' -H 'Authorization: Bearer eyJhbGciOiJIUzI1NiJ9.eyJpZCI6NiwicmVhbG0iOiJtZXJjaGFudF9rZXkifQ.hTdblIJI6mcPCyZ5u9eiYI8e6tRJ5VRtU0AbLQ8WlAw'`

# API calls description

Input is passed as query parameters for GET requests and as JSON encoded object in request body for POST.

Output is always JSON encoded object.

Errors are always reported as JSON encoded object with "error" key. E.g.:

    {
      "error": "error_description"
    }

## GET `/token`

Returns JWT token for given credential.

### Output:

Object with properties:

* `token`

    JWT token.

    *Validation*: string.

### Example:

`curl 'https://username:password@app.paytomat.com/api/v1/token'`

    {
      "token": "eyJhbGciOiJIUzI1NiJ9.eyJpZCI6NiwicmVhbG0iOiJtZXJjaGFudF9rZXkifQ.hTdblIJI6mcPCyZ5u9eiYI8e6tRJ5VRtU0AbLQ8WlAw"
    }

## GET `/currencies`

Returns available currency codes.

### Output:

An array of currency records.

### Example:

`curl 'https://username:password@app.paytomat.com/api/v1/currencies'`

    {
      "currencies": [
        {
          "code": "BTC",
          "in": true,
          "out": true,
          "title": "Bitcoin"
        },
        {
          "code": "UAH",
          "in": true,
          "out": false,
          "title": "Hryvnia"
        }
      ]
    }

## GET `/exchange_rates`

Returns currency exchange rates.

### Input:

* `currency`

    Currency code.

    *Validation*: **required**, text.

### Output:

An object of exchange rates for chosen currency.

### Example:

`curl 'https://username:password@app.paytomat.com/api/v1/exchange_rates?currency=BTC'`

    {
      "rates": {
        "BTC":   "0.020903",
        "DASH":  "0.3338",
        "USD":   "241.81",
        "WAVES": "29.874"
      }
    }

## GET `/balance`

Returns current balances for currencies.

### Output:

Object with amounts for every available currency.

### Example:

`curl 'https://username:password@app.paytomat.com/api/v1/balance'`

    {
      "balance": {
        "USD": 10000
      }
    }

## POST `/create_transaction`

### Input:

* `order_id`

  Reference order ID.

  *Validation*: **required**, alphanumeric.

  **NOTE**: If not defined, the `ref` input will be used.

* `ref`

    Merchant specific data.

    *Validation*: **optional**.

    **NOTE**: If not defined, the `order_id`**_UnixTimestamp** input will be used.

* `currency_in`

    Code for send currency.

    *Validation*: **required**, text.

* `currency_out`

    Code for receive currency

    *Validation*: **required**, text.

* `amount_in`

    Amount to send.

    *Validation*: **required**, float.

* `callback_url`

    And URL to post back to after transactions is completed.

    *Validation*: **optional**.

    Postback structure:

      {
        "merchant_id": <merchant.id>,
        "product_id": <product.id>,
        "tx_code": <tx.code>,
        "tx_id": <tx.id>,
        "amount": <tx.amount_out>,
        "currency": <tx.currency.code>,
        "qr_code": <qr_code>,
        "details": <tx.details>,
        "tx_status": <tx.status>,
        "custom_css": <product.custom_css>
      }

### Output:

Object with properties:

* `id`

    Internal transaction ID.

    *Validation*: integer, unique.

* `rate`

    Exchange rate.

    *Validation*: float.

* `currency_out`

    Currency to receive. See `currencies` for available options.

    *Validation*: text.

* `amount_out`

    Amount to receive.

    *Validation*: float.

* `qr_code`

    Text for QR code generation.

    *Validation*: text.

* `address`

    Address to receive currency.

    *Validation*: text.

* `note`

    Address notification.

    *Validation*: text.

### Example:

`curl 'https://username:password@app.paytomat.com/api/v1/create_transaction' -d '{ "ref": 125, "currency_in": "USD", "currency_out": "BTC", "amount_in": 100 }'`

    {
      "amount_out": 0.041896,
      "currency_out": "BTC",
      "id": 3,
      "qr_code": "bitcoin:38iub2TZpsTGQjZc7zhGcBcphxPk6eGpHa?amount=0.041896",
      "rate": 0.00041896,
      "address": "38iub2TZpsTGQjZc7zhGcBcphxPk6eGpHa",
      "note": "Please turn off transaction encryption if you encounter public key error"
    }

## POST `/cancel_transaction`

Manually cancel pending transaction.

### Input:

* `id`

    Internal transaction ID.

    *Validation*: **required**, integer.


### Output:

* `status`

    Action status.

    *Validation*: text.


* `status_description`

    Status description.

    *Validation*: text.

### Example:

`curl 'https://username:password@app.paytomat.com/api/v1/cancel_transaction' -d '{ "id": 9 }'`

    {
      "status": "cancelled",
      "status_description": "Transaction has been cancelled"
    }

## POST `/chargeback_transaction`

Request chargeback for transaction.

### Input:

* `id`

    Internal transaction ID.

    *Validation*: **required**, integer.

### Output:

* `status`

    Action status.

    *Validation*: text.

* `status_description`

    Status description.

    *Validation*: text.

### Example:

`curl 'https://username:password@app.paytomat.com/api/v1/chargeback_transaction' -d '{ "id": 3 }'`

    {
      "status": "chargeback_scheduled",
      "status_description": "Transaction has been scheduled for chargeback"
    }

## GET `/transaction`

Get single transaction data.

### Input:

* `id`

    Internal transaction ID.

    *Validation*: **required**, integer, unique.

### Output:

Object with transaction properties:

* `id`

    Internal transaction ID.

    *Validation*: integer, unique.

* `ref`

    Merchant specific data.

    *Validation*: optional.

* `status`

    Current transaction status.

    *Validation*: text.

* `comment`

    Transaction status comment.

    *Validation*: text.

* `created`

    Date/time of transaction creation.

    *Validation*: ISO datetime.

* `updated`

    Date/time of last transaction update.

    *Validation*: ISO datetime.

* `address`

    Address to receive currency.

    *Validation*: text.

* `currency_in`

    Code for send currency. See `currencies` for available options.

    *Validation*: text.

* `currency_out`

    Code for receive currency. See `currencies` for available options.

    *Validation*: text.

* `amount_in`

    Amount to send.

    *Validation*: float.

* `amount_out`

    Amount to receive.

    *Validation*: float.

* `rate`

    Exchange rate.

    *Validation*: float.

* `qr_code`

    Text for QR code generation.

    *Validation*: text.

* `note`

    Address notification.

    *Validation*: text.

* `product_id`

    Product id (if transaction is attached to a product).

    *Validation*: optional, integer.

* `callback_url`

    And URL to post back to after transactions is completed.

    *Validation*: optional, text.


### Example:

`curl 'https://username:password@app.paytomat.com/api/v1/transaction?id=1'`

    {
      "transaction": {
        "address": "33CyuJsTYQ3h6kPkDPT1Mawx7qZouVzmHe",
        "amount_in": "100",
        "amount_out": "0.041896",
        "comment": "Requested chargeback",
        "created": "2017-05-24 15:12:39",
        "currency_in": "USD",
        "currency_out": "BTC",
        "id": 1,
        "rate": "0.00041896",
        "ref": "CK00004",
        "status": "cancellation",
        "updated": "2017-05-24 16:34:24",
        "qr_code": "bitcoin:2N1sNUrHUmMw61rSyFPPmvDYHJrCk7QCy8x?amount=0.0000000511751",
        "note": "Please turn off transaction encryption if you encounter public key error"
      }
    }

## GET `/transactions`

Get a list of transactions.

### Input:

* `status`

    Filter the list by this status.

    *Available*: new, pending, confirmation, cancellation, success, cancelled, failure, chargeback, timeout.

    *Validation*: optional.

* `limit`

    Limit the number of transactions.

    *Validation*: optional, integer, max 100.

* `offset`

    Offset the transaction list by this value.

    *Validation*: optional, depends on limit, integer.

* `order_by`

    Sort order by one of these fields: id, status, updated, created, order_id, ref.
    By default, sort direction is ascending. Add - character before field name to sort by descending order.

    *Validation*: optional, string.

### Output:

Array of transaction records (see `transaction` ,)..


### Example:

`curl 'https://username:password@app.paytomat.com/api/v1/transactions?order_by=-updated'`

    {
      "transactions": [
        {
          "address": "33CyuJsTYQ3h6kPkDPT1Mawx7qZouVzmHe",
          "amount_in": "100",
          "amount_out": "0.041896",
          "comment": "Requested chargeback",
          "created": "2017-05-24 15:12:39",
          "currency_in": "USD",
          "currency_out": "BTC",
          "id": 1,
          "rate": "0.00041896",
          "ref": "CK00004",
          "status": "cancellation",
          "updated": "2017-05-24 16:34:24",
          "qr_code": "bitcoin:2N1sNUrHUmMw61rSyFPPmvDYHJrCk7QCy8x?amount=0.0000000511751",
          "note": "Please turn off transaction encryption if you encounter public key error"
        },
        {
          "address": "3DahmyivDiKeAo67FREygXJEZgD5G4q96u",
          "amount_in": "100",
          "amount_out": "0.041896",
          "comment": "Payment received",
          "created": "2017-05-24 15:13:11",
          "currency_in": "USD",
          "currency_out": "BTC",
          "id": 2,
          "rate": "0.00041896",
          "ref": "CK00006",
          "status": "success",
          "updated": "2017-05-24 16:34:24",
          "qr_code": "bitcoin:2N1sNUrHUmMw61rSyFPPmvDYHJrCk7QCy8x?amount=0.0000000511751",
          "note": "Please turn off transaction encryption if you encounter public key error"
        }
      ]
    }


## POST /broadcast_transaction

Broadcast transaction

### Input:

* `currency`

  Transaction currency code.

  *Validation*: **required**, string.

* `txid`

  Transaction id.

  *Validation*: **required**, string.

* `receiver`

  Receiver address.

    *Validation*: **required**, string.

* `amount`

    Transaction amount.

    *Validation*: **required**, decimal, greater than 0.

### Example:

`curl 'https://username:password@app.paytomat.com/api/v1/broadcast_transaction' -d '{"currency": "BTC", "txid": "f39448872b469a6f49b7d8b037adf810519f1f2b583cba852e7f5128f53daf29", "receiver": "1ApkXfxWgJ5CBHzrogVSKz23umMZ32wvNA", "amount": 0.07227808}'`

    {
      "success": 1
    }

## GET `/product`

Get product data

### Input:

* `id`

    Internal transaction ID.

    *Validation*: **required**, integer, unique.

### Output:

Object with product properties:

* `id`

    Internal product ID.

    *Validation*: integer, unique.

* `amount`

    Product price.

    *Validation*: decimal, greater than 0.

* `currency`

    Price currency code.

    *Validation*: text.

* `title`

    Product title

    *Validation*: text.

* `tx_success`

    Number of successful transactions for this product.

    *Validation*: integer.

* `unit`

    Product unit.

    "single" - customer will not be able to change bought quantity
    "piece" - product is sold by piece
    "decimal" - product is sold of any amount

    *Validation*: text.

* `callback_url`

    And URL to post back to after transactions is completed. See `POST /create_product`.

    *Validation*: URL.

* `skip_email`

    Set this to 1 if you don't want the customer to input their email.

    *Validation*: 1 or 0.

* `custom_css`

    Custom css style.

    *Validation*: text.

* `fields`

    Array of objects describing product fields. See `POST /create_product`.

### Example:

`curl 'https://username:password@app.paytomat.com/api/v1/product?id=1'`

    {
      "product": {
        "id": 1,
        "amount": "5.00",
        "currency": "USD",
        "title": "Croissant",
        "unit": "piece"
        "fields": [],
        "callback_url": "",
        "tx_success": 7,
        "custom_css": "https://somesite.com/css/style.css",
      }
    }

## POST `/create_product`

Create new product.

### Input:

* `id`

    Internal transaction ID.

    *Validation*: **required**, integer, unique.

* `amount`

    Product price.

    *Validation*: **required**, decimal, greater than 0.

* `currency`

    Price currency code.

    *Validation*: **required**, text.

* `title`

    Product title

    *Validation*: **required**, text.

* `unit`

    Product unit.

    "single" - customer will not be able to change bought quantity
    "piece" - product is sold by piece
    "decimal" - product is sold of any amount

    *Validation*: **required**, text.

* `callback_url`

    And URL to post back to after transactions is completed.

    *Validation*: URL.

    Postback structure:

      {
        "merchant_id": <merchant.id>,
        "product_id": <product.id>,
        "tx_code": <tx.code>,
        "tx_id": <tx.id>,
        "amount": <tx.amount_out>,
        "currency": <tx.currency.code>,
        "qr_code": <qr_code>,
        "details": <tx.details>,
        "tx_status": <tx.status>,
        "custom_css": <product.custom_css>
      }

* `custom_css`

    Custom css style.

    *Validation*: text.

* `fields`

    Array of objects describing product fields.

    Field structure sample:

      [
        {
          "id": "shipping_address",
          "title": "Shipping Address",
          "type": "text",
          "min_length": "10",
          "max_length": "42"
          "is_required": 1,
          "options": "",
        }
      ]

### Output:

Object with properties:

* `id`

    Internal product ID.

    *Validation*: integer, unique.

### Example:

`curl 'https://username:password@app.paytomat.com/api/v1/create_product' -d '{ "title": "Pierogi", "amount": 15, "currency": "UAH", "unit": "piece", "fields": [ { "id": "comment", "title": "Comment", "is_required": 0, "type": "text", "custom_css": "is_optional" } ] }'`

    {
      "id": 12
    }

## PUT `/product/{id}`

Update product properties.

### Input:

* `id`

    Internal transaction ID.

    *Validation*: **required**, integer, unique.

Product fields are validated as per `POST /create_product`, but optional.

### Output:

Object with properties:

* `updated`

    Flag of whether product has been updated.

    *Validation*: boolean.

### Example:

`curl 'https://username:password@app.paytomat.com/api/v1/product/12' -X PUT -d '{ "amount": 20 }'`

    {
      "updated": 1
    }


## GET `/payment_url`

Returns URL to merchant payment widget for given credentials.

### Output:

Object with properties:

* `url`

    URL to merchant payment widget.

    *Validation*: string.

### Example:

`curl 'https://username:password@app.paytomat.com/api/v1/payment_url'`

    {
      "url": "https://app.paytomat.com/pay/2mVQcpFLk9f"
    }
