---
实现客户端和服务端简单的TCP通信，
网络编程中我们一般会使用 C/S 架构，即包含服务器端和客户端。

* TCP 网络编程的服务器端

服务器端一般先用 socket 创建一个套接字，然后用 bind 给这个套接字绑定地址（即 ip+端口号），然后调用 listen 把这个套接字置为监听状态，随后调用 accept 函数从已完成连接队列中取出成功建立连接的套接字，以后就在这个新的套接字上调用 send、recv 来发送数据、接收数据，最后调用 close 来断开连接释放资源即可。


### server.cpp
#### Procedure
1. 构造一个套接字描述符；  
   ==> `int socket(int domain, int type, int protocol);`
2. 构造一个网络地址A以存储服务端地址；  
   ==> `struct sockaddr_in;`
3. 绑定上述套接字描述符和网络地址A；   
   ==> `int bind(int sock_fd, struct sockaddr * sock_addr, int addr_len);`
4. 将套接字描述符置于监听状态；    
   ==> `int listen(int sock_fd, int backlog);`
5. 构造一个网络地址B用以存储客户端地址；    
   ==> `struct sockaddr_in;`
6. 将申请与服务端建立连接的客户端地址存储到网络地址B中，并为该客户端套接字分配描述符；   
   ==> `int accept(int server_fd, struct sockaddr * sock_add, int* addr_len);`
7. 建立缓冲区；   
   ==> `char buffer[1000];`
8. 创建循环：   
   1. 重新初始化缓冲区；   
   ==> `void*  memset(void *s, int ch, size_t n);`
   2. 从客户端读取数据存到缓冲区中；   
   ==> `int recv(int cli_fd, void* buffer, size_t length, unsigned int flag);`
   3. 检查读取数据长度，判断客户端是否发出退出提示；
   4. 输出客户端数据；
9. 关闭本进程服务端和客户端套接字描述符；    
   ==> `int close(int fd);`

#### Code
```c++
#include <sys/types.h>
#include <sys/socket.h>
#include <stdio.h>
#include <netinet/in.h>
#include <arpa/inet.h>
#include <unistd.h>
#include <string.h>
#include <stdlib.h>
#include <fcntl.h>
#include <sys/shm.h>
#include <iostream>
using namespace std;

int main()
{
    //1. 构造一个套接字描述符；
    //定义sockfd
    int server_sockfd = socket(AF_INET,SOCK_STREAM, 0);
    /*
        It's Ok to use PF_INET(Protocol Family) to initialize a socket as well as AF_INET(Address Family)
        int socket(int domain, int type, int Protocol);
        需要指明使用的协议栈， 服务类型（数据报服务: SOCK_DGRAM，数据流服务: SOCK_STREAM），
        使用的协议（一般取0，即系统默认）; 返回一个整型即Socket描述符。
    */

    // 2. 构造一个网络地址A以存储服务端地址；
    //定义sockaddr_in
    struct sockaddr_in server_sockaddr;
    server_sockaddr.sin_family = AF_INET;//TCP/IP协议族
    server_sockaddr.sin_port = htons(8023);//端口号 host to net, short; 从主机字节序转化为网络字节序
    server_sockaddr.sin_addr.s_addr = inet_addr("127.0.0.1");//ip地址，127.0.0.1是环回地址，相当于本机ip

    //3. 绑定上述套接字描述符和网络地址A；
    //bind，成功返回0，出错返回-1
    if(bind(server_sockfd,(struct sockaddr *)&server_sockaddr,sizeof(server_sockaddr))==-1)
    {
        perror("bind");//输出错误原因到stderr
        exit(1);//结束程序
    }
    /*
        int bind(int sock_fd, struct sockaddr * sock_adrr, int length);
        指明要绑定的套接字，地址，地址长度
    */

    //4. 将套接字描述符置于监听状态；
    //listen，成功返回0，出错返回-1
    if(listen(server_sockfd,20) == -1)
    {
        perror("listen");//输出错误原因到stderr
        exit(1);//结束程序
    }
    /*
        int listen(int sock_fd, int backlog);
        将套接字置于监听状态，backlog为accept队列长度
    */

    //5. 构造一个网络地址B用以存储客户端地址；
    //客户端网络地址
    struct sockaddr_in client_addr;
    socklen_t length = sizeof(client_addr);

    //6. 将申请与服务端建立连接的客户端地址存储到网络地址B中，并为该客户端套接字分配描述符；
    //成功返回非负描述字，出错返回-1
    int conn = accept(server_sockfd, (struct sockaddr*)&client_addr, &length);
    if(conn<0)
    {
        perror("connect");//输出错误原因
        exit(1);//结束程序
    }
    cout<<"客户端成功连接\n";
    /*
        int accept(int server_sock_fd, struct sockaddr * client_addr, int * addr_length);
        指明服务器套接字描述符，用于存放客户端地址的地址，地址长度, 返回客户端套接字描述符
    */

    //7. 建立缓冲区；
    //接收缓冲区
    char buffer[1000];

    //8. 创建循环：
    //不断接收数据
    while(1)
    {
        //1. 重新初始化缓冲区；
        memset(buffer,0,sizeof(buffer));
        //2. 从客户端读取数据存到缓冲区中；
        int len = recv(conn, buffer, sizeof(buffer),0);
        //3. 检查读取数据长度，判断客户端是否发出退出提示；
        //客户端发送exit或者异常结束时，退出
        if(strcmp(buffer,"exit")==0 || len<=0)
            break;
        //4. 输出客户端数据；
        cout<<"收到客户端信息："<<buffer<<endl;
    }
    /*
        int recv(int sock_fd, void* buffer, int buff_len, unsigned int flag);
        指明套接字描述符，用于存放的缓存地址，缓存区大小，flag默认0，返回接收到的数据长度 
    */
    //9. 关闭本进程服务端和客户端套接字描述符
    close(conn);
    close(server_sockfd);
    // 可能会有多个进程共享同一个套接字，close会将套接字的引用减一，当引用数量为0时，关闭连接并撤销关键字
    return 0;
}
```

