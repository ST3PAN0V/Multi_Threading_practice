# Asynchronous C++ Practice

An educational C++20 and Boost.Asio project that models restaurant orders. It explores how timer-based waiting and work on multiple threads can share one event loop.

## The model

An order contains a burger with or without onion:

| Step | Modelled duration | Implementation |
| --- | --- | --- |
| Roast the cutlet | 1 second | Asynchronous steady timer |
| Marinate the onion, when requested | 2 seconds | Asynchronous steady timer |
| Pack the burger | 0.5 seconds | CPU work simulated by a busy loop |

Cutlet and onion preparation can overlap. Packaging starts once the required ingredients are ready. These durations are simulation inputs, not benchmark results.

## Concepts demonstrated

- A shared `boost::asio::io_context` processed by four worker threads.
- Worker lifetime management with `std::jthread`.
- Non-blocking waits through `boost::asio::steady_timer::async_wait`.
- Object lifetime management across callbacks with `shared_from_this()`.
- A strand and `dispatch()` to serialize insertion into the shared results collection.
- Timestamped logging with `std::osyncstream`.

The example submits **16 orders** and checks their final state with assertions.

## Stack and layout

| Path | Purpose |
| --- | --- |
| [src/main.cpp](src/main.cpp) | Burger and order models, callbacks, worker setup, and assertions |
| [CMakeLists.txt](CMakeLists.txt) | C++20 executable and thread-library configuration |
| [conanfile.txt](conanfile.txt) | Boost 1.78.0 and the Conan `cmake` generator |

Requirements:

- A compiler and standard library with C++20 support, including `std::jthread` and `std::osyncstream`.
- CMake 3.11 or newer.
- Conan 1.x, using its legacy [`cmake` generator](https://docs.conan.io/en/1.65/reference/generators/cmake.html).

## Build and run

The following commands assume Linux/GCC. Build in Debug mode to keep the assertions enabled.

```bash
git clone https://github.com/ST3PAN0V/Multi_Threading_practice.git
cd Multi_Threading_practice
mkdir -p build
cd build
conan install .. --build=missing -s compiler.libcxx=libstdc++11 -s build_type=Debug
cmake .. -DCMAKE_BUILD_TYPE=Debug
cmake --build .
./bin/restaurant
```

Conan's `cmake` generator places the executable under `build/bin/`. No command-line arguments are required.

## Reading the output

Each log line includes an order ID and elapsed time. Follow the messages from starting an order through preparing ingredients, packing, and completion. Their order can vary between runs because several threads execute the handlers.

The final assertions check that all 16 orders produced results, no result contains an error, each cutlet was roasted, every burger was packed, and onion matches the request. There is no separate test target in the current CMake configuration.

## Synchronization scope

The strand protects insertion into the final `orders` collection. The timer callbacks that update an individual `Order` are not bound to a per-order strand, so that shared state still needs synchronization before applying this pattern to a concurrent service.

The packing loop intentionally consumes CPU time to make the difference between waiting and active work visible. This repository is a learning example for inspecting asynchronous control flow and concurrency.
