# Migrating from Bitstamp's API to CoinFLEX's Bitstamp-like API

CoinFLEX offers an API gateway that approximately emulates [Bitstamp's v1 HTTP API](https://www.bitstamp.net/api/). This document details the differences, the most notable of which are related to CoinFLEX's use of different currency codes.

Bitstamp was not a multi-currency exchange at the time this emulation layer was implemented. Because CoinFLEX is a multi-currency exchange, CoinFLEX emulates the entire Bitstamp v1 API at multiple separate endpoints, one for each market that CoinFLEX operates.

The Bitstamp-like API can be accessed at the following API endpoints:

DEMO site
* `https://demowebapi.coinflex.com/bist/<base>/<counter>/`

LIVE site
* `https://webapi.coinflex.com/bist/<base>/<counter>/`  (go-live settings - currently turned off)

`<base>` and `<counter>` are placeholders for asset codes listed in the "Asset Type" column of [SCALE.md](SCALE.md).

These endpoint URIs replace the `https://www.bitstamp.net/api/` portion of Bitstamp's URIs.


## Public Data Functions


### Ticker

#### Functional differences

None.

#### Structural differences

* Bitstamp includes an undocumented `timestamp` field in the returned JSON object. CoinFLEX does not include this field. Please refer to the `Date` HTTP response header.
* CoinFLEX returns **null** for any ticker fields that are not populated, as may happen if no trade has occurred in the past 24 hours.


### Order Book

#### Functional differences

None.

#### Structural differences

* Bitstamp includes an undocumented `timestamp` field in the returned JSON object. CoinFLEX does not include this field. Please refer to the `Date` HTTP response header.


### Transactions

#### Functional differences

None.

#### Structural differences

* Bitstamp encodes the `date` field as a string, even though its value is a Unix timestamp. CoinFLEX encodes the `date` field as an integer.


### EUR/USD Conversion Rate

CoinFLEX does not implement this method.


## API Authentication

### Functional differences

* Bitstamp authenticates requests using an API key and an HMAC (which Bitstamp calls a "signature"). The HMAC is computed using a tonce (which Bitstamp calls a "nonce," even though it isn't one), a client ID, an API key, and a secret key as inputs. The API key, tonce, and HMAC are then passed as POST parameters.
* CoinFLEX authenticates requests using standard HTTP Basic authentication. The username portion of the Basic credentials is constructed by concatenating the numeric user ID, a slash, and the user's API authentication cookie. The password portion is simply the user's login passphrase.

### Structural differences

* Bitstamp passes authentication credentials in POST parameters. Custom software is needed to generate the required HMAC.
* CoinFLEX passes authentication credentials in the HTTP standard `Authorization` request header. Many off-the-shelf HTTP clients (including cURL and all web browsers) can generate the necessary request header.


## Private Functions


### Account Balance

#### Functional differences

* CoinFLEX does not return information about trading fees.

#### Structural differences

* Bitstamp returns fields whose names are constructed by appending `_balance`, `_reserved`, or `_available` to a currency code, which is either `btc` or `usd`. CoinFLEX constructs these field names using currency codes `xbt`, `eur`, `gbp`, `usd`, and/or `pln`, depending on the specific API endpoint being accessed.
* CoinFLEX does not include the `fee` field in the response.


### User Transactions

#### Functional differences

None.

#### Structural differences

* Bitstamp returns the quantity and total of each trade in the `btc` and `usd` fields, respectively, and returns the trade price in the (undocumented) `btc_usd` field. CoinFLEX names these fields using currency codes `xbt`, `eur`, `gbp`, `usd`, and/or `pln`, depending on the specific API endpoint being accessed.


### Open Orders

#### Functional differences

None.

#### Structural differences

None.


### Cancel Order

#### Functional differences

None.

#### Structural differences

* If the specified order could not be found, Bitstamp returns `{"error": "Order not found"}`. CoinFLEX returns `false` in this case.
* If the specified order was found and canceled, Bitstamp returns `true` with a `Content-Type: application/json` header, even though `true` is not a valid JSON encoding. CoinFLEX returns `true` with a `Content-Type: text/plain` header in this case.


### Buy/Sell Limit Order

#### Functional differences

* CoinFLEX supports the use of tonces on orders via the optional `nonce` request parameter (so named to emulate Bitstamp). When tonces are used, each order placed must specify a tonce that is greater than that specified on any previous order since the tonce counter was last reset. Placing an order without a tonce resets the tonce counter.
* CoinFLEX supports an optional time-to-live parameter on orders. If given, the placed order will be canceled automatically after the specified time if it has not already been closed otherwise.
    * Note that extenuating circumstances (such as heavy traffic or system upgrades) may force orders placed with times-to-live to be canceled sooner than specified. (This proviso does not apply to orders placed without a time-to-live.)

#### Structural differences

* Bitstamp returns a `datetime` field with microsecond precision. CoinFLEX returns a `datetime` field with whole-second precision.
* CoinFLEX accepts an optional `ttl` request parameter that specifies a time-to-live for the order in seconds. If given, it must be positive and must not exceed 86400 (24 hours).


### Buy/Sell Market Order

Bitstamp does not implement this method.

#### Structure

```
POST <endpoint>/buy_market/
POST <endpoint>/sell_market/
```

##### Request parameters:

* Exactly one of:
  * `quantity` – an amount of the base asset to buy or sell
  * `total` – an amount of the counter asset with which to buy the base asset or for which to sell the base asset

##### Response:

Returns a JSON object containing a single field `remaining`, which communicates how much of the requested quantity or total could not be traded.


### Estimate Buy/Sell Market Order

Bitstamp does not implement this method.

#### Structure

```
POST <endpoint>/estimate_buy_market/
POST <endpoint>/estimate_sell_market/
```

##### Request Parameters:

* Exactly one of:
  * `quantity` – an amount of the base asset that would be bought or sold
  * `total` – an amount of the counter asset with which the base asset would be bought or for which the base asset would be sold

##### Response:

Returns a JSON object containing fields `quantity` and `total`, which reflect how much of the base and counter assets, respectively, would have been traded if a market order had been executed.


### Withdrawal Requests

CoinFLEX does not implement this method.


### Bitcoin Withdrawal

#### Functional differences

* CoinFLEX does not support withdrawing bitcoins to arbitrary Bitcoin addresses. All withdrawals are made to the address on file for the requesting user.

#### Structural differences

* CoinFLEX does not support the `address` parameter and will reject any request that specifies it.
* CoinFLEX does not return a withdrawal identifier. A successful call returns a `204 No Content` status.


### Bitcoin Deposit Address

CoinFLEX does not implement this method.


### Unconfirmed Bitcoin Deposits

CoinFLEX does not implement this method.


### Ripple Withdrawal

CoinFLEX does not implement this method.


### Ripple Deposit Address

CoinFLEX does not implement this method.
