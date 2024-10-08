## 项目概述
临床端将采集的皮肤镜图像上传到服务器，服务器对图像进行解析处理，处理完之后将远程分析结果返回给临床端。
图像的上传是通过浏览器发送POST请求，服务器接收到请求后，解析请求报文，提取出请求的图像数据，然后调用TensorRT接口对图像数据进行处理，处理完之后将预测结果返回给临床端。
服务器框架：使用C++编写，采用线程池 + epoll IO多路复用的方式处理客户端请求。
TensorRT使用流程：将.path转换为onnx格式，然后将onnx格式的模型转换为trt格式，最后使用TensorRT对图像数据进行处理。
创建推理引擎，创建TensorRT执行上下文对象，将图像数据输入到引擎中，执行推理，获取输出结果。

## 1.  I/O多路复用？

IO即为网络网络 I/O,多路即为多个tcp连接,复用即为共用一个线程或者进程,模型最大的优势是系统开销小,不必创建也不必维护过多的线程或者进程。
## 2.  select , poll 和 epoll的区别？

### 2.1 select

当client连接到server后，server会创建一个该连接的描述符fd，fd有三种状态，分别是读、写、异常，存放在三个fd_set（其实就是三个long型数组）中。select本质是通过设置和检查fd的三个fd_set来进行下一步操作。
<div align=center><img src=".\fig\fd_set.png"height="265"/> </div>

**select大体执行步骤** ：

1. 已连接的Socket都放到一个fd_set集合里面, 然后调用 select 函数将文件描述符集合拷贝到内核里，让内核来检查是否有网络事件产生。
2. 内核遍历[0,nfds)范围内的每个fd,调用fd所对应的设备驱动poll函数,poll函数可以检测fd的可用流, (读流,写流,异常流),轮询方式O(n)从所有文件描述符查找已经就绪的Socket文件描述符。
3. 接着再把整个文件描述符集合拷贝回用户态里，然后用户态还需要再通过遍历的方法找到可读或可写的 Socket，然后再对其处理。

**select缺点：**

1. 需要进行 2 次「遍历」文件描述符集合，一次是在内核态里，一个次是在用户态里 
2. 还会发生 2 次「拷贝」文件描述符集合，先从用户空间传入内核空间，由内核修改后，再传出到用户空间中。
3. select支持的文件描述符数量太小了，默认是1024

select 使用固定长度的 BitsMap（位图），表示文件描述符集合，每个文件描述符对应一个 Bit，当文件描述符对应的 Bit 为 1 时，表示该文件描述符对应的事件发生了，为 0 时表示事件未发生。
fd_set结构体：
```c
typedef struct {
    //__FD_SETSIZE是1024, 在内核头文件中定义无法修改
    // 64 位系统下，__NFDBITS的值通常为sizeof(unsigned long) * 8，也就是一个无符号长整数的位数，一般为 64。
    //所以fds_bits是大小为16的unsigned long数组，每个元素有64位（可以表示64个fd的状态）总共可以监视 16 * 64 = 1024个文件描述符
    unsigned long fds_bits[__FD_SETSIZE / __NFDBITS];
} fd_set;
```

使用示例：
```cpp
const int BUFFER_SIZE = 1024;
const int PORT = 8888;

int main() {
    int serverSocket = socket(AF_INET, SOCK_STREAM, 0);

    sockaddr_in serverAddress;
    serverAddress.sin_family = AF_INET;
    serverAddress.sin_addr.s_addr = INADDR_ANY;
    serverAddress.sin_port = htons(PORT);

    bind(serverSocket, (struct sockaddr *)&serverAddress, sizeof(serverAddress))
    listen(serverSocket, 10)

    fd_set masterSet, workingSet;
    //把当前fd_set所有位的数字都置为0。
    FD_ZERO(&masterSet);
    //FD_SET实现了句柄和fd_set的联系，可以把fd，也就是句柄加入到fd_set中。
    FD_SET(serverSocket, &masterSet);

    int maxSocketDescriptor = serverSocket;
    timeval timeout;
    timeout.tv_sec = 5;  // 等待 5 秒
    timeout.tv_usec = 0;

    while (true) {
        workingSet = masterSet;
        //int select(int maxfd, fd_set* readset, fd_set* writeset, fd_set* excepset, const struct timeavl* timeout) 
        //select函数返回一个整数值，表示准备好进行 I/O 操作的文件描述符的数量。如果返回值为 0，表示在指定的时间内没有任何文件描述符准备好进行 I/O 操作。如果返回值为 -1，表示发生了错误，可以通过检查errno来确定错误原因。
        //maxfd:监视对象文件描述符的数量(由于文件描述符从0开始所以要加1) 
        //readfds、writefds、exceptfds这三个参数是指向描述符集的指针，描述符集说明了我们关心的 可读、可写、异常 的结合
        //timeout设置超时时间，防止无限阻塞
        int activity = select(maxSocketDescriptor + 1, &workingSet, nullptr, nullptr, &timeout);
        //复杂度O(N), N表示总共监视的fd的总数
        for (int i = 0; i <= maxSocketDescriptor; i++) {
            //检测fd是否在set集合中，不在则返回0
            if (FD_ISSET(i, &workingSet)) {                 
                if (i == serverSocket) {
                    //它从已完成连接队列中取出一个连接请求，并创建一个新的套接字来与客户端进行通信。
                    int clientSocket = accept(serverSocket, nullptr, nullptr);
                    FD_SET(clientSocket, &masterSet);
                    //更新需要监视的fd数量
                    if (clientSocket > maxSocketDescriptor) {
                        maxSocketDescriptor = clientSocket;
                    }
                    std::cout << "New client connected." << std::endl;
                } else {
                    char buffer[BUFFER_SIZE];
                    int bytesRead = recv(i, buffer, BUFFER_SIZE - 1, 0);
                    if (bytesRead <= 0) {
                        if (bytesRead == 0) {
                            std::cout << "Client disconnected." << std::endl;
                        } else {
                            std::cerr << "Error reading from client." << std::endl;
                        }
                        close(i);
                        FD_CLR(i, &masterSet);
                    } else {
                        buffer[bytesRead] = '\0';
                        std::cout << "Received from client: " << buffer << std::endl;
                        send(i, buffer, bytesRead, 0);
                    }
                }
            }
        }
    }

    close(serverSocket);
    return 0;
}
```

