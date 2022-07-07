---
title: socket编程实践
date: 2022-03-11 19:20:27
tags:
     - c++
categories: 
     - socket
     - socket编程实践
comments: true
---

> ​                                                                       socket编程实践

<!-- more -->

#### socket编程实践

##### 服务器端代码 server.cpp

```c++
#include <stdio.h>
#include <string.h>
#include <stdlib.h>
#include <unistd.h>
#include <arpa/inet.h>
#include <sys/socket.h>
#include <netinet/in.h>

int main() {
    //创建套接字
    int listenfd, connfd;

    if ((listenfd = socket(AF_INET, SOCK_STREAM, 0)) == -1) {
        cout << "create socket error" << endl;
        exit(0);
    }

    //将套接字和IP、端口绑定
    struct sockaddr_in serv_addr;
    memset(&serv_addr, 0, sizeof(serv_addr));    //每个字节都用0填充
    serv_addr.sin_family = AF_INET;               //使用IPv4地址
    serv_addr.sin_addr.s_addr = inet_addr(INADDR_ANY);    
    serv_addr.sin_port = htons(1234);              //端口1234

    if (bind(listenfd, (struct sockaddr*)&serv_addr, sizeof(serv_addr)) == -1) {
        cout << "bind socket error" << endl;
        exit(0);
    }

    //进入监听状态，等待用户发起请求
    if (listen(listenfd, 10) == -1) {
        cout << "listen socket error" << endl;
        exit(0);
    }

    //等待接收客户端请求
    while (1) {
        struct sockaddr_in client_addr;
        if ((connfd = accept(listenfd, (struct sockaddr*)&client_addr,       sizeof(client_addr))) == -1) {
            cout << "accept socket error" << endl;
            continue;
        }
        //有客户端建立连接之后
        if ((childpid = fork()) == 0) { //创建子进程，子进程运行区域
            close(listenfd);  // 关闭监听套接字
            func();    // 自行定义处理客户端请求的函数
            close(connfd);  // 子进程执行完，关闭连接套接字
        }
        else { // 父进程运行区域
            close(connfd); //  调用fork()后，父进程关闭连接套接字，等待其他连接的到来
        }
    }
    close(listenfd); //整个关掉
    
    void func(int sockfd){ //自定义处理请求的函数
        // 函数体
    }

    return 0;
}
```

##### 客户端代码 client.cpp

```c++
#include <stdio.h>
#include <string.h>
#include <stdlib.h>
#include <unistd.h>
#include <arpa/inet.h>
#include <sys/socket.h>

int main(){
    //创建套接字
    int sockfd;
    
    if((sockfd = socket(AF_INET, SOCK_STREAM, 0)) == -1){
        cout << "create socket error" << endl;
        exit(0);
    }
    
    //向服务器（特定的IP和端口）发起请求
    struct sockaddr_in serv_addr;
    memset(&serv_addr, 0, sizeof(serv_addr));            //每个字节都用0填充
    serv_addr.sin_family = AF_INET;                      //使用IPv4地址
    serv_addr.sin_addr.s_addr = inet_addr("127.0.0.1");  //具体的IP地址
    serv_addr.sin_port = htons(1234);                    //端口1234
    connect(sock, (struct sockaddr*)&serv_addr, sizeof(serv_addr));
    
    if( connect(sockfd, (struct sockaddr*)&ser_vaddr, sizeof(ser_vaddr)) == -1){
        cout << "connect error" << endl;
        exit(0);
    }
    
    //连接成功后开始发送数据
    char message[4096];
    for(int i=0;  ;i++){  //可以一直发
        send(sockfd, &message, sizeof(message));
    }
    
    //读取服务器传回的数据
    char buffer[40];
    for(int i=0;  ;i++){   // 一直接收
        read(sockfd, &buffer, sizeof(buffer));
    }
   
    //关闭套接字
    close(sockfd);

    return 0;
}
```