* TCP 网络编程的客户端

与服务器不同，客户端并不需要 bind 绑定地址，因为端口号是系统自动分配的，而且客户端也不需要设置监听的套接字，因此也不需要 listen。客户端在用 socket 创建套接字后直接调用 connect 向服务器发起连接即可，connect 函数通知 Linux 内核完成 TCP 三次握手连接，最后把连接的结果作为返回值。成功建立连接后我们就可以调用 send 和 recv 来发送数据、接收数据，最后调用 close 来断开连接释放资源。


### client.cpp
#### Procedure
1. 创建客户端套接字描述符；    
   ==> `int socket(int domain, int type, int protocol);`
2. 创建网络地址并初始化为服务端网络地址；    
   ==> `struct sockaddr_in;`
3. 申请创建客户端到服务端的连接；   
   ==> `int connect(int client_fd, struct sockaddr_in * server_addr, size_t addr_len);`
4. 创建发送缓冲区；    
   ==> `char buffer[1000];`
5. 创建循环：   
   1. 重置发送缓冲区；   
   ==> `void* memset(void* s, int ch, size_t n);`
   2. 获取外部输入到发送缓冲区；  
   3. 将发送缓冲区内容写入客户端套接字描述符；   
   ==> `int send(int client_fd, const void* buffer, size_t length);`
   4. 判断是否输入“exit”；
6. 关闭本进程客户端套接字描述符；   
   ==> `int close(int fd);`

#### Code
```c++
#include <sys/types.h>
#include <sys/socket.h>
#include <stdio.h>
#include <netinet/in.h>
#include <arpa/inet.h>
#include <unistd.h>
#include <string.h>
#include <stdlib.h>
#include <fcntl.h>
#include <sys/shm.h>
#include <iostream>
using namespace std;

int main()
{
    // 1. 创建客户端套接字描述符；
    //定义sockfd
    int sock_cli = socket(AF_INET,SOCK_STREAM, 0);

    // 2. 创建网络地址并初始化为服务端网络地址；
    //定义sockaddr_in
    struct sockaddr_in servaddr;
    memset(&servaddr, 0, sizeof(servaddr));
    servaddr.sin_family = AF_INET;//TCP/IP协议族
    servaddr.sin_port = htons(8023);  //服务器端口
    servaddr.sin_addr.s_addr = inet_addr("127.0.0.1");  //服务器ip

    // 3. 申请创建客户端到服务端的连接;
    //连接服务器，成功返回0，错误返回-1
    if (connect(sock_cli, (struct sockaddr *)&servaddr, sizeof(servaddr)) < 0)
    {
        perror("connect");
        exit(1);
    }
    cout<<"连接服务器成功！\n";
    /*
        int connect(int cli_sock_fd, struct sockaddr * ser_addr, int addr_len);
    */

    // 4. 创建发送缓冲区；
    char sendbuf[100];
    char recvbuf[100];
    // 5. 创建循环：
    while (1)
    {
       // 1. 重置发送缓冲区；
        memset(sendbuf, 0, sizeof(sendbuf));
       // 2. 获取外部输入到发送缓冲区；
        cin>>sendbuf;
       // 3. 将发送缓冲区内容写入客户端套接字描述符；
        send(sock_cli, sendbuf, strlen(sendbuf),0); //发送
       // 4. 判断是否输入“exit”；
        if(strcmp(sendbuf,"exit")==0)
            break;
    }
    // 6. 关闭本进程客户端套接字描述符；
    close(sock_cli);
    return 0;
}
```

下一篇：[服务端(多线程, 面向对象)](https://errorbeep.github.io/服务端(多线程,-面向对象))

