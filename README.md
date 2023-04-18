# xuexi
import okex
import numpy as np
import pandas as pd
import talib
import time
import datetime
from talib import abstract
import matplotlib.pyplot as plt
from matplotlib.pylab import date2num
import mpl_finance as mpf

# Connect to OKEx API
okex_api_key = 'YOUR_API_KEY'
okex_secret_key = 'YOUR_SECRET_KEY'
okex_passphrase = 'YOUR_PASSPHRASE'
okex_api = okex.TradeAPI(okex_api_key, okex_secret_key, okex_passphrase, True)

# Technical indicators to use
indicators = ['rsi', 'macd', 'ma', 'ema', 'boll', 'sar', 'kdj']

# Time intervals to predict
time_intervals = [300, 900, 3600, 14400, 86400, 2592000]

# Moving average periods
ma_periods = [5, 10, 20, 60]

# Exponential moving average periods
ema_periods = [12, 26]

# Bollinger Bands periods
boll_periods = [20]

# SAR parameters
sar_acceleration = 0.02
sar_maximum = 0.2

# KDJ parameters
kdj_periods = [9, 3, 3]

# Create empty dataframes to store prediction results
df_results_short = pd.DataFrame(columns=['time', 'close', 'prediction'])
df_results_long = pd.DataFrame(columns=['time', 'close', 'prediction'])

# Initialize previous prediction variables
prev_pred_short = 0
prev_pred_long = 0

# Loop indefinitely
while True:
    # Get current time
    current_time = int(time.time())
    current_dt = datetime.datetime.fromtimestamp(current_time)
    print('Current time:', current_dt)

    # Loop over time intervals
    for interval in time_intervals:
        # Get historical data
        start_time = current_time - interval * 200
        end_time = current_time
        ohlc_data = okex_api.get_kline_data('BTC-USDT', interval, start_time, end_time)
        df_ohlc = pd.DataFrame(ohlc_data, columns=['time', 'open', 'high', 'low', 'close', 'volume'])
        df_ohlc['time'] = pd.to_datetime(df_ohlc['time'], unit='s')

        # Loop over technical indicators
        for indicator in indicators:
            # Skip SAR for intervals shorter than 4 hours
            if indicator == 'sar' and interval < 14400:
                continue

            # Calculate technical indicator
            ti_data = abstract.Function(indicator)(df_ohlc)
            ti_name = indicator.upper()
            if isinstance(ti_data, np.ndarray):
                ti_data = pd.Series(ti_data, name=ti_name)
            elif isinstance(ti_data, pd.DataFrame):
                ti_data.columns = [c.upper() for c in ti_data.columns]

            # Add technical indicator to dataframe
            df_ohlc = pd.concat([df_ohlc, ti_data], axis=1)

        # Loop over moving average periods
        for ma_period in ma_periods:
            # Calculate moving average
            ma_data = talib.MA(df_ohlc['close'].values, timeperiod=ma_period)
            ma_name = f'MA_{ma_period}'
            df_ohlc[ma_name] = ma_data

        # Loop over exponential moving average periods
        for ema_period in ema_periods:
            # Calculate exponential moving
