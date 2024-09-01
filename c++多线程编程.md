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
        //调用者线程首先释放mutex,然后阻塞，等待被别的线程唤醒,当调用者线程被唤醒后，调用者线程会再次获取mutex
        cv.wait(lock, [] { return turn == 0; });
    //  while (turn != 0) {
    //      cv.wait(lock); 
    // }
        std::cout << "a";
        turn = 1;
        cv.notify_all();
    }//离开作用域自动释放锁（RAII）
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

## 线程池

```cpp
#include <condition_variable>
#include <functional>
#include <iostream>
#include <mutex>
#include <queue>
#include <thread>
#include <vector>
using namespace std;

class ThreadPool {
public:
  ThreadPool(int threadnum) : started(false), thread_num(threadnum) {}

  ~ThreadPool() {
    stop();
    for (int i = 0; i < thread_num; ++i) {
      if (threadlist[i]->joinable()) {
        threadlist[i]->join();
      }
      delete threadlist[i];
    }
    threadlist.clear();
  }

  void threadFunc() {
    while (true) {
      std::function<void()> task;
      {
        std::unique_lock<std::mutex> lock(queueMutex);
        condition.wait(lock, [this] { return !tasks.empty() || !started; });
        if (!started && tasks.empty())
          return;
        task = std::move(tasks.front());
        tasks.pop();
      }
      task();
    }
  }

  int getThreadNum() { return thread_num; }

  void start() {
    if (thread_num > 0) {
      started = true;
      for (int i = 0; i < thread_num; ++i) {
        thread *pthread = new thread(&ThreadPool::threadFunc, this);
        threadlist.push_back(pthread);
      }
    }
  }

  void stop() {
    started = false;
    condition.notify_all();
  }

  //   template <class F> void addTask(F &&f) {
  //     {
  //       std::lock_guard<std::mutex> lock(queueMutex);
  //       tasks.emplace(std::forward<F>(f));
  //     }
  //     condition.notify_one();
  //   }
  void addTask(std::function<void()> f) {
    {
      std::lock_guard<std::mutex> lock(queueMutex);
      tasks.emplace(std::move(f));
    }
    condition.notify_one();
  }

private:
  int thread_num;
  bool started;
  vector<thread *> threadlist;
  condition_variable condition;
  queue<function<void()>> tasks;
  mutex queueMutex;
};

int main() {
  // 创建一个包含4个工作线程的线程池
  ThreadPool pool(4);
  pool.start();
  // 向线程池中添加了8个任务，每个任务都会输出一条信息
  for (int i = 0; i < 8; ++i) {
    pool.addTask([] {
      std::cout << "Task executed by thread: " << std::this_thread::get_id()
                << std::endl;
    });
  }

  return 0;
}
```
## 无锁循环打印AB(原子操作)

```cpp
#include <iostream>
#include <thread>
#include <atomic>

std::atomic<bool> flag(false);

void printA() {
    while (true) {
        while (flag.load());
        std::cout << "A" << std::endl;
        flag = true;
    }
}

void printB() {
    while (true) {
        while (!flag.load());
        std::cout << "B" << std::endl;
        flag = false;
    }
}

int main() {
    std::thread t1(printA);
    std::thread t2(printB);

    t1.detach();
    t2.detach();

    while (true);

    return 0;
}
```