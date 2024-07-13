##  1.  I/O多路复用？

IO即为网络网络 I/O,多路即为多个tcp连接,复用即为共用一个线程或者进程,模型最大的优势是系统开销小,不必创建也不必维护过多的线程或者进程。
## 2.  select , poll 和 epoll的区别？
### 2.1 select

当client连接到server后，server会创建一个该连接的描述符fd，fd有三种状态，分别是读、写、异常，存放在三个fd_set（其实就是三个long型数组）中。select本质是通过设置和检查fd的三个fd_set来进行下一步操作。
<div align=center><img src=".\fig\fd_set.png"height="265"/> </div>

**select大体执行步骤** ：

1. 使用copy_from_user从用户空间拷贝fd_set到内核空间(每次循环都需要要有用户切换内核的开销)
1. 内核遍历[0,nfds)范围内的每个fd,调用fd所对应的设备驱动poll函数,poll函数可以检测fd的可用流, (读流,写流,异常流),轮询方式O(n)从所有文件描述符查找已经就绪的文件描述符
1.  io方式返回后,用户要找到就绪描述符,需要遍历所有文件描述符O(n)

**select缺点：**

1. 每次调用select，都需要把fd集合从用户态拷贝到内核态，这个开销在fd很多时会很大
2. 每次调用select都需要在内核遍历传递进来的所有fd，这个开销在fd很多时也很大
3. select支持的文件描述符数量太小了，默认是1024
### 2.2 poll
poll大体执行和select差不多 poll使用pollfd结构类型的数组pollfd将文件描述符和事件类型放在一起,任何事件都可以统一处理,使得api接口简单,而select是3个fd_set,文件描述符并没有与事件类型绑定,仅仅是三个文件描述符集,所以他不能处理更多类型事件。他的最大文件描述符上限也达到了最高，其余和select一样。
### 2.3 epoll

epoll是Linux特有的IO复用函数，它在实现和使用上与select和poll有很大差异。epoll把用户关心的文件描述符上的事件放在内核上的一个事件表中，从而无须像select和poll那样每次调用都要重复传入文件描述符集合事件表。但epoll需要使用一个额外的文件描述符，来唯一标识内核中这个事件表。

**epoll 功能的实现分了三个系统调用完成：**

- epoll_create:创建内核事件表(epoll 池)用于存放描述符和关注的事件，主要数据结构有： struct eventpoll；其中有两个重要的成员 struct rb_root rbr;和 struct list_head rdllist;前者是一棵**红黑树**，也就是内核事件表，后者是就绪事件存放的队列(**双向链表**)。epoll 池只需要遍历这个就绪链表，就能给用户返回所有已经就绪的 fd 数组；
- epoll_ctl:为红黑树添加 ep_insert、移除 ep_remove、修改 ep_modify 节点的操作，每个节点就是一个描述符和事件的结构体，数据结构如：struct epitem;
- epoll_wait:负责收集就绪事件.当关注的描述符上有事件就绪,就会调用回调函数(ep_poll_callback)，把**对应 fd 的结构体**放入就绪队列中，从而把 epoll 从 epoll_wait处唤醒。

 1int range = min(A_right, B_right) - max(A_left, B_left) cpp

- 本身没有最大并发连接的限制，仅受系统中进程能打开的最大文件数目限制（65535);
- 采用回调方式来检测就绪事件，算法时间复杂度为O(1)。
- 如果在并发量低，但socket都比较活跃的情况下，select就不见得比epoll慢了（回调函数触发得过于频繁）。
- epoll的跨平台性不够用 只能工作在linux下。

## 3. epoll中水平触发（LT）和边沿触发（ET）的区别？

- recv的时候:
如果设置为LT，只要 接受缓冲 不为空，就会一直触发EPOLLIN，直到 接受缓冲 为空
如果设置为ET，只要 客户端 发送一次数据，就会触发一次EPOLLIN
- send的时候:
如果设置为LT，只要发送缓冲不满，就会一直触发EPOLLOUT
如果设置为ET，有注册EPOLLOUT事件，才会一次触发一次EPOLLOUT

ET模式 效率要比 LT模式高, 小数据使用边沿触发，大数据使用水平触发
比如listenfd，接受缓冲区 可能存放多个客户端连接请求的信息，这时候要使用水平触发（LT），因为accept每次只能处理一个，需要多次触发。如果用边沿触发（ET）可能会漏掉一些连接。

**为什么ET模式下一定要设置非阻塞？**

在ET模式下，一般会设置非阻塞io, 也就是说，在当前没有可读取数据的情况下:
- 如果是阻塞io，recv()会阻塞
- 如果是非阻塞io，recv()会返回-1
因为ET模式下是无限循环读，直到出现错误为EAGAIN或者EWOULDBLOCK，这两个错误表示socket为空，不用再读了，然后就停止循环了，如果不是非阻塞，循环读在socket为空的时候就会阻塞到那里.

## 3. 同步IO和异步IO的区别？

- 同步异步都是针对于真正发起操作之后的行为。
- 同步是指要等待I/O操作完毕，当数据就绪后，也就是有数据可读时，要将process阻塞，一直到读完为止。
- 而对于异步，用户发起操作之后，可以立即去处理其他操作。全程交给内核处理。

