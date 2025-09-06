---
layout: post
title: IO多路复用模型
slug: linux-multiplexing
categories: [网络编程, Linux]
tags: [Linux]
---
I/O多路复用实现**单个线程同时监控多个socket的状态**，当**任何一个**socket文件描述符就绪（可读、可写）时，内核会通知应用程序，从而避免了阻塞和轮询的低效。

Linux中提供了select、poll和epoll三种IO多路复用机制。

## select

`select` 函数原型：

```c
int select(int nfds, fd_set *readfds, fd_set *writefds,
           fd_set *exceptfds, struct timeval *timeout);
```

+   **`nfds`**：监控的文件描述符集里最大文件描述符加1。
+   **`readfds`**：指向一个文件描述符集合，用于**监视可读**事件。
+   **`writefds`**：指向一个文件描述符集合，用于**监视可写**事件。
+   **`exceptfds`**：指向一个文件描述符集合，用于**监视异常**事件。
+   **`timeout`**：超时时间，如果为 `NULL`，则无限期阻塞；如果为0，则检查文件描述符后立即返回。

函数返回值：

+   大于0：成功，返回集合中已就绪的IO总个数。
+   等于-1：调用失败。
+   等于0：没有就绪的IO。

**`fd_set`**是一个**位图**数据结构，用于存放一组文件描述符。**每个位**代表一个文件描述符。例如，`fd_set` 中的第 `n` 个位如果被设置为 `1`，则表示文件描述符 `n` 在这个集合中。

`fd_set`本质是一个数组，为了方便操作数组，Linux提供了以下函数：

```cpp
// 将文件描述符fd添加到set集合中
void FD_SET(int fd, fd_set *set);

// 将set集合中, 所有文件描述符对应的标志位设置为0
void FD_ZERO(fd_set *set);

// 将文件描述符fd从set集合中删除
void FD_CLR(int fd, fd_set *set);

// 判断文件描述符fd是否在set集合中
int  FD_ISSET(int fd, fd_set *set);
```

### 工作原理

当用户调用 `select()` 时，将控制权交给内核。

1.   内核会将所有 `fd_set` **全量拷贝**到内核空间。
1.   内核检查传入的所有`fd_set`，对每个 fd，内核会遍历并检查其当前状态：
     +   可读 fd：是否有数据可读。
     +   可写 fd：是否有空间可写。
     +   异常 fd：是否有异常发生。
1.   如果至少有一个fd已经就绪，`select`会**立即返回**，进程不会被阻塞。
1.   如果没有任何 fd 准备好，内核会**阻塞进程**，直到：
     +   至少一个fd状态变化（可读/可写/异常）。
     +   超时。
     +   被信号中断。

1.   进入阻塞状态后，内核会**遍历**传入 `fd_set` 中的每一个fd，对于每个fd都会关联一个等待队列，然后将**当前调用 `select()` 的进程**添加到该等待队列中。（可能会有多个进程调用select()，**一个fd的等待队列可能包含多个进程**）
1.   内核将当前进程的状态设置为**休眠**，并将其从调度器的运行队列中移除。
1.   当fd状态发生变化后，内核找到这个fd对应的等待队列，并遍历这个等待队列上的所有进程，并将这些进程设置为**可运行状态**。
1.   进程被调度执行后，`select()` 系统调用会从阻塞状态中返回。
1.   内核会重新扫描并修改 `fd_set` 集合，**只保留那些已经就绪的fd对应的位**，而将未就绪的位清零。
1.   内核将修改完成后的 `fd_set` 集合**从内核空间拷贝到用户空间**。
1.   `select` 返回后，应用程序需要**再次遍历**这三个 `fd_set` 集合，使用 `FD_ISSET()` 宏来检查每个文件描述符是否仍然被设置。如果 `FD_ISSET()` 返回真，那么就表示文件描述符已经就绪。
> 内核在`select`执行期间，可能会多次遍历`fd_set`


select进程启动时，没有数据到达网卡：

![](/assets/images/select1.png)


有数据到达时，唤醒进程
![](/assets/images/select2.png)

### select的缺陷
1. 性能开销大
+ 调用select时需要将参数中的fd_set从用户空间拷贝到内核空间，select执行完后，还需将fd_set从内核空间拷贝回用户空间，高并发场景下这样的拷贝会消耗极大资源。
+ 进程被唤醒后，不知道哪些连接已就绪，需要遍历传递进来的所有fd_set的每一位。
+ select只返回就绪fd的个数，具体哪个fd就绪还需要遍历。

2. 同时能够监听的文件描述符数量太少。受限于sizeof(fd_set)的大小，在编译内核时就确定了且无法更改。一般32位操作系统是1024，64位是2048。

## poll

`poll` 函数原型：

