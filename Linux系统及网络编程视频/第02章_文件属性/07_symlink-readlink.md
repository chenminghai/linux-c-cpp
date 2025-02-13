# 11. symlink、readlink

这两个函数与符号链接文件有关，讲《Linux基础初级》时，我们有详细的介绍符号链接文件，不过这里还是要先回顾一下符号链接这个东西

## 11.1 链接文件 

符号链接文件也被称为软链接文件。

### 11.1.1 使用ln -s就可以创建符号链接文件

演示：

```shell
ln -s xxx.txt pxxx
```

### 11.1.2 什么是符号链接文件

符号链接文件就是一个快捷图标，它指向了另一个文件. 

### 11.1.3 符号链接 与 硬链接的对比

#### （1）创建硬连接

同一个文件有多个不同的名字，它们指向是同一个inode节点(**删一个硬链接就全删除了**)

#### （2）创建符号链接文件

符号链接文件与它所指向的文件，是两个完全不同的独立的文件，拥有自己独立的inode节点(**删除软链接不影响原始文件**)

符号链接文件的数据就是指向文件的文件名，**文件大小就是名字的字符个数**

#### （3）不能给目录创建硬链接，但是可以给目录创建符号链接

只要你有需要，可以给任何文件创建符号链接文件

#### （4）可以给符号链接文件，创建硬链接吗

可以

## 11.2 symlink

### 11.1.1 函数原型

```c
#include <unistd.h>

int symlink(const char *oldpath, const char *newpath);
```

+ （1）功能：为oldpath，创建符号连接文件newpath。
  + 使用`ln`创建硬链接时，调用的是link函数。
  + 使用`ln -s`创建符号链接时，调用的是symlink，sym就是符号的意思。

+ （2）函数返回值
  + 调用成功返回0
  + 失败返回-1, errno被设置

### 11.1.2 代码演示

## 11.3 readlink

+ (0) 函数原型

  ```c
  #include <unistd.h>
  ssize_t readlink(const char *path, char *buf, size_t bufsiz);
  ```

+ （1）功能：读符号链接文件的数据（指向文件的名字）。
  + 1）`const char *path`：符号连接文件的路径名。
  + 2）`char *buf`：存放名字缓存的地址。
  + 3）`size_t bufsiz`：缓存的大小
  readlink命令，就是调用这个函数实现的。
+ （2）返回值
  + 调用成功，返回读到的字节数
  + 失败返回-1，errno被设置
+ （3）代码演示
  ```c
  #include <stdio.h>
  #include <sys/types.h>
  #include <sys/stat.h>
  #include <fcntl.h>
  #include <unistd.h>
  #include <stdlib.h>


  #define print_error(str) \
  do{\
      fprintf(stderr, "File %s, Line %d, Function %s error\n",__FILE__, __LINE__, __func__);\
      perror(str);\
      exit(-1);\
  }while(0);


  int main(void)
  {
      int ret = symlink("file.txt", "pfile"); // 创建软链接
      if(ret < 0)print_error("symlink error");

      char buf[30] = {0};
      ret = readlink("pfile", buf, sizeof(buf)); // 读取软链接指向的文件的文件名
      if(ret < 0) print_error("readlink error");

      printf("symbol link is %s\n", buf); // 读取软链接ppfile得到它实际指向的文件pfile

  }
  ```
  回显为：
  
  ```shell
  symbol link is file.txt
  ```
## 11.4 符号跟随函数 与 符号不跟随函数

### (1) 符号跟随函数

调用某个函数操作文件，当指定的路径名是符号链接文件时，如果函数最后操作的是符号链接文件所指向的文件，而不是“符号链接文件”本身，这个函数就是符号跟随函数，因为它跟到符号链接文件所指向的背后去了

比如stat、open就是符号跟随函数，因为这些操作的都是符号链接文件所指向的文件

### （2）符号不跟随函数

当路径名是符号链接文件时，函数操作的就是符号链接文件本身，不会跟随。
比如lstat就是符号不跟随函数，因为获取文件属性时，如果操作的是符号链接文件的话，那么获取的是符号链接文本身的属性。

### （3）需要记住哪些是符号链接文件，哪些不是吗？

#### 1）不要记，有规律。

凡是需要指定“文件路径名”函数，只要函数名字是l打头的，比如像lstat，基本都是符号不跟随函数。

如果没有l，比如stat、open、truncate这种的，就是符号跟随函数。

疑问：lseek是符号跟随函数吗？

lseek不需要指定路径名，而是通过文件描述符（fd）操作的，不存在符号跟随与不跟随的问题。

#### 2）你要是实在拿不准

你就自己去操作一个符号链接文件测试下，看看这个函数是跟随的还是不跟随的就知道了，然后你就知道了.
有关符号跟随问题，理解即可，在后续学习中，涉及到的不多。
