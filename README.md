# Imaar Chauthani October 2024
# This program is written in Python to access online Alpaca trading platform

import pandas as pd
import time
from datetime import datetime, timedelta
from alpaca_trade_api.rest import REST, TimeFrame

# Alpaca API credentials
API_KEY = 'PKL7AFB8RQH0SZ0X45JY'
API_SECRET = 'LNrdS3fF73DqZvwht5edWRP2v0nr5NrsBwTO6Wdy'
BASE_URL = 'https://paper-api.alpaca.markets'

# Initialize Alpaca API
api = REST(API_KEY, API_SECRET, base_url=BASE_URL)

def get_stock_data(symbol, timeframe=TimeFrame.Day, limit=100):
    """
    Get historical stock data for the last 100 days.

    Args:
        symbol (str): Stock symbol
        timeframe (TimeFrame): Timeframe for the data (default: Day)
        limit (int): Number of days to retrieve data for (default: 100)

    Returns:
        pd.DataFrame: Historical stock data
    """
    start = datetime.now() - timedelta(days=limit)
    start_str = start.strftime('%Y-%m-%d')
    bars_iter = api.get_bars_iter(symbol, timeframe, start=start_str)
    bars_df = pd.DataFrame(bars_iter)
    return bars_df

def calculate_sma(data, period):
    """
    Calculate Simple Moving Average.

    Args:
        data (pd.DataFrame): Stock data
        period (int): Period for the SMA

    Returns:
        pd.Series: Simple Moving Average
    """
    return data['close'].rolling(window=period).mean()

def calculate_rsi(data, period=14):
    """
    Calculate Relative Strength Index.

    Args:
        data (pd.DataFrame): Stock data
        period (int): Period for the RSI (default: 14)

    Returns:
        pd.Series: Relative Strength Index
    """
    delta = data['close'].diff()
    gain = (delta.where(delta > 0, 0)).rolling(window=period).mean()
    loss = (-delta.where(delta < 0, 0)).rolling(window=period).mean()
    rs = gain / loss
    return 100 - (100 / (1 + rs))

def analyze_stock(symbol):
    """
    Analyze stock using SMA and RSI.

    Args:
        symbol (str): Stock symbol

    Returns:
        str: Recommendation based on SMA and RSI
    """
    data = get_stock_data(symbol)
    if data.empty:
        return "No data available for symbol {}".format(symbol)

    data['SMA_20'] = calculate_sma(data, 20)
    data['SMA_50'] = calculate_sma(data, 50)
    data['RSI'] = calculate_rsi(data)

    current_price = data['close'].iloc[-1]
    sma_20 = data['SMA_20'].iloc[-1]
    sma_50 = data['SMA_50'].iloc[-1]
    rsi = data['RSI'].iloc[-1]

    print(f"\nAnalysis for {symbol}:")
    print(f"Current Price: ${current_price:.2f}")
    print(f"20-day SMA: ${sma_20:.2f}")
    print(f"50-day SMA: ${sma_50:.2f}")
    print(f"RSI: {rsi:.2f}")

    if sma_20 > sma_50 and rsi < 70:
        return "Buy Signal: Price is above 20-day SMA, which is above 50-day SMA. RSI is below overbought levels."
    elif sma_20 < sma_50 and rsi > 30:
        return "Sell Signal: Price is below 20-day SMA, which is below 50-day SMA. RSI is above oversold levels."
    elif rsi > 70:
        return "Overbought: Consider selling or waiting for a pullback."
    elif rsi < 30:
        return "Oversold: Consider buying or waiting for a bounce."
    else:
        return "No clear signal. The stock is in a neutral zone."

def place_order(symbol, qty, side, type, time_in_force):
    """
    Place a new order.

    Args:
        symbol (str): Stock symbol
        qty (int): Quantity to buy or sell
        side (str): Side of the order (buy or sell)
        type (str): Type of the order (market or limit)
        time_in_force (str): Time in force for the order (day or gtc)

    Returns:
        dict: Order details
    """
    try
