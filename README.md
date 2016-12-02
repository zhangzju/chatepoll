# Chat E-poll
E-poll模型的简单运用，编译时不需要makefile，但是utility.h文件需要和所使用的文件在统一路径下。



## E-poll

 epoll的接口非常简单，一共就三个函数：
1. ```c
     int epoll_create(int size); 
   ```


     创建一个epoll的句柄，size用来告诉内核这个监听的数目一共有多大。这个参数不同于select()中的第一个参数，给出最大监听的 fd+1的值。需要注意的是，当创建好epoll句柄后，它就是会占用一个fd值，在linux下如果查看**/proc/进程id/fd/**，是能够看到这个 fd的，所以在使用完epoll后，必须调用close()关闭，否则可能导致fd被耗尽。

2. ```c
   int epoll_ctl(int epfd, int op, int fd, struct epoll_event *event);
   ```

   epoll的事件注册函数，它不同与select()是在监听事件时告诉内核要监听什么类型的事件，而是在这里先注册要监听的事件类型。第一个参数是epoll_create()的返回值，第二个参数表示动作，用三个宏来表示：
   EPOLL_CTL_ADD：注册新的fd到epfd中；
   EPOLL_CTL_MOD：修改已经注册的fd的监听事件；
   EPOLL_CTL_DEL：从epfd中删除一个fd；
   第三个参数是需要监听的fd，第四个参数是告诉内核需要监听什么事，struct epoll_event结构如下：
   ​

   ```c
   1. typedef union epoll_data {
       void *ptr;
       int fd;
       _uint32t u32;
       _uint64t u64;
      } epoll_data_t;

   struct epoll_event {

       __uint32_t events; /* Epoll events */
       epoll_data_t data; /* User data variable */

   };

   ```

events可以是以下几个宏的集合：

* EPOLLIN ：表示对应的文件描述符可以读（包括对端SOCKET正常关闭）；
* EPOLLOUT：表示对应的文件描述符可以写；
* EPOLLPRI：表示对应的文件描述符有紧急的数据可读（这里应该表示有带外数据到来）；
* EPOLLERR：表示对应的文件描述符发生错误；
* EPOLLHUP：表示对应的文件描述符被挂断；
* EPOLLET： 将EPOLL设为边缘触发(Edge Triggered)模式，这是相对于水平触发(Level Triggered)来说的。
* EPOLLONESHOT：只监听一次事件，当监听完这次事件之后，如果还需要继续监听这个socket的话，需要再次把这个socket加入到EPOLL队列里



1. ```c
     int epoll_wait(int epfd, struct epoll_event * events, int maxevents, int timeout);
   ```

等待事件的产生，类似于select()调用。参数events用来从内核得到事件的集合，maxevents告之内核这个events有多大，这 个 maxevents的值不能大于创建epoll_create()时的size，参数timeout是超时时间（毫秒，0会立即返回，-1将不确定，也有 说法说是永久阻塞）。该函数返回需要处理的事件数目，如返回0表示已超时。



## 使用方法

由于同时使用了Utility的部分函数，因此需要在同一个路径下编译，这个项目的结构比较简单，只需按照如下方式编译即可：

```shell
	gcc -o client client.c
```



编译server时将client替换成server即可。

编译完成后的可执行文件直接运行即可，但是不要忘了添加用户权限。