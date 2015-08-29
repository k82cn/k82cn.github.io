---
layout: post
categories: tech
title: Mesos Tips
---

## Compile Mesos UT cases without running them

    make check -j8 GTEST_FILTER=-"*"

## Run compiled Mesos UT cases

    cd build
    src/mesos-tests --gtest_filter=FetcherTest.OSNetUriTest
