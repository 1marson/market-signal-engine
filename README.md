# Live Market Data & Signal Engine (PocketOption)

*Русская версия ниже ⬇️*

A real-time market data infrastructure built for the **PocketOption** broker, powering AI-driven trading-signal Telegram bots.

## What it does

- **Live quote streaming** — continuously tracks live prices for hundreds of trading instruments (forex, crypto, commodities, stocks) from PocketOption, in real time.
- **Tick-to-candle reconstruction** — aggregates raw price ticks into OHLC candlestick data across multiple timeframes (1m, 5m, 15m, 1h and more), the same way a trading terminal builds its chart.
- **Signal generation** — analyzes the live candle stream with technical indicators to produce trade signals with a confidence/accuracy score.
- **Multi-consumer serving layer** — a single data feed serves several downstream consumers simultaneously: Telegram bots, web-based trading terminals, and chart rendering.
- **High availability** — runs as a persistent background service (systemd-managed), auto-reconnecting on any interruption so the data feed never goes stale.

## Why it's non-trivial

PocketOption, like most retail broker platforms, doesn't expose a public, documented API for live quotes — the data only exists inside their own web trading terminal. This project solves that by building a resilient, always-on bridge that extracts the live price feed and re-serves it in a clean, structured way that any downstream application (bot, dashboard, chart) can consume.

## Project structure

*(simplified overview)*

```
market-signal-engine/
├── bridge/             # maintains a live broker session and captures the raw price stream
├── server/             # aggregates ticks into candles and serves them to consumers
│   ├── candles/          # tick → OHLC aggregation, per timeframe
│   ├── api/               # HTTP endpoints (candles, signals, chart images)
│   └── ws/                 # WebSocket broadcast to connected clients
├── signals/            # technical-indicator based signal scoring
├── deploy/             # systemd service definitions, Nginx config templates
└── README.md
```

## How it works — illustrative examples

*(Simplified examples for demonstration only — not the production source.)*

**1. Turning a raw price stream into candles:**

```python
class CandleAggregator:
    """Turns a stream of live price ticks into OHLC candles."""

    def __init__(self, timeframe_seconds: int):
        self.timeframe = timeframe_seconds
        self.current = None

    def add_tick(self, price: float, ts: float):
        bucket = int(ts // self.timeframe) * self.timeframe
        if self.current is None or self.current["time"] != bucket:
            self._flush()
            self.current = {"time": bucket, "open": price, "high": price, "low": price, "close": price}
        else:
            c = self.current
            c["high"], c["low"], c["close"] = max(c["high"], price), min(c["low"], price), price

    def _flush(self):
        if self.current:
            self._on_candle_closed(self.current)   # persist / broadcast the finished candle
```

**2. A simple example signal score:**

```python
def score_signal(candles: list[dict]) -> dict:
    """Example: naive trend + momentum based confidence score."""
    closes = [c["close"] for c in candles[-14:]]
    trend = closes[-1] - closes[0]
    momentum = sum(b - a for a, b in zip(closes, closes[1:]))
    direction = "CALL" if trend > 0 else "PUT"
    confidence = min(95, 50 + abs(momentum) * 10)
    return {"direction": direction, "confidence": round(confidence, 1)}
```

**3. Example of the data shape served to consumers:**

```json
GET /api/candles?symbol=EURUSD&timeframe=60&limit=3

[
  {"time": 1755000000, "open": 1.0932, "high": 1.0935, "low": 1.0929, "close": 1.0931},
  {"time": 1755000060, "open": 1.0931, "high": 1.0938, "low": 1.0930, "close": 1.0936},
  {"time": 1755000120, "open": 1.0936, "high": 1.0940, "low": 1.0933, "close": 1.0938}
]
```

## Stack

Python, WebSockets, headless browser automation, in-memory time-series aggregation, systemd, Nginx.

---
---

# Движок рыночных данных и сигналов (PocketOption)

Инфраструктура сбора рыночных данных в реальном времени для брокера **PocketOption**, лежащая в основе Telegram-ботов с ИИ-сигналами для трейдинга.

## Что делает

