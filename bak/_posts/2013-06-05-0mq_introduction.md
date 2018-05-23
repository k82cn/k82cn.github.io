---
layout: post
title: 0MQ – Introduction
categories: tech
---

0MQ (also spelled ZeroMQ, 0MQ or ZMQ) was originally a high-performance asynchronous messaging library aimed at use in scalable distributed or concurrent applications. It provides a message queue, but unlike message-oriented middleware, a 0MQ system can run without a dedicated message broker. The library is designed to have a familiar socket-style API, and was developed by iMatix Corporation.

## How to install

ZeroMQ comes as source code licensed under LGPLv3+, which can be got from ZeroMQ. For the papers, ZeroMQ 3.2.3 is used to show the feature and source code.

Here are quick steps to install the ZMQ:

Get the source code tar from the offical web site:

    wget http://download.zeromq.org/zeromq-3.2.3.tar.gz

Decompress the tar to get the source code:

    tar zxvf zeromq-3.2.3.tar.gz

Generate the Makefiles by autotools:

    ./configure --enable-debug --prefix=/opt/zmq

`–enable-debug` is not necessary; it will enable debug information, so gdb works.

Build and install the software:

    make && make install

Check the directory:

    /opt/zmq
        |-include
            |-lib
                |-share
                        |-man

## Samples

Here is a sample to show ZMQ’s basic APIs and how it works; and based on this sample, let show

* how data exchanged between server and client
* how many threads are started
* how server/client works
* what’s technology used
* ….

### Server code:

    #include <zmq.h>
    #include <stdio.h>
    #include <unistd.h>
    #include <string.h>
    #include <assert.h>
    
    int main (void)
    {
        // Socket to talk to clients
        void *context = zmq_ctx_new ();
        void *responder = zmq_socket (context, ZMQ_REP);
        int rc = zmq_bind (responder, "tcp://*:5555");
        assert (rc == 0);
        while (1) {
            char buffer [10];
            zmq_recv (responder, buffer, 10, 0);
            printf ("Received Hello\n");
            zmq_send (responder, "World", 5, 0);
            sleep (1);
            // Do some 'work'
        }
        return 0;
    }

### Client code:

    #include <zmq.h>
    #include <string.h>
    #include <stdio.h>
    #include <unistd.h>
    
    int main (void)
    {
        printf ("Connecting to hello world server...\n");
        void *context = zmq_ctx_new ();
        void *requester = zmq_socket (context, ZMQ_REQ);
        zmq_connect (requester, "tcp://localhost:5555");
    
        int request_nbr;
        for (request_nbr = 0; request_nbr != 2; request_nbr++) {
            char buffer [10];
            printf ("Sending Hello %d...\n", request_nbr);
            zmq_send (requester, "Hello", 5, 0);
            zmq_recv (requester, buffer, 10, 0);
            printf ("Received World %d\n", request_nbr);
        }
        zmq_close (requester);
        zmq_ctx_destroy (context);
        return 0;
    }

### Makefile:

    all:
        g++ -g server.cpp -o serv -I/opt/zmq/include/ -L/opt/zmq/lib -lzmq
        g++ -g client.cpp -o client -I/opt/zmq/include/ -L/opt/zmq/lib -lzmq

### Output:

    Connecting to hello world server...
    Sending Hello 0...
    Received Hello
    Received World 0
    Sending Hello 1...
    Received Hello
    Received World 1

