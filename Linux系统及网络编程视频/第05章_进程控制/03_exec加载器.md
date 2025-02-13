# 3. exec加载器

exec加载器就是我们之前介绍的加载函数。

## 3.1 exec的作用

父进程fork复制出子进程的内存空间后，子进程内存空间的代码和数据和父进程是相同的，这样没有太大的意义，我们需要在子进程空间里面运行全新的代码，这样才有意义。

怎么运行新代码？  
我们可以在if(ret==0){}里面直接写新代码，但是这样子很麻烦，如果新代码有上万行甚至更多的话，这种做法显然是不行的，因此就有了exec加载器。

有了exec后，**我们可以单独的另写一个程序，将其编译好后，使用exec来加载即可**

## 3.2 exec函数族

exec的函数有很多个，它们分别是execve、execl、execv、execle、execlp、execvp，都是加载函数。

其中execve是系统函数，其它的execl、execv、execle、execlp、execvp都是**基于execve封装得到的库函数**

因此我们这里重点介绍execve函数，这个函数懂了，其它的函数原理是一样的

### 3.2.1 execve函数原型

```c
#include <unistd.h>
int execve(const char *filename, char **const argv, char **const envp);
```

+ （1）功能
    向子进程空间加载新程序代码（编译后的机器指令）。
+ （2）参数  
  + 1）filename：新程序（可执行文件）所在的路径名
    可以是任何编译型语言所写的程序，比如可以是c、c++、汇编等，这些语言所写的程序被编译为机器指令后，都可以被execve这函数加载执行。正是由于这一点特性，我们才能够在C语言所实现的OS上，运行任何一种编译型语言所编写的程序

    疑问：java可以吗？  
    java属于解释性语言，它所写的程序被编译后只是字节码，并不是能被CPU直接执行的机器指令，所以不能被execve直接加载执行，而是被虚拟机解释执行。execve需要先加载运行java虚拟机程序，然后再由虚拟机程序去将字节码解释为机器指令，再有cpu去执行，在后面还会详细讨论这个问题。

  + 2）argv：传给main函数的参数，比如我可以将命令行参数传过去
  + 3）envp：环境变量表

+ （3）返回值：函数调用成功不返回，失败则返回-1，且errno被设置

+ （4）代码演示

    ```c
    // fork.c 父进程调用子进程
    #include <stdio.h>
    #include <unistd.h>

    int main(int argc, char **argv, char **environ)
    {
        pid_t ret = -1;
        ret = fork();// 创建子进程

        if(ret > 0){
            // 父进程必须sleep一下，要不父进程先退出子进程就不会执行完了
            sleep(1);
        }else if(ret==0){
            // 子进程
            execve("./new_process", argv, environ); 
        }
    }
    ```

    ```c

    // new_process.c，用于编译生成可执行文件然后给fork.c调用
    #include <stdio.h>

    int main(int argc, char **argv, char **environ)
    {

        printf("当前所在程序：new_process.c\n");
        int i = 0;

        printf("\n-----------命令行参数----------\n");

        for(i = 0; i < argc; i++){
            printf("%s\n", argv[i]);
        }

        printf("\n-----------环境变量----------\n");
        for(i = 0; NULL != environ[i]; i++){
            printf("%s\n", environ[i]);
        }
        return 0;
    }
    ```

    编译后执行`fork name lsg`结果如下：

    ```shell
    当前所在程序：new_process.c

    -----------命令行参数----------
    /workspace/exec/fork

    -----------环境变量----------
    HOSTNAME=9bcde94681d8
    SHELL=/bin/sh
    TERM=xterm-256color
    ISOUTPUTPANE=1
    TMUX=/tmp/tmux-0/cloud91.9,1303,37
    PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
    SUPERVISOR_GROUP_NAME=cloud9
    PWD=/workspace/exec
    NODE_PATH=/usr/lib/nodejs:/usr/lib/node_modules:/usr/share/javascript
    TMUX_PANE=%37
    NODE_ENV=production
    SUPERVISOR_ENABLED=1
    SHLVL=1
    HOME=/root
    SUPERVISOR_PROCESS_NAME=cloud9
    SUPERVISOR_SERVER_URL=unix:///var/run/supervisor.sock
    BASE_PROC=/workspace
    CUSTOM=43
    _=/usr/bin/node
    ```

    也可以自定义argv和environ

    ```c
    #include <stdio.h>
    #include <stdlib.h>
    #include <sys/types.h>
    #include <unistd.h>

    int main(int argc, char **argv)
    {
        pid_t ret = 0;
  
        ret = fork();
        if(ret > 0){
            sleep(1);
        }else if(ret == 0){
            extern char **environ; // 系统环境变量数组
            //int execve(const char *filename, char **const argv, char **const envp);
            char *my_argv[] = {"fds", "dsfds", NULL};
            char *my_env[] = {"AA=aaaaa", "BB=bbbbb", NULL};
            execve("./new_process", my_argv, my_env);
        }

      return 0;
    }
    ```

    new_process不变，执行结果如下，可见这样就完全分隔了父子进程的实现：

    ```shell
    当前所在程序：new_process.c

    -----------命令行参数----------
    fds
    dsfds

    -----------环境变量----------
    AA=aaaaa
    BB=bbbbb
    ```

    ```c
                 命令行参数/环境表                    命令行参数/环境表                  命令行参数/环境表
    终端窗口进程————————————————————>a.out（父进程）——————————————————————>a.out（子进程）————————————————>新程序
                                                         fork                             exec
    ```

  exec的作用：将新程序代码加载（拷贝）到子进程的内存空间，替换掉原有的与父进程一模一样的代码和数据，让子进程空间运行全新的程序。

## 3.3 在命令行执行./a.out，程序是如何运行起来的

+ （1）窗口进程先fork出子进程空间
+ （2）调用exec函数加载./a.out程序，并把命令行参数和环境变量表传递给新程序的main函数的形参

## 3.4 双击快捷图标，程序是怎么运行起来的

+ （1）图形界面进程fork出子进程空间
+ （2）调用exec函数，加载快捷图标所指向程序的代码
  以图形界面方式运行时，就没有命令行参数了，但是会传递环境变量表。
