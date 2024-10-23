# Multi_Threading_practice

## About
There is an abstraction of the work of a restaurant preparing two types of burgers:
 - with onion
 - without onion

Burger contains following ingredients:
 1. *Bread* (already cooked)
 2. *Onion* (must be marinate - 2 seconds, can be executed in parallel, can only be marinate once)
 3. *Cutlet* (must be fried - 1 second, can be executed in parallel, can only be fried once)

Once all the ingredients are ready, the burger can be assembled and packaged (packaging requires full process participation and takes 0.5 seconds).

## Logical idea
Of course, you can cook the burgers one at a time. But this can be optimized by adding multithreading:
```cpp
#include <thread>
#include <vector>

template <typename Fn>
void RunWorkers(unsigned n, const Fn& fn) {
    n = std::max(1u, n);
    std::vector<std::jthread> workers;
    workers.reserve(n - 1);
    // Запускаем n-1 рабочих потоков, выполняющих функцию fn
    while (--n) {
        workers.emplace_back(fn);
    }
    fn();
}
```
Now we have `n` processes that will use the `fn` function. 
Great! Now each process deals with its own burger. But this code can also be *optimized*!
If you think about it, you can understand that the process does nothing while it waits for the cutlet or onion to fry. These operations do not require process participation. In this case we can use `boost::asio::timer`, so as not to wait for the process to complete, but simply start the timer and switch to another process. A problem appears. Processes enter a race condition, to avoid this we use **Strand** - a sequential executor. From there, processes will take tasks and solve them sequentially. To add to the queue, the execution **post, defer, dispatch**:
- `Post` - Adds a task to a queue for later asynchronous execution.
- `Defer` - Delays execution of a task until the current operation completes.
- `Dispatch` - Sends an event for immediate processing by a handler.


*___P.S___. The idea is presented briefly and simply; some points may be missed.*

## How to run

bash:
```bash
git clone https://github.com/ST3PAN0V/Multi_Threading_practice
cd Multi_Threading_practice
conan install ..
cmake ..
cmake --build .
./bin/restaurant
```
Pay attention to the timestamps and in what order the processes are executed.
