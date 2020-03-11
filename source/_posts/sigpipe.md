---
title: SIGPIPE的产生
date: 2020-03-11 15:22:27
categories:
- unix
tags:
- unix
- 网络编程
- CSAPP
- webserver
---

当一个进程向已经收到`RST`的`socket`写数据时，`SIGPIPE`信号会被发送给该进程。

<!--more-->

以下是一段服务端向客户端发送响应报文的代码，第13行调用`write`向`fd`写入报文首部，第21行调用`write`向`fd`写入报文主体。若客户端连接断开，第一次`write`后，`fd`会收到`RST`，第二次`write`将产生`SIGPIPE`信号。
```c
void serve_static(int fd, char *filename, int filesize)
{
    int srcfd;
    char *srcp, filetype[MAXLINE], buf[MAXBUF];
 
    /* Send response headers to client */
    get_filetype(filename, filetype);
    sprintf(buf, "HTTP/1.0 200 OK\r\n");
    sprintf(buf, "%sServer: Tiny Web Server\r\n", buf);
    sprintf(buf, "%sConnection: close\r\n", buf);
    sprintf(buf, "%sContent-length: %d\r\n", buf, filesize);
    sprintf(buf, "%sContent-type: %s\r\n\r\n", buf, filetype);
    Rio_writen(fd, buf, strlen(buf));
    printf("Response headers:\n");
    printf("%s", buf);

    /* Send response body to client */
    srcfd = Open(filename, O_RDONLY, 0);
    srcp = Mmap(0, filesize, PROT_READ, MAP_PRIVATE, srcfd, 0);
    Close(srcfd);
    Rio_writen(fd, srcp, filesize);
    Munmap(srcp, filesize);
}
```