## 4. 阻塞I/O和非阻塞I/O的区别？

在处理 IO 的时候，阻塞和非阻塞都是同步 IO。

- **阻塞**是指 **IO 操作**需要彻底**完成后才返回到用户空间。** 
- 而非阻塞是指 **IO 操作被调用后**立即返回给用户一个状态值**，无需等到 IO 操作彻底完成。(比如调用read读socket的fd，有数据就读取返回读到的数据长度，没数据就返回-1 **errno = EAGAIN)**

## 5. 睡眠、挂起、阻塞和终止的区别？

睡眠、挂起和终止是一种主动的行为，而阻塞则是一种被动的行为，也可以将阻塞理解为是一种状态。睡眠、阻塞、挂起这三者的共同本质就是正在执行的进程/线程，由于某些原因(主、被动)释放CPU，暂停执行。
- 睡眠会使线程进入到睡眠状态指定的时间，过了指定时间后就会被唤醒继续向下执行。睡眠能够使线程在执行时间内暂停执行，过了指定时间之后就会自动恢复执行。此时线程也在内存中。
- 挂起会让线程进入挂起状态，恢复之后线程才会继续向下执行。挂起能够使线程暂停执行，并且暂停执行期间线程被转移到了外存(磁盘)中。
- 阻塞会使线程进入到阻塞状态，恢复之后线程才会继续向下执行。阻塞能够使线程暂停执行，此时线程在内存中阻塞等待。
- 终止就是让线程彻底停止，不管run方法是否执行完了，终止以后下次该线程需要从头开始启动执行。

## 6. 为什么使用线程池？

1. **降低资源消耗**。通过重复利用已创建的线程降低线程创建和销毁造成的消耗。
2. **提高响应速度**。当任务到达时，任务可以不需要等到线程创建就能立即执行。
3. **提高线程的可管理性**。线程是稀缺资源，如果无限制的创建，不仅会消耗系统资源，还会降低系统的稳定性，使用线程池可以进行统一的分配，调优和监控。

**如何创建线程池：**
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

## 7.  服务器工作方式
- 这个项目主要的目的是对临床客户端的上传的皮肤镜图像进行解析处理，处理完之后将远程分析结果返回给客户端，如分割后的皮肤镜图像和一些评价指标。

- 服务器后端的处理方式使用socket通信，利用多路IO复用，可以同时处理多个请求，请求的解析使用预先准备好的线程池，使用**模拟proactor**模式，主线程负责监听，监听到有事件之后，从socket中循环读取数据，然后将读取到的数据封装成一个请求对象插入请求队列。线程池中的工作线程从请求队列中取出一个任务进行处理（解析请求报文），处理的方式用**状态机**。
## 8. 同步I/O方式模拟proactor
由于异步I/O并不成熟，实际中使用较少，这里将使用同步I/O模拟实现proactor模式。

原理是：主线程执行数据读写操作，读写完成之后，主线程向工作线程通知这一”完成事件“。那么从工作线程的角度来看，它们就直接获得了数据读写的结果，接下来要做的只是对读写的结果进行逻辑处理。使用同步 I/O 模型（以 epoll_wait为例）模拟出的 Proactor 模式的工作流程如下：

- 主线程往 epoll 内核事件表中注册 socket 上的读就绪事件。
- 主线程调用 epoll_wait 等待 socket 上有数据可读。
- 当 socket 上有数据可读时，epoll_wait 通知主线程。主线程从 socket 循环读取数据，直到没有更多数据可读，然后将读取到的数据封装成一个请求对象并插入请求队列。
- 睡眠在请求队列上的某个工作线程被唤醒，它获得请求对象并处理客户请求，然后往 epoll 内核事件表中注册 socket 上的写就绪事件。
- 主线程调用 epoll_wait 等待 socket 可写。
- 当 socket 上有数据可写时，epoll_wait 通知主线程。主线程往 socket 上写入服务器处理客户请求的结果。

## 9.  为什么要用状态机，什么是状态机？
因为传统应用程序的控制流程基本是按顺序执行的：遵循事先设定的逻辑，从头到尾地执行。简单来说如果想在不同状态下实现代码跳转时，就需要破坏一些代码，这样就会造成代码逻辑混乱。所以我们必须采取不同的技术来处理这些情况。
有限状态机能处理任何顺序的事件，并能提供有意义的响应——即使这些事件发生的顺序和预计的不同，每个状态都有一系列的转移，每个转移与输入和另一状态相关。当输入进来，如果它与当前状态的某个转移相匹配，机器转换为所指的状态，然后执行相应的代码。

**项目中状态机的工作流程**：

从状态机负责读取报文的一行，主状态机负责对该行数据进行解析，主状态机内部调用从状态机，从状态机驱动主状态机。

**主状态机的三种状态**：

- CHECK_STATE_REQUESTLINE，解析请求行

- CHECK_STATE_HEADER，解析请求头

- CHECK_STATE_CONTENT，解析消息体，仅用于解析POST请求

<div align=center><img src=".\fig\http请求报文.jpg"height="465"/> </div>

**从状态机的三种状态**：

- LINE_OK，完整读取一行
- LINE_BAD，报文语法有误
- LINE_OPEN，读取的行不完整
