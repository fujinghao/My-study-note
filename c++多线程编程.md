## 循环打印abc
```cpp
#include <iostream>
#include <thread>
#include <mutex>
#include <condition_variable>

std::mutex mtx;
std::condition_variable cv;
int turn = 0;
const int NUM_CYCLES = 10; // 控制循环次数

void printA() {
    for (int i = 0; i < NUM_CYCLES; ++i) {
        std::unique_lock<std::mutex> lock(mtx);
        cv.wait(lock, [] { return turn == 0; });
    //  while (turn != 0) {
    //      cv.wait(lock);
    // }
        std::cout << "a";
        turn = 1;
        cv.notify_all();
    }
}

void printB() {
    for (int i = 0; i < NUM_CYCLES; ++i) {
        std::unique_lock<std::mutex> lock(mtx);
        cv.wait(lock, [] { return turn == 1; });
    //  while (turn != 1) {
    //      cv.wait(lock);
    // }
        std::cout << "b";
        turn = 2;
        cv.notify_all();
    }
}

void printC() {
    for (int i = 0; i < NUM_CYCLES; ++i) {
        std::unique_lock<std::mutex> lock(mtx);
        cv.wait(lock, [] { return turn == 2; });
    //  while (turn != 2) {
    //      cv.wait(lock);
    // }
        std::cout << "c";
        turn = 0;
        cv.notify_all();
    }
}

int main() {
    std::thread t1(printA);
    std::thread t2(printB);
    std::thread t3(printC);

    t1.join();
    t2.join();
    t3.join();

    return 0;
}
```