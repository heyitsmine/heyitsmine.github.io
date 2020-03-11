---
title: 记录一个CSAPP中Tiny Web Server的问题
date: 2020-03-11 10:26:13
categories:
- unix
tags:
- unix
- 网络编程
- CSAPP
- webserver
---

Tiny Web Server读取客户端发送来的请求头并将其输出到Terminal，此外不对首部的字段做其他处理。读取首部的代码如下：

<!--more-->

```c
void read_requesthdrs(rio_t *rp) 
{
    char buf[MAXLINE];

    Rio_readlineb(rp, buf, MAXLINE);
	printf("%s\n", buf);
    while(strcmp(buf, "\r\n")) {
		Rio_readlineb(rp, buf, MAXLINE);
		printf("%s", buf);
    }
    return;
}
```

该函数存在的一个问题是，若客户端的连接断开，`Rio_readlineb(rp, buf, MAXLINE)`将读取到`EOF`，而循环判断条件为`strcmp(buf, "\r\n")`，由于无法从客户端读取到数据，`buf`内容将永远不会更新，该循环成为死循环。为避免这种情况，`Rio_readlineb(rp, buf, MAXLINE)`返回`0`时函数应返回：

```c
void read_requesthdrs(rio_t *rp) 
{
    char buf[MAXLINE];
	
    while(Rio_readlineb(rp, buf, MAXLINE) && strcmp(buf, "\r\n")) {
		printf("%s", buf);
    }
    return;
}
```

