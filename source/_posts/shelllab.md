---
title: csapp shell lab 相关
date: 2019-12-05 16:05:58
categories:
- unix
tags:
- unix
- shell
---

CS: APP 的[Shell Lab](http://csapp.cs.cmu.edu/3e/labs.html)，要求实现一个支持作业控制的Unix shell。

<!--more-->

# 基本框架

1. 安装信号捕捉函数
2. 在循环中读取命令，创建子进程运行命令。

```c
/*
 * main - The shell's main routine 
 */
int main(int argc, char **argv)
{
    /* Install the signal handlers */
    /* Execute the shell's read/eval loop */
    while (1) {
        /* Read command line */
        if ((fgets(cmdline, MAXLINE, stdin) == NULL) && ferror(stdin))
            app_error("fgets error");
        if (feof(stdin)) { /* End of file (ctrl-d) */
            fflush(stdout);
            exit(0);
        }
        /* Evaluate the command line */
        eval(cmdline);
        fflush(stdout);
    }

    exit(0); /* control never reaches here */
}
```

# 调用waitpid获取子进程终止状态

进程停止（stopped）或终止（terminated）时，会向父进程发送SIGCHLD信号，该信号的默认动作是忽略；父进程可以捕捉SIGCHLD信号，在信号处理函数中根据子进程状态做相应处理。
```c
pid_t waitpid(pid_t pid, int *wstatus, int options);

void sigchld_handler(int sig) 
{
    pid_t pid;
    int status;

    struct job_t *job;

    while ((pid = waitpid(-1, &status, WNOHANG|WUNTRACED)) > 0) { /* Reap a zombie child */
        if((job = getjobpid(jobs, pid)) != NULL) {
            /* Should always get here */
            if (WIFEXITED(status)) {
                deletejob(jobs, pid);
            } else if (WIFSIGNALED(status)) {
                printf("Job [%d] (%d) terminated by signal %d (from pid %d)\n",
                        pid2jid(pid), pid, WTERMSIG(status), getpid());
                deletejob(jobs, pid);
            } else if (WIFSTOPPED(status)) {
                job->state = ST;
                printf("Job [%d] (%d) stoped by signal %d (from pid %d)\n",
                        pid2jid(pid), pid, WSTOPSIG(status), getpid());
            }
        }
    }
    return;
}
```

- `waitpid`的参数`options`或者是0，或者是以下几个常量按位或运算的结果：
  - `WNOHANG`：若没有由`pid`指定的进程状态发生改变，`waitpid`立刻返回，返回值为0；
  - `WUNTRACED`：若有由`pid`指定的进程停止（stopped），`waitpid`也会返回；
  - `WCONTINUED`：若有由`pid`指定的进程在停止（stopped）状态下收到`SIGCONT`信号而继续运行，`waitpid`也会返回。
  
- 一个以及发出但未被接收的信号叫做**待处理信号**（pending signal）。在任何时刻，每种类型的信号都至多只有一个待处理信号。如果一个进程有一个类型为*k*的待处理信号，那么接下来任何发送到这个进程的类型为*k*的信号都不会排队等待，它们都会被直接丢弃。

  因此，shell主进程接收到一个来自子进程的SIGCHLD信号并运行信号捕捉函数时，可能会有其他子进程的运行状态发生改变并向shell主进程发送SIGCHLD信号，shell主进程需要处理所有运行状态发生改变的子进程。

  `sigchld_handler`第10行的`while ((pid = waitpid(-1, &status, WNOHANG|WUNTRACED)) > 0)`，将`waitpid`的返回值作为循环判断条件，只有处理完所有停止（stopped）或终止（terminated）的子进程时`waitpid`返回0，循环结束。

# 前台与后台 

- shell用作业（job）来表示执行一个命令行而创建的进程（一个或多个），这些进程属于同一个进程组。任何时候，至多可以有一个前台作业以及0个或多个后台作业，下图是一个有一个前台作业和两个后台作业的shell。
<div align=center> {% asset_img 8-28.png%} </div>
由键盘输入产生的信号（SIGINT、SIGTSTP等）会发送给前台进程组中的所有进程。函数`tcsetpgrp`可以将指定进程组设置为前台进程组。

# SIGTTIN与SIGTTOU

`SIGTTIN`与`SIGTTOU`属于作业控制信号，当任何一个后台作业中进程试图读终端时，`SIGTTIN`信号会被发送给该作业中的所有进程；类似的，当任何一个后台作业中进程试图写终端时，`SIGTTOU`信号会被发送给该作业中的所有进程。shell主进程应该忽略这些作业控制信号，以避免被意外地停止。

# 使用`sigprocmask`同步进程

函数`sigprocmask`可以获取和改变信号屏蔽字，通过设置信号屏蔽字可以实现进程的同步。

```c
initjobs(); /* Initialize the job list */

while (1) {
    Sigprocmask(SIG_BLOCK, &mask_one, &prev_one); /* Block SIGCHLD */
    if ((pid = Fork()) == 0) { /* Child process */
        Sigprocmask(SIG_SETMASK, &prev_one, NULL); /* Unblock SIGCHLD */
        Execve("/bin/date", argv, NULL);
    }
    Sigprocmask(SIG_BLOCK, &mask_all, NULL); /* Parent process */
    addjob(pid); /* Add the child to the job list */
    Sigprocmask(SIG_SETMASK, &prev_one, NULL); /* Unblock SIGCHLD */
}
```

主进程在创建子进程之前屏蔽了`SIGCHLD`信号，在执行完`addjob(pid)`后再解除屏蔽`SIGCHLD`，保证了在执行`SIGCHLD`的信号处理函数之前已经执行了`addjob(pid)`。