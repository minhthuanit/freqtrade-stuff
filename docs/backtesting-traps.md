---
title: Backtesting Traps
layout: post
---

Backtesting is important to understand the performance and behaviour of your strategy, but there are a number of issues which can result in a backtest producing much better results than are likely once the strategy is run live (or dry-run).

This information applies to the Hyperopt feature as well, which processes strategies in the same way as Backtesting.

To start with, make sure to familiarize yourself with the [official documentation on Backtesting](https://www.freqtrade.io/en/stable/backtesting/#assumptions-made-by-backtesting) and it's assumptions.  

## Basics

- The Future Will Be Different From The Past

  Backtesting tests use historical data, so even if your strategy avoids all of the other problems, there's no guarantee that it will succeed in the future, even if it succeeded on past data.

- Trade Timeouts

  When backtesting, all of your trade orders are magically filled at the exact price you wanted.  This doesn't happen in real life, and if you place limit orders, it's likely that some percentage of them go unfilled and time out, causing you to miss entering or exiting a trade.  The likelyhood of this happening depends on the volume of the pair you're trading, when your strategy places orders, whether the market is moving up or down, which side of the bid/ask spread you place your orders on, etc...

- Slippage

  Regardless of which order type you use, the market will continue to move between the time your strategy decides to place and order, and that order actually getting placed, and hopefully filled on the exchange.  Particularly when using market orders, this causes price slippage and it is likely that you will get a somewhat worse price than the moment the strategy decided to place an order.  How much worse will depend on the exact market conditions, spread, volume, the size of your order, price direction, etc...

Refer to [Prices Used For Orders](https://www.freqtrade.io/en/stable/configuration/#prices-used-for-orders) and [Market Order Pricing](https://www.freqtrade.io/en/stable/configuration/#market-order-pricing) in the Configuration documentation for details.



## Reading Future Candles

Something that can be difficult to get used to when backtesting is that signals are generated from the entire time range at one time.  The bot does not start at the first candle and process each candle in order.

`populate_indicators`, `populate_buy_trend`, and `populate_sell_trend` are called once for a given pair, and generate all of the signals for the entire duration.  This means that the "end" of the Dataframe is always the final candle of the backtesting time range, and is not the "most recent" candle as it would be when running live or dry.

When creating the signals for any given candle it is therefor easy to accidentally read from the candles which follow in the Dataframe, effectively "seeing into the future".  When running dry or live, the future is not available.

Read the official documentation on [common mistakes made when developing strategies](https://www.freqtrade.io/en/stable/strategy-customization/#common-mistakes-when-developing-strategies) for details on things to avoid.

## Trailing Stops

> *"i keep tightening trailing stoploss and backtest profits go up. that cant be right."*

One backtesting anomaly that seems to produce incredible, one might even say *unbelieveable*, results is the use of very small trailing stops.

Here's an example of a strategy that exploits backtesting to report incredible results:

![20210412-173449-powershell](https://user-images.githubusercontent.com/323682/114368495-6b373500-9bb8-11eb-8cc5-0e8103c31d9a.png)

In only 11 days, we've gone from $2,500 to $65,150, a 2506% profit! Amazing!

Sadly, this strategy will not produce similar results when run live.
