import matplotlib
matplotlib.use('TkAgg')
import matplotlib.pyplot as plt
import matplotlib.dates as mdates
from matplotlib.widgets import Button
import pandas as pd
import numpy as np
import requests

# Function to fetch SOL/USDT OHLC data from KuCoin API
def get_solana_data(interval='4hour', days=60):
    url = "https://api.kucoin.com/api/v1/market/candles"
    params = {
        'symbol': 'SOL-USDT',
        'type': interval,
        'startAt': int(pd.Timestamp.utcnow().timestamp()) - (days * 24 * 60 * 60),
        'endAt': int(pd.Timestamp.utcnow().timestamp())
    }
    try:
        response = requests.get(url, params=params)
        response.raise_for_status()
        return response.json().get('data', [])
    except Exception as e:
        print(f"API Error: {e}")
        return []

# Process data and calculate EMAs
def process_data(data):
    try:
        df = pd.DataFrame(data, columns=['timestamp', 'open', 'close', 'high', 'low', 'volume', 'turnover'])
        df['timestamp'] = pd.to_datetime(df['timestamp'], unit='s')
        df.set_index('timestamp', inplace=True)
        df = df.apply(pd.to_numeric).dropna()

        df['EMA_50'] = df['close'].ewm(span=50, adjust=False).mean()
        df['EMA_200'] = df['close'].ewm(span=200, adjust=False).mean()

        return df
    except Exception as e:
        print(f"Processing Error: {e}")
        return pd.DataFrame()

# Determine candle width
def get_candle_width(interval):
    return 0.02 if interval == "1hour" else 0.08

# Plot chart with EMAs
def plot_chart(ax, df, interval):
    ax.clear()
    candle_width = get_candle_width(interval)

    for idx, row in df.iterrows():
        color = '#2ecc71' if row['close'] > row['open'] else '#e74c3c'
        ax.plot([idx, idx], [row['low'], row['high']], color='black', linewidth=0.8)
        ax.fill_between([idx - pd.Timedelta(hours=candle_width * 12), idx + pd.Timedelta(hours=candle_width * 12)],
                        row['open'], row['close'], color=color, edgecolor=color)

    ax.plot(df.index, df['EMA_50'], label='EMA 50', color='blue', linewidth=1.5)
    ax.plot(df.index, df['EMA_200'], label='EMA 200', color='black', linewidth=1.5)

    ax.set_title(f'SOL/USDT Chart ({interval})', fontsize=16, pad=20)
    ax.xaxis.set_major_formatter(mdates.DateFormatter('%m/%d'))
    ax.legend()
    plt.draw()

# Update chart with new interval
def update_chart(interval):
    days = 30 if interval == "1hour" else 60
    df = process_data(get_solana_data(interval=interval, days=days))
    plot_chart(ax_price, df, interval)

# Set up plot and initial data
fig, ax_price = plt.subplots(figsize=(16, 8))
df = process_data(get_solana_data(interval="4hour", days=60))
plot_chart(ax_price, df, "4hour")

# Add buttons
ax_1h = plt.axes([0.82, 0.01, 0.08, 0.05])
btn_1h = Button(ax_1h, "1H")
btn_1h.on_clicked(lambda event: update_chart("1hour"))

ax_4h = plt.axes([0.91, 0.01, 0.08, 0.05])
btn_4h = Button(ax_4h, "4H")
btn_4h.on_clicked(lambda event: update_chart("4hour"))

plt.show()
