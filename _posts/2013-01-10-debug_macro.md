---
layout: post
categories: tech
title: Debug Macro
---

## Code slice

    #define debug(fmt,...) do {                                             \
        fprintf(stderr,"[ DEBUG ] : [ %s, %d ] ",__FILE__,__LINE__);    \
        fprintf(stderr,fmt,##__VA_ARGS__);                              \
        fprintf(stderr,"\n");                                           \
    } while(0)

## How to use

    #include <stdio.h>
    
    #define debug(fmt,...) do {                                             \
        fprintf(stderr,"[ DEBUG ] : [ %s, %d ] ",__FILE__,__LINE__);    \
        fprintf(stderr,fmt,##__VA_ARGS__);                              \
        fprintf(stderr,"\n");                                           \
    } while(0)
    
    int main(int argc, char** argv)
    {
        debug("this is a debug sample with <%d> args", argc);
        debug("debug sample exit.");
        return 0;
    }

## Output of the sample

    [ DEBUG ] : [ main.cpp, 12 ] this is a debug sample with <1> args
    [ DEBUG ] : [ main.cpp, 13 ] debug sample exit.
