---
layout: post
categories: tech
title: 在Linux中打印函数调用堆栈（一）
---

在编写Java程序时，Exception类的printStacktrace()可以打印异常堆栈，这个小工具极大的提高了调试效率；虽然不是一个好习惯，却很实用。习惯了Java编程，很希望 C/C++里也有这样的小工具可以帮助调试程序. 经过几天查找，发现其实每个系统都提供了打印调用堆栈的函数；这些函数是系统相关，这里仅以Linux下的函数作说明. Linux中共提供了三个函数用于打印调用堆栈：

    /*
     * 函数说明： 取得当前函数的调用堆栈
     * 参数：
     *     buffer：用于存储函数地址的数组
     *     size：buffer数组的长度
     * 返回值：
     *      存储到数组中的函数个数
     */
    int backtrace(void **buffer, int size);
    
    /*
     *
     * 函数说明：将一组函数地址转换为字符串
     * 参数:
     *      buffer: 经由backtrace得到的函数地址
     *      size: buffer数组的长度
     * 返回值:
     *       函数在系统中对应用字符串
     */
    char **backtrace_symbols(void *const *buffer, int size);
    
    /*
     * 函数说明：将一组函数地址转换为字符串
     * 参数:
     *      buffer: 经由backtrace得到的函数地址
     *      size: buffer数组的长度
     *      fd: 输出结果文件描述符
     */
    void backtrace_symbols_fd(void *const *buffer, int size, int fd);

## 示例程序：

    #include <execinfo.h>
    #include <stdio.h>
    #include <stdlib.h>
    #include <unistd.h>
    
    void myfunc3(void)
    {
        int j, nptrs;
        #define SIZE 100
        void *buffer[100];
        char **strings;
    
        nptrs = backtrace(buffer, SIZE);
        printf("backtrace() returned %d addresses\n", nptrs);
    
        backtrace_symbols_fd(buffer, nptrs, STDOUT_FILENO);
    }
    
    void myfunc(void)
    {
        myfunc3();
    }
    
    int main(int argc, char *argv[])
    {
        myfunc();
        return 0;
    }

## 程序运行结果:

    [dma@bp860-10 ~]$ g++ -rdynamic t.cpp -o t #这里的参数 -rdynamic 是必须
    
    [dma@bp860-10 ~]$ ./t
    backtrace() returned 5 addresses
    ./t(_Z7myfunc3v+0x1c)[0x4008c4]
    ./t(_Z6myfuncv+0x9)[0x4008f9]
    ./t(main+0x14)[0x400910] /lib64/tls/libc.so.6(__libc_start_main+0xdb)[0x3f37c1c40b]
    ./t(__gxx_personality_v0+0x3a)[0x40081a]
    [dma@bp860-10 ~]$

虽然现在的程序可以输出函数调用的堆栈，但是函数多了一些前缀，比如：./t(_Z7myfunc3v+0x1c)；这个问题可以通过c++fileter这个工具来解决：

    [dma@bp860-10 ~]$ ./t | c++filt
    ./t(myfunc3()+0x1c)[0x4008c4]
    ./t(myfunc()+0x9)[0x4008f9]
    ./t(main+0x14)[0x400910] /lib64/tls/libc.so.6(__libc_start_main+0xdb)[0x3f37c1c40b]
    ./t(__gxx_personality_v0+0x3a)[0x40081a]
    backtrace() returned 5 addresses

__（未完，待续：C++ ABI: 应用程序二进制接口）__
