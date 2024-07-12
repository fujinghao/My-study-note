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

### 协程循环打印abc
```cpp
#include <iostream>
#include <functional>

class Coroutine {
public:
    enum class State { Running, Suspended, Finished };

    Coroutine(std::function<void(Coroutine&)> func) : func_(func), state_(State::Suspended) {}

    void resume() {
        if (state_ == State::Finished) return;
        state_ = State::Running;
        func_(*this);
        if (state_ == State::Running) {
            state_ = State::Finished;
        }
    }

    void yield() {
        state_ = State::Suspended;
    }

    bool isFinished() const {
        return state_ == State::Finished;
    }

private:
    std::function<void(Coroutine&)> func_;
    State state_;
};
```
```cpp
#include <vector>

void printA(Coroutine& co) {
    while (true) {
        std::cout << "a";
        co.yield();
    }
}

void printB(Coroutine& co) {
    while (true) {
        std::cout << "b";
        co.yield();
    }
}

void printC(Coroutine& co) {
    while (true) {
        std::cout << "c";
        co.yield();
    }
}

int main() {
    Coroutine coroutineA(printA);
    Coroutine coroutineB(printB);
    Coroutine coroutineC(printC);

    std::vector<Coroutine*> coroutines = { &coroutineA, &coroutineB, &coroutineC };
    size_t current = 0;

    for (int i = 0; i < 30; ++i) { // 打印 30 次
        coroutines[current]->resume();
        current = (current + 1) % coroutines.size();
    }

    std::cout << std::endl;
    return 0;
}
```