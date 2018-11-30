# Intrinio Java SDK for Real-Time Stock Prices

[Intrinio](https://intrinio.com/) provides real-time stock prices via a two-way WebSocket connection. To get started, [subscribe to a real-time data feed](https://intrinio.com/marketplace/data/prices/realtime) and follow the instructions below.

## Requirements

- Java 7+

## Installation

Go to [Release](https://github.com/intrinio/intrinio-realtime-java-sdk/releases), download the JAR, reference it in your project. The JAR contains dependencies necessary to the SDK.

## Sample Project

For a sample Java project see: [intrinio-realtime-java-sample](https://github.com/intrinio/intrinio-realtime-java-sample)

For a sample Android project see: [intrinio-realtime-android-sample](https://github.com/intrinio/intrinio-realtime-android-sample)

## Features

* Receive streaming, real-time price quotes (last trade, bid, ask)
* Subscribe to updates from individual securities
* Subscribe to updates for all securities (contact us for special access)

## Example Usage
```java
String api_key = "YOUR_API_KEY";

try (RealTimeClient client = new RealTimeClient(api_key, RealTimeClient.Provider.IEX)) {
    client.registerQuoteHandler(new QuoteHandler() {
        public void onQuote(Quote quote) {
            System.out.println(quote.toString());
        }
    });

    String[] channels = new String[]{"AAPL", "MSFT", "GE"};
    client.join(channels);

    client.connect();
}
```

## Handling Quotes and the Queue

When the Intrinio Realtime library receives quotes from the websocket connection, it places them in an internal queue. This queue has a default maximum size of 10,000 quotes. You can modify this value by specifying a `maxQueueSize` parameter in the client constructor, as your environment memory contraints allow. Once a quote has been placed in the queue, a registered `QuoteHandler` will receive it and pass it along to its `onQuote` method. Your overriden `onQuote` method should process quotes quickly, so that the queue does not reach its maximum size (at which point, the system will log an error and ignore any incoming quotes until the queue has space). We recommend registering multiple `QuoteHandler` instances for operations such as writing quotes to a database (or anything else involving time-consuming I/O). The client also has a `remainingQueueCapacity()` method, which returns an integer specifying the approximate length of the quote queue. Monitor this to make sure you are processing quotes quickly enough.

## Providers

Currently, Intrinio offers realtime data for this SDK from the following providers:

* IEX - [Homepage](https://iextrading.com/)
* QUODD - [Homepage](http://home.quodd.com/)
* Cryptoquote - [Homepage](https://cryptoquote.io/)


Each has distinct price channels and quote formats, but a very similar API.

## Quote Data Format

Each data provider has a different format for their quote data.

### QUODD

NOTE: Messages from QUOOD reflect _changes_ in market data. Not all fields will be present in every message. Upon subscribing to a channel, you will receive one quote and one trade message containing all fields of the latest data available.

#### Trade Message

```java
{ "ticker": "AAPL.NB",
  "root_ticker": "AAPL",
  "protocol_id": 301,
  "last_price_4d": 1594850,
  "trade_volume": 100,
  "trade_exchange": "t",
  "change_price_4d": 24950,
  "percent_change_4d": 15892,
  "trade_time": 1508165070052,
  "up_down": "v",
  "vwap_4d": 1588482,
  "total_volume": 10209883,
  "day_high_4d": 1596600,
  "day_high_time": 1508164532269,
  "day_low_4d": 1576500,
  "day_low_time": 1508160605345,
  "prev_close_4d": 1569900,
  "volume_plus": 6333150,
  "ext_last_price_4d": 1579000,
  "ext_trade_volume": 100,
  "ext_trade_exchange": "t",
  "ext_change_price_4d": 9100,
  "ext_percent_change_4d": 5796,
  "ext_trade_time": 1508160600567,
  "ext_up_down": "-",
  "open_price_4d": 1582200,
  "open_volume": 100,
  "open_time": 1508141103583,
  "rtl": 30660,
  "is_halted": false,
  "is_short_restricted": false }
```

* **ticker** - Stock Symbol for the security
* **root_ticker** - Underlying symbol for a particular contract
* **last_price_4d** - The price at which the security most recently traded
* **trade_volume** - The number of shares that that were traded on the last trade
* **trade_exchange** - The market center where the last trade occurred
* **trade_time** - The time at which the security last traded in milliseconds
* **up_down** - Tick indicator - up or down - indicating if the last trade was up or down from the previous trade
* **change_price_4d** - The difference between the closing price of a security on the current trading day and the previous day's closing price.
* **percent_change_4d** - The percentage at which the security is up or down since the previous day's trading
* **total_volume** - The accumulated total amount of shares traded
* **volume_plus** - NASDAQ volume plus the volumes from other market centers to more accurately match composite volume. Used for NASDAQ Basic
* **vwap_4d** - Volume weighted Average Price. VWAP is calculated by adding up the dollars traded for every transaction (price multiplied by number of shares traded) and then dividing by the total shares traded for the day.
* **day_high_4d** - A security's intra-day high trading price.
* **day_high_time** - Time that the security reached a new high
* **day_low_4d** - A security's intra-day low trading price.
* **day_low_time** - Time that the security reached a new low
* **ext_last_price_4d** - Extended hours last price (pre or post market)
* **ext_trade_volume** - The amount of shares traded for a single extended hours trade
* **ext_trade_exchange** - Extended hours exchange where last trade took place (Pre or post market)
* **ext_trade_time** - Time of the extended hours trade in milliseconds
* **ext_up_down** - Extended hours tick indicator - up or down
* **ext_change_price_4d** - Extended hours change price (pre or post market)
* **ext_percent_change_4d** - Extended hours percent change (pre or post market)
* **is_halted** - A flag indicating that the stock is halted and not currently trading
* **is_short_restricted** - A flag indicating the stock is current short sale restricted - meaning you can not short sale the stock when true
* **open_price_4d** - The price at which a security first trades upon the opening of an exchange on a given trading day
* **open_time** - The time at which the security opened in milliseconds
* **open_volume** - The number of shares that that were traded on the opening trade
* **prev_close_4d** - The security's closing price on the preceding day of trading
* **protocol_id** - Internal Quodd ID defining Source of Data
* **rtl** - Record Transaction Level - number of records published that day

#### Quote Message

```java
{ "ticker": "AAPL.NB",
  "root_ticker": "AAPL",
  "bid_size": 500,
  "ask_size": 600,
  "bid_price_4d": 1594800,
  "ask_price_4d": 1594900,
  "ask_exchange": "t",
  "bid_exchange": "t",
  "quote_time": 1508165070850,
  "protocol_id": 302,
  "rtl": 129739 }
```

* **ticker** - Stock Symbol for the security
* **root_ticker** - Underlying symbol for a particular contract
* **ask_price_4d** - The price a seller is willing to accept for a security
* **ask_size** - The amount of a security that a market maker is offering to sell at the ask price
* **ask_exchange** - The market center from which the ask is being quoted
* **bid_price_4d** - A bid price is the price a buyer is willing to pay for a security.
* **bid_size** - The bid size number of shares being offered for purchase at a specified bid price
* **bid_exchange** - The market center from which the bid is being quoted
* **quote_time** - Time of the quote in milliseconds
* **rtl** - Record Transaction Level - number of records published that day
* **protocol_id** - Internal Quodd ID defining Source of Data

### IEX

```java
{ "type": "ask",
  "timestamp": 1493409509.3932788,
  "ticker": "GE",
  "size": 13750,
  "price": 28.97 }
```

*   **type** - the quote type
  *    **`last`** - represents the last traded price
  *    **`bid`** - represents the top-of-book bid price
  *    **`ask`** - represents the top-of-book ask price
*   **timestamp** - a Unix timestamp (with microsecond precision)
*   **ticker** - the ticker of the security
*   **size** - the size of the `last` trade, or total volume of orders at the top-of-book `bid` or `ask` price
*   **price** - the price in USD

### Cryptoquote

#### Book Update
```java
{ "type": "book_update",
  "pair_name": "BTCUSD",
  "pair_code": "btcusd",
  "exchange_name": "Gemini",
  "exchange_code": "gemini",
  "side": "buy",
  "price": 6337.4,
  "size": 0.3 }
```

*   **type** - the type of message this is
  *    **`book_update`** - a message that denotes a change to an order book
  *    **`ticker`** - a snapshot of the market as depicted by the Exchange
  *    **`trade`** - a trade message (updating `last_trade_price`, `last_trade_time`, and `last_trade_size`)
*   **pair_name** - the name of the currency pair
*   **pair_code** - the code of the currency pair
*   **exchange_name** - the name of the exchange
*   **exchange_code** - the code of the exchange
*   **side** - the side of the book this update is for
  *    **`buy`** - this is an update to the buy side of the book
  *    **`sell`** - this is an update to the sell side of the book
*   **price** - the price of this book entry
*   **size** - the size of this book entry

#### Ticker
```java
{ "type": "ticker",
  "pair_name": "BTCUSD",
  "pair_code": "btcusd",
  "exchange_name": "Binance",
  "exchange_code": "binance",
  "last_updated": "2018-10-29 23:08:02.277Z",
  "bid": 6326,
  "bid_size": 6.51933000,
  "ask": 6326.97,
  "ask_size": 6.12643000,
  "change": -151.6899999999996,
  "change_percent": -2.340895061728389,
  "volume": 13777.232772,
  "open": 6480,
  "high": 6505.01,
  "low": 6315,
  "last_trade_time": "2018-10-29 23:08:01.834Z",
  "last_trade_side": null,
  "last_trade_price": 6326.97000000,
  "last_trade_size": 0.00001200 }
```

*   **type** - the type of message this is
  *    **`book_update`** - a message that denotes a change to an order book
  *    **`ticker`** - a snapshot of the market as depicted by the Exchange
  *    **`trade`** - a trade message (updating `last_trade_price`, `last_trade_time`, and `last_trade_size`)
*   **pair_name** - the name of the currency pair
*   **pair_code** - the code of the currency pair
*   **exchange_name** - the name of the exchange
*   **exchange_code** - the code of the exchange
*   **last_updated** - a UTC timestamp of when the ticker was last updated
*   **ask** - the ask for the currency pair on the exchange
*   **ask_size** - the size of the ask for the currency pair on the exchange
*   **bid** - the bid for the currency pair on the exchange
*   **bid_size** - the size of the bid for the currency pair on the exchange
*   **change** - the notional change in price since the last ticker
*   **change_percent** - the percent change in price since the last ticker
*   **volume** - the volume of the currency pair on the exchange
*   **open** - the opening price of the currency pair on the exchange
*   **high** - the highest price of the currency pair on the exchange
*   **low** - the lowest price of the currency pair on the exchange
*   **last_trade_time** - a UTC timestamp of the last trade for the currency pair on the exchange
*   **last_trade_side** - the side of the last trade
  *    **`buy`** - this is an update to the buy side of the book
  *    **`sell`** - this is an update to the sell side of the book
*   **last_trade_price** - the price of the last trade for the currency pair on the exchange
*   **last_trade_size** - the size of the last trade for the currency pair on the exchange

#### Trade
```java
{ "type": "trade",
  "pair_name": "BTCUSD",
  "par_code": "btcusd",
  "exchange_name": "Gemini",
  "exchange_code": "gemini",
  "last_updated": "2018-10-29 23:08:02.277Z",
  "bid": null,
  "bid_size": null,
  "ask": null,
  "ask_size": null,
  "change": -133.40760000000046,
  "change_percent": -2.059762280845255,
  "volume": 22121.79710206001,
  "open": 6476.8445,
  "high": 6506.2724,
  "low": 6311,
  "last_trade_time": "2018-10-29 23:08:01.834Z",
  "last_trade_side": "sell",
  "last_trade_price": 6343.7124,
  "last_trade_size": 1.6045 }
```

*   **type** - the type of message this is
  *    **`book_update`** - a message that denotes a change to an order book
  *    **`ticker`** - a snapshot of the market as depicted by the Exchange
  *    **`trade`** - a trade message (updating `last_trade_price`, `last_trade_time`, and `last_trade_size`)
*   **pair_name** - the name of the currency pair
*   **pair_code** - the code of the currency pair
*   **exchange_name** - the name of the exchange
*   **exchange_code** - the code of the exchange
*   **last_updated** - a UTC timestamp of when the ticker was last updated
*   **ask** - the ask for the currency pair on the exchange
*   **ask_size** - the size of the ask for the currency pair on the exchange
*   **bid** - the bid for the currency pair on the exchange
*   **bid_size** - the size of the bid for the currency pair on the exchange
*   **change** - the notional change in price since the last ticker
*   **change_percent** - the percent change in price since the last ticker
*   **volume** - the volume of the currency pair on the exchange
*   **open** - the opening price of the currency pair on the exchange
*   **high** - the highest price of the currency pair on the exchange
*   **low** - the lowest price of the currency pair on the exchange
*   **last_trade_time** - a UTC timestamp of the last trade for the currency pair on the exchange
*   **last_trade_side** - the side of the last trade
  *    **`buy`** - this is an update to the buy side of the book
  *    **`sell`** - this is an update to the sell side of the book
*   **last_trade_price** - the price of the last trade for the currency pair on the exchange
*   **last_trade_size** - the size of the last trade for the currency pair on the exchange

## Channels

### QUODD

To receive price quotes from QUODD, you need to instruct the client to "join" a channel. A channel can be
* A security ticker with data feed designation (`AAPL.NB`, `MSFT.NB`, `GE.NB`, etc)

### IEX

To receive price quotes from IEX, you need to instruct the client to "join" a channel. A channel can be
* A security ticker (`AAPL`, `MSFT`, `GE`, etc)
* The security lobby (`$lobby`) where all price quotes for all securities are posted
* The security last price lobby (`$lobby_last_price`) where only last price quotes for all securities are posted

Special access is required for both lobby channels. [Contact us](mailto:sales@intrinio.com) for more information.

### Cryptoquote

To receive price quotes from Cryptoquote, you need to instruct the client to "join" a channel. A channel can be
* The lobby (`crypto:lobby`) where all message types for all currency pairs are posted
* The type lobby (`crypto:lobby:{message_type}`) where all messages for the given type for all currency pairs are posted (i.e. `crypto:lobby:trade`)
* The pair lobby (`crypto:pair:{pair_code}`) where all message types for the provided currency pair are posted (i.e. `crypto:pair:btcusd`)
* The book_update pair lobby (`crypto:pair:book_update:{pair_code}`) where book_updates for the provided currency pair are posted (i.e. `crypto:pair:book_update:btcusd`)
* The ticker pair lobby (`crypto:pair:ticker:{pair_code}`) where tickers for the provided currency pair are posted (i.e. `crypto:pair:ticker:btcusd`)
* The trade pair lobby (`crypto:pair:trade:{pair_code}`) where trades for the provided currency pair are posted (i.e. `crypto:pair:trade:btcusd`)

## API Keys
You will receive your Intrinio API Key after [creating an account](https://intrinio.com/signup). You will need a subscription to a [realtime data feed](https://intrinio.com/marketplace/data/prices/realtime) as well.

## Documentation

### Methods

`RealTimeClient client = new RealTimeClient(String api_key, RealTimeClient.Provider provider)` - Creates an Intrinio Real-Time client
* **Parameter** `api_key`: Your Intrinio API Key
* **Parameter** `provider`: The real-time data provider to use

```java
String api_key = "YOUR_INTRINIO_API_KEY";

RealTimeClient client = new RealTimeClient(api_key, RealTimeClient.Provider.IEX);
```

---------

`RealTimeClient client = new RealTimeClient(String api_key, RealTimeClient.Provider provider, Integer maxQueueSize)` - Creates an Intrinio Real-Time client
* **Parameter** `api_key`: Your Intrinio API Key
* **Parameter** `provider`: The real-time data provider to use
* **Parameter** `maxQueueSize`: The maximum size of the quote queue (default size is 10,000)

```java
String api_key = "YOUR_INTRINIO_API_KEY";
Integer maxQueueSize = 50000;

RealTimeClient client = new RealTimeClient(api_key, RealTimeClient.Provider.IEX, maxQueueSize);
```

---------

`client.connect()` - Opens the WebSocket connection and joins the requested channels. This method blocks indefinitely.

---------

`client.connectAsync()` - Opens the WebSocket connection and joins the requested channels. This method returns after a connection has been established.

---------

`client.disconnect()` - Closes the WebSocket.

---------

`client.close()` - Closes the WebSocket, and disposes all internal resources. You _must_ either call this to dispose of the client or use try-with-resource.

---------

`client.registerQuoteHandler(QuoteHandler quoteHandler)` - Adds a QuoteHandler for handling quotes. Each quote handler will wait to receive a quote from the client's queue. Note that all quote handlers will not receive all quotes. Each handler receives the next quote in the queue once the handler finishes handling its current quote. Register multiple quote handlers to handle quotes quicker in cases of I/O.

```java
client.registerQuoteHandler(new QuoteHandler() {
    public void onQuote(Quote quote) {
        // do something with the quote
    }
});
```

Or with Java 8 lambdas:

```java
client.registerQuoteHandler(quote ->
    // do something with quote
);
```

---------

`client.getNextQuote()` - Blocks until the queue provides the next quote, then returns the quote.

---------

`client.remainingQueueCapacity()` - Returns the remaining queue capacity. Check this periodically to make sure you are processing quotes quickly enough.

---------

`client.join(String channel)` - Joins the given channel. This can be called at any time. The client will automatically register joined channels and establish the proper subscriptions with the WebSocket connection.
* **Parameter** `channel` - The channel to join

```java
client.join("AAPL");
```

---------

`client.join(String[] channels)` - Joins the given channels.
* **Parameter** `channels` - The channels to join

```java
client.join(new String[]{"AAPL", "MSFT"});
```

---------

`client.leave(String channel)` - Leaves the given channel.
* **Parameter** `channel` - The channel to leave

```java
client.leave("AAPL");
```

---------

`client.leave(String[] channels)` - Leaves the given channels.
* **Parameter** `channel` - The channels to leave

```java
client.leave(new String[]{"AAPL", "MSFT"});
```

---------

`client.leaveAll()` - Leaves all channels.

---------

`client.setChannels(String[] channels)` - Joins/leaves channels so that only the given channels are joined.
* **Parameter** `channels` - The only channels to join

```java
client.setChannels(new String[]{"AAPL", "MSFT"});
```

---------

`client.setLogger(Logger logger)` - Replaces the internal logger with the provided java.util.logging.Logger instance.
* **Parameter** `logger` - A java.util.logging.Logger instance