```c
int poll(struct pollfd *fds, nfds_t nfds, int timeout);
```

+   **`fds`**： `pollfd` 结构体数组。
+   **`nfds`**：`fds` 数组中元素的数量。
+   **`timeout`**：超时时间，单位为毫秒。设为0表示立即返回不等待；设为-1表示无期限等待。

其中`pollfd`结构体定义如下：
```c
struct pollfd {
    int fd;           // 要监听的文件描述符
    short events;     // 要监听的事件
    short revents;    // 文件描述符fd上实际发生的事件
};
// events
#define POLLIN		0x001		/* There is data to read.  */
#define POLLPRI		0x002		/* There is urgent data to read.  */
#define POLLOUT		0x004		/* Writing now will not block.  */

// revents返回的事件宏与events几乎相同,但还会包括一些错误事件
#define POLLERR		0x008		/* Error condition.  */
#define POLLHUP		0x010		/* Hung up. 对端关闭连接或半关闭 */
#define POLLNVAL	0x020		/* Invalid polling request.  */
```

### 工作原理

`poll`与 `select`的工作原理基本相同，除了：

+   **文件描述符数量**：`poll`使用动态数组，不受`FD_SETSIZE`的限制。

### poll的缺陷

虽然`poll`解决了`select`的数量限制，但它仍然存在**全量拷贝**和**线性遍历**的问题。当监视的文件描述符数量非常多时，`poll`的性能瓶颈会非常明显。


## epoll

`epoll`是性能最高的 I/O 多路复用机制，它解决了`select`和`poll`的所有性能瓶颈。

`epoll`不是一个函数，而是**一组系统调用**：

### 创建epoll实例
```cpp
int epoll_create(int size);

int epoll_create1(int flags);
```

+   `size`：这个参数目前会被内核忽略，内核会动态地、按需地分配内存来管理文件描述符，不再依赖于这个初始值。但必须传入一个大于0的值。
+   `flags`：通常传入`EPOLL_CLOEXEC`，当进程调用 `exec`系列函数来执行一个新的程序时，这个 `epoll` 文件描述符会被自动关闭。
+   返回值：`epoll` 实例的**文件描述符**，**使用完`epoll`必须需要调用`close`关闭。**

### sockfd管理控制

`epoll_ctl()`用于向 `epoll` 实例中**注册、修改或删除**要监视的文件描述符。

```cpp
/*
 * epfd: epoll实例的文件描述符
 * op: 操作类型
 	* EPOLL_CTL_ADD : 添加
 	* EPOLL_CTL_MOD : 修改
 	* EPOLL_CTL_DEL : 删除
 * fd: 要操作的sockfd
 * event: 指定要监视的事件
 */
int epoll_ctl(int epfd, int op, int fd, struct epoll_event *event);
```

其中`epoll_event`结构如下：

```c
typedef union epoll_data {
  void *ptr;
  int fd;
  uint32_t u32;
  uint64_t u64;
} epoll_data_t;

struct epoll_event {
  uint32_t events;	    /* Epoll events */
  epoll_data_t data;	/* User data variable */
};
```

`EPOLL_EVENTS`说明：

| `EPOLL_EVENTS`       | 作用                                 | 触发条件                                                     |
| -------------------- | ------------------------------------ | ------------------------------------------------------------ |
| **`EPOLLIN`**        | **文件描述符可读。**                 | 文件描述符上有数据可供读取。<br>1. **TCP Socket**：对端发送了数据，可以调用 `read()` 或 `recv()`。<br>2. **监听 Socket**：有新的连接请求到达，可以调用 `accept()`。 |
| **`EPOLLPRI`**       | **文件描述符有紧急数据可读。**       | 对端发送了带外（Out-of-Band）数据。<br>通常用于 TCP 的紧急数据。 |
| **`EPOLLOUT`**       | **文件描述符可写。**                 | 文件描述符可以写入数据而不会阻塞。<br>1. **TCP Socket**：发送缓冲区有空闲空间。<br>2. **非阻塞 `connect()`**：连接建立成功。 |
| **`EPOLLERR`**       | **文件描述符发生错误。**             | 1. `read()` 或 `write()` 失败。<br>2. TCP 对端发送了 RST 包。<br>**注意**：`EPOLLERR` 通常伴随 `EPOLLHUP` 一起返回。 |
| **`EPOLLHUP`**       | **对端挂断或错误。**                 | 1. 对端关闭连接（发送 FIN）。<br>2. 管道或 FIFO 的写入端被关闭。 |
| **`EPOLLRDHUP`**     | **对端半关闭或关闭连接。**           | TCP 对端关闭了连接的写端，或整个连接被关闭。在 `epoll_wait` 中，这通常意味着对端已经关闭了。 |
| **`EPOLLEXCLUSIVE`** | **独占模式。**                       | 当多个 `epoll` 实例监视同一个文件描述符时，只唤醒其中一个阻塞的进程。<br> |
| **`EPOLLWAKEUP`**    | **唤醒模式。**                       | 当进程被此事件唤醒后，内核会暂时禁止进入休眠。用于确保在事件处理完成前，进程不会被意外休眠。 |
| **`EPOLLONESHOT`**   | **一次性模式。**                     | 仅在第一次事件发生时报告一次。之后该文件描述符上的事件不再被监视。常与多线程配合使用，以避免多个线程同时处理同一事件。 |
| **`EPOLLET`**        | **边缘触发模式（Edge Triggered）。** | 只在文件描述符状态发生**改变**时报告一次。<br>例如，从不可读变为可读时，只报告一次 `EPOLLIN`。即使缓冲区还有数据，也不会再次报告。 |

