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

## GET `token`

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

## GET `currencies`

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

## GET `exchange_rates`

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

## GET `balance`

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

## POST `create_transaction`

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

### Example:

`curl 'http://username:password@app.paytomat.com/api/v1/create_transaction' -d '{ "ref": 125, "currency_in": "USD", "currency_out": "BTC", "amount_in": 100 }'`

    {
      "amount_out": 0.041896,
      "currency_out": "BTC",
      "id": 3,
      "qr_code": "bitcoin:38iub2TZpsTGQjZc7zhGcBcphxPk6eGpHa?amount=0.041896",
      "rate": 0.00041896,
      "address": "38iub2TZpsTGQjZc7zhGcBcphxPk6eGpHa"
    }

## POST `cancel_transaction`

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

`curl 'http://username:password@app.paytomat.com/api/v1/cancel_transaction' -d '{ "id": 9 }'`

    {
      "status": "cancelled",
      "status_description": "Transaction has been cancelled"
    }

## POST `chargeback_transaction`

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

`curl 'http://username:password@app.paytomat.com/api/v1/chargeback_transaction' -d '{ "id": 3 }'`

    {
      "status": "chargeback_scheduled",
      "status_description": "Transaction has been scheduled for chargeback"
    }

## GET `transaction`

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

### Example:

`curl 'http://username:password@app.paytomat.com/api/v1/transaction?id=1'`

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
        "qr_code": "bitcoin:2N1sNUrHUmMw61rSyFPPmvDYHJrCk7QCy8x?amount=0.0000000511751"
      }
    }

## GET `transactions`

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

### Output:

Array of transaction records (see `transaction` call).

### Example:

`curl 'http://username:password@app.paytomat.com/api/v1/transactions'`

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
          "qr_code": "bitcoin:2N1sNUrHUmMw61rSyFPPmvDYHJrCk7QCy8x?amount=0.0000000511751"
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
          "qr_code": "bitcoin:2N1sNUrHUmMw61rSyFPPmvDYHJrCk7QCy8x?amount=0.0000000511751"
        }
      ]
    }

## POST `widget`

Get ready to use iframe with embedded parameters and signature.

### Input:

* `public_key`

    Internal public_key ID .

    *Validation*: **required**, alphanumeric.

* `order_id`

    Reference ID.

    *Validation*: **required**, alphanumeric, unique.

* `currency_in`

    Code for send currency.

    *Validation*: **required**, text.

* `amount_in`

    Amount to send.

    *Validation*: **required**, float.

* `callback_url`

    Callback url .

    *Validation*: optional, text.

### Output:

Ready to use iframe string (HTML format).

### Example:

`curl 'http://username:password@app.paytomat.com/api/v1/widget/<public_key>' -d '{ "order_id": 125, "currency_in": "USD", "amount_in": 100, "callback_url": "http://mysupersite.com/paytomat_callback" }'`

    <iframe src="http://app.paytomat.com/gateway/PT6J4MCX6Q?order_id=125&currency_in=USD&amount_in=100&callback_url=http://mysupersite.com/paytomat_callback&signature=VERY0LONG0SUPERSICRET0SIGNATURE"></iframe>