### 2.2 poll
poll使用链表来管理pollfd结构体，突破了 select 的文件描述符个数限制（当然还会受到系统文件描述符限制）。
而select是3个fd_set,文件描述符并没有与事件类型绑定,仅仅是三个文件描述符集,所以他不能处理更多类型事件。
但是 poll 和 select 并没有太大的本质区别，都是需要遍历文件描述符集合来找到可读或可写的 Socket，时间复杂度为 O(n)，而且也需要在用户态与内核态之间拷贝文件描述符集合，这种方式随着并发数上来，性能的损耗会呈指数级增长。
```c
struct pollfd {
    int fd; // 文件描述符
    short events; // 我们感兴趣的事件
    short revents; // 实际发生了的事件
};
```
### 2.3 epoll

epoll是Linux特有的IO复用函数，它在实现和使用上与select和poll有很大差异。epoll把用户关心的文件描述符上的事件放在内核上的一个事件表中，从而无须像select和poll那样每次调用都要重复传入文件描述符集合事件表。但epoll需要使用一个额外的文件描述符，来唯一标识内核中这个事件表。

**epoll 功能的实现分了三个系统调用完成：**

- epoll_create:创建内核事件表(epoll 池)用于存放描述符和关注的事件，主要数据结构有： struct eventpoll；其中有两个重要的成员 struct rb_root rbr;和 struct list_head rdllist;前者是一棵**红黑树**，也就是内核事件表(用于存放待检测的描述符)，后者是就绪事件存放的队列(**双向链表**)。epoll 池只需要遍历这个就绪链表，就能给用户返回所有已经就绪的 fd 数组；
- epoll_ctl:为红黑树添加 ep_insert、移除 ep_remove、修改 ep_modify 节点的操作，每个节点就是一个描述符和事件的结构体，数据结构如：struct epitem;
- epoll_wait:负责收集就绪事件.当关注的描述符上有事件就绪,就会调用回调函数(ep_poll_callback)，把**对应 fd 的结构体**放入就绪队列中，从而把 epoll 从 epoll_wait处唤醒。

**epoll监视过程**：
1. epoll 在内核里使用红黑树来跟踪进程所有待监控的文件描述字，把需要监控的 socket 通过 epoll_ctl() 函数加入内核中的红黑树里，而 select/poll 内核里没有类似 epoll 红黑树这种保存所有待检测的 socket 的数据结构，所以 select/poll 每次操作时都传入整个 socket 集合给内核，而 epoll 因为在内核维护了红黑树，可以保存所有待检测的 socket ，所以只需要传入一个待检测的 socket，减少了内核和用户空间大量的数据拷贝和内存分配。
2. epoll 使用事件驱动的机制，内核里维护了一个链表来记录就绪事件，当某个 socket 有事件发生时，通过回调函数内核会将其加入到这个就绪事件列表中，当用户调用 epoll_wait() 函数时，内核将就绪列表中的数据传入到epoll_event数组，并返回给用户就绪事件的数量，用户通过遍历epoll_event数组来获取就绪事件，而 select/poll 每次操作时都需要遍历整个 socket 集合，效率较低。

**epoll的优缺点：**

