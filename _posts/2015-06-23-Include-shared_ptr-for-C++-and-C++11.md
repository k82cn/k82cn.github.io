---
layout: post
title: Include shared_ptr for C++ and C++11
---

    #if __cplusplus >= 201103L
    #include <memory>
    #else // __cplusplus >= 201103L
    #include <tr1/memory>
    #endif // __cplusplus >= 201103L