`epoll_data_t`说明：
`epoll_data_t`的核心价值在于**避免在`epoll_wait()`返回后，再次通过fd去查找数据**。
+ **`ptr`**：通常指向一个自定义结构体，包含了与该文件描述符相关的信息。
+ **`fd`**：通常用于直接存储文件描述符。
+ **`u32`和`u64`**：用于存储一些标志或ID。


### 等待事件发生

```cpp
/*
 * epfd: epoll实例的文件描述符
 * event: epoll_event数组，用于存放已就绪的事件
 * events: 数组的大小
 * timeout: 超时时间，单位为毫秒
 * 返回值: 就绪的事件总数
 */
int epoll_wait(int epfd, struct epoll_event *events,
               int maxevents, int timeout);
```
### 工作原理

#### 核心数据结构

`epoll` 在内核中主要使用了两个数据结构：

1.  **红黑树**：用于存放所有**被监视**的文件描述符。查找、插入和删除的效率都很高 `O(logN)`。
1.  **就绪列表**：一个双向链表，用于存放所有**已经就绪**的文件描述符。

#### 核心流程

1.  **`epoll_create()`**：当调用`epoll_create()`时，内核会创建一个`epoll`实例，并在内部创建一个`eventpoll`结构。

    ```cpp
    struct eventpoll {
        /* ... */
        wait_queue_head_t wq;      // 等待队列链表，存放阻塞的进程
        struct list_head rdllist;  // 就绪列表
        struct rb_root rbr;        // 红黑树
        /* ... */
    };
    ```
    + wq：如果当前进程没有数据需要处理，会把当前进程描述符和回调函数构造一个等待队列，放入当前wq等待队列。
    + rdllist：当有连接数据就绪，内核会把就绪的连接放到rdllist链表里。
    + rbr：管理用户进程下添加进来的所有socket连接。
    ![](/assets/images/epoll.png)

1.  **`epoll_ctl()`**：内核会将`fd`和你指定的事件作为一个节点插入到**红黑树**中， 同时会为`fd`关联一个**回调函数**。
    1. 创建一个`epitem`对象，主要包含两个字段，分别存放`sockfd`和所属的`eventpoll`对象的指针；
    2. 将一个数据到达时用到的回调函数添加到`socket`的进程等待队列中，注意，进程是放在`eventpoll`的等待队列中，等待被``epoll_wait`函数唤醒，而不是放在`socket`的进程等待队列中；
    3. 将`epitem`对象插入红黑树；
    ![](/assets/images/epoll2.png)
1.  **`epoll_wait()`**：检查内核中的**就绪列表**，如果不为空，它会立即返回；如果为空，则会阻塞进程，直到有事件发生。
    1. 检查`eventpoll`对象的就绪的连接`rdllist`上是否有数据到达，如果没有就把当前的进程描述符添加到一个等待队列项里，加入到eventpoll的进程等待队列里，然后阻塞当前进程，等待数据到达时通过回调函数被唤醒。
    1. 当`eventpoll`监控的连接上有数据到达时：
        1. socket的数据接收队列有数据到达，会通过进程等待队列的回调函数唤醒红黑树中的节点epitem；
        1. 回调函数将有数据到达的epitem添加到eventpoll对象的就绪队列rdllist中；
        1. 回调函数检查eventpoll对象的进程等待队列上是否有等待项，通过回调函数default_wake_func唤醒这个进程，进行数据的处理；
        1. 当进程被唤醒后，继续从epoll_wait时暂停的代码继续执行，把rdlist中就绪的事件返回给用户进程。

    ![](/assets/images/epoll3.png)
1.  **返回**：
    +   当`epoll_wait()`返回时，只需遍历它返回的**就绪事件数组**，而不需要遍历所有监视的文件描述符。

## 参考
[](https://www.cnblogs.com/88223100/p/Deeply-learn-the-implementation-principle-of-IO-multiplexing-select_poll_epoll.html)
