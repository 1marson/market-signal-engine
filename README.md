# Live Market Data & Signal Engine

A real-time market data infrastructure that powers AI-driven trading-signal Telegram bots.

## What it does

- **Live quote streaming** — continuously tracks live prices for hundreds of trading instruments (forex, crypto, commodities, stocks) from broker platforms, in real time.
- **Tick-to-candle reconstruction** — aggregates raw price ticks into OHLC candlestick data across multiple timeframes (1m, 5m, 15m, 1h and more), the same way a trading terminal builds its chart.
- **Signal generation** — analyzes the live candle stream with technical indicators to produce trade signals with a confidence/accuracy score.
- **Multi-consumer serving layer** — a single data feed serves several downstream consumers simultaneously: Telegram bots, web-based trading terminals, and chart rendering.
- **High availability** — runs as a persistent background service (systemd-managed), auto-reconnecting on any interruption so the data feed never goes stale.

## Why it's non-trivial

Most retail broker platforms don't expose a public, documented API for live quotes — the data only exists inside their own web trading terminal. This project solves that by building a resilient, always-on bridge that extracts the live price feed and re-serves it in a clean, structured way that any downstream application (bot, dashboard, chart) can consume.

## Stack

Python, WebSockets, headless browser automation, in-memory time-series aggregation, systemd, Nginx.

---

*This repository is a public overview only. Implementation details, endpoints, and credentials are not included.*
