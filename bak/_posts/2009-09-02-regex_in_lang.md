---
layout: post
categories: tech
title: 正则表达式
---

各个语言的正则表达式使用，在这些记录一下：

## Javascript

Javascript的正则函数好像是最简单的了，也可能是因为本身就是一种弱类型的语言：看一下使用吧：

    var pattern = /test$/;
    pattern.match("test"); // 这个会匹配全串，返回true or false;
    pattern.exec("test"); //这个呢则会查找每个匹配的部分，返回值是匹配的字符串

## Java

Java语言的也不太麻烦：

    Matcher matcher = Pattern.compile("\\s+where\\s*$").matcher("text where");
    matcher.matches(); // 这个匹配全串，返回true or false;
    matcher.find(); // 这个也返回true or false，但是这个是查找串的一部分

## C

挺麻烦，估计是c的一惯风格

    regex_t re;
    regmatch_t   pmatch;
    
    int rc = regcomp(&re, "a$", 0);
    assert(rc >= 0);
    while(regexec(&re, "adfasfa", 1, &pmatch, 0)>=0)
    // c库就一个啊，没有别的了，查串中的每一个，rm_so是串的开始位置，
    // rm_eo是串的结束位置；需要自己拆串了。
    printf("%d %d", pmatch.rm_so, pmatch.rm_eo); 
