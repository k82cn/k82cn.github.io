---
layout: post
title: ACE (1) ACE_Message_Block
categories: tech
---

本以为ACE_Message_Block只是对void*一个简单的封装, 查看了源码发现里还有一层ACE_Data_Block:是一个带引用记数的数据区；ACE_Message_Block::duplicate会对 ACE_Message_Block进行”浅复制”,即两个ACE_Message_Block对象引用同一个ACE_Data_Block对象,但 ACE_Data_Block的引用记数为2；

参照ACE tutorial里写了一个段代码,但是发现原版本中对read_n使用方法不正确,见下漂红处:

    #include <ace/Message_Block.h>
    #include <ace/ACE.h>
    
    using namespace std;
    
    int main(int argc, char** argv)
    {
        ACE_Message_Block* head = new ACE_Message_Block(10);
        ACE_Message_Block* mblk = head;
    
        while (1) {
            size_t nbytes = -1;
            ssize_t rc = ACE::read_n(ACE_STDIN, mblk->wr_ptr(), mblk->size(), &nbytes);
            if (nbytes > 0)
                mblk->wr_ptr(nbytes);
            if (rc <= 0)
                break;
            mblk->cont(new ACE_Message_Block(BUFSIZ));
            mblk = mblk->cont();
        }
    
        for ( mblk = head ; mblk!=0; mblk = mblk->cont()) {
            ACE::write_n(ACE_STDOUT, mblk->rd_ptr(), mblk->length());
        }
    
        head->release();
    
        return 0;
    }

这里说一下ACE::read_n 的行为:

ACE::read_n 会试图读取buf长度的数据.如果遇到文件结束(EOF)或者错误则返回 0 或 -1；如果先到达了buf长度则返回数据区长度；问题来了:如果数据读取成功,但是没有到达buf长度怎么办? 如何拿到已读数据的长度? 这就要用到ACE::read_n的第4个参数,这个参数记录了实际读取的数据长度.

在上面的code里还用到了几个函数:

ACE_Message_Block::size 指数据区的长度, 就是初始化时指定的长度,这里是10;

ACE_Message_Block::length 指数据的长度, 是 wr_ptr() – rd_ptr()的结果.

注意数据区和数据的区别….

ACE_Message_Block::cont ACE_Message_Block还实现了内存的链表结构.
