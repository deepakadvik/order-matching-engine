# order-matching-engine
Every trade on a stock exchange goes through an order matching engine. This is mine — built from scratch in C++ with a focus on low latency, correctness, and real exchange-grade matching logic.

High-Performance Order Matching Engine
A low-latency, multi-threaded order matching engine written in modern C++, simulating the core execution system of a real-time financial exchange.

# What Is This?
When you place a stock order on an exchange, a matching engine is the system that finds a counterparty and executes the trade in microseconds. This project implements that system from scratch, focusing on correctness, performance, and real-world exchange semantics.


# Architecture

┌─────────────────────────────────────────────────┐
│                  Order Gateway                  │
│         (Incoming BUY / SELL orders)            │
└────────────────────┬────────────────────────────┘
                     │
                     ▼
┌─────────────────────────────────────────────────┐
│             Order Book (Per Symbol)             │
│                                                 │
│   BID SIDE (BUY)        ASK SIDE (SELL)         │
│   ┌──────────────┐      ┌──────────────┐        │
│   │ Price: 105   │      │ Price: 106   │        │
│   │ Price: 104   │      │ Price: 107   │        │
│   │ Price: 103   │      │ Price: 108   │        │
│   └──────────────┘      └──────────────┘        │
│      (sorted DESC)         (sorted ASC)         │
└────────────────────┬────────────────────────────┘
                     │
                     ▼
┌─────────────────────────────────────────────────┐
│              Matching Engine Core               │
│    Price-Time Priority FIFO Matching Logic      │
└────────────────────┬────────────────────────────┘
                     │
                     ▼
┌─────────────────────────────────────────────────┐
│                 Trade Log                       │
│     (Matched orders, prices, timestamps)        │
└─────────────────────────────────────────────────┘

#Features

- **Limit Orders** — Buy/Sell at a specified price
- **Market Orders** — Execute immediately at best available price
- **Order Cancellation** — O(1) cancellation using iterator caching
- **Price-Time Priority** — Strict FIFO matching within each price level
- **Multi-threaded Pipeline** — Producer/consumer architecture with thread-safe order queue
- **Latency Benchmarking** — Measures throughput in orders/second using `std::chrono`

# Core Data Structures

| Structure | Purpose | Complexity |
|---|---|---|
| `std::map<double, std::list<Order>>` | Price-level order book (bids & asks) | O(log n) insert |
| `std::unordered_map<int, iterator>` | Order ID → position cache for cancellation | O(1) cancel |
| `std::queue<Order>` | Thread-safe incoming order queue | O(1) enqueue |
| `std::mutex` + `std::condition_variable` | Producer-consumer synchronization | — |

# Getting Started

# Prerequisites
- C++17 or later
- CMake 3.15+
- Linux / macOS (or WSL on Windows)

# Build

```bash
git clone https://github.com/deepakadvik/order-matching-engine.git
cd order-matching-engine
mkdir build && cd build
cmake ..
make
./ome
```

# Project Structure

```
order-matching-engine/
├── src/
│   ├── Order.h           # Order struct definition
│   ├── OrderBook.h       # Order book interface
│   ├── OrderBook.cpp     # Matching logic implementation
│   ├── Engine.h          # Multithreaded engine
│   ├── Engine.cpp
│   └── main.cpp          # Entry point & benchmarks
├── tests/
│   └── test_matching.cpp # Unit tests
├── CMakeLists.txt
└── README.md
```

# How Matching Works

**Price-Time Priority (FIFO)**

1. Incoming BUY order arrives at price `P`
2. Engine checks lowest ASK price
3. If `lowest ask ≤ P` → **match executes**
4. Partial fills supported — remaining quantity stays in book
5. If no match → order rests in book at its price level

```
Example:
  Order Book:          Incoming Order:
  ASK: 101 (50 qty)    BUY 102, qty 80
  ASK: 102 (40 qty)
  
  Result:
    Match at 101: 50 qty filled
    Match at 102: 30 qty filled
    BUY order fully filled. Traded 80 qty.
```

# Benchmarks

  Metric              Value
Order insertion,     O(log n)
Order matching,      O(1) per fill
Order cancellation,  O(1)
Throughput,          TBD after Phase 3


# Roadmap

- Phase 1 — Core order book with limit orders
- Phase 2 — Market orders + cancellation
- Phase 3 — Multithreaded producer-consumer pipeline
- Phase 4 — Latency benchmarking & profiling
- Phase 5 — Lock-free queue with atomics


# References

- [How Exchanges Work — Investopedia](https://www.investopedia.com)
- [C++ Concurrency in Action — Anthony Williams]
- [Linux Kernel Development — Robert Love]


# Author

**Deepak Kesharwani**  
M.Tech — Industrial Engineering & Data Analytics
डॉ बी आर अम्बेडकर राष्ट्रीय प्रौद्योगिकी संस्थान जालंधर, पंजाब (भारत)
