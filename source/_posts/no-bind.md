---
title: 如果socket服务端程序未调用bind
date: 2020-02-03 22:12:29
tags:
- unix
- 网络编程
---

unp的习题4.5：如果socket在服务端程序中，移除对`bind`的调用但继续调用`listen`，会发生什么？

<!--more-->

结果是：在未调用`bind`的情况下，调用`listen`会为监听套接字分配临时端口号(ephemeral port)。

用以下代码进行验证：

```c
#include	"unp.h"
#include	<time.h>

int
main(int argc, char **argv)
{
    int					listenfd, connfd;
    socklen_t			len;
    struct sockaddr_in	servaddr, cliaddr;
    char				buff[MAXLINE];
    time_t				ticks;
    char buf[MAXLINE];

    listenfd = Socket(AF_INET, SOCK_STREAM, 0);

    len = sizeof(servaddr);
    getsockname(listenfd, (SA *)&servaddr, &len);
    printf("Local addr: %s\n", Sock_ntop((SA *) &servaddr, len));

//    bzero(&servaddr, sizeof(servaddr));
//    servaddr.sin_family      = AF_INET;
//    servaddr.sin_addr.s_addr = htonl(INADDR_ANY);
//    servaddr.sin_port        = htons(9999);	/* daytime server */

//    Bind(listenfd, (SA *) &servaddr, sizeof(servaddr));

    Listen(listenfd, LISTENQ);

    len = sizeof(servaddr);
    getsockname(listenfd, (SA *)&servaddr, &len);
    printf("Local addr: %s\n", Sock_ntop((SA *) &servaddr, len));

    for ( ; ; ) {
        len = sizeof(cliaddr);
        connfd = Accept(listenfd, (SA *) &cliaddr, &len);
        printf("connection from %s, port %d\n",
               Inet_ntop(AF_INET, &cliaddr.sin_addr, buff, sizeof(buff)),
               ntohs(cliaddr.sin_port));

        ticks = time(NULL);
        snprintf(buff, sizeof(buff), "%.24s\r\n", ctime(&ticks));
        Write(connfd, buff, strlen(buff));

        Close(connfd);
    }
}

```

运行结果为：

```shell
▶ ./daytimetcpsrv
Local addr: 0.0.0.0:44833
connection from 127.0.0.1, port 9999
connection from 127.0.0.1, port 9999

▶ ./daytimetcpsrv
Local addr: 0.0.0.0:43641
connection from 127.0.0.1, port 9999
connection from 127.0.0.1, port 9999
```