- **Поток живых котировок** — непрерывно отслеживает цены по сотням торговых инструментов (форекс, крипта, товары, акции) у PocketOption в реальном времени.
- **Сборка свечей из тиков** — агрегирует сырые тики цены в OHLC-свечи по разным таймфреймам (1м, 5м, 15м, 1ч и больше) — так же, как это делает сам торговый терминал.
- **Генерация сигналов** — анализирует поток свечей техническими индикаторами и выдаёт торговые сигналы с оценкой уверенности/точности.
- **Единый источник для нескольких потребителей** — один и тот же поток данных одновременно обслуживает Telegram-ботов, веб-терминалы и отрисовку графиков.
- **Высокая доступность** — работает как постоянный фоновый сервис (через systemd), с автопереподключением при любом сбое, чтобы данные никогда не устаревали.

## Почему это нетривиально

PocketOption, как и большинство брокеров, не предоставляет публичный документированный API для живых котировок — данные существуют только внутри их собственного веб-терминала. Проект решает эту задачу, строя устойчивый, постоянно работающий мост, который извлекает живой поток цен и отдаёт его в чистом, структурированном виде любому потребителю (боту, дашборду, графику).

## Структура проекта

*(упрощённый обзор)*

```
market-signal-engine/
├── bridge/             # держит живую сессию с брокером и забирает сырой поток цен
├── server/             # агрегирует тики в свечи и отдаёт их потребителям
│   ├── candles/          # сборка тиков в OHLC-свечи по таймфреймам
│   ├── api/               # HTTP-эндпоинты (свечи, сигналы, изображения графиков)
│   └── ws/                 # рассылка по WebSocket подключённым клиентам
├── signals/            # сигнальный скоринг на технических индикаторах
├── deploy/             # описания systemd-сервисов, шаблоны конфигов Nginx
└── README.md
```

## Как это работает — примеры

*(Упрощённые примеры для демонстрации, не боевой исходный код.)*

**1. Превращение потока цен в свечи:**

```python
class CandleAggregator:
    """Превращает поток тиков цены в OHLC-свечи."""

    def __init__(self, timeframe_seconds: int):
        self.timeframe = timeframe_seconds
        self.current = None

    def add_tick(self, price: float, ts: float):
        bucket = int(ts // self.timeframe) * self.timeframe
        if self.current is None or self.current["time"] != bucket:
            self._flush()
            self.current = {"time": bucket, "open": price, "high": price, "low": price, "close": price}
        else:
            c = self.current
            c["high"], c["low"], c["close"] = max(c["high"], price), min(c["low"], price), price

    def _flush(self):
        if self.current:
            self._on_candle_closed(self.current)   # сохранить / разослать закрытую свечу
```

**2. Пример простого сигнального скоринга:**

```python
def score_signal(candles: list[dict]) -> dict:
    """Пример: наивная оценка уверенности по тренду и моментуму."""
    closes = [c["close"] for c in candles[-14:]]
    trend = closes[-1] - closes[0]
    momentum = sum(b - a for a, b in zip(closes, closes[1:]))
    direction = "CALL" if trend > 0 else "PUT"
    confidence = min(95, 50 + abs(momentum) * 10)
    return {"direction": direction, "confidence": round(confidence, 1)}
```

**3. Пример формата данных, который отдаётся потребителям:**

```json
GET /api/candles?symbol=EURUSD&timeframe=60&limit=3

[
  {"time": 1755000000, "open": 1.0932, "high": 1.0935, "low": 1.0929, "close": 1.0931},
  {"time": 1755000060, "open": 1.0931, "high": 1.0938, "low": 1.0930, "close": 1.0936},
  {"time": 1755000120, "open": 1.0936, "high": 1.0940, "low": 1.0933, "close": 1.0938}
]
```

## Стек

Python, WebSockets, автоматизация headless-браузера, агрегация временных рядов в памяти, systemd, Nginx.

---

*This repository is a public overview with illustrative examples only. Real endpoints, credentials, and production source are not included. / Этот репозиторий — публичное описание с иллюстративными примерами. Реальные эндпоинты, учётные данные и боевой код не включены.*
