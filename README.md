import ccxt
import talib
import numpy as np
import time

def create_exchange(api_key, api_secret, exchange_id='binance'):
    # Create and return an exchange instance based on exchange_id
    exchange_class = getattr(ccxt, exchange_id)
    exchange = exchange_class({
        'apiKey': api_key,
        'secret': api_secret,
    })
    return exchange

def trading_bot(exchange, symbol, quantity, stop_loss_percentage):
    in_position = False

    while True:
        try:
            # Fetch historical data
            ohlcv = exchange.fetch_ohlcv(symbol, '5m', limit=50)  # 5-minute chart, adjust limit as needed

            # Extract closing prices
            closes = np.array([c[4] for c in ohlcv])

            # Calculate moving averages
            short_ma = talib.SMA(closes, timeperiod=20)
            long_ma = talib.SMA(closes, timeperiod=50)

            # Buy condition (crossover)
            if not in_position and short_ma[-1] > long_ma[-1] and short_ma[-2] <= long_ma[-2]:
                order = exchange.create_market_buy_order(symbol, quantity)
                print(f"Buy Order Executed: {order}")
                in_position = True

            # Sell condition (crossunder) with stop-loss
            elif in_position and (short_ma[-1] < long_ma[-1] or short_ma[-1] < long_ma[-1] and short_ma[-2] >= long_ma[-2]):
                order = exchange.create_market_sell_order(symbol, quantity)
                print(f"Sell Order Executed: {order}")
                in_position = False

        except Exception as e:
            print(f"An error occurred: {e}")

        time.sleep(300)  # Adjust the sleep interval based on your strategy for a 5-minute chart

# Example usage:
api_key = 'your_api_key'
api_secret = 'your_api_secret'
exchange_id = 'binance'  # Change to the desired exchange, e.g., 'deribit', 'kraken', etc.
symbol = 'BTC/USDT'  # Adjust the trading pair
quantity = 0.01
stop_loss_percentage = 0.002

# Create the exchange instance
exchange = create_exchange(api_key, api_secret, exchange_id)

# Run the trading bot
trading_bot(exchange, symbol, quantity, stop_loss_percentage)
