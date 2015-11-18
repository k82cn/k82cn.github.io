---
layout: post
title: protobuf&#58; retrieve the name of enum in C++
categories: tech
---

## Protobuf file

    package MyPackage;
    
    message MyMessage
    {
      enum RequestType {
        Login = 0;
        Logout = 1;
      }
    
      optional RequestType requestType = 1;
    }

## C++ Source Code

    #include<iostream>
    
    #include "my.pb.h"
    
    using namespace std;
    using namespace MyPackage;
    
    int main(int argc, char** argv)
    {
      MyMessage::RequestType type = MyMessage::Login;
      // Print the name of enum item
      cout << MyMessage::RequestType_Name(type) << endl;
    }

## Compile & Run

    $g++ main.cpp my.pb.cc -lprotobuf
    $./a.out
    Login