- 本身没有最大并发连接的限制，仅受系统中进程能打开的最大文件数目限制（65535）;
- 采用回调方式来检测就绪事件，算法时间复杂度为O(1)。
- 如果在并发量低，但socket都比较活跃的情况下，select就不见得比epoll慢了（回调函数触发得过于频繁）。
- epoll的跨平台性不够用 只能工作在linux下。
```cpp
int serverSocket = socket(AF_INET, SOCK_STREAM, 0);
bind(serverSocket, ...);
listen(serverSocket, ...)

int epfd = epoll_create(EPOLL_SIZE); //创建epoll句柄
struct epoll_event event;
event.events = EPOLLIN;
event.data.fd = serverSocket;
epoll_ctl(epoll_fd, EPOLL_CTL_ADD, serverSocket, &event); //将所有需要监听的socket添加到epfd中
struct epoll_event events[EPOLL_SIZE]; //用于接收就绪事件的数组
while(1) {
    int n = epoll_wait(...);
    for(int i = 0; i < n; i++) {
        //有新的连接
        if(events[i].data.fd == serverSocket) {
            int clientSocket = accept(serverSocket, ...);
            event.events = EPOLLIN;
            event.data.fd = clientSocket;
            epoll_ctl(epoll_fd, EPOLL_CTL_ADD, clientSocket, &event);
        } else {
            //处理clientSocket的读写事件
        }
    }
}
```

## 3. epoll中水平触发（LT）和边沿触发（ET）的区别？

- recv的时候:
如果设置为LT，只要**接受缓冲**不为空，就会一直触发EPOLLIN，直到**接受缓冲**为空
如果设置为ET，客户端每发送一次数据，才触发一次EPOLLIN。并且如果这次事件没有把所有数据读完，那么在数据没有被再次更新之前，后续的epoll_wait调用将不会再触发该文件描述符的可读事件。
- send的时候:
如果设置为LT，只要发送缓冲不满，就会一直触发EPOLLOUT
如果设置为ET，有注册EPOLLOUT事件，才会一次触发一次EPOLLOUT

**LT优缺点**：
- 程序简单，会完整地读取所有数据。
- 重复地事件触发会影响高并发服务器地性能
**ET优缺点**：
- 每次epoll_wait只用触发一次，通过程序逻辑实现读取缓冲区的所有数据，工作效率高，大大提升了服务器性能；
- 数据的完整性需要手动保证，如果read一次没有读尽buffer中的数据，那么下次将得不到读就绪的通知，造成buffer中已有的数据无机会读出，除非有新的数据再次到达。

**总结**：ET模式 效率要比 LT模式高, 小数据使用边沿触发，大数据使用水平触发
比如listenfd，接受缓冲区可能存放多个客户端连接请求的信息，这时候要使用水平触发（LT），因为accept每次只能处理一个，需要多次触发。如果用边沿触发（ET）可能会漏掉一些连接（需要用while循环来保证）。
在高并发服务器中边沿触发(ET) 的效率更高，因为边沿触发只在数据到来的一刻才触发，很多时候服务器在接受大量数据时会先接受数据头部，接着服务器通过解析头部决定要不要接这个数据。此时，如果不接受数据，水平触发需要手动清除（水平触发当有数据时，会一直触发，直到没有数据可读），而边沿触发可以将清除工作交给一个定时的清除程序去做（只触发一次，不需要的数据可以不读），自己立刻返回。

**为什么ET模式下一定要设置非阻塞？**

在ET模式下，一般会设置非阻塞io, 也就是说，在当前没有可读取数据的情况下:
- 如果是阻塞io，recv()会阻塞
- 如果是非阻塞io，recv()会返回-1
因为ET模式下是无限循环读，直到出现错误为EAGAIN或者EWOULDBLOCK，这两个错误表示socket为空，不用再读了，然后就停止循环了，如果不是非阻塞，循环读在socket为空的时候就会阻塞到那里.

**使用EPOLLONESHOT的原因及优点**

epoll在某次循环中唤醒一个事件，并用某个工作进程去处理该fd，而在数据的处理过程中该fd上又有新数据可读（EPOLLIN再次被触发），主线程仍会唤醒这个事件，并另派线程池中另一个线程也来处理这个fd。

为了避免这种情况，需要在注册时间时加上EPOLLSHOT标志，EPOLLSHOT相当于说，某次循环中epoll_wait唤醒该事件fd后，就会从注册中删除该fd,也就是说以后不会epollfd的表格中将不会再有这个fd,也就不会出现多个线程同时处理一个fd的情况。
同时要注意，注册了EPOLLONESHOT事件的socket一旦被某个线程处理完毕，该线程就应该立即重置这个socket的EPOLLONESHOT的事件，以确保这个socket下次可读时，其EPOLLIN事件被触发，进而让其他的工作线程有机会继续处理这个socket。


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


## 7.  服务器工作方式
- 服务器后端的处理方式使用socket通信，利用多路IO复用，可以同时处理多个请求，请求的解析使用预先准备好的线程池，使用**reactor**模式，主线程负责监听，监听到有事件之后，从socket中循环读取数据，然后将读取到的数据封装成一个请求对象插入请求队列。线程池中的工作线程从请求队列中取出一个任务进行处理（解析请求报文），处理的方式用**状态机**。
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

<div align=center><img src=".\fig\get请求报文.jpg"height="465"/> </div>

**从状态机的三种状态**：

- LINE_OK，完整读取一行
- LINE_BAD，报文语法有误
- LINE_OPEN，读取的行不完